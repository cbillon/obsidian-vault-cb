---
link: https://blog.dochia.dev/blog/http_edge_cases/
site: Dochia CLI Blog
excerpt: "Most HTTP vulnerabilities don't come from sophisticated attacks. They come from misunderstanding where your framework stops protecting you. This covers the edge cases that actually bite production APIs: Range headers, path traversal, encoding conflicts, and request smuggling"
twitter: https://twitter.com/@dochiadev
slurped: 2026-05-10T13:00
title: Nine HTTP Edge Cases Every API Developer Should Understand | Dochia CLI Blog
tags:
  - api
  - http
---

## Introduction

Last February, [CVE-2024-26141](https://nvd.nist.gov/vuln/detail/CVE-2024-26141) was discovered for Rack, the web server interface that powers nearly every Ruby on Rails application. The vulnerability was simple: send a file request with hundreds of byte ranges, and Rack would generate a unexpectedly large response. Production servers got hammered by single HTTP requests until they ran out of memory or bandwidth.

What made this worse was the affected version range: 1.3.0 and up, which meant applications from 2011 onwards were vulnerable. Many engineers everywhere spent their weekend(s) deploying patches.

This is a simple example on how **a simple mishandled HTTP edge case can cause significant damage**. And not because we’re bad developers, but because HTTP is complex. The happy path works fine. Then production happens.

Range headers exist for specific reasons: resuming downloads, seeking in videos, downloading file chunks, etc. They’re part of HTTP/1.1, defined in RFC 7233, and most of us never think about them until something breaks.

The Rack vulnerability worked because crafted Range headers could trigger excessive computation when Rack parsed and assembled the response: misaligned or extreme ranges caused the server to allocate or calculate more data than expected. With enough ranges requested, the processing cost dominated the actual content. The attack looked like this:

```
GET /report.pdf HTTP/1.1
Range: bytes=0-999,1000000-2000000,999999999-...
```

Keep adding ranges and the parsing overhead accumulates. A 1MB PDF with malicious ranges might generate multi-megabyte responses or force expensive range calculations. While not catastrophic for one request, multiply this across hundreds of concurrent requests and you amplify CPU and memory consumption significantly, degrading availability.

Static file servers like nginx and Apache handle this correctly. S3, CloudFront also fine. The problem appears when we write custom file download endpoints for access control, watermarking, or dynamic generation.

But mitigation is not as simple as limiting range count. Even five ranges could generate problematic responses if the file is large enough. You need to validate both the number of ranges and the total projected response size. Spring Boot’s ResourceHttpRequestHandler does this for static resources, but custom controllers don’t inherit that protection:

```
@GetMapping("/api/files/{filename}")
public ResponseEntity<?> downloadFile(
    @PathVariable String filename,
    @RequestHeader(value = "Range", required = false) String rangeHeader) throws IOException {

    Resource file = fileService.loadAsResource(filename);
    long size = file.contentLength();

    if (rangeHeader == null || rangeHeader.isBlank()) {
        return ResponseEntity.ok()
            .contentType(MediaType.APPLICATION_OCTET_STREAM)
            .body(file);
    }

    List<HttpRange> ranges;
    try {
        ranges = HttpRange.parseRanges(rangeHeader); // this will throw an exception on bad syntax or >100 ranges
    } catch (IllegalArgumentException ex) {
        // RFC 7233: 416 for unsatisfiable/malformed; include Content-Range with '*'
        return ResponseEntity.status(HttpStatus.REQUESTED_RANGE_NOT_SATISFIABLE)
            .header(HttpHeaders.CONTENT_RANGE, "bytes */" + size)
            .build();
    }

    // Cap ranges and total bytes to defend resources
    final int MAX_RANGES = 5;
    final long MAX_BYTES = Math.min(size, 8L * 1024 * 1024);

    if (ranges.size() > MAX_RANGES) {
        // Option A: ignore ranges -> return full content (200)
        // Option B: 416 to force client to retry sanely, pick a consistent policy
        return ResponseEntity.ok().body(file);
    }

    long totalBytes = 0L;
    for (HttpRange r : ranges) {
        ResourceRegion rr = r.toResourceRegion(file);
        totalBytes += rr.getCount();
        if (totalBytes > MAX_BYTES) {
            return ResponseEntity.ok().body(file); // ignore ranges if too large
        }
    }

    // Let Spring write 206 / multipart or single-part automatically
    return ResponseEntity.status(HttpStatus.PARTIAL_CONTENT)
        .contentType(MediaType.APPLICATION_OCTET_STREAM)
        .body(HttpRange.toResourceRegions(ranges, file));
}
```

Similar example for Express.js:

```
import rangeParser from 'range-parser';
import fs from 'fs';
import path from 'path';

app.get('/files/:filename', (req, res) => {
  const filename = req.params.filename;
  const filePath = path.join(__dirname, 'files', filename);
  const stat = fs.statSync(filePath);
  const fileSize = stat.size;
  const rangeHeader = req.headers.range;

  if (rangeHeader) {
    // Parse with range-parser (same as express.static)
    const ranges = rangeParser(fileSize, rangeHeader, { combine: true });

    if (ranges === -1) {
      // Unsatisfiable -> 416 per RFC
      return res.status(416)
        .set('Content-Range', `bytes */${fileSize}`)
        .end();
    }
    if (ranges === -2) {
      // Malformed -> ignore Range, serve full
      return res.sendFile(filePath);
    }

    // Limit ranges & aggregate size
    const MAX_RANGES = 5;
    const MAX_BYTES = 8 * 1024 * 1024;

    if (ranges.length > MAX_RANGES) {
      // Too many -> ignore Range, serve full
      return res.sendFile(filePath);
    }

    let totalBytes = 0;
    for (const r of ranges) {
      totalBytes += (r.end - r.start + 1);
      if (totalBytes > MAX_BYTES) {
        // Too heavy -> ignore Range
        return res.sendFile(filePath);
      }
    }

    // If one range, serve 206 single part
    if (ranges.length === 1) {
      const { start, end } = ranges[0];
      res.status(206)
        .set('Content-Range', `bytes ${start}-${end}/${fileSize}`)
        .set('Accept-Ranges', 'bytes')
        .set('Content-Length', end - start + 1);
      return fs.createReadStream(filePath, { start, end }).pipe(res);
    }

    // If multiple ranges, you'd need to construct multipart/byteranges.
    // Node/Express doesn't do this automatically; consider rejecting or serving full content
    return res.sendFile(filePath);
  }

  // No Range -> serve full
  res.sendFile(filePath);
});
```

The fix isn’t complicated, but it’s easy to forget when you’re focused on access control logic and database queries. Range headers feel like infrastructure concerns until they’re your problem.

## Content-Type Enforcement Prevents Weird Parser Behavior

JSON **must** be UTF-8 encoded according to RFC 8259 Section 8.1. There’s no “charset parameter” because UTF-8 is mandatory. Any JSON with `Content-Type: application/json; charset=iso-8859-1` is technically malformed.

But there’s a subtler problem. Consider this request:

```
POST /api/users HTTP/1.1
Content-Type: text/plain
Content-Length: 87

{"name": "Robert'; DROP TABLE users;--", "email": "[email protected]"}
```

Some frameworks see JSON-shaped text and parse it anyway. Others correctly reject it. The behavior varies, and that inconsistency creates security gaps. An attacker might craft payloads that only parse in specific framework versions or configurations.

Spring Boot enforces Content-Type when you’re explicit about what you accept:

```
@PostMapping(value = "/api/users", consumes = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
    // Returns 415 Unsupported Media Type if Content-Type doesn't match
    return ResponseEntity.ok(userService.create(user));
}
```

Express requires explicit middleware configuration. By default it does not parse bodies at all. With explicit configuration, you can enforce strict media types:

```
// Only accepts application/json, rejects everything else
app.use(express.json({ type: 'application/json' }));

app.post('/api/users', (req, res) => {
  // req.body only populated for correct Content-Type
  // Wrong Content-Type? req.body is undefined, preventing silent failures
});
```

Django REST Framework validates Content-Type by default, which is one reason it’s harder to shoot yourself in the foot with Django:

```
from rest_framework.decorators import api_view
from rest_framework.parsers import JSONParser

@api_view(['POST'])
@parser_classes([JSONParser])  # Only accepts application/json
def create_user(request):
    # Returns 415 for incorrect Content-Type
    # No ambiguity, no silent parsing
    pass
```

**Always declare what Content-Types you accept**. Permissive APIs create attack surface. If you only handle JSON, reject anything that isn’t explicitly `application/json`. Don’t try to be helpful by accepting “close enough” content types.

Accept headers let clients specify format preferences with quality values:

```
Accept: application/json;q=1.0, application/xml;q=0.5, */*;q=0.1
```

This says “I prefer JSON, XML is acceptable, and I’ll take anything if necessary.” Quality values range from 0.0 to 1.0, with higher being more preferred. Most APIs ignore this entirely and return whatever format matches first in their routing logic.

Content negotiation bugs surface when routing logic doesn’t respect quality values. If an API adds XML support to accommodate one client, but the routing logic checks for “xml” substring presence before parsing quality values, clients that prefer JSON but list XML as acceptable (`Accept: application/json;q=1.0, application/xml;q=0.5`) might unexpectedly receive XML. The JSON parser receives XML, parsing fails, and the integration breaks.

Spring Boot handles content negotiation automatically with proper quality value parsing:

```
@GetMapping(value = "/api/data", produces = MediaType.APPLICATION_JSON_VALUE)
public Data getData() {
    return dataService.fetch();
}

// Multiple formats - Spring negotiates based on quality values
@GetMapping(value = "/api/data/{id}", 
            produces = {MediaType.APPLICATION_JSON_VALUE, MediaType.APPLICATION_XML_VALUE})
public Data getDataById(@PathVariable Long id) {
    return dataService.findById(id);
}
```

Client asks for something unsupported? Spring returns 406 Not Acceptable with no ambiguity.

Express needs manual handling, and you have to get the parsing right:

```
app.get('/api/data', (req, res) => {
  // req.accepts() properly parses quality values
  const accepts = req.accepts(['json', 'xml']);
  
  if (accepts === 'json') {
    res.json(data);
  } else if (accepts === 'xml') {
    res.type('application/xml').send(toXml(data));
  } else {
    res.status(406).send('Not Acceptable');
  }
});
```

**Pick your supported formats deliberately**. Don’t try supporting everything. If you only handle JSON in 99% of endpoints, don’t add XML support to one endpoint because a legacy system asked for it. That’s how you create the production incident I described.

## Method Not Allowed Should Tell You What Works

HTTP 405 responses should include an `Allow` header listing valid methods. This turns error responses into documentation. When developers get 405 instead of 200, they immediately know what methods are supported. Without proper `Allow` headers, 404 and 405 become indistinguishable, forcing developers to guess whether endpoints exist or they’re using the wrong method.

Spring Boot handles this automatically for controller methods:

```
@RestController
@RequestMapping("/api/payments")
public class PaymentController {
    
    @GetMapping("/{id}")
    public Payment getPayment(@PathVariable String id) {
        return paymentService.findById(id);
    }
    
    @PostMapping
    public Payment createPayment(@Valid @RequestBody PaymentRequest request) {
        return paymentService.create(request);
    }
}
```

If you send DELETE to `/api/payments/123`? You get:

```
HTTP/1.1 405 Method Not Allowed
Allow: GET, POST, HEAD, OPTIONS
```

Express needs explicit handling:

```
app.route('/api/payments/:id')
  .get(getPayment)
  .post(createPayment)
  .all((req, res) => {
    res.set('Allow', 'GET, POST')
       .status(405)
       .send('Method Not Allowed');
  });
```

It’s a small detail, but it’s the difference between a great API and a good one.

## Compression Configuration Is Never Where You Think

Compression seems straightforward: enable gzip, save bandwidth, everybody wins. Then you discover your Spring Boot compression config does nothing in production because nginx (or another proxy) terminates the connection before it reaches your application.

Spring Boot’s compression only affects the embedded servlet container:

```
server.compression.enabled=true
server.compression.mime-types=application/json,application/xml,text/plain
server.compression.min-response-size=1024
```

In production, you typically have nginx, HAProxy, or an API Gateway handling the TLS connection. They see the request, process it, and proxy to your backend. If nginx isn’t configured for compression, the response goes out uncompressed regardless of your Spring Boot settings. If nginx is configured for compression, it compresses the response whether Spring Boot is configured or not.

Configure compression where it actually happens - at the edge. Also, disable app-level compression or strip Accept-Encoding upstream to avoid double-compression:

nginx:

```
gzip on;
gzip_types application/json application/xml text/plain text/css application/javascript;
gzip_min_length 1024;
gzip_vary on;       # Tells caches compression varies by Accept-Encoding
gzip_proxied any;   # Enable gzip for proxied responses
gzip_comp_level 6;  # Balance CPU and ratio
```

Apache:

```
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE application/json application/xml text/plain text/css application/javascript
    DeflateCompressionLevel 6
    Header append Vary "Accept-Encoding"
</IfModule>
```

AWS API Gateway:

```
# Enable compression in API Gateway settings
minimumCompressionSize: 1024
```

Don’t compress everything indiscriminately: for small responses under ~1KB, compression overhead exceeds bandwidth savings. The CPU cycles and latency aren’t worth saving 200 bytes. Also consider disabling compression on sensitive endpoints (e.g., pages reflecting secrets) to reduce BREACH-style risks.

## Character Encoding Silently Corrupts Your Database

**Encoding mismatches destroy data permanently**. A registration form accepts user data. Server assumes UTF-8. Client sends ISO-8859-1. Names get corrupted. The corruption is permanent because you can’t reliably detect encoding issues after the fact. You don’t know if is corrupted UTF-8 or correctly encoded text in another charset?

The problem happens with form data:

```
POST /api/registration HTTP/1.1
Content-Type: application/x-www-form-urlencoded; charset=iso-8859-1

name=Jos%E9+Garc%EDa&city=S%E3o+Paulo
```

IF the server assumes UTF-8, decodes the percent-encoded bytes as UTF-8, stores it. **The corruption is permanent**.

Use UTF-8 everywhere and enforce it:

Spring Boot:

```
server.tomcat.uri-encoding=UTF-8
spring.http.encoding.charset=UTF-8
spring.http.encoding.force=true
```

Express:

```
app.use(express.json({ defaultCharset: 'utf-8' }));
app.use(express.urlencoded({ defaultCharset: 'utf-8', extended: true }));
```

Django defaults to UTF-8, but be explicit:

```
DEFAULT_CHARSET = 'utf-8'
```

nginx:

```
charset utf-8;
```

**For JSON endpoints, UTF-8 is mandatory per RFC 8259. Reject any request claiming non-UTF-8 encoding**. It’s malformed per spec and processing it will corrupt data.

## Path Traversal Lets Attackers Read Arbitrary Files

[CVE-2019-19781](https://nvd.nist.gov/vuln/detail/cve-2019-19781) showed how path traversal vulnerabilities compromise entire networks. The Citrix ADC vulnerability let attackers read arbitrary files through URL manipulation:

```
GET /vpn/../vpns/cfg/smb.conf HTTP/1.1
GET /vpn/%2e%2e%2fvpns%2fcfg%2fsmb.conf HTTP/1.1
```

Walk up the directory tree, read configuration files containing credentials, move laterally through the network. Enterprise networks got compromised because of improper path validation. Citrix issued emergency patches, but exploits were already there.

Any endpoint mapping user input to filesystem paths needs validation. The pattern is universal: normalize the path, resolve it to an absolute path, verify it stays inside your allowed directory!

Some examples on how to handle this below:

Java:

```
@GetMapping("/api/files/**")
public ResponseEntity<Resource> serveFile(HttpServletRequest request) throws IOException {
    String requestPath = request.getRequestURI().substring("/api/files/".length());
    
    Path baseDir = Paths.get("/var/app/uploads").toAbsolutePath().normalize();
    Path requestedFile = baseDir.resolve(requestPath).normalize();
    
    // Critical: verify resolved path stays inside base directory
    if (!requestedFile.startsWith(baseDir)) {
        return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
    }
    
    if (!Files.exists(requestedFile) || !Files.isReadable(requestedFile)) {
        return ResponseEntity.notFound().build();
    }
    
    return ResponseEntity.ok().body(new FileSystemResource(requestedFile));
}
```

Node.js:

```
const path = require('path');

app.get('/files/*', (req, res) => {
  const baseDir = path.resolve('/var/app/uploads');
  const requestedPath = path.join(baseDir, req.params[0]);
  const resolved = path.resolve(requestedPath);
  
  if (!resolved.startsWith(baseDir)) {
    return res.status(403).send('Forbidden');
  }
  
  res.sendFile(resolved);
});
```

Python:

```
from pathlib import Path

def serve_file(filename):
    base_dir = Path('/var/app/uploads').resolve()
    requested = (base_dir / filename).resolve()
    
    # Verify resolved path is inside base directory
    if not requested.is_relative_to(base_dir):
        return 403, "Forbidden"
    
    return send_file(requested)
```

**Never trust user input for filesystem operations**. Always normalize, resolve, and validate.

## Request Size Limits Prevent Memory Exhaustion

Unlimited request sizes are the simplest DoS attack available. Send 100MB POST requests until server memory is exhausted. No sophistication required, no distributed attack needed. One attacker, one script, one poorly configured server.

Set conservative limits everywhere:

Spring Boot:

```
server.tomcat.max-http-request-header-size=8KB
spring.servlet.multipart.max-file-size=10MB
spring.servlet.multipart.max-request-size=10MB
```

Express:

```
app.use(express.json({ limit: '1mb' }));
app.use(express.urlencoded({ limit: '1mb', extended: true }));
```

Django:

```
DATA_UPLOAD_MAX_MEMORY_SIZE = 1048576  # 1MB
FILE_UPLOAD_MAX_MEMORY_SIZE = 10485760  # 10MB
```

nginx:

```
client_max_body_size 10M;
client_body_buffer_size 128K;
```

Apache:

```
LimitRequestBody 10485760
```

Most APIs don’t need more than 1-10MB per request. Need larger uploads? Use chunked uploads, resumable protocols, or direct-to-S3 with presigned URLs. **Don’t buffer entire uploads in memory**.

The Rack Range vulnerability combined with unlimited response sizes would have been catastrophic. Each layer of validation matters. Set limits, enforce them, test them.

## Transfer-Encoding Enables Request Smuggling

Request smuggling exploits disagreements between proxies and application servers about where one request ends and another begins. As per RFC 7230 Section 3.3.3: if Transfer-Encoding is present, Content-Length must be ignored.

An attack sends conflicting headers:

```
POST /api/submit HTTP/1.1
Content-Length: 44
Transfer-Encoding: chunked

0

POST /api/admin/delete HTTP/1.1
```

Proxy processes `Content-Length` and sees one request. Application server processes chunked encoding and sees two requests. The second “smuggled” request bypasses authentication because the proxy already validated the first request.

Modern HTTP servers implement RFC 7230 correctly, but bugs still appear in specific configurations. [CVE-2020-7238](https://nvd.nist.gov/vuln/detail/cve-2020-7238) affected Netty 4.1.43 because it mishandled `Transfer-Encoding` headers with leading whitespace. [CVE-2021-43797](https://nvd.nist.gov/vuln/detail/cve-2021-43797) showed that control characters in header names could enable smuggling in specific proxy configurations.

You don’t need to implement request smuggling protection in your application code. What you need is defense in depth—multiple layers that each validate HTTP compliance:

- nginx normalizes requests automatically and rejects malformed ones. No special configuration needed—it’s built into the HTTP parser.
- HAProxy validates HTTP compliance strictly by default and rejects requests with both `Content-Length` and `Transfer-Encoding` headers.
- Springboot and Tomcat validate this at the connector level. The servlet container rejects smuggling attempts before your application code runs.

The vulnerability appears when proxies and application servers disagree about the spec. Your defense is ensuring each layer properly implements RFC 7230. **Don’t try to add your own request smuggling detection**. You’ll get it wrong and add complexity without security benefit.

## Testing Edge Cases Reveals Real Behavior

Ideally, all these cases should be handled by automated tests. Initially, use `curl` to understand how your stack handles edge cases, then automate the successful tests:

```
# Test Range header abuse
curl -v -H "Range: bytes=0-100,200-300,400-500,600-700,800-900,1000-1100" \
     http://localhost:8080/api/files/test.pdf

# Test Content-Type enforcement
curl -X POST -H "Content-Type: text/plain" \
     -d '{"valid": "json"}' http://localhost:8080/api/users

# Test path traversal
curl "http://localhost:8080/api/files/../../../etc/passwd"
curl "http://localhost:8080/api/files/%2e%2e%2f%2e%2e%2fetc%2fpasswd"

# Test size limits - generate 11MB to exceed 10MB limit
dd if=/dev/zero bs=1M count=11 2>/dev/null | \
curl -X POST -H "Content-Type: application/octet-stream" \
     --data-binary @- http://localhost:8080/api/upload

# Test method validation
curl -X DELETE -v http://localhost:8080/api/payments/123

# Test encoding conflicts
curl -X POST -H "Content-Length: 10" -H "Transfer-Encoding: chunked" \
     -d "test" http://localhost:8080/api/submit
```

These tests gives you an understanding on how your API fails. Things to check:

- Are error messages useful or do they leak implementation details?
- Does validation work consistently?
- What information appears in logs when attacks happen?

## HTTP/2 and HTTP/3 Change the Attack Surface

The edge cases discussed focus on HTTP/1.1, but HTTP/2 and HTTP/3 introduce different vulnerabilities. HTTP/2’s binary framing and header compression (HPACK) enable new attacks like compression bombs (large, carefully crafted header tables) and stream abuse. HTTP/3’s QUIC transport adds concerns such as connection migration and 0-RTT resumption, which can enable replay attacks if not carefully handled.

Most frameworks abstract protocol differences, but some edge cases remain protocol-specific:

- **Range headers**: Work identically across HTTP/1.1, HTTP/2, and HTTP/3.
- **Request smuggling**: Common in HTTP/1.1 due to chunked encoding ambiguities, but not applicable in HTTP/2’s strict binary framing.
- **Rapid reset attacks**: Unique to HTTP/2’s multiplexing, where attackers can open many streams and cancel them immediately, consuming server resources.
- **Header compression exploits**: HPACK (HTTP/2) and QPACK (HTTP/3) can be abused for memory pressure attacks if implementations don’t enforce strict limits.
- **QUIC transport risks**: HTTP/3 inherits issues like replay with 0-RTT and potential misuse of connection migration.

Framework abstractions handle most protocol differences, but understanding which vulnerabilities apply to which protocol version matters when designing defenses. A reverse proxy that translates HTTP/2 to HTTP/1.1 (or HTTP/3 to HTTP/2) introduces another layer where disagreements about request boundaries or header parsing can re-introduce old classes of bugs.

## Where Frameworks End and Your Responsibility Begins

**Most modern frameworks handle HTTP correctly**. Custom validation code is rarely needed and often introduces more bugs than it prevents. The vulnerabilities presented above - CVE-2024-26141 (Rack Range headers), CVE-2019-19781 (Citrix path traversal), CVE-2020-7238 (Netty smuggling) - all exploited gaps between specification and implementation.

**Frameworks typically handle**:

- Content-Type validation
- Accept header negotiation
- HTTP method validation
- Transfer-Encoding validation
- Basic path normalization
- Compression (when configured at the right layer)
- Character encoding defaults

**You must implement**:

- Range header validation for custom file endpoints
- Path traversal prevention for dynamic file serving
- Request size limits (via configuration)
- Proper error responses with Allow headers

**Handle at infrastructure level**:

- Rate limiting
- DDoS protection
- TLS termination
- Request normalization
- Geographic routing

**Your defense isn’t more code**. It’s understanding HTTP deeply, **knowing what your framework handles**, using infrastructure layers for redundancy, and writing custom validation only where genuinely needed. Most security vulnerabilities come from unnecessary custom code that reimplements (incorrectly) what the framework already does correctly.