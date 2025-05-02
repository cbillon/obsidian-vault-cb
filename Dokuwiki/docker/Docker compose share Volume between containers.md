---
tags:
  - docker
---
## 1. Overview

Docker containers are isolated environments. However, containers sometimes need to persist and share data. It may happen when a second container needs to access a shared cache or use database data. We may also need to backup or perform operations on user-generated data.

In this short tutorial, we’ll see how to share data between Docker containers with an example using Docker Compose.

## 2. Persist and Share Data with Docker Storage

**When containers run, all files get a writable space. However, they no longer exist once we stop the container.**

[Docker](https://docs.docker.com/) uses [Storage](https://docs.docker.com/storage/) with persistent and in-memory options if we need to save our data.

Storing files also improves performance because it writes directly to the host filesystem instead of using the container’s writable layer.
### 2.1. Docker Volumes

Let’s have a quick look at [Docker Volumes](https://www.baeldung.com/ops/docker-volumes). For example, let’s run an Nginx container with a named volume.

First, let’s [create](https://docs.docker.com/engine/reference/commandline/volume_create/) our volume:

```bash
docker volume create --name volume-data
```

Then, let’s run our container:

```bash
docker run -d -v volume-data:/data --name nginx-test nginx:latest
```

In this case, Docker will mount in the container’s _/data_ folder. It will also **copy the directory’s contents into the volume if the container has files or directories in the path to be mounted.**

We can also have a look at [_bind mounts_](https://docs.docker.com/storage/bind-mounts/) for persistent storage.
### 2.2. Share Data with Volumes

**Multiple containers can run with the same volume when they need access to shared data.**

For example, let’s start our web application:

```bash
docker run -d -v volume-data:/usr/src/app/public --name our-web-app web-app:latest
```

Docker creates a _local_ volume by default. However, we can use a [volume diver](https://docs.docker.com/storage/volumes/#use-a-volume-driver) to share data across multiple machines.

Finally, Docker also has [_–volumes-from_](https://docs.docker.com/storage/volumes/#backup-restore-or-migrate-data-volumes) to link volumes between running containers. It may help for data sharing or more general backup usage.

## 3. Share Data with Docker Compose

We’ve seen how to create volumes with Docker. **[Docker Compose](https://www.baeldung.com/ops/docker-compose) also supports the _[volumes](https://docs.docker.com/compose/compose-file/compose-file-v3/#volume-configuration-reference)_ keyword within the YAML template definition.**

Let’s create a _docker-compose.yml_ to run both an Nginx container and our web application sharing the same volume:

```yaml
services:
  nginx:
    container_name: nginx
    build: ./nginx/
    volumes:
      - shared-volume:/usr/src/app

  web-app:
    container_name: web-app
    env_file: .env
    volumes:
      - shared-volume:/usr/src/app/public
    environment:
      - NODE_ENV=production

volumes:
  shared-volume:
```

Again, in Docker Compose, the default _driver_ will be _local_. We can also specify a driver to use for this volume:

```yaml
volumes:
  db:
    driver: some-driver
```

We may also need to use a volume external to Docker Compose:

```yaml
volumes:
  data:
    external: true
    name: shared-data
```

## 4. Conclusion

In this article, we’ve seen how to share Docker containers’ data using volumes. We have also seen the same concept with a simple example using Docker Compose.