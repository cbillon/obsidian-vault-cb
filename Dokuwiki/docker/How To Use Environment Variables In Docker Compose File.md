---
link: https://hackernoon.com/how-to-use-environment-variables-in-docker-compose-file-l2n32ou
byline: Igor Fomin
date: 2020-02-26T00:00
excerpt: If I want to use different credentials on different servers, environment variables work great with docker compose.
slurped: 2024-06-09T15:58:17.156Z
title: How To Use Environment Variables In Docker Compose File
tags:
  - docker-compose
  - variable
  - tips
---

**Notes**:

- HTTP_USER=${REDIS_USER:-admin} - here I use ${REDIS_USER:-admin} syntax, which means - use REDIS_USER variable from .env file, in case it is missing, use "admin" instead.
- .env file gets picked up automatically when running "docker-compose up -d", there is no need to specify it somehow

If I now try this docker-compose.yml file without .env file, credentials to redis-commander will be **admin/qwerty**.

If I create .env file next to my yml file and put the following inside, credentials to redis-commander will be **test/1234**:

```
REDIS_USER=test 
```

```
REDIS_PASSWORD=1234
```