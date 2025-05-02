---
link: https://hackernoon.com/how-to-setup-cron-and-docker-correctly-a-how-to-guide-9v5f36kz
byline: Igor Fomin
date: 2020-01-29T00:00
excerpt: What do I do if I need to use linux cron and execute command in one of my containers?
slurped: 2024-06-09T16:04:39.188Z
title: How To Setup Cron And Docker Correctly [A How To Guide]
tags:
  - docker
  - cron
  - tips
---

What do I do if I need to use linux cron and execute command in one of my containers?

When I started to google this, I found 2 solutions right away, but both seemed quite bad:

## Bad option 1:

Build custom crontab image, copy crontab file inside, and then run a command towards other container.

**Problem with this approach is** - I will need to rebuild this image every time I need to change my cron command or add a new one. Also it will be quite problematic to make calls from my cron container to other containers, where I really need command to run.

## Bad option 2:

Modify container to run crontab.

**Problem with this approach is** - my container will have to run 2 commands (crontab + whatever container used to run) - this contradicts philosophy of docker, each container is supposed to be responsible only for 1 task.

## Good option:

Run cron on host machine where I start my docker containers.

I can do it using command like this, right in crontab:

```
* * * * * docker exec -t {containerID} {command} >> /dev/null 2>&1
```

For example:

```
* * * * * docker exec -t {containerID} {command} >> /dev/null 2>&1

For example:
* * * * * docker exec -t $(docker ps -qf "name=docker_php_1") php artisan schedule:run >> /dev/null 2>&1
```

**Notes:**

- normally I would run "**docker exec -t <container_id> php artisan schedule:run**" - but container id will change every time I restart container, and I do not want to change crontab everytime
- $(**docker ps -qf "name=docker_php_1"**) - allows to get container id based on search by container name
- $(**docker ps -qf "ancestor=php:7.2-fpm"**) - allows to get container id based on search by image name
- **php artisan schedule:run** - Laravel specific command to run scheduled tasks, used here just as example