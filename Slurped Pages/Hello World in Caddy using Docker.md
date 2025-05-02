---
link: https://thedevelopercafe.com/articles/hello-world-in-caddy-using-docker-c8b18ee67f2f
excerpt: In this article I am going to show you how to run a simple webserver
  using Caddy and Docker.
slurped: 2024-05-13T14:27:58.518Z
title: Hello World in Caddy using Docker
---

In this article I am going to show you how to run a simple webserver using Caddy and Docker.

## Caddyfile 

Let's write a `Caddyfile` that returns a simple "Hello World".

```
:80 {
	respond "Hello World"
}
```

Now using **docker**, run the `caddy:latest` container while mounting the above `Caddyfile` as a volume.

```
docker run --rm --name caddy -p 8080:80 -v ${PWD}/Caddyfile:/etc/caddy/Caddyfile caddy:latest
```

## Docker Compose 

When developing locally you would want to run Caddy using Docker Compose. Using the above `Caddyfile`, let's write a `docker-compose.yaml` file.

```
version: '3.8'

services:
  caddy:
    image: caddy:latest
    ports:
      - 8080:80
    volumes:
      - ${PWD}/Caddyfile:/etc/caddy/Caddyfile
```

Now run using `docker compose`.

```
docker compose up -d
```

Once the container is running test it out using curl.

```
curl localhost:8080
Hello World
```

**Reference Links**

[Caddy](https://caddyserver.com/v2)  
[Docker Compose](https://docs.docker.com/compose/)