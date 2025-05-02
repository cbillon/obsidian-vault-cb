---
link: https://modulitos.com/blog/lets-deploy-part-1/
byline: modulitos
date: 2016-03-23T00:00
excerpt: >-
  Note that this post assumes a basic understanding of Docker Engine and Docker
  Compose. Also, this workflow mostly applies when configuring and deploying
  applications in a production environment, while keeping costs and maintenance
  overhead down. This is not set up to scale - but feel free to get in touch if
  you have any ideas!

  Feel free to review and re-purpose the code here:
  https://github.com/modulitos/docker-shareabouts/

  Let’s deploy using a single script Link to heading This project began as a
  re-usable template for managing my Docker containers, which helped me to
  answer a question on StackOverflow.
tags:
  - blog
  - developer
  - personal
slurped: 2024-11-26T18:13
title: Let's Deploy! (Part 1)
---

![compose](https://modulitos.com/blog/lets-deploy-part-1/docker-compose.png)

**Note** that this post assumes a basic understanding of Docker Engine and Docker Compose. Also, this workflow mostly applies when configuring and deploying applications in a production environment, while keeping costs and maintenance overhead down. This is not set up to scale - but feel free to get in touch if you have any ideas!

Feel free to review and re-purpose the code here: [https://github.com/modulitos/docker-shareabouts/](https://github.com/modulitos/docker-shareabouts/)

## Let’s deploy using a single script [Link to heading](https://modulitos.com/blog/lets-deploy-part-1/#lets-deploy-using-a-single-script)

This project began as a re-usable template for managing my Docker containers, which helped me to [answer a question on StackOverflow](https://stackoverflow.com/a/33186458/1884158). After noticing significant traction on that answer, I decided to re-frame my answer into a longer form blog post.

I have everything bundled to where I can just run `./letsdeploy.sh`, which will fill in the environment variables for all of my templates and configs, then deploy my containers.

Here is the script:

_letsdeploy.sh_

```

#!/bin/bash

# Let's delete all containers that have a `my-project_` prefix:
# (by deleting/re-deploying our containers, this script is now idempotent!):
docker rm -f `docker ps -aq -f name=my-project_*`

# variables defined in .env will be exported into this script's environment:
set -a
source .env

# To avoid substituting nginx variables, which also use the shell syntax,
# we'll specify only the variables that will be used in our nginx config:
NGINX_VARS='$DOMAINS:$APP_CONTAINER_NAME'
# Now lets populate our nginx config templates to get an actual nginx config
# (which will be loaded into our nginx container):
envsubst "$NGINX_VARS" < nginx.conf > nginx-envsubst.conf

# Let's populate the variables in our compose file template,
# then deploy it!
cat compose.yml | envsubst | docker-compose -f - -p my-project_ up -d
```

## Let’s configure our compose and nginx templates: [Link to heading](https://modulitos.com/blog/lets-deploy-part-1/#lets-configure-our-compose-and-nginx-templates)

Here is a compose template below, which is basically a regular compose file but allows for variable substitution using shell syntax like `$MY_VAR` or `${MY_VAR}`:

_compose.yml_

```
version: '2'
services:
  ${DATABASE_CONTAINER_NAME}:
    image: kartoza/postgis:9.4-2.1
    volumes:
      - ~/postgres_data/smartercleanup-api:/var/lib/postgresql
      - ./start-postgis.sh:/start-postgis.sh
      - ./set-timezone.sh:/set-timezone.sh
    ports:
      - 25432:5432
    environment:
      - POSTGRES_PASS=${POSTGRES_PASS}
      - POSTGRES_USER=${POSTGRES_USER}
      - TZ=${TZ}
    restart: always
    command: sh -c "/set-timezone.sh && echo \"host all all 0.0.0.0/0 md5\" >> /etc/postgresql/9.4/main/pg_hba.conf && /start-postgis.sh"

  ${APP_CONTAINER_NAME}:
    image: smartercleanup/api:release-0.6.2
    depends_on:
      - ${DATABASE_CONTAINER_NAME}
    restart: always
    env_file: ./.env-smartercleanup-api
    volumes:
      - ./set-timezone.sh:/set-timezone.sh
    environment:
      - PASS=${POSTGRES_PASS}
      - USERNAME=${POSTGRES_USER}
      - HOST=${DATABASE_CONTAINER_NAME}
      - PORT=${POSTGRES_PORT}
      - TZ=${TZ}
    command: sh -c "/set-timezone.sh && git fetch && git checkout lukeswart/1.7-dependencies-upgrade && git pull --rebase && /api/start.sh"

  nginx:
    image: nginx
    volumes:
      # This is our nginx template with the variables substituted:
      - ./nginx-envsubst.conf:/etc/nginx/nginx.conf
    links:
      - ${APP_CONTAINER_NAME}
    volumes_from:
      - ${APP_CONTAINER_NAME}
    ports:
      - 80:80
    restart: always
```

Our `./letdeploy.sh` script will substitute all variables in that compose file, allowing us to abstract the variables from our template. This is especially useful when re-configuring a template for other deployment scenarios, or concealing passwords while version controlling the template.

Our nginx template is below, which allows us to substitute variables the same way:

_nginx.conf_

```
worker_processes 1;
error_log stderr notice;

events {
    worker_connections 1024;
}

http {

    include /etc/nginx/mime.types;
    charset utf-8;

    proxy_set_header Host $host;

    gzip_static on;
    gzip on;
    gzip_min_length  1100;
    gzip_buffers  4 32k;
    gzip_types    text/plain application/x-javascript text/xml text/css;
    gzip_vary on;

    server {
        listen 80 default_server;
        return 444;
    }
    server {
        listen 80;
        server_name ${DOMAINS};

        location /static/ {
            root /api;
            try_files $uri $uri/;
        }
        location / {
            proxy_pass http://${APP_CONTAINER_NAME}:8010;
        }
    }
}
```

Note that we are using Compose version 2, which simplifies the way we connect to our nginx containers in our nginx config template. Basically, this allows us to pass in our container name as the host name, shown in the `proxy_pass http://${APP_CONTAINER_NAME}:8010` line of the nginx config above.

Before version 2, linking containers was much more difficult, requiring the use of extra Nginx features through Lua extensions like [the OpenResty project](http://openresty.org/), or through more complex proxying that parses container metadata, like this one: [https://github.com/jwilder/nginx-proxy](https://github.com/jwilder/nginx-proxy).

## Let’s define our config variables [Link to heading](https://modulitos.com/blog/lets-deploy-part-1/#lets-define-our-config-variables)

Finally, we dial in our config using a single `.env` file. This will allow anyone to deploy the application by customizing this file and running `./letsdeploy.sh`:

_.env_

```
# Timezone variable for our database and api container:
TZ=America/Los_Angeles
##### Database settings:
DATABASE_CONTAINER_NAME=postgis
POSTGRES_USER=postgres
POSTGRES_PASS=postgres-password
POSTGRES_PORT=5432

#### App-specific settings:
APP_CONTAINER_NAME=smartercleanup-api

##### nginx server settings:
# domains for our nginx config:
DOMAINS="www.mysite.com mysite.com"
```

This setup has worked well for me, and I hope it helps someone out there. Feel free to leave a reply below, and let’s continue sharing what we know!

Want to see more? Follow this post to Part 2 (coming soon!) where I share my LetsEncrypt configuration to automatically generate and renew SSL certs through Docker containers.