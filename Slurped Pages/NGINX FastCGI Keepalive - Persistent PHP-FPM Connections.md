---
link: https://www.getpagespeed.com/server-setup/nginx/nginx-fastcgi-keepalive
byline: Danila Vershinin
site: GetPageSpeed
date: 2026-01-21T17:28
updated: 2026-06-07T15:38
excerpt: Configure NGINX FastCGI keepalive for persistent PHP-FPM connections. Complete guide with fastcgi_keep_conn, upstream keepalive, and tuning tips.
twitter: https://twitter.com/@getpagespeed
tags:
  - nginx
  - configuration
slurped: 2026-07-05T20:13
title: "NGINX FastCGI Keepalive: Persistent PHP-FPM Connections"
---

We have by far the [largest RPM repository](https://www.getpagespeed.com/redhat) with [NGINX module packages](https://www.getpagespeed.com/nginx-extras) and VMODs for Varnish. If you want to install NGINX, Varnish, and lots of useful performance/security software with smooth `yum` upgrades for production use, this is the repository for you.  
_Active [subscription](https://www.getpagespeed.com/repo-subscribe) is required._

**📅 Updated:** June 7, 2026 (Originally published: January 21, 2026)

Every PHP request on your server involves a hidden cost: establishing a connection between NGINX and PHP-FPM. By default, NGINX closes this connection after each request, only to open a brand new one for the next. This constant connect-disconnect cycle wastes CPU cycles and adds latency to every single request. The solution? **NGINX FastCGI keepalive connections**.

If you have ever wondered why your PHP application feels slower than it should be, or why your server CPU spikes under moderate load, the culprit might be connection overhead. In this comprehensive guide, you will learn how to configure persistent connections between NGINX and PHP-FPM using the `fastcgi_keep_conn` directive and the upstream `keepalive` module.

We will dive deep into the FastCGI protocol, examine the actual NGINX source code to understand what happens under the hood, and provide production-ready configurations that have been tested and verified on Rocky Linux 9 and AlmaLinux 9.

## What Are FastCGI Keepalive Connections?

Before diving into configuration, let us understand the problem we are solving. When NGINX proxies a request to PHP-FPM, it communicates using the FastCGI protocol. This protocol, designed in the 1990s, allows web servers to communicate with application servers like PHP-FPM efficiently.

Without keepalive connections, this is what happens for every single PHP request on your server:

1. NGINX opens a new connection to PHP-FPM (via Unix socket or TCP)
2. The connection is established and resources are allocated
3. NGINX sends the FastCGI BEGIN_REQUEST record
4. NGINX sends FastCGI PARAMS and STDIN records with the request data
5. PHP-FPM processes the request and sends STDOUT records back
6. PHP-FPM sends an END_REQUEST record
7. NGINX closes the connection and releases resources

This entire process repeats thousands of times per minute on a busy server. Each connection establishment involves system calls, memory allocation, and socket setup overhead. On a server handling 1,000 requests per second, that means 1,000 connection establishments and teardowns every second.

With **FastCGI keepalive connections** enabled, NGINX fundamentally changes this behavior by maintaining a pool of persistent connections to PHP-FPM:

1. NGINX checks the connection pool for an available connection
2. If found, NGINX reuses the existing connection
3. NGINX sends the request over the persistent connection
4. PHP-FPM processes and responds
5. Instead of closing, the connection returns to the pool for the next request

This eliminates the connection establishment overhead entirely, reducing latency and CPU usage significantly.

## How fastcgi_keep_conn Works Under the Hood

The `fastcgi_keep_conn` directive controls a single bit in the FastCGI protocol. This might seem trivial, but understanding this mechanism helps you troubleshoot issues and optimize your configuration.

Looking at the [NGINX source code](https://github.com/nginx/nginx/blob/master/src/http/modules/ngx_http_fastcgi_module.c), we can see exactly how this works internally.

The FastCGI protocol specification defines a BEGIN_REQUEST record that NGINX sends at the start of each request. This record contains a flags field:

```
typedef struct {
    u_char  role_hi;
    u_char  role_lo;
    u_char  flags;        /* The KEEP_CONN flag lives here */
    u_char  reserved[5];
} ngx_http_fastcgi_begin_request_t;
```

The `flags` field can contain the `FCGI_KEEP_CONN` constant (defined as 1). When this bit is set, it signals to the FastCGI application: “Do not close this connection after responding. Keep it open for more requests.”

In the NGINX source code, this is implemented simply:

```
#define NGX_HTTP_FASTCGI_KEEP_CONN      1

/* ... */

ngx_http_fastcgi_request_start.br.flags =
    flcf->keep_conn ? NGX_HTTP_FASTCGI_KEEP_CONN : 0;
```

When you set `fastcgi_keep_conn on` in your [NGINX configuration](https://www.getpagespeed.com/check-nginx-config), the `keep_conn` flag becomes true (1), and NGINX sets the KEEP_CONN bit in every FastCGI request to PHP-FPM.

However, `fastcgi_keep_conn on` alone is not enough. This directive only tells PHP-FPM to keep its end of the connection open. Without a connection pool on the NGINX side, NGINX would still close connections after each request. You need **both** the `keepalive` directive in the upstream block AND `fastcgi_keep_conn on` in the location block for persistent connections to actually work.

## Complete Configuration for NGINX FastCGI Keepalive

Now that you understand the mechanism, let us configure it properly. This configuration has been tested on Rocky Linux 9 and AlmaLinux 9 with the system NGINX and PHP-FPM packages.

### Step 1: Configure the Upstream Block with Connection Pooling

Create or edit the upstream configuration file. On RHEL-based systems, this is typically `/etc/nginx/conf.d/php-fpm.conf`:

```
# PHP-FPM FastCGI server with keepalive connections
# for optimal performance

upstream php-fpm {
    server unix:/run/php-fpm/www.sock;

    # Number of idle keepalive connections to cache per worker process
    # Calculate: pm.max_children / worker_processes
    keepalive 16;

    # Maximum requests per connection before recycling
    # Higher values reduce connection cycling overhead
    keepalive_requests 10000;

    # Idle connection timeout (default: 60s)
    # Connections idle longer than this are closed
    keepalive_timeout 60s;
}
```

### Step 2: Configure the FastCGI Location Block

In your server block configuration, add or modify the PHP location. This can be in your main server block or in a separate file like `/etc/nginx/default.d/php.conf`:

```
# PHP processing with keepalive connections
location ~ .php$ {
    # Security: return 404 for non-existent PHP files
    try_files $uri =404;

    # Pass to the upstream with connection pooling
    fastcgi_pass php-fpm;
    fastcgi_index index.php;

    # Required FastCGI parameters
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;

    # CRITICAL: Enable persistent FastCGI connections
    # This tells PHP-FPM to keep the connection open
    # Must be used with keepalive in the upstream block
    fastcgi_keep_conn on;
}
```

### Step 3: Verify and Apply the Configuration

Always test your NGINX configuration before reloading:

```
nginx -t
```

You should see:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If the test passes, reload NGINX to apply the changes:

```
systemctl reload nginx
```

## Understanding Each Upstream Keepalive Directive

Let us examine each directive in the upstream block in detail. Understanding these helps you tune the configuration for your specific workload.

### The keepalive Directive

```
keepalive 16;
```

This is the most important directive. It sets the maximum number of idle keepalive connections preserved in the cache of each NGINX worker process. When this number is exceeded, the least recently used connections are closed to make room for new ones.

**How to calculate the optimal value:**

The ideal `keepalive` value depends on your PHP-FPM and NGINX configuration:

```
keepalive = pm.max_children / worker_processes
```

For example, if your PHP-FPM configuration has:

```
pm.max_children = 50
```

And your NGINX configuration has:

```
worker_processes 4;
```

Then the calculation is:

```
keepalive = 50 / 4 = 12.5 ≈ 16 (round up)
```

**Why this formula works:** Each NGINX worker process maintains its own connection pool. If you have 50 PHP-FPM workers and 4 NGINX workers, each NGINX worker needs at most 50/4 = 12.5 connections to reach all PHP-FPM workers. Rounding up to 16 provides some buffer.

For more details on tuning worker_processes, see our guide on [tuning NGINX worker_processes](https://www.getpagespeed.com/server-setup/nginx/tuning-nginx-worker-processes).

**Common mistakes:**  
– Setting too high: Wastes memory storing unused connection handles  
– Setting too low: Causes frequent connection cycling, defeating the purpose

### The keepalive_requests Directive

```
keepalive_requests 10000;
```

Added in NGINX 1.15.3, this directive sets the maximum number of requests that can be served through one keepalive connection. After this limit is reached, the connection is closed and recycled.

The default value was 100 until NGINX 1.19.10, when it was increased to 1000. For high-traffic sites, setting this even higher (10000 or more) reduces connection recycling overhead.

**Why limit requests per connection?**

Some PHP applications have memory leaks or accumulate state over time. Recycling connections periodically ensures that any accumulated issues are cleared. It also helps with connection distribution across PHP-FPM workers.

### The keepalive_timeout Directive

```
keepalive_timeout 60s;
```

Also added in NGINX 1.15.3, this sets the maximum time an idle keepalive connection stays open. Connections that sit idle longer than this value are closed. The default is 60 seconds.

**Tuning considerations:**

- Low traffic sites: Consider reducing to 30s to free resources sooner
- High traffic sites: 60s is usually fine; connections get reused before timing out
- Bursty traffic: Consider increasing to handle traffic spikes that follow quiet periods

### The keepalive_time Directive

```
keepalive_time 1h;
```

Added in NGINX 1.19.10, this directive limits the maximum total time a connection can be used, regardless of how active it is. Even if a connection is constantly serving requests, it will be closed after this time period.

This helps with:  
– Graceful connection rotation  
– Memory leak mitigation in PHP-FPM  
– Load distribution across PHP-FPM workers over time

## PHP-FPM Process Manager Configuration

The PHP-FPM process manager mode significantly impacts how well NGINX FastCGI keepalive connections work. This is one of the most critical aspects to get right. For a deeper dive into PHP-FPM tuning, see our comprehensive guide on [optimizing NGINX for high-performance PHP websites](https://www.getpagespeed.com/server-setup/nginx/optimize-nginx-for-high-performance-php-websites).

### Dynamic Mode (Recommended for Keepalive)

```
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
```

Dynamic mode is the ideal choice for use with NGINX FastCGI keepalive connections. PHP-FPM maintains a pool of worker processes that persist between requests. The number of workers scales up and down based on demand, but there are always at least `pm.min_spare_servers` workers available.

When NGINX sends a request over a keepalive connection, there is always a PHP-FPM worker ready to handle it. The connection pool maps cleanly to the worker pool.

### Static Mode (Also Works Well)

```
pm = static
pm.max_children = 50
```

Static mode maintains a fixed number of worker processes at all times. This mode also works well with keepalive connections because workers never exit unexpectedly. The trade-off is higher memory usage during low traffic periods.

Static mode is preferred for high-traffic production servers where you want predictable resource usage and maximum performance.

### Ondemand Mode (Problematic with Keepalive)

```
pm = ondemand
pm.max_children = 50
pm.process_idle_timeout = 10s
pm.max_requests = 500
```

**Warning:** Ondemand mode can cause serious problems with NGINX FastCGI keepalive connections.

In ondemand mode, PHP-FPM workers spawn only when needed and exit after `pm.process_idle_timeout` seconds of inactivity. Here is the problem:

1. NGINX caches a keepalive connection to PHP-FPM worker #1
2. During a quiet period, worker #1 exits due to idle timeout
3. A new request arrives, and NGINX tries to use the cached connection
4. The connection fails because worker #1 no longer exists
5. You see a 502 Bad Gateway error

If you must use ondemand mode for memory savings, you have two options:

**Option 1: Disable keepalive entirely**

```
upstream php-fpm {
    server unix:/run/php-fpm/www.sock;
    # No keepalive directive
}

location ~ .php$ {
    fastcgi_pass php-fpm;
    # No fastcgi_keep_conn directive
    # ...
}
```

**Option 2: Align timeouts carefully**

Set `pm.process_idle_timeout` higher than `keepalive_timeout`:

```
# PHP-FPM configuration
pm.process_idle_timeout = 120s
```

```
# NGINX configuration
keepalive_timeout 60s;
```

This ensures PHP-FPM workers outlive the cached connections.

## Unix Socket vs TCP Connections

Both Unix domain sockets and TCP connections support NGINX FastCGI keepalive. Here is when to use each:

### Unix Socket Configuration (Same Server)

```
upstream php-fpm {
    server unix:/run/php-fpm/www.sock;
    keepalive 16;
}
```

Use Unix sockets when NGINX and PHP-FPM run on the same server. Benefits:

- No TCP handshake overhead
- No network stack processing
- No port allocation
- Slightly lower latency
- Slightly lower CPU usage

On RHEL-based systems, the default PHP-FPM socket path is `/run/php-fpm/www.sock`.

### TCP Connection Configuration (Remote or Load Balanced)

```
upstream php-fpm {
    server 127.0.0.1:9000;
    keepalive 16;
}
```

Use TCP connections when:

- PHP-FPM runs on a different server
- You need to load balance across multiple PHP-FPM instances
- You are using containerized PHP-FPM (where sockets may not be accessible)

For TCP connections, consider also enabling:

```
upstream php-fpm {
    server 10.0.0.2:9000;
    server 10.0.0.3:9000;
    keepalive 32;
}

location ~ .php$ {
    fastcgi_pass php-fpm;
    fastcgi_keep_conn on;

    # Enable TCP keepalive probes on the socket
    fastcgi_socket_keepalive on;
}
```

## Do Not Confuse These Two Directives

There are two similarly-named directives that serve completely different purposes. Understanding the difference prevents configuration mistakes.

### fastcgi_keep_conn (FastCGI Protocol Level)

```
fastcgi_keep_conn on;
```

This controls the **FastCGI protocol’s KEEP_CONN flag**. It tells PHP-FPM to keep the connection open after sending the response. This is required for connection pooling to work.

Default: `off`

### fastcgi_socket_keepalive (TCP Socket Level)

```
fastcgi_socket_keepalive on;
```

Added in NGINX 1.15.6, this enables **TCP keepalive probes** on the underlying socket. These are OS-level packets sent periodically to detect dead connections at the network layer.

Default: `off`

**When to use fastcgi_socket_keepalive:**

- TCP connections to remote PHP-FPM servers
- Network environments where connections may die silently
- Connections through firewalls that drop idle connections

**When it does not help:**

- Unix socket connections (no TCP stack involved)
- Local TCP connections on loopback (127.0.0.1)

## Performance Impact and Benchmarking

The performance improvement from enabling NGINX FastCGI keepalive depends on your specific workload. Here is what to expect:

### High Impact Scenarios

**High request volume:** Sites handling thousands of requests per second benefit most. Connection overhead is constant per request, so eliminating it multiplies across all requests.

**Short PHP execution times:** When PHP scripts complete in 10-50ms, the 1-2ms connection overhead represents 2-20% of total request time. Eliminating it provides noticeable improvement.

**CPU-constrained servers:** Connection establishment requires CPU cycles for socket operations and memory allocation. Keepalive frees these cycles for actual request processing.

### Lower Impact Scenarios

**Long-running PHP scripts:** If your scripts take 500ms+ to execute, the 1-2ms connection overhead is negligible (0.2-0.4%).

**Low traffic sites:** With few requests per minute, the benefits are minimal, though there is no downside to enabling it.

### Measuring the Improvement

Before enabling keepalive, establish a baseline:

```
# Check current connection states
ss -s

# Check PHP-FPM status if enabled
curl http://localhost/status?full

# Benchmark current performance
ab -n 10000 -c 100 http://localhost/test.php
```

After enabling keepalive, repeat the measurements. You should observe:

- Reduced TIME_WAIT connection states
- Lower CPU usage under the same load
- Improved requests per second in benchmarks
- Reduced average response time

## Troubleshooting Common Issues

### Problem: Intermittent 502 Bad Gateway Errors

**Symptoms:** Random 502 errors that come and go, especially after traffic spikes. For a comprehensive guide to diagnosing these errors, see our article on [500 Internal Server Error in NGINX: PHP-FPM causes](https://www.getpagespeed.com/server-setup/nginx/500-internal-server-error).

**Likely causes:**

1. **PHP-FPM ondemand mode:** Workers exit and invalidate cached connections. Switch to dynamic mode.
2. **pm.max_requests too low:** PHP-FPM workers restart after serving N requests, closing connections. Increase the value or align it with `keepalive_requests`.
3. **Mismatched timeouts:** NGINX keeps connections longer than PHP-FPM keeps workers alive.

**Diagnostic commands:**

```
# Check error logs
tail -f /var/log/nginx/error.log

# Check PHP-FPM worker count
ps aux | grep php-fpm | wc -l
```

### Problem: No Performance Improvement

**Symptoms:** Benchmarks show the same performance before and after enabling keepalive.

**Likely causes:**

1. **Missing one of the required directives:** You need BOTH `keepalive` in the upstream block AND `fastcgi_keep_conn on` in the location.
2. **Using IP address directly instead of upstream:** This bypasses connection pooling:
    

```
# WRONG - no connection pooling
fastcgi_pass 127.0.0.1:9000;

# CORRECT - uses upstream with pooling
fastcgi_pass php-fpm;
```

1. **Bottleneck elsewhere:** If PHP execution time dominates, connection savings are negligible.

### Problem: Memory Usage Keeps Growing

**Symptoms:** NGINX or PHP-FPM memory usage increases over time.

**Solutions:**

1. Lower the `keepalive` pool size
2. Set `keepalive_time` to force periodic connection recycling
3. Reduce `pm.max_requests` in PHP-FPM to restart workers periodically

## NGINX Version Requirements

These directives have different minimum NGINX version requirements:

|Directive|Minimum NGINX Version|Notes|
|---|---|---|
|`fastcgi_keep_conn`|1.1.4 (Sep 2011)|Core FastCGI keepalive|
|`keepalive`|1.1.4 (Sep 2011)|Upstream connection pooling|
|`keepalive_requests`|1.15.3 (Aug 2018)|Per-connection request limit|
|`keepalive_timeout`|1.15.3 (Aug 2018)|Idle connection timeout|
|`keepalive_time`|1.19.10 (Apr 2021)|Maximum connection lifetime|
|`fastcgi_socket_keepalive`|1.15.6 (Nov 2018)|TCP keepalive probes|

Check your installed version:

```
nginx -v
```

On current RHEL-based distributions (Rocky Linux 9, AlmaLinux 9), the system NGINX packages (version 1.20+) support all these directives.

## Production Deployment Checklist

Before deploying NGINX FastCGI keepalive in production, verify these items:

- [ ] Configuration tested with `nginx -t`
- [ ] PHP-FPM uses dynamic or static process manager (not ondemand)
- [ ] `keepalive` value calculated: `pm.max_children / worker_processes`
- [ ] Both `keepalive` and `fastcgi_keep_conn on` are configured
- [ ] `fastcgi_pass` uses an upstream name, not a direct address
- [ ] Baseline performance metrics recorded before deployment
- [ ] Error log monitoring enabled after deployment
- [ ] Rollback plan ready if issues occur

## Summary

Enabling NGINX FastCGI keepalive connections is a straightforward optimization that eliminates connection establishment overhead between NGINX and PHP-FPM. While the concept is simple, correct implementation requires understanding both NGINX upstream configuration and PHP-FPM process management.

The key takeaways from this guide:

1. **Use both directives together:** `keepalive` in the upstream block creates the connection pool. `fastcgi_keep_conn on` in the location block tells PHP-FPM to keep connections open. Both are required.
2. **Size the pool appropriately:** Calculate `keepalive` as `pm.max_children / worker_processes` for optimal resource usage.
    
3. **Match your PHP-FPM process manager:** Dynamic and static modes work well with keepalive. Ondemand mode requires special consideration or should avoid keepalive entirely.
    
4. **Monitor after deployment:** Watch for 502 errors and verify the expected performance improvement through benchmarking.
    

With the production-ready configurations provided in this guide, you can reduce latency, lower CPU usage, and improve the overall throughput of your PHP applications running on NGINX.

**Tuning today fixes today’s problem.** Tomorrow’s regression undoes it silently. [**GetPageSpeed Amplify**](https://amplify.getpagespeed.com/) runs scheduled gixy scans across every host and ties findings to live NGINX runtime metrics. Drop-in compatible with the deprecated `nginx-amplify-agent` (EOL January 2026).

D

Danila Vershinin

Founder & Lead Engineer

NGINX configuration and optimizationLinux system administrationWeb performance engineering

10+ years NGINX experience • Maintainer of GetPageSpeed RPM repository • Contributor to open-source NGINX modules