---
link: https://ntietz.com/blog/til-docker-log-rotation/
excerpt: Monday, February 16, 2026
slurped: 2026-03-16T14:49
title: "TIL: Docker log rotation | nicole@web"
tags:
  - docker
  - log
---


Docker doesn't automatically rotate log files! As long as a container exists, the logs will keep growing for it. This means even if you stop a container and start it again, the logs are still there and getting bigger. I'd not thought about this before, but it turns out that when my blog is seeing heavy traffic, the logs can grow in order of megabytes per hour. And that really adds up over time.

* * *

First I did a quick check of how the logs are configured to start with. You can see the log configuration by using `docker inspect`.

```
$ docker inspect --format='{{.HostConfig.LogConfig}}' my-poor-container
{json-file map[]}
```

And my container's logging was totally unconfigured! That explained a lot.

Now the fix is pretty quick. The [docs](https://docs.docker.com/engine/logging/drivers/json-file/) show us an example that works well enough here. `/etc/docker/daemon.json` didn't exist yet, so I created it and added this log configuration in.

```
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "3"
  }
}
```

The original example had `10m` for log files, but I want a little more than that. I have the disk space, and I'd like longer to investigate logs before they are truncated away.

After setting that up, I restarted the docker daemon by calling `systemctl restart docker`. But logs don't rotate yet, no! The docs told us that this applies for _new_ containers after Docker is restarted, but not for existing containers. So the final step was to stop and remove any containers I wanted rotation to work on, then recreate them.

After that, a quick check, and we've got log rotation.

```
$ docker inspect --format='{{.HostConfig.LogConfig}}' my-happy-container
{json-file map[max-file:3 max-size:100m]}
```
