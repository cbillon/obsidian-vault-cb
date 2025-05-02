---
tags:
  - docker-compose
  - containers
---
## 1. Overview

In this tutorial, we’ll see how to rebuild a container independently from the others with _[docker-compose](https://www.baeldung.com/ops/docker-compose)_.

## 2. Presentation of the Problem[

**Let’s define a _docker-compose.yml_ configuration file with two containers**: One will refer to the latest _ubuntu_ image and the other one to the latest _alpine_ image. We’ll add pseudo-terminals for each with “_tty: true_” to prevent the containers from exiting directly on launch:

```yaml
version: "3.9"
services:
  ubuntu:
    image: "ubuntu:latest"
    tty: true
  alpine:
    image: "alpine:latest"
    tty: true
```

Let’s now build the containers and start them. We’ll use the _docker-compose up_ command with the _-d_ option to let them run in the background:

```plaintext
$ docker-compose up -d

Container {folder-name}-alpine-1  Creating
Container {folder-name}-ubuntu-1  Creating
Container {folder-name}-ubuntu-1  Created
Container {folder-name}-alpine-1  Created
Container {folder-name}-ubuntu-1  Starting
Container {folder-name}-alpine-1  Starting
Container {folder-name}-alpine-1  Started
Container {folder-name}-ubuntu-1  Started
```

We can quickly check that our containers are running as expected:

```plaintext
$ docker-compose ps
NAME                         COMMAND             SERVICE             STATUS              PORTS
{folder-name}-alpine-1   "/bin/sh"           alpine              running
{folder-name}-ubuntu-1   "bash"              ubuntu              running
```

We’ll now see how we can rebuild and restart the _ubuntu_ container without impacting the _alpine_ container.

## 3. Rebuild and Restart a Container Independently

**Adding the name of the container to the _docker-compose up_ command will do the trick.** We’ll add the _build_ option to build the image before starting the container. We’ll also add the _force-recreate_ flag because we haven’t changed the image:

```plaintext
$ docker-compose up -d --force-recreate --build ubuntu
Container {folder-name}-ubuntu-1  Recreate
Container {folder-name}-ubuntu-1  Recreated
Container {folder-name}-ubuntu-1  Starting
Container {folder-name}-ubuntu-1  Started
```

As we can see, the _ubuntu_ container was rebuilt and restarted without any impact on the _alpine_ container.

## 4. If the Container Depends on Another Container

Let’s now slightly update our _docker-compose.yml_ file to make the _ubuntu_ container depend on the _alpine_ one:

```yaml
version: "3.9"
services:
  ubuntu:
    image: "ubuntu:latest"
    tty: true
    depends_on:
      - "alpine"
  alpine:
    image: "alpine:latest"
    tty: true
```

We’ll **stop the previous containers and rebuild them from scratch** with the new configuration:

```plaintext
$ docker-compose stop
Container {folder-name}-alpine-1  Stopping
Container {folder-name}-ubuntu-1  Stopping
Container {folder-name}-ubuntu-1  Stopped
Container {folder-name}-alpine-1  Stopped

$ docker-compose up -d
Container {folder-name}-alpine-1  Created
Container {folder-name}-ubuntu-1  Recreate
Container {folder-name}-ubuntu-1  Recreated
Container {folder-name}-alpine-1  Starting
Container {folder-name}-alpine-1  Started
Container {folder-name}-ubuntu-1  Starting
Container {folder-name}-ubuntu-1  Started
```

In this case, we need to **add the _no-deps_ option to explicitly tell _docker-compose_ not to restart linked containers**:

```java
$ docker-compose up -d --force-recreate --build --no-deps ubuntu
Container {folder-name}-ubuntu-1  Recreate
Container {folder-name}-ubuntu-1  Recreated
Container {folder-name}-ubuntu-1  Starting
Container {folder-name}-ubuntu-1  Started
```

## 5. Conclusion

In this tutorial, we’ve seen how to rebuild a container with _docker-compose_.

As always, the code is available [over on GitHub](https://github.com/Baeldung/ops-tutorials/tree/main/docker-modules/docker-compose).