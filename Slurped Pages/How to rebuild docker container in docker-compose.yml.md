---
link: https://stackoverflow.com/questions/36884991/how-to-rebuild-docker-container-in-docker-compose-yml
byline: |-
  yuklia
          
              8,02355 gold badges2323 silver badges2727 bronze badges
site: Stack Overflow
excerpt: |-
  There are scope of services which are defined in docker-compose.yml. These services have been started. I need to rebuild only one of these and start it without up other services.
  I run the following
slurped: 2025-12-07T10:51
title: How to rebuild docker container in docker-compose.yml?
tags:
  - docker-compose
---

There are scope of services which are defined in docker-compose.yml. These services have been started. I need to rebuild only one of these and start it without up other services. 

```
$ docker compose up -d --no-deps --build <service_name>
```

> --no-deps - Don't start linked services.

> --build - Build images before starting containers.


```
Options:
    -d, --detach        Detached mode: Run containers in the background,
                        print new container names. Incompatible with
                        --abort-on-container-exit.
    --no-deps           Don't start linked services.
    --force-recreate    Recreate containers even if their configuration
                        and image haven't changed.
    --build             Build images before starting containers.
```

## Without cache

To force a rebuild to ignore cached layers, we have to first [build](https://docs.docker.com/compose/reference/build/) a new image

```
docker compose build --no-cache [<service_name>..]
```

From the help menu

```
Options:
    --force-rm              Always remove intermediate containers.
    -m, --memory MEM        Set memory limit for the build container.
    --no-cache              Do not use cache when building the image.
    --no-rm                 Do not remove intermediate containers after a successful build.
```

Then recreate the container

```
docker compose up --force-recreate --no-deps [-d] [<service_name>..]
```


This should fix your problem:

```
docker-compose ps # lists all services (id, name)
docker-compose stop <id/name> #this will stop only the selected container
docker-compose rm <id/name> # this will remove the docker container permanently 
docker-compose up # builds/rebuilds all not already built container 
```


if you want to `stop` and `rm` all containers then you can use the command `docker-compose down`. Your solution is better if you only want to get rid of some.


Simply use :

```
docker-compose build [yml_service_name]
```

Replace `[yml_service_name]` with your service name in `docker-compose.yml` file. You can use `docker-compose restart` to make sure changes are effected. You can use `--no-cache` to ignore the cache.
