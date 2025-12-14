---
link: https://www.digitalocean.com/community/tutorials/nginx-location-directive
byline: Meghna Gangwar
site: DigitalOcean
date: 2022-08-04T00:17
excerpt: Learn how to use the Nginx location directive with real-world examples for static files, regex matching, reverse proxy setups, and common configurations.
slurped: 2025-12-07T11:38
title: "Nginx Location Directive: Exact, Regex & Proxy Configs Explained"
tags:
  - nginx
  - configuration
---

### [Introduction](https://www.digitalocean.com/community/tutorials/nginx-location-directive#introduction)

The `location` directive is [Nginx](https://www.digitalocean.com/community/tags/nginx)’s core routing mechanism, determining how requests are matched and where to serve content or forward traffic. Understanding location directive syntax and match precedence is essential for configuring static file serving, reverse proxies, API routing, and complex URL rewriting scenarios.

**Why location directives matter:** Every request to your Nginx server must match a location block. Without proper configuration, requests may route to the wrong backend, fail to find static files, or bypass security rules. Mastering location matching ensures your web server routes traffic correctly, efficiently, and securely.

This tutorial covers location directive fundamentals, match order, real-world examples including `proxy_pass` configurations, side-by-side comparisons of `root` versus `alias`, debugging techniques, and production best practices. Whether you’re serving static files, proxying to application servers, or routing API endpoints, this guide provides the patterns you need.

## [Key Takeaways](https://www.digitalocean.com/community/tutorials/nginx-location-directive#key-takeaways)

Before diving into examples, here’s what you’ll learn:

- **Location directive syntax and modifiers:** Learn how to write and structure Nginx location directives using different modifiers such as prefix matching (default), exact match (`=`), case-sensitive and case-insensitive regex (`~`, `~*`), and the prefix-stopping `^~`. This section clarifies what each modifier does, when to use it, and how these choices affect how Nginx routes incoming requests to handlers or file paths, ensuring the most precise and efficient rule is used.
    
- **Nginx location match order and precedence:** Discover the exact algorithm Nginx uses when determining which location block should process a given request. This includes exploring the step-by-step search order: checking for exact matches, longest matching prefixes, the effects of modifiers like `^~`, and finally, the evaluation of regular expressions. Understanding these rules is essential for predicting traffic flow, preventing routing errors, and optimizing your configuration for clarity and speed.
    
- **Real-world configurations:** Put theory into practice by analyzing complete configuration examples that solve daily problems. You’ll see how to set up static file serving securely, how to use `proxy_pass` for forwarding requests to application servers or APIs, and how to nest location blocks for scenarios where complex routing or layered control is needed. These patterns are illustrated with sample configs you can adapt to your own production environments.
    
- **Root versus alias:** Learn the differences between the `root` and `alias` directives when mapping request URIs to file system paths. This section offers clear, side-by-side configuration examples to highlight their behaviors, reveal common mistakes, and help you decide which directive best fits different use cases. Understanding these nuances is crucial to preventing broken links or unexpected file exposures in your site or app.
    
- **Debugging and troubleshooting:** Master practical techniques for identifying which location block handles a specific URL, using Nginx logs, debug modules, and manual test requests. You’ll also learn systematic methods for diagnosing and correcting issues like mismatched routes, permission errors, and accidental fall-throughs to the wrong handler, helping you iterate faster and keep your server secure and reliable.
    
- **Performance optimization:** Explore best practices for maximizing the efficiency of your Nginx server. Apply caching directives to reduce backend load and speed up responses, carefully design and order your location blocks for faster matches, and craft regular expressions to minimize processing overhead. These strategies ensure your configuration remains robust as traffic grows and requirements scale.
    

## [Prerequisites](https://www.digitalocean.com/community/tutorials/nginx-location-directive#prerequisites)

- Nginx installed on Ubuntu or a compatible Linux distribution. If you need installation guidance, follow our tutorial on [how to install Nginx on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04).
- Basic familiarity with Nginx configuration files and server blocks. Review [Understanding the Nginx Configuration File Structure and Configuration Contexts](https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts) if needed.
- Access to edit Nginx configuration files (typically requires `sudo` privileges).

**Test your configuration safely:** Always run `sudo nginx -t` before reloading Nginx. This validates your syntax and prevents breaking your web server. Never reload without testing first.

## [Understanding Location Directive Syntax](https://www.digitalocean.com/community/tutorials/nginx-location-directive#understanding-location-directive-syntax)

The `location` directive defines how Nginx matches incoming request URIs against file system paths or proxy targets. Location blocks can be placed inside `server` blocks or nested inside other location blocks (with restrictions).

**Basic syntax:**

```
location [modifier] [URI] {
    # directives
}
```

The modifier is optional but fundamentally changes matching behavior. The URI can be a literal string or a regular expression pattern.

### [Location Modifiers Explained](https://www.digitalocean.com/community/tutorials/nginx-location-directive#location-modifiers-explained)

Nginx supports four modifiers that control matching behavior:

|Modifier|Name|Matching Behavior|Priority|
|---|---|---|---|
|`=`|Exact match|Matches URI exactly, stops all further searching|Highest|
|`^~`|Prefix match stop|Longest prefix match, stops regex evaluation|High|
|`~`|Case-sensitive regex|Regular expression match, case-sensitive|Medium|
|`~*`|Case-insensitive regex|Regular expression match, case-insensitive|Medium|
|_(none)_|Prefix match|Matches URI prefix, continues searching|Lowest|

**Why modifiers matter:** Without modifiers, Nginx performs prefix matching that continues evaluating regex blocks. Using `^~` stops regex checks after a prefix match, improving performance. The `=` modifier is the highest priority but only matches exact URIs—use it for specific endpoints like `/favicon.ico` or `/health`.

**Common modifier patterns:**

```
# Exact match - highest priority, no further matching
location = /images {
    # Only matches /images, not /images/ or /images/logo.png
}

# Prefix match that stops regex evaluation
location ^~ /images {
    # Matches /images, /images/, /images/logo.png
    # Stops checking regex locations after match
}

# Case-sensitive regex
location ~ \.(jpg|png|gif)$ {
    # Matches .jpg, .png, .gif (case-sensitive)
}

# Case-insensitive regex  
location ~* \.(jpg|png|gif)$ {
    # Matches .JPG, .PNG, .GIF, .jpg, .png, .gif
}
```

## [How Nginx Chooses a Location Block](https://www.digitalocean.com/community/tutorials/nginx-location-directive#how-nginx-chooses-a-location-block)

Understanding Nginx location match order is critical for predictable routing. Nginx evaluates location blocks in a specific sequence, not simply “first match wins.” The algorithm ensures exact matches and longest prefixes take precedence over regex patterns.

### [Location Matching Algorithm](https://www.digitalocean.com/community/tutorials/nginx-location-directive#location-matching-algorithm)

Nginx follows this exact sequence when selecting a location block:

#### Step 1: Exact Match (`=`)

- Nginx first checks all `location = /path` blocks.
- If an exact match is found, that block is selected immediately and no further matching occurs.
- Example: `location = /api` only matches `/api`, not `/api/users` or `/api/`.

#### Step 2: Longest Prefix Match (without `^~`)

- Nginx finds the longest prefix match among non-regex locations.
- If the longest match uses `^~`, Nginx stops here and uses that block.
- If the longest match doesn’t use `^~`, Nginx stores it temporarily and continues to Step 3.

#### Step 3: Regex Evaluation

- Nginx evaluates regex locations (`~` and `~*`) in the order they appear in the configuration file.
- The first regex that matches wins.
- If a regex matches, Nginx uses that block (overriding the stored prefix match from Step 2).

#### Step 4: Fallback to Prefix

- If no regex matches in Step 3, Nginx uses the stored longest prefix match from Step 2.
- If no prefix matches exist, Nginx falls back to the `location /` block (the catch-all).

**Critical Note:** Regex locations are evaluated in configuration file order, not by specificity. If you have `location ~ /api` before `location ~ /api/users`, a request to `/api/users` matches the first regex (`/api`), not the more specific one. Always order regex locations from most specific to least specific, or use prefix matching with `^~` to avoid regex evaluation entirely.

### [Match Precedence Visualization](https://www.digitalocean.com/community/tutorials/nginx-location-directive#match-precedence-visualization)

```
Request: GET /images/logo.png

1. Check exact matches
   location = /images/logo.png  [No match]
   
2. Find longest prefix
   location /images/          [Match - longest]
   location /                 [Match - but shorter, stored as fallback]
   
3. Check if longest prefix uses ^~
   location ^~ /images/       [Uses ^~ → STOP, use this block]
   
   (If no ^~, continue to regex...)
   
4. Evaluate regex (in config order)
   location ~ \.png$          [Would match - but skipped due to ^~]
   
Result: location ^~ /images/ is selected
```

### [Practical Example: Understanding Match Order](https://www.digitalocean.com/community/tutorials/nginx-location-directive#practical-example-understanding-match-order)

Consider this configuration:

```
server {
    listen 80;
    server_name example.com;
    
    # Regex location - evaluated after prefix matches
    location ~ /api/v1 {
        return 200 "API v1";
        add_header Content-Type text/plain;
    }
    
    # Prefix match with ^~ - stops regex evaluation
    location ^~ /api {
        return 200 "API prefix";
        add_header Content-Type text/plain;
    }
    
    # Exact match - highest priority
    location = /api {
        return 200 "API exact";
        add_header Content-Type text/plain;
    }
    
    # Catch-all
    location / {
        return 200 "Default";
        add_header Content-Type text/plain;
    }
}
```

**Test results:**

```
curl http://example.com/api
# Response: "API exact" (exact match wins)

curl http://example.com/api/users  
# Response: "API prefix" (^~ stops regex, prefix wins)

curl http://example.com/api/v1/users
# Response: "API prefix" (^~ matches /api, stops before checking regex)

```

**Why this matters:** Changing the order of location blocks or adding `^~` can completely change which block handles a request. Always test location matching after configuration changes using `curl` or your browser’s developer tools.

## [Basic Location Block Examples](https://www.digitalocean.com/community/tutorials/nginx-location-directive#basic-location-block-examples)

These examples demonstrate fundamental location directive patterns you’ll use in production configurations.

### [Example 1: Catch-All Location](https://www.digitalocean.com/community/tutorials/nginx-location-directive#example-1-catch-all-location)

The `location /` block matches all requests that don’t match more specific locations. It serves as the default fallback.

```
location / {
    root /var/www/html;
    index index.html index.htm;
    try_files $uri $uri/ =404;
}
```

**When to use:** As a default handler for unmatched requests, typically for serving a main website or application.

**Note:** Because `location /` matches everything, it has the lowest precedence. More specific locations (exact matches, longer prefixes, or matching regex) will take precedence.

### [Example 2: Exact Match Location](https://www.digitalocean.com/community/tutorials/nginx-location-directive#example-2-exact-match-location)

Exact match locations (`=`) have the highest priority and match only when the URI exactly matches the specified path.

```
location = /favicon.ico {
    access_log off;
    log_not_found off;
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

**When to use:** For specific endpoints like `/favicon.ico`, `/robots.txt`, or health check endpoints like `/health` where you want absolute precision.

**Test it:**

```
# This matches
curl http://example.com/favicon.ico

# These do NOT match
curl http://example.com/favicon.ico/
curl http://example.com/favicon.ico?v=1

```

**Exact match limitation:** The `=` modifier matches the URI path only, ignoring query strings. So `location = /api` matches `/api` and `/api?key=value` equally. To exclude query strings from matching, use the `$request_uri` variable in your logic instead.

### [Example 3: Directory Prefix Match](https://www.digitalocean.com/community/tutorials/nginx-location-directive#example-3-directory-prefix-match)

Prefix matching without modifiers matches any URI that starts with the specified path.

```
location /images/ {
    root /var/www;
    expires 30d;
    add_header Cache-Control "public";
}
```

**Behavior:** Matches `/images/`, `/images/logo.png`, `/images/photos/2024.jpg`, but not `/images` (without trailing slash).

**File resolution:** With `root /var/www`, a request to `/images/logo.png` resolves to `/var/www/images/logo.png`.

### [Example 4: Prefix Match with Stop (`^~`)](https://www.digitalocean.com/community/tutorials/nginx-location-directive#example-4-prefix-match-with-stop)

The `^~` modifier performs prefix matching but prevents Nginx from evaluating regex locations after a match.

```
location ^~ /images {
    root /var/www;
    expires 30d;
}
```

**Performance benefit:** Stops regex evaluation immediately, reducing CPU overhead when serving many requests.

**Use case:** When you have regex locations elsewhere that might accidentally match paths under `/images`, `^~` ensures `/images/*` requests never reach regex blocks.

### [Example 5: Case-Insensitive Regex Match](https://www.digitalocean.com/community/tutorials/nginx-location-directive#example-5-case-insensitive-regex-match)

Case-insensitive regex matching (`~*`) is useful for file extensions or flexible path patterns.

```
location ~* \.(jpg|jpeg|png|gif|ico|svg|webp)$ {
    root /var/www;
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

**What it matches:** Any URI ending with the specified extensions, regardless of case: `.jpg`, `.JPG`, `.Png`, `.SVG`, etc.

**Regex performance:** Regex matching is slower than prefix matching. For high-traffic static file serving, prefer prefix matching with `^~` over regex when possible. Reserve regex for cases where pattern matching is truly needed.

### [Example 6: Case-Sensitive Regex Match](https://www.digitalocean.com/community/tutorials/nginx-location-directive#example-6-case-sensitive-regex-match)

Case-sensitive regex (`~`) matches patterns only when the case matches exactly.

```
location ~ /API/ {
    return 403;
    add_header Content-Type text/plain;
}
```

**What it matches:** `/API/users`, `/API/v1/data`, but NOT `/api/users` or `/Api/users`.

**Common use:** Blocking requests with incorrect casing, enforcing case-sensitive API paths, or matching specific patterns that require exact case.

## [Real-World Location Block Configurations](https://www.digitalocean.com/community/tutorials/nginx-location-directive#real-world-location-block-configurations)

Production configurations combine multiple location blocks to handle static files, API routing, reverse proxying, and security rules.

### [Static File Serving with Root and Alias](https://www.digitalocean.com/community/tutorials/nginx-location-directive#static-file-serving-with-root-and-alias)

Two directives serve static files: `root` and `alias`. Understanding their difference prevents common configuration errors.

**Root directive behavior:**

```
location /static/ {
    root /var/www/html;
}
```

With `root`, Nginx appends the location path to the `root` path:

- Request: `/static/css/style.css`
- File path: `/var/www/html/static/css/style.css`

**Alias directive behavior:**

```
location /static/ {
    alias /var/www/assets/;
}
```

With `alias`, Nginx replaces the location path with the `alias` path:

- Request: `/static/css/style.css`
- File path: `/var/www/assets/css/style.css` (note: `/static/` is replaced)

**Critical difference:** `alias` requires a trailing slash when the location block has one. Without it, Nginx may return 404 errors or serve incorrect files:

- Correct: `location /static/ { alias /var/www/assets/; }`
- Wrong: `location /static/ { alias /var/www/assets; }` (missing trailing slash)

**Side-by-side comparison:**

|Scenario|Root Configuration|Alias Configuration|
|---|---|---|
|Location|`location /images/`|`location /images/`|
|Directive|`root /var/www/html;`|`alias /var/www/photos/;`|
|Request URI|`/images/logo.png`|`/images/logo.png`|
|Resolved path|`/var/www/html/images/logo.png`|`/var/www/photos/logo.png`|
|Use case|Files are in `/var/www/html/images/`|Files are elsewhere, like `/var/www/photos/`|

### [Reverse Proxy Configuration with proxy_pass](https://www.digitalocean.com/community/tutorials/nginx-location-directive#reverse-proxy-configuration-with-proxy_pass)

The `proxy_pass` directive forwards requests to backend application servers. Location blocks route different paths to different backends.

**Basic proxy_pass:**

```
location /api/ {
    proxy_pass http://127.0.0.1:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

**Note:** The presence or absence of a trailing slash in `proxy_pass` dramatically changes how the URI is forwarded:

```
# With trailing slash - location path is stripped
location /api/ {
    proxy_pass http://127.0.0.1:8000/;  # Note trailing slash
}
# Request: /api/users → Backend receives: /users

# Without trailing slash - full path is forwarded  
location /api/ {
    proxy_pass http://127.0.0.1:8000;  # No trailing slash
}
# Request: /api/users → Backend receives: /api/users
```

Always test your proxy_pass configuration to ensure the backend receives the expected path.

**Complete reverse proxy example:**

```
server {
    listen 80;
    server_name example.com;
    
    # Serve static files directly
    location /static/ {
        alias /var/www/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Proxy API requests to backend
    location /api/ {
        proxy_pass http://127.0.0.1:8000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # Proxy WebSocket connections
    location /ws/ {
        proxy_pass http://127.0.0.1:8001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # Default: serve main application
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
    }
}
```

For a comprehensive reverse proxy setup, including HTTPS and load balancing, see our guide on [How To Configure Nginx as a Reverse Proxy on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-reverse-proxy-on-ubuntu-22-04).

### [Nested Location Blocks](https://www.digitalocean.com/community/tutorials/nginx-location-directive#nested-location-blocks)

Nginx allows nested location blocks with restrictions. Nested blocks inherit settings from parent locations and can override them.

```
location /files/ {
    root /var/www;
    
    # Nested location for specific file types
    location ~ \.(jpg|png|gif)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Nested location for other files
    location ~ \.(pdf|doc)$ {
        add_header Content-Disposition "attachment";
    }
}
```

**Nesting limitations:** Not all directives can be used in nested locations. For example, you cannot use `root` or `alias` in nested blocks when the parent already uses them. Review the [Nginx documentation](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) for specific restrictions.

## [Common Mistakes and How to Fix Them](https://www.digitalocean.com/community/tutorials/nginx-location-directive#common-mistakes-and-how-to-fix-them)

These mistakes cause routing failures, 404 errors, or security issues. Learn to recognize and fix them quickly.

### [Mistake 1: Confusing `root` and `alias`](https://www.digitalocean.com/community/tutorials/nginx-location-directive#mistake-1-confusing-root-and-alias)

**Problem:** Files return 404 errors even though they exist, often due to incorrect usage of the `alias` directive, especially missing or misplaced trailing slashes.

```
# WRONG - Missing trailing slash in alias
location /static/ {
    alias /var/www/assets;
}
```

**Diagnosis:** Examine Nginx error logs (`/var/log/nginx/error.log`) for clues on failed file lookups. Check how Nginx resolves the file system path and ensure it matches the actual structure. The missing slash causes Nginx to append the URI after `/static/` _directly_ to `/var/www/assets`, which breaks path resolution.

**Solution:** When using `alias` for directories, always include a trailing slash on both the `location` _and_ the `alias` path to ensure correct mapping. For example, `/static/logo.png` should resolve to `/var/www/assets/logo.png`:

```
# CORRECT - Both location and alias end with slashes
location /static/ {
    alias /var/www/assets/;  # Trailing slash required
}
```

**Tip:** Use `root` for prefix mapping and `alias` for remapping to a different directory, but always double-check slashes with `alias`. If mapping a single file, do not use a trailing slash.

### [Mistake 2: Regex Matching Unintended Paths](https://www.digitalocean.com/community/tutorials/nginx-location-directive#mistake-2-regex-matching-unintended-paths)

**Problem:** A regex location matches more paths than intended.

```
# WRONG - Matches /api, /api/users, AND /api-backup/users
location ~ /api {
    proxy_pass http://127.0.0.1:8000;
}

# CORRECT - More specific regex
location ~ ^/api/ {
    proxy_pass http://127.0.0.1:8000;
}

# BETTER - Use prefix with ^~ to avoid regex entirely
location ^~ /api/ {
    proxy_pass http://127.0.0.1:8000;
}
```

**Solution:** Write regex patterns as specifically as possible. Use anchors like `^` to match the start of the URI, and favor prefix matches (`^~`) when regular expressions are not needed. This prevents Nginx from matching unintended paths (like `/api-backup/users`). Always test your location matches with possible edge-case URLs.

### [Mistake 3: Incorrect proxy_pass URI Handling](https://www.digitalocean.com/community/tutorials/nginx-location-directive#mistake-3-incorrect-proxy_pass-uri-handling)

**Problem:** Backend receives wrong paths due to a trailing slash in `proxy_pass`.

```
# Request: /api/users

# Backend receives: /api/users (location path included)
location /api/ {
    proxy_pass http://127.0.0.1:8000;
}

# Backend receives: /users (location path stripped)
location /api/ {
    proxy_pass http://127.0.0.1:8000/;
}
```

**Solution:** The presence or absence of a trailing slash in the `proxy_pass` target changes how Nginx rewrites the request URI. Without a trailing slash, the original URI is passed (e.g., `/api/users`); with a trailing slash, the matching part of the location is replaced (resulting in `/users`). Decide which behavior matches your backend’s expected URL structure and be consistent throughout your configuration. Test with `curl` and your backend logs to ensure routes resolve as intended.

### [Mistake 4: Location Order Causing Wrong Matches](https://www.digitalocean.com/community/tutorials/nginx-location-directive#mistake-4-location-order-causing-wrong-matches)

**Problem:** Regex location appears before a more specific prefix, causing incorrect routing.

```
# WRONG ORDER
location ~ /api {
    return 403;  # Blocks /api/users incorrectly
}

location /api/users {
    proxy_pass http://127.0.0.1:8000;
}

# CORRECT ORDER - Most specific first
location /api/users {
    proxy_pass http://127.0.0.1:8000;
}

location ~ /api {
    return 403;
}

# BETTER - Use prefix matching to avoid regex
location ^~ /api/users {
    proxy_pass http://127.0.0.1:8000;
}
```

**Solution:** Always order your location blocks from most specific to least specific in your configuration file. When possible, avoid regex matches in favor of prefix matches, as they are more predictable and performant. Remember that prefix and exact matches have higher precedence over regex, but if regex matches are necessary, ensure general regexes are listed last. Regularly review and refactor your location block ordering to prevent unintentional route blocking or exposure.

### [Mistake 5: Missing try_files for SPA Routing](https://www.digitalocean.com/community/tutorials/nginx-location-directive#mistake-5-missing-try_files-for-spa-routing)

**Problem:** Single Page Application (SPA) routes return 404 when accessed directly.

```
# WRONG - 404 on /dashboard/settings
location / {
    root /var/www/html;
    index index.html;
}

# CORRECT - Fallback to index.html for SPA routing
location / {
    root /var/www/html;
    try_files $uri $uri/ /index.html;
}
```

**Solution:** For SPAs that handle routing client-side, configure `try_files $uri $uri/ /index.html;` in your root `location /` block. This ensures that direct navigation to any route in your SPA (like `/dashboard/settings`) falls back to `index.html`, letting your front-end router take over. Without this, non-root routes will return 404 errors when accessed directly or on page reload.

### [Method 1: Add Unique Response Headers](https://www.digitalocean.com/community/tutorials/nginx-location-directive#method-1-add-unique-response-headers)

Add distinct headers to each location block to identify matches:

```
location /api/ {
    add_header X-Location-Match "api-prefix" always;
    proxy_pass http://127.0.0.1:8000;
}

location ~ /api {
    add_header X-Location-Match "api-regex" always;
    return 403;
}
```

Test with curl:

```
curl -I http://example.com/api/users
# Look for X-Location-Match header in response

```

### [Method 2: Use Return for Testing](https://www.digitalocean.com/community/tutorials/nginx-location-directive#method-2-use-return-for-testing)

Temporarily replace complex logic with `return` statements to verify matching:

```
location /images/ {
    return 200 "Images location matched";
    add_header Content-Type text/plain;
}
```

**Don’t forget to revert:** After testing, restore your original configuration. Leaving `return` statements in production will break normal functionality.

### [Method 3: Enable Debug Logging](https://www.digitalocean.com/community/tutorials/nginx-location-directive#method-3-enable-debug-logging)

Enable debug-level logging to see Nginx’s location matching process:

```
error_log /var/log/nginx/error.log debug;
```

Then check the logs after making a request:

```
tail -f /var/log/nginx/error.log
# Make a request and watch for location matching details

```

**Performance impact:** Debug logging generates extensive output and impacts performance. Only enable it for troubleshooting, then revert to `error_log /var/log/nginx/error.log warn;` for production.

### [Method 4: Use Nginx Location Testing Tools](https://www.digitalocean.com/community/tutorials/nginx-location-directive#method-4-use-nginx-location-testing-tools)

Several online tools and command-line utilities can simulate Nginx location matching:

- Test configurations locally with `nginx -T` to validate syntax
- Use `curl -v` to inspect full request/response headers
- Create a test server block with a unique server_name for experimentation

## [Advanced Location Patterns](https://www.digitalocean.com/community/tutorials/nginx-location-directive#advanced-location-patterns)

These advanced patterns solve complex routing requirements in production environments.

### [API Versioning with Location Blocks](https://www.digitalocean.com/community/tutorials/nginx-location-directive#api-versioning-with-location-blocks)

Route different API versions to different backends:

```
location /api/v1/ {
    proxy_pass http://127.0.0.1:8001/;
}

location /api/v2/ {
    proxy_pass http://127.0.0.1:8002/;
}

location /api/ {
    # Default to latest version
    proxy_pass http://127.0.0.1:8002/;
}
```

### [Blocking Access by File Extension](https://www.digitalocean.com/community/tutorials/nginx-location-directive#blocking-access-by-file-extension)

Prevent access to sensitive file types:

```
location ~ \.(env|ini|log|sql|bak)$ {
    deny all;
    return 404;
}
```

### [Conditional Routing Based on Request Method](https://www.digitalocean.com/community/tutorials/nginx-location-directive#conditional-routing-based-on-request-method)

While location blocks don’t directly support HTTP method matching, combine with `if` (use sparingly) or use separate location blocks with method-specific upstreams:

```
# GET requests to /api/read
location = /api/read {
    proxy_pass http://127.0.0.1:8000;
    limit_except GET {
        deny all;
    }
}

# POST requests to /api/write  
location = /api/write {
    proxy_pass http://127.0.0.1:8000;
    limit_except POST {
        deny all;
    }
}
```

**Avoid `if` when possible:** Nginx’s `if` directive has many caveats and can cause unexpected behavior. Prefer `map` directives, separate location blocks, or `limit_except` for conditional logic. See the [If Is Evil](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/) guide for details.

### [Serving Different Content Based on Domain](https://www.digitalocean.com/community/tutorials/nginx-location-directive#serving-different-content-based-on-domain)

While domain routing typically uses separate `server` blocks, location blocks can handle path-based routing within a server:

```
server {
    server_name api.example.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
    }
}

server {
    server_name www.example.com;
    
    location / {
        root /var/www/html;
        try_files $uri $uri/ /index.html;
    }
}
```
## [Performance Optimization Best Practices](https://www.digitalocean.com/community/tutorials/nginx-location-directive#performance-optimization-best-practices)

Location block structure and directive choices impact Nginx performance. Apply these optimizations for high-traffic sites.

### [Minimize Regex Evaluation](https://www.digitalocean.com/community/tutorials/nginx-location-directive#minimize-regex-evaluation)

Prefer prefix matching with `^~` over regex when possible:

```
# SLOWER - Regex evaluation for every request
location ~ \.(jpg|png|gif)$ {
    root /var/www;
}

# FASTER - Prefix match stops regex checks
location ^~ /images/ {
    root /var/www;
}
```

### [Order Locations by Specificity](https://www.digitalocean.com/community/tutorials/nginx-location-directive#order-locations-by-specificity)

Place more specific locations before general ones to reduce evaluation:

```
# CORRECT ORDER
location = /favicon.ico { }      # Checked first (exact)
location ^~ /static/ { }         # Checked second (specific prefix)
location ~ \.css$ { }             # Checked third (regex)
location / { }                    # Checked last (catch-all)
```

### [Cache Static Files Aggressively](https://www.digitalocean.com/community/tutorials/nginx-location-directive#cache-static-files-aggressively)

Use location blocks to apply different caching strategies:

```
location ~* \.(jpg|jpeg|png|gif|ico|svg|webp)$ {
    root /var/www;
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}

location ~* \.(css|js)$ {
    root /var/www;
    expires 30d;
    add_header Cache-Control "public";
}
```

### [Use Alias for Performance](https://www.digitalocean.com/community/tutorials/nginx-location-directive#use-alias-for-performance)

When serving files from non-standard paths, `alias` avoids unnecessary directory traversal:

```
# More efficient for paths outside document root
location /assets/ {
    alias /var/cdn/assets/;
}
```

## [Docker Example Configuration](https://www.digitalocean.com/community/tutorials/nginx-location-directive#docker-example-configuration)

Test Nginx location directives in an isolated [Docker](https://www.digitalocean.com/community/tutorials/how-to-run-nginx-in-a-docker-container-on-ubuntu-22-04) environment without modifying your production server.

**Create `nginx.conf`:**

```
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name localhost;
        
        location = /health {
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
        
        location /static/ {
            alias /usr/share/nginx/html/;
        }
        
        location /api/ {
            proxy_pass http://app:8000/;
            proxy_set_header Host $host;
        }
        
        location / {
            root /usr/share/nginx/html;
            try_files $uri $uri/ /index.html;
        }
    }
}
```

**Create `Dockerfile`:**

```
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY index.html /usr/share/nginx/html/
```

**Run with Docker Compose:**

```
version: '3.8'
services:
  nginx:
    build: .
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
  app:
    image: node:18
    # Your app server configuration
```

**Testing workflow:** Use Docker to test location configurations safely. Make changes to `nginx.conf`, rebuild the container, and test with `curl` without affecting your production Nginx installation.

## [Troubleshooting Common Issues](https://www.digitalocean.com/community/tutorials/nginx-location-directive#troubleshooting-common-issues)

Diagnose and fix location directive problems with these troubleshooting techniques.

### [Issue: Location Block Not Matching](https://www.digitalocean.com/community/tutorials/nginx-location-directive#issue-location-block-not-matching)

**Symptoms:** Requests return 404 or hit the wrong location block.

**Diagnosis steps:**

1. **Verify syntax:** Run `sudo nginx -t` to check for configuration errors.
    
2. **Check match precedence:** Review the location matching algorithm. An exact match or `^~` prefix might be intercepting your intended block.
    
3. **Test with curl:** Use `curl -v http://example.com/path` to inspect full request/response.
    
4. **Check error logs:** Review `/var/log/nginx/error.log` for path resolution details.
    

**Common fixes:**

- Add `^~` modifier if regex locations are matching first
- Reorder location blocks (most specific first)
- Verify URI path matches exactly (trailing slashes matter)

### [Issue: proxy_pass Sending Wrong Path](https://www.digitalocean.com/community/tutorials/nginx-location-directive#issue-proxy_pass-sending-wrong-path)

**Symptoms:** Backend receives incorrect URIs, causing 404 errors on the application server.

**Diagnosis:**

```
# Check what path backend actually receives
# Add logging in your backend application
# Or use tcpdump/wireshark to inspect proxied requests

```

**Fix:** Adjust trailing slash in `proxy_pass`:

```
# If backend expects /users (not /api/users)
location /api/ {
    proxy_pass http://127.0.0.1:8000/;  # Trailing slash strips /api/
}

# If backend expects /api/users
location /api/ {
    proxy_pass http://127.0.0.1:8000;  # No trailing slash forwards full path
}
```

### [Issue: Static Files Not Found](https://www.digitalocean.com/community/tutorials/nginx-location-directive#issue-static-files-not-found)

**Symptoms:** 404 errors for files that exist in the file system.

**Diagnosis:**

1. Check file permissions: `ls -la /var/www/html/images/logo.png`
2. Verify `root`/`alias` path resolution
3. Confirm location block path matches request URI

**Common causes:**

- Trailing slash mismatch in `alias` directive
- Incorrect `root` path (Nginx appends location path to root)
- File permissions are preventing the Nginx worker process from reading files
- SELinux/AppArmor restrictions (check with `getenforce` or `aa-status`)

### [Issue: Regex Location Too Greedy](https://www.digitalocean.com/community/tutorials/nginx-location-directive#issue-regex-location-too-greedy)

**Symptoms:** Regex matches unintended paths.

**Fix:** Make the regex more specific with anchors:

```
# TOO BROAD - Matches /api, /api-backup, /my-api
location ~ /api {
    # ...
}

# SPECIFIC - Only matches /api/ or paths starting with /api/
location ~ ^/api/ {
    # ...
}

# BEST - Use prefix match to avoid regex entirely
location ^~ /api/ {
    # ...
}
```

## [Location Directive vs Rewrite Rules](https://www.digitalocean.com/community/tutorials/nginx-location-directive#location-directive-vs-rewrite-rules)

Understand when to use `location` blocks versus `rewrite` rules. Both route requests serve different purposes.

|Feature|Location Directive|Rewrite Rule|
|---|---|---|
|**Purpose**|Match and route requests|Modify request URI|
|**When to use**|Route to different backends/files|Change URL structure, redirect|
|**Performance**|Faster (native matching)|Slower (regex processing)|
|**Use case**|`proxy_pass` to backend, serve static files|Redirect old URLs, clean URLs|

### [Example: When to Use Each](https://www.digitalocean.com/community/tutorials/nginx-location-directive#example-when-to-use-each)

```
# Use location for routing
location /api/ {
    proxy_pass http://127.0.0.1:8000;
}

# Use rewrite for URL transformation
location /blog/ {
    rewrite ^/blog/(.*)$ /posts/$1 permanent;
}

# Combine both
location /old-api/ {
    rewrite ^/old-api/(.*)$ /api/v1/$1 break;
    proxy_pass http://127.0.0.1:8000;
}
```

For detailed rewrite rule patterns, see [Nginx Rewrite URL Rules](https://www.digitalocean.com/community/tutorials/nginx-rewrite-url-rules).

## [Frequently Asked Questions](https://www.digitalocean.com/community/tutorials/nginx-location-directive#frequently-asked-questions)

### [What is a location directive in Nginx?](https://www.digitalocean.com/community/tutorials/nginx-location-directive#what-is-a-location-directive-in-nginx)

A location directive defines how Nginx matches incoming request URIs and determines where to serve content or forward traffic. Location blocks can serve static files from the file system, proxy requests to backend servers, or apply specific configurations to matched paths.

Location directives use modifiers (`=`, `~`, `~*`, `^~`) to control matching behavior, with exact matches having the highest priority and regex patterns evaluated in configuration file order.

### [How does Nginx choose which location block to use?](https://www.digitalocean.com/community/tutorials/nginx-location-directive#how-does-nginx-choose-which-location-block-to-use)

Nginx follows a four-step algorithm:

1. **Exact match check:** `location = /path` blocks are checked first. If matched, that block is used immediately.
    
2. **Longest prefix match:** Nginx finds the longest matching prefix. If it uses `^~`, Nginx stops and uses that block.
    
3. **Regex evaluation:** If the longest prefix doesn’t use `^~`, Nginx evaluates regex locations (`~` and `~*`) in configuration order. The first matching regex wins.
    
4. **Fallback:** If no regex matches, Nginx uses the stored longest prefix match, or the `location /` catch-all block.
    

This algorithm ensures predictable routing while allowing flexible pattern matching.

### [What is the difference between root and alias in Nginx?](https://www.digitalocean.com/community/tutorials/nginx-location-directive#what-is-the-difference-between-root-and-alias-in-nginx)

`root` appends the location path to the root path, while `alias` replaces the location path with the alias path.

**Root example:**

```
location /static/ {
    root /var/www/html;
}
# Request: /static/file.css → File: /var/www/html/static/file.css
```

**Alias example:**

```
location /static/ {
    alias /var/www/assets/;
}
# Request: /static/file.css → File: /var/www/assets/file.css
```

**Key difference:** Use `root` when files are in a subdirectory matching the location path. Use `alias` when files are in a different directory structure. `alias` requires a trailing slash when the location block has one.

### [How do regex location blocks work in Nginx?](https://www.digitalocean.com/community/tutorials/nginx-location-directive#how-do-regex-location-blocks-work-in-nginx)

Regex location blocks (`~` for case-sensitive, `~*` for case-insensitive) use Perl-compatible regular expressions to match request URIs. They’re evaluated in configuration file order after prefix matches (unless `^~` stops evaluation).

```
# Case-sensitive regex
location ~ \.(jpg|png)$ {
    # Matches .jpg, .png (not .JPG, .PNG)
}

# Case-insensitive regex
location ~* \.(jpg|png)$ {
    # Matches .jpg, .png, .JPG, .PNG
}
```

Regex locations are slower than prefix matching, so prefer prefix with `^~` when pattern matching isn’t strictly needed.

### [How can I test which location block matches a URL?](https://www.digitalocean.com/community/tutorials/nginx-location-directive#how-can-i-test-which-location-block-matches-a-url)

Several methods:

1. **Add unique headers:** Include `add_header X-Location "name" always;` in each location block, then check headers with `curl -I`.
    
2. **Use return statements:** Temporarily replace logic with `return 200 "location name";` to see which block matches.
    
3. **Enable debug logging:** Set `error_log /var/log/nginx/error.log debug;` to see matching details (disable after testing).
    
4. **Inspect with curl:** Use `curl -v` to see full request/response and identify routing behavior.
    

### [What’s the difference between `location /` and `location = /`?](https://www.digitalocean.com/community/tutorials/nginx-location-directive#what-s-the-difference-between-location-and-location)

`location /` is a prefix match that matches any URI starting with `/` (essentially all requests). `location = /` is an exact match that only matches the root URI `/` exactly.

```
# Exact match - only matches /
location = / {
    return 200 "Root exact";
}

# Prefix match - matches /, /about, /api/users, everything
location / {
    return 200 "Root prefix";
}
```

Use `location = /` when you want to handle only the root path specially, while `location /` serves as a catch-all for unmatched requests.

### [Can I nest multiple location directives in Nginx?](https://www.digitalocean.com/community/tutorials/nginx-location-directive#can-i-nest-multiple-location-directives-in-nginx)

Yes, but with restrictions. Nested location blocks inherit settings from parent locations and can override some directives (like `expires`), but cannot use `root` or `alias` if the parent already defines them.

```
location /files/ {
    root /var/www;
    
    location ~ \.(jpg|png)$ {
        expires 1y;  # Override parent's expires
    }
}
```

Nesting is useful for applying different configurations to file types within a directory, but keep nesting shallow to avoid complexity.

### [How do I handle trailing slashes in location blocks?](https://www.digitalocean.com/community/tutorials/nginx-location-directive#how-do-i-handle-trailing-slashes-in-location-blocks)

Trailing slashes affect matching and file resolution:

- `location /images` matches `/images` and `/images/` (Nginx normalizes)
- `location /images/` matches `/images/` and `/images/file.jpg`, but NOT `/images` (without slash)

For directory serving, include the trailing slash:

```
# Correct for directory serving
location /static/ {
    alias /var/www/assets/;  # Both location and alias have trailing slashes
}
```

Use `try_files` to handle both cases:

```
location /images {
    try_files $uri $uri/ =404;
}
```

### [What’s the performance impact of using regex in location blocks?](https://www.digitalocean.com/community/tutorials/nginx-location-directive#what-s-the-performance-impact-of-using-regex-in-location-blocks)

Regex matching is computationally more expensive than prefix matching. For high-traffic sites serving many static files, regex evaluation can add noticeable overhead.

**Performance tips:**

- Prefer prefix matching with `^~` when possible
- Order regex locations from most specific to least specific
- Cache regex compilation results (Nginx does this automatically)
- Use `access_log off;` for high-volume static file locations to reduce I/O

Benchmark shows regex locations can be 2-3x slower than prefix matches under load, though modern Nginx versions have optimized regex engines significantly.

## [Conclusion](https://www.digitalocean.com/community/tutorials/nginx-location-directive#conclusion)

The Nginx location directive is fundamental to routing requests efficiently and correctly. By understanding match precedence, choosing appropriate modifiers, and applying best practices for static file serving and reverse proxying, you can build robust web server configurations that scale.

Key principles to remember:

- **Match order matters:** Exact matches (`=`) win, followed by `^~` prefixes, then regex in config order
- **Root vs alias:** `root` appends paths, `alias` replaces them—choose based on your file structure
- **Test your configurations:** Always use `nginx -t` and verify with `curl` before deploying
- **Optimize for performance:** Prefer prefix matching over regex, order locations by specificity, cache aggressively

For deeper dives into Nginx configuration, explore our guides on [Understanding Nginx Server and Location Block Selection Algorithms](https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms) and [Understanding the Nginx Configuration File Structure and Configuration Contexts](https://www.digitalocean.com/community/tutorials/understanding-the-nginx-configuration-file-structure-and-configuration-contexts).

Whether you’re serving static sites, proxying to application servers, or building complex routing rules, location directives provide the flexibility and performance you need for production web infrastructure.

[![Creative Commons](https://www.digitalocean.com/api/static-content/v1/images?src=%2F_next%2Fstatic%2Fmedia%2Fcreativecommons.c0a877f1.png&width=384)](https://creativecommons.org/licenses/by-nc-sa/4.0/)This work is licensed under a Creative Commons Attribution-NonCommercial- ShareAlike 4.0 International License.