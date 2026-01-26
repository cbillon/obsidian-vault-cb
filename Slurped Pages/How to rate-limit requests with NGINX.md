---
link: https://joshtronic.com/2025/11/09/rate-limiting-nginx/
byline: Josh Sherman
excerpt: |-
  Posted 09-Nov-2025
              
              in Servers / Serverless
              
                tagged
                
                  Web Servers
slurped: 2026-01-22T10:10
title: How to rate-limit requests with NGINX
tags:
  - nginx
---

Posted 09-Nov-2025 in [Servers / Serverless](https://joshtronic.com/category/servers-serverless) tagged [Web Servers](https://joshtronic.com/tag/web-servers)

This so-called "AI Era" is a fucking mess. The web is full of low-quality slop articles and made up imagery. [Self-hosting](https://joshtronic.com/2025/07/06/technical-independence/) is more of an exercise in DDoS mitigation than anything else.

What a time to be alive.

## All you do is take, take, take

Due to these disrespectful AI company scrapers, I've been through a handful of completely unnecessary issues during my self hosting journey. Pegged CPUs and full disks are the norm if you let these companies have their way. Doesn't even matter that you've tried to communicate same sanity via `robots.txt`.

I could easily outsource a lot of these issues over to Cloudflare, but that defeats the purpose of self-hosting for me. I'd much prefer to omit the additional layers they provide. While miniscule, they do add latency, and vendor lock in. While I don't run my own DNS server(s) yet, I'm utilizing more traditional providers.

## One step forward, two steps back

Initially I went a bit scorched-earth, [rigidly blocking a ton of known user-agents](https://joshtronic.com/2025/08/03/wordpress-login-security-fail2ban/). This always felt like a losing battle, as there's a new scraper born every minute. Not just that, there's no guarantee that some of the sketchier companies won't just change their user-agent strings to get around this kind of protection.

Not completely satisfied with that solution, I decided to take a different approach. Where user-agents themselves are mutable and not guaranteed, the velocity at which my sites are crawled is not. I've since changed course and moved away from caring about the user-agents directly, and implemented some rate limiting within NGINX.

## Burstable rate-limiting to the rescue

NGINX has a lovely [rate-limiting module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html), that is typically precompiled with it. This module lets you configure the limits, how to handle bursts, and custom return codes. If desired, you can even configure different rates for different routes.

For the sake of what I am trying to accomplish, implementing system wide rate limits was the quickest and easiest approach. To do so, I added a file named `limits.conf` to `/etc/nginx/conf.d/` which automatically gets sourced from within `nginx.conf`. In that file, I added the following lines:

```
limit_req_zone $binary_remote_addr zone=req_per_ip:20m rate=5r/s;
limit_req zone=req_per_ip burst=20;
limit_req_status 429;
```

What this allows is up to 5 requests a second without any penalty. All IP addresses and how many requests they've made will be tracked to a 20 MB buffer, enough for like ~320k IP addresses.

If you exceed the 5 requests per second, we won't immediately block you either. This is definitely an option, but I did want to allow for bursts because _shit happens_.

Once you reach the limit, we'll allow you to make an additional 20 requests before serving up a lovely `429 Too Many Requests` response.

Don't forget to `nginx -t` and restart your server/service.

## Test that it's working as expected

This is all well and good, but waiting until some sketchy spider comes and decimates your box isn't a good test strategy. Instead, you can reach for the [Apache HTTP server benchmarking tool](https://httpd.apache.org/docs/2.4/programs/ab.html), or `ab` for short.

### Happy path

First test, aside from just clicking around on your site, as fast as possible, would be to test up to the rate limit - 1,000 requests, 5 concurrent:

```
ab -n 1000 -c 5 https://joshtronic.com/
```

It takes a minute, feel free to adjust the `-n`. If all goes according to plan, you should see `Failed requests: 0` showing that no requests failed at that clip.

### Sad path

To simulate a bad actor, simply push up the concurrency:

```
ab -n 1000 -c 50 https://joshtronic.com/
```

This will run a lot faster, due to the higher concurrency. Depending on which concurrency value you use, you'll see some failed requests. Push it high enough, and you'll see most requests failing.

This should be enough peace of mind to know that things are working as expected.