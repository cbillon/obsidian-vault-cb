---
link: https://hackernoon.com/how-to-extend-docker-compose-file-jc723ypq
byline: Igor Fomin
date: 2020-03-05T00:00
excerpt: |-
  How to use different/multiple docker compose files on different servers
  ?
slurped: 2024-06-09T16:12:08.409Z
title: How To Extend Docker Compose File
tags:
  - docker-compose
  - extension
---

How to use different/multiple docker compose files on different servers ?

There is a page in official documentation that describes this: [https://docs.docker.com/compose/extends/](https://docs.docker.com/compose/extends/?ref=hackernoon.com)

However, this page only describes first 2 options, and completely skips option 3, which is the most versatile and convenient.

It comes down to the following options:

## 1. docker-compose.override.yml

By default, when you execute -

```
docker-compose up
```

- Docker compose reads two files, a **docker-compose.yml** and an optional **docker-compose.override.yml**.

So I might have only **docker-compose.yml** on my local machine, and **docker-compose.yml** + **docker-compose.override.yml** - on my dev/prod servers.

This is not very convenient, and will be difficult to have correct **docker-compose.override.yml** in my repository, especially when my servers will have differences in setup.

## 2. Override syntax:

```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

This is much better, and I can keep multiple different override files, like **docker-compose.prod.yml**, **docker-compose.dev.yml**, etc

But there are 2 problems with the above approach:

- files are combined at runtime, so I do not see resulting merged docker-compose.yml file, and can not verify correctness of the merge
- this syntax does not work in docker swarm mode

## 3. Docker compose config command

```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml config > docker-compose.stack.yml
```

This works the same exact way as the above 2 options, except it provides me with the resulting yml file -

```
docker-compose.stack.yml
```

, which I can review and check for correctness. Also now I can use this approach in docker swarm.

Now I can use it like:

```
docker-compose -f docker-compose.stack.yml up -d
```

Or in swarm:

```
docker stack deploy -c docker-compose.stack.yml mystack
```

Stay Tuned!