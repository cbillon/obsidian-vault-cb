---
link: https://blog.wijman.net/automatic-docker-images-update/
byline: Stephan Wijman
site: Stephan Wijman's Blog
date: 2018-02-16T14:16
excerpt: |-
  Docker images are a good way of separating application code and keep a system
  clean. The downside is that Docker doesn't support an automated method to update
  images. This procedure describes installing an extra image which will monitor
  all running images for updates.

  WatchTower [https://hub.docker.com/r/v2tec/watchtower/] will do the following:

   1. Monitor all running containers
   2. Download updates when available
   3. Restart the container(s) with the new image(s)

  Information

  This procedure
slurped: 2024-04-28T15:57:22.626Z
title: Automatic Docker Images Update
tags:
  - docker
  - image
  - update
---

Docker images are a good way of separating application code and keep a system clean. The downside is that Docker doesn't support an automated method to update images. This procedure describes installing an extra image which will monitor all running images for updates.

[WatchTower](https://hub.docker.com/r/v2tec/watchtower/?ref=blog.wijman.net) will do the following:

1. Monitor all running containers
2. Download updates when available
3. Restart the container(s) with the new image(s)

Information

This procedure expects that docker is already installed and running.

Warning!

The WatchTower image requires access to the docker socket (`/var/run/docker.sock`) to monitor the running containers.

### Steps to install WatchTower

1. Download the WatchTower image:

```

docker pull v2tec/watchtower
```

2. Build the container:

```

docker create --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  v2tec/watchtower
```

Information

This is the default running mode, which will do the following:

- Monitor all running containers
- Check for updates every 5 minutes

Note

Additional examples are shown below.

Start the container:

```

docker start watchtower
```

### Additional Arguments

The following table shows the available arguments which are available:

|Argument|Description|
|---|---|
|--host / -h|Docker daemon socket to connect to. Defaults to "unix:///var/run/docker.sock" but can be pointed at a remote Docker host by specifying a TCP endpoint as "tcp://hostname:port".|
|--interval / -i|Poll interval (in seconds). This value controls how frequently watchtower will poll for new images. Defaults to 300 seconds (5 minutes).  <br>Mutual exclusive with --schedule|
|--schedule / -s|Cron expression which defines when and how often to check for new images.  <br>Mutual exclusive with --interval|
|--no-pull|Do not pull new images.|
|--cleanup|Remove old images after updating.|
|--tlsverify|Use TLS when connecting to the Docker socket and verify the server's certificate.|
|--debug|Enable debug mode.|
|--help|Show documentation about the supported flags.|

### Additional example for WatchTower arguments

The examples below show different combinations of arguments with explanation of the behaviour.

#### Example 1:

```

docker create --name watchtower --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  v2tec/watchtower
```

Behaviour:

- Monitor all running containers
- Check for updates every 5 minutes
- Restart automatically on docker start

#### Example 2:

```

docker create --name watchtower--restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  v2tec/watchtower --cleanup
```

Behaviour:

- Monitor all running containers
- Check for updates every 5 minutes
- Restart automatically on docker restart
- Cleanup old images after update

#### Example 3: ( my preferred method)

```

docker create --name watchtower --restart always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  v2tec/watchtower --cleanup --interval @daily
```

Behaviour:

- Monitor all running containers
- Check for updates daily (at 00:00)
- Restart automatically on docker restart
- Cleanup old images after update

#### Example 4:

```

docker create --name watchtower \
  -v /var/run/docker.sock:/var/run/docker.sock \
  v2tec/watchtower --cleanup collabora
```

Behaviour:

- Monitor only collabora container
- Check for updates every 5 minutes
- Cleanup old images after update