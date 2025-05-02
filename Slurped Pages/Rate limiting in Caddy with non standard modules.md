---
link: https://thedevelopercafe.com/articles/rate-limiting-in-caddy-with-non-standard-modules-7cdedba46785
excerpt: In this article you are going to learn how to setup request rate
  limiting in Caddy using a non standard module.
slurped: 2024-05-13T14:41:36.728Z
title: Rate limiting in Caddy with non standard modules
---

In this article you are going to learn how to setup request rate limiting in [Caddy](https://caddyserver.com/) using a non standard module.

This articles assumes you have a basic understanding of how modules work in Caddy. You can read more about modules [here](https://caddyserver.com/docs/extending-caddy).

If you know nothing about modules, you can still follow the article and get rate limiting working.

## Rate Limiting Module [#](https://thedevelopercafe.com/articles/rate-limiting-in-caddy-with-non-standard-modules-7cdedba46785#rate-limiting-module)

You are going to use [https://github.com/mholt/caddy-ratelimit](https://github.com/mholt/caddy-ratelimit) non-standard module to implement rate limiting.

## Building Caddy with Module [#](https://thedevelopercafe.com/articles/rate-limiting-in-caddy-with-non-standard-modules-7cdedba46785#building-caddy-with-module)

When using non-standard module you have to build Caddy along with the module. I am going to use **Docker** and build a custom container image that includes the module.

```
FROM caddy:builder-alpine AS builder

RUN xcaddy build \
		--with github.com/mholt/caddy-ratelimit

FROM caddy:latest

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

```
docker build -t caddy:custom .
```

The above uses the builder image `caddy:builder-alpine`, builds Caddy using the [xcaddy](https://github.com/caddyserver/xcaddy) tool and then copies the built binary in a regular `caddy:latest` image.

`caddy-custom` image contains Caddy along with the rate limiting module.

## Writing Caddyfile [#](https://thedevelopercafe.com/articles/rate-limiting-in-caddy-with-non-standard-modules-7cdedba46785#writing-caddyfile)

Set up a simple Caddyfile with basic response.

```
:80 {
		handle /api* {
				respond "API 200"
		}
}
```

The above returns a simple **API 200** response when sending a request to any endpoint starting with `/api`.

```
curl -i localhost:80/api
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Server: Caddy
Date: Sun, 16 Jul 2023 08:11:03 GMT
Content-Length: 7

API 200
```

## Add Rate Limiting [#](https://thedevelopercafe.com/articles/rate-limiting-in-caddy-with-non-standard-modules-7cdedba46785#add-rate-limiting)

The rate limiting module provides a directive called `rate_limit`.

```
{
		order rate_limit before basicauth
}

:80 {
		rate_limit {
				zone dynamic_zone {
						key {http.request.remote_ip}
						events 2
						window 5s
				}
		}

		handle /api* {
				respond "API 200"
		}
}
```

Run the above `Caddyfile` using the previously built docker image.

```
docker run --rm -v ${PWD}/Caddyfile:/etc/caddy/Caddyfile -p 80:80 caddy:custom
```

Let's break down the above `Caddyfile`.

## Directive Ordering [#](https://thedevelopercafe.com/articles/rate-limiting-in-caddy-with-non-standard-modules-7cdedba46785#directive-ordering)

```
{
		order rate_limit before basicauth
}
```

Since this is a non-standard module, you need to tell Caddy in which order to evaluate the `rate_limit` directive. Read more about directive ordering [here](https://caddyserver.com/docs/caddyfile/directives#directive-order).

## Rate Limit Directive [#](https://thedevelopercafe.com/articles/rate-limiting-in-caddy-with-non-standard-modules-7cdedba46785#rate-limit-directive)

```
rate_limit {
		zone dynamic_zone {
				key {http.request.remote_ip}
				events 2
				window 5s
		}
}
```

The basic configuration of a rate limit includes `key`, `events` and `window`.

- `key` is how you identify/group realted requests together.
- `events` refers to the total number of requests.
- `window` refers to the time period of the rate limit.

You can read the above as "Based on the request **key** allow only **events** requests in a period of **window**."

Fill in the values and it makes more sense, "Based on the request **ip address** allow only **2** request in a period of **5 seconds**".

So if you send more than 2 requests in a period of 5 seconds from the same ip address, you will get back a `429 Too Many Request` error.

![Caddy Rate Limiting](https://tdc-public.nyc3.digitaloceanspaces.com/photos/ebaf8bd7-daa7-42fb-8ca1-8d73d5575e98-caddy-rate-limiting.png)

```
$ curl -i localhost:80/api
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Server: Caddy
Date: Sun, 16 Jul 2023 08:27:30 GMT
Content-Length: 7

API 200

$ curl -i localhost:80/api
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Server: Caddy
Date: Sun, 16 Jul 2023 08:27:30 GMT
Content-Length: 7

API 200

$ curl -i localhost:80/api
HTTP/1.1 429 Too Many Requests
Retry-After: 4
Server: Caddy
Date: Sun, 16 Jul 2023 08:27:31 GMT
Content-Length: 0
```

Read about various other options available in this module [here](https://github.com/mholt/caddy-ratelimit).