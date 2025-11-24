---
link: https://dev.to/iacons/faster-docker-builds-with-composer-install-3opj
byline: Iacovos Constantinou
site: DEV Community
date: 2021-10-28T10:17
excerpt: How to optimize your docker build time when using composer install
twitter: https://twitter.com/@iacons
tags:
  - docker
  - composer
slurped: 2025-11-21T10:05
title: Faster Docker builds with composer install ⚡
---


When working with PHP projects and Docker, it is a good practice to include all the third party dependencies coming from `composer install`.

This really helps to reduce the time needed to spin off a new container. However, when not done right, it can significantly increase the build time.

Throughout this post we will see how we can avoid this and optimize `composer install` for docker builds.

Consider the following simple example, where we copy our source code before executing `composer install`  

```
WORKDIR /my-cool-app
COPY . ./
RUN composer install --no-dev --no-interaction --optimize-autoloader
```

 

While this works, there is a major flaw; we don't leverage Docker cache. Every time there is a change in our source code, the docker cache for `composer install` will be invalidated.

In real life, this will happen with almost every build, causing `composer install` to be executed even when there are no changes in our dependencies.

Not efficient, that's for sure. But how can we avoid this? Consider the following example:  

```
WORKDIR /my-cool-app
COPY composer.json ./
COPY composer.lock ./
RUN composer install --no-dev --no-interaction --no-autoloader --no-scripts
COPY . ./
RUN composer dump-autoload --optimize
```

 

Here, we are copying only `composer.json` and `composer.lock` (instead of copying the entire source) right before doing `composer install`. This is enough to take advantage of docker cache and `composer install` will be executed only when `composer.json` or `composer.lock` have indeed changed!

One downside is that we have to skip autoload generation and script execution since the source code is not available yet. Easy fix though. This can be done as the last step and it shouldn't affect the performance anyway.

As part of this post we saw how we can re-structure our Dockerfile for PHP projects and take better advantage of Docker cache. I hope that's helpful for your project!
