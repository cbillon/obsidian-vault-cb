---
link: https://github.com/Hipjea/docker-moodle
site: GitHub
excerpt: A Docker Compose stack for deploying Moodle with a Traefik proxy. - Hipjea/docker-moodle
twitter: https://twitter.com/@github
slurped: 2026-03-31T19:48
title: "GitHub - Hipjea/docker-moodle: A Docker Compose stack for deploying Moodle with a Traefik proxy."
tags:
  - moodle
  - traefik
  - docker_compose
---

A Docker Compose stack for deploying Moodle with a Traefik proxy.

## Docker Compose breakdown

[](https://github.com/Hipjea/docker-moodle#docker-compose-breakdown)

### Stacks:

[](https://github.com/Hipjea/docker-moodle#stacks)

- `moodle` - The Moodle PHP-FPM application container
- `moodle-web` - Nginx container exposing Moodle to Traefik
- `db` - MariaDB container for Moodle's database
- `redis` - Redis container for caching and session storage
- `traefik` - Traefik reverse proxy handling routing and HTTPS

### Volumes:

[](https://github.com/Hipjea/docker-moodle#volumes)

|Volume|Purpose|
|---|---|
|`moodle_src`|The Moodle application source code. Mounted inside the Moodle container at `/var/www/html`.|
|`moodle_data`|The moodledata (cache, sessions, files, uploaded content). Mounted at `/var/moodledata` in the Moodle container.|
|`db_data`|The MariaDB database persistence.|
|`redis_data`|Redis data persistence (if configured) for caching and session storage.|

### Moodle sources:

[](https://github.com/Hipjea/docker-moodle#moodle-sources)

Download and place the Moodle source files in the [moodle/src](https://github.com/Hipjea/docker-moodle/blob/main/moodle/src) folder.

You should have a structure like: `moodle/src/[ Moodle sources files]`.

## Installation

[](https://github.com/Hipjea/docker-moodle#installation)

1. Copy the sample file [.env-sample](https://github.com/Hipjea/docker-moodle/blob/main/.env-sample) as `.env` and fill in the missing information.
    
2. Create the network:
    

docker network create web

3. Prepare the Docker stack:

- Edit your `.env` file
- Configure the `traefik/acme.json` file with the right permissions:

chmod 600 traefik/acme.json
chown $USER:$USER traefik/acme.json

- Download and prepare Moodle sources:

wget -P moodle/src https://download.moodle.org/download.php/direct/stable501/moodle-latest-501.tgz

tar -xf moodle/src/moodle-latest-501.tgz -C moodle/src

mv moodle/src/moodle/{*,.*} moodle/src/ 2>/dev/null && \
  rmdir moodle/src/moodle 2>/dev/null && \
  rm moodle/src/moodle-latest-*.tgz

4. Build the stack:

docker compose up --build

5. Launch the app:

## Traefik

[](https://github.com/Hipjea/docker-moodle#traefik)

Uncomment the following lines in `docker-compose.yml` to let Traefik display debug logs:

- "--log.level=DEBUG"
- "--accesslog=true"
- "--accesslog.format=json"

## Database

[](https://github.com/Hipjea/docker-moodle#database)

Log into the MariaDB container:

docker compose exec db mariadb -u moodle -p moodle

## Stats

[](https://github.com/Hipjea/docker-moodle#stats)

Real time stats:

Observe dockerd processes:

ps aux | grep dockerd | grep -v grep

Memory use by dockerd daemon:

ps aux | grep dockerd | awk '{print $4, $11}' | sort -nr
ps aux --sort=-%mem | head -20

App & Traefik logs:

docker compose logs -f moodle
docker logs traefik

## Backups

[](https://github.com/Hipjea/docker-moodle#backups)

### Create backups of volumes:

[](https://github.com/Hipjea/docker-moodle#create-backups-of-volumes)

docker volume ls

docker run --rm -v docker-moodle_db_data:/volume -v /opt/backup:/backup alpine tar czvf /backup/backup_moodle_db_data.tar.gz -C /volume .

### Restore volumes:

[](https://github.com/Hipjea/docker-moodle#restore-volumes)

Remove the containers:

Delete the volumes:

docker volume rm docker-moodle_db_data

Create the volumes:

docker volume create docker-moodle_db_data

Restore the backups:

docker run --rm -v docker-moodle_db_data:/volume -v /opt/backup:/backup alpine sh -c "tar xzvf /backup/backup_moodle_db_data.tar.gz -C /volume"

## Localhost setup

[](https://github.com/Hipjea/docker-moodle#localhost-setup)

Install `mkcert` and its dependencies:

Install the Docker compose stack:

docker compose -f docker-compose.local.yml build

Generate the certificates for localhost:

docker compose -f docker-compose.local.yml run --rm mkcert

Install the local certificates:

Run the stack:

docker compose -f docker-compose.local.yml up -d