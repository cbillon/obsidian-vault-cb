---
link: https://fariszr.com/set-cache-header-caddy-files/
site: FarisZR
date: 2022-09-30T02:00
excerpt: A quick way to set the Cache-Control ; header in Caddy for static files
slurped: 2024-05-13T15:07:21.151Z
title: How to set "Cache-Control" Header for static files in Caddy
---
## Cache-Control

To set a `Cache-Control` header only for your static files like, .js, .jpg, and others, just add this to the relevant part of your Caddyfile(under the site block if the Caddyfile includes multiple websites)
```
@static {
  file
  path *.ico *.css *.js *.gif *.webp *.avif *.jpg *.jpeg *.png *.svg *.woff *.woff2
}
header @static Cache-Control max-age=5184000

```
The files are going to be cached for `5184000` seconds, so 60 days, you can change it if you want.

## Source

[https://caddy.community/t/correct-way-to-set-expires-on-caddy-2/7914/13](https://caddy.community/t/correct-way-to-set-expires-on-caddy-2/7914/13)