---
link: https://laurent-bel.medium.com/waiting-for-postgresql-to-start-in-docker-compose-c72271b3c74a
byline: Laurent Bel
site: Medium
date: 2022-08-20T11:27
excerpt: Waiting for PostgreSQL to start in Docker Compose In some cases, you need your PostgreSQL database to be up and running before starting another container such as a web application. Here are different …
twitter: https://twitter.com/@Medium
slurped: 2026-02-16T17:42
title: Waiting for PostgreSQL to start in Docker Compose
tags:
  - postgresql
  - docker_compose
---

In some cases, you need your PostgreSQL database to be up and running before starting another container such as a web application. Here are different options to achieve this.

Press enter or click to view image in full size

## Using depends_on directive is not sufficient

The most basic option is to use the depends_on directive, that let’s you indicate that your web container should be started after your database since it depends on it:
```
services:  
  web:  
    build: .  
    ports:  
      - "80:80"  
    depends_on:  
      - "db"
  db:  
    image: postgres

```



While this is fine, it is not actually sufficient in most case: the web container will only wait for the database container to be running, but it does not mean that the database as started yet !

## Using wait-for-it

Another option is to use the [wait-for-it](https://github.com/vishnubob/wait-for-it) script that will check for TCP connections on port 5432 to be accepted:
```
services:  
  web:  
    build: .  
    ports:  
      - "80:80"  
    depends_on:  
      - "db"  
    command: ["./wait-for-it.sh", "db:5432", "--", "python", "app.py"]
  
  db:  
    image: postgres
    
```


This way, the command “python app.py” will only be executed after TCP port 5432 is available. More [info here](https://docs.docker.com/compose/startup-order/) on the official Docker Compose documentation.

## Using “healthcheck” instruction

Another option, which is actually my preferred one since it is a “native” one, is to use the healthcheck instruction from Docker Compose. It is a simple and efficient mechanism to tell if a container is healthy or not:

```
services:  
  web:  
    build: .  
    ports:  
      - "80:80"  
    depends_on:  
      db:  
        condition: service_healthy
    restart: always
    
  db:  
    image: postgres  
    healthcheck:  
      test: ["CMD-SHELL", "pg_isready -U postgres"]  
      interval: 5s  
      timeout: 5s  
      retries: 5
```

The healthcheck test will rely on [pg_isready](https://www.postgresql.org/docs/current/app-pg-isready.html) utility. You can adjust the interval, timeout and retries to whatever is convenient in your situation. The web container will wait for the database to be healthy.

## Don’t forget to use restart instruction

Another additional trick, that I am using in conjunction with the above is to use the “restart : always” instruction. This way, if my container crashes because the database is not available for example, it will be automatically restarted:
## Conclusion

“Et voilà”, some simple instructions to ensure that your database is available before you start additional containers that rely on it. This example is illustrated with PostgreSQL but you can indeed apply that to other databases whether its MySQL, SQL Server, MariaDB, MongoDB, … and indeed also other services such as Redis, Memcached, … simply be inventive !