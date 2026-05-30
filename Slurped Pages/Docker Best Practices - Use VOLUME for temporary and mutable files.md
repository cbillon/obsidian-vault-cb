---
link: https://gagor.pro/2022/09/docker-best-practices-use-volume-for-temporary-and-mutable-files/
byline: Tom
site: Tom's Blog
date: 2022-09-12T02:00
excerpt: Learn best practices for writing Dockerfiles by using VOLUME for all mutable, temporary file locations to enhance performance and maintain cleaner images.
tags:
  - docker
  - dockerfile
  - best_pratices
slurped: 2026-05-27T09:02
title: Docker Best Practices - Use VOLUME for temporary and mutable files
---

IMO people don’t understand how `VOLUME`[1](#fn:1) works so they don’t use it. It’s generally used far too rarely!
[doc volume](https://docs.docker.com/engine/reference/builder/#volume)

In short `VOLUME` means two things:

1. Whatever is left in directory marked as `VOLUME`, stays there and can’t be changed in later layers (actually it can be changed but changes won’t be persistent).
2. Volumes are not part of layered image FS. They’re mounted as anonymous volumes located on standard file system. This means they’re working much faster.

Let me explain it a bit.

## Use VOLUME for temporary files directories

That’s one of things that piss me off in the [official base images](#dont-rely-on-official-base-images) . They just don’t set up volumes for temporary directories by default. That’s why in my base images I always have something like that, somewhere at beginning:

```
# on Debian
VOLUME ["/tmp", "/var/tmp", "/var/cache/apt", "/var/lib/apt/lists"]

# on CentOS
VOLUME ["/tmp", "/var/tmp", "/var/cache/yum", "/var/cache/dnf"]
```

What it does? It keeps images clean. Earlier, I was installing packages like this:

How everybody do it - without VOLUMEs

```
FROM debian

RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

Debian by default saves package lists (that’s what `apt-get update` downloads) in `/var/cache/apt/lists`. Then it’s downloading `deb` packages to `/var/cache/apt/archives` and all of this rubbish will be left in image by default, unless you remove it. But by marking `/var/cache/apt` as a `VOLUME`, all content became immutable and is removed/dropped at the end of each directive (like each RUN). So I can do it right now like this:

How I do it - with VOLUMEs

```
FROM debian

VOLUME ["/tmp", "/var/tmp", "/var/cache/apt", "/var/lib/apt/lists"]

RUN apt-get update && \
    apt-get install -y nginx
```

It just makes life easier 😄

## Add clean-up task for anonymous volumes

Info

Sadly, those anonymous volumes are just left after you deploy new image version or shut it down. You have to clean them from time to time!

Which you probably should already know, if you manage cluster of docker hosts. It might be as simple as running in cron:

```
docker volume prune -f

```
