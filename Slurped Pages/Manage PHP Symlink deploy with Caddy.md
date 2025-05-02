---
link: https://endelwar.it/2020/09/manage-php-symlink-deploy-with-caddy-2.2/
excerpt: Sending PR to OSS projects for fun and profit
slurped: 2024-05-13T13:57:29.842Z
title: Manage PHP Symlink deploy with Caddy 2.2
---

I use [Deployer](https://deployer.org/) as the deployment tool for websites and web apps that I develop: it’s a fine piece of software that permits releasing software very quickly, and the whole process becomes a no-brainer.

Deployer uses the following directory structure to organize your code:

```
.
├── current -> releases/91
├── releases
│   ├── 87
│   ├── 88
│   ├── 89
│   ├── 90
│   └── 91
└── shared
    ├── config
    ├── private
    ├── public
    └── var
```

The last 5 (quantity is configurable) releases are in `releases` directory, and the last one is symlinked to `current`, which, probably, contains `public` directory that is the root directory for your web server.

Because of PHP OPCode caching, the root path, i.e. `current` directory as seen above, is cached and won’t change until PHP-FPM daemon is restarted or cache is flushed in some other way[1](#fn:1).

This could be handled by the webserver by automatically resolving `current` symlink before passing the information to PHP-FPM but [Caddy](https://caddyserver.com/) don’t do this out-of-the-box. A couple of months ago [I submitted a PR](https://github.com/caddyserver/caddy/pull/3587) that adds this functionality to Caddy’s `php_fastcgi` directive by adding a new `resolve_root_symlink` subdirective:

```
example.test {
    root * /var/www/deploy/current/public
    php_fastcgi 127.0.0.1:9000 {
        resolve_root_symlink
    }
}
```

The PR was merged and will be part of upcoming Caddy 2.2 release, go [check the last beta](https://github.com/caddyserver/caddy/releases) to test this new feature.

> UPDATE: Caddy 2.2.0 was released on 24/Sept/2020, go get it while it’s hot!

It was fun working with [Go](https://golang.org/) (it’s not my primary development language) and the process of submitting an idea first, and a PR after positive feedback was satisfying.

I like working with open source software, especially when you get your hands dirty by fixing a bug or adding a feature that’s important for you, while giving back to community.