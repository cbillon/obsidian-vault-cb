---
link: https://erseco.github.io/alpine-moodle/
excerpt: Lightweight Moodle Docker image built on Alpine Linux — official documentation.
slurped: 2026-10-14T18:45
title: Home - Alpine Moodle
---

[](https://github.com/erseco/alpine-moodle/edit/main/docs/index.md "Edit this page")

A lightweight **Moodle** Docker image built on [Alpine Linux](https://alpinelinux.org/).

[![Docker Pulls](https://img.shields.io/docker/pulls/erseco/alpine-moodle.svg)](https://hub.docker.com/r/erseco/alpine-moodle/) ![Docker Image Size](https://img.shields.io/docker/image-size/erseco/alpine-moodle) ![License MIT](https://img.shields.io/badge/license-MIT-blue.svg)

## What is this image?[¶](https://erseco.github.io/alpine-moodle/#what-is-this-image "Permanent link")

`erseco/alpine-moodle` packages Moodle into a single, small (~100 MB) container based on [`erseco/alpine-php-webserver`](https://github.com/erseco/alpine-php-webserver). It runs Nginx + PHP-FPM under a non-privileged user, includes [Moosh CLI](https://github.com/tmuras/moosh), and is configured entirely through environment variables.

Highlights:

- PHP 8.4 FPM with `ondemand` process manager — legacy Moodle tags retain PHP 8.3 ([runtime policy](https://erseco.github.io/alpine-moodle/php84/))
- Works with PostgreSQL, MariaDB/MySQL, or SQLite (single-container dev mode)
- Optional Redis session handler for HA deployments
- Supports Moodle **4.x**, **5.0**, **5.1** (with `/public` directory) and `main`
- **Version-aware code sync** (`SYNC_MOODLE_CODE=auto`) — upgrade Moodle core by changing the image tag even with a persistent `moodlehtml` volume ([Upgrading](https://erseco.github.io/alpine-moodle/upgrading/), [#103](https://github.com/erseco/alpine-moodle/issues/103))
- Multi-arch images: `amd64`, `arm64`, `arm/v7`, `arm/v6`, `386`, `ppc64le`, `s390x`
- Internal cron via `runit` (configurable)
- Logs go straight to `docker logs`
- Extensible via pre/post configuration hooks and `POST_CONFIGURE_COMMANDS`

## Where to start[¶](https://erseco.github.io/alpine-moodle/#where-to-start "Permanent link")

- **[Quick Start](https://erseco.github.io/alpine-moodle/quick-start/)** — get a Moodle instance running in under a minute.
- **[Docker Compose](https://erseco.github.io/alpine-moodle/docker-compose/)** — practical stacks for dev, production and proxied setups.
- **[Reverse Proxy](https://erseco.github.io/alpine-moodle/reverse-proxy/)** — Traefik, Nginx, NPM, Apache, Caddy recipes.
- **[Environment Variables](https://erseco.github.io/alpine-moodle/environment-variables/)** — every supported knob, with defaults.
- **[Persistence & Volumes](https://erseco.github.io/alpine-moodle/persistence/)** — what to mount and what to back up.
- **[PHP runtime policy](https://erseco.github.io/alpine-moodle/php84/)** — PHP 8.4 for 5.3+, legacy tags retain PHP 8.3.
- **[Upgrading](https://erseco.github.io/alpine-moodle/upgrading/)** — how to move between Moodle versions safely.
- **[Troubleshooting](https://erseco.github.io/alpine-moodle/troubleshooting/)** — solutions to the most common deployment issues.
- **[FAQ](https://erseco.github.io/alpine-moodle/faq/)** — short answers to recurring questions.

## Minimal example[¶](https://erseco.github.io/alpine-moodle/#minimal-example "Permanent link")

docker run -d \
  -p 80:8080 \
  -e MOODLE_DATABASE_TYPE=sqlite3 \
  -e MOODLE_PASSWORD=ChangeMe123! \
  -v moodledata:/var/www/moodledata \
  erseco/alpine-moodle``


Open [http://localhost](http://localhost)
  and log in with `moodleuser` / `ChangeMe123!`.

SQLite is for development and demos only

Use PostgreSQL or MariaDB for any real deployment. See [Docker Compose](https://erseco.github.io/alpine-moodle/docker-compose/) for production-grade examples.

## Project links
[¶](https://erseco.github.io/alpine-moodle/#project-links "Permanent link")

- Source code: [https://github.com/erseco/alpine-moodle](https://github.com/erseco/alpine-moodle)
- Docker Hub: [https://hub.docker.com/r/erseco/alpine-moodle](https://hub.docker.com/r/erseco/alpine-moodle)
- GitHub Container Registry: `ghcr.io/erseco/alpine-moodle`
- Issue tracker: [https://github.com/erseco/alpine-moodle/issues](https://github.com/erseco/alpine-moodle/issues)