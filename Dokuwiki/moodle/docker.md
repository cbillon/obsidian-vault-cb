---
tags:
  - moodle
  - docker
---

====== Moodle Docker ======

[[https://github.com/jmhardison/docker-moodle.git]]

A Dockerfile that installs and runs the latest Moodle stable, with external MySQL Database. 
Installé test OK  

[[https://github.com/zakery1369/moodle/]]

** manque le Dockerfile

[[https://github.com/42ae/moodle-docker]]

3 container : mysql moodle phpmyadmin cron

The Moodle cron process is a PHP script (part of the standard Moodle installation) that must be run regularly in the background. The Moodle cron script runs different tasks at differently scheduled intervals.

In order to run cron, we use the useful [[https://github.com/funkyfuture/deck-chores|funkyfuture/deck-chores]] image which allows us to define regular cron jobs to run within a container context via container labels.

    labels:
      cron.moodle.command: "/usr/local/bin/php /var/www/html/admin/cli/cron.php"
      cron.moodle.interval: "every minute"

[[https://github.com/erseco/alpine-moodle/]]

[[https://github.com/fondation-unit/docker-moodle]]

[[https://github.com/jobcespedes/docker-compose-moodle]]

