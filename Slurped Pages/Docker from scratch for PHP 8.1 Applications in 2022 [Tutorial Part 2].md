---
link: https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/
byline: Pascal Landau
excerpt: How to set up a repository with Docker 'from scratch' to develop PHP 8.1 applications in 2022.
slurped: 2026-06-12T14:59
title: Docker from scratch for PHP 8.1 Applications in 2022 [Tutorial Part 2]
tags:
  - docker
  - dockerfile
---

In this part of my tutorial series on developing PHP on Docker we will revisit the previous tutorials and update some things to be up-to-date in 2022.

**All code samples are publicly available** in my [Docker PHP Tutorial repository on Github](https://github.com/paslandau/docker-php-tutorial/).  
You find the branch for this tutorial at [part-4-1-docker-from-scratch-for-php-applications-in-2022](https://github.com/paslandau/docker-php-tutorial/tree/part-4-1-docker-from-scratch-for-php-applications-in-2022)

**All published parts of the Docker PHP Tutorial** are collected under a dedicated page at [Docker PHP Tutorial](https://www.pascallandau.com/docker-php-tutorial/). The previous part was [Structuring the Docker setup for PHP Projects](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/) and the following one is [PhpStorm, Docker and Xdebug 3 on PHP 8.1 in 2022](https://www.pascallandau.com/blog/phpstorm-docker-xdebug-3-php-8-1-in-2022/).

If you want to follow along, please subscribe to the [RSS feed](https://www.pascallandau.com/feed.xml) or [via email](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#newsletter) to get **automatic notifications** when the next part comes out :)

## Table of contents

- [Introduction](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#introduction)
- [Local docker setup](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#local-docker-setup)
- [Docker](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#docker)
    - [docker-compose](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#docker-compose)
        - [`.docker/.env` file and required ENV variables](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#docker-env-file-and-required-env-variables)
    - [Images](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#images)
        - [PHP images](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#php-images)
            - [ENV vs ARG](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#env-vs-arg)
        - [Image naming convention](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#image-naming-convention)
        - [Environments and build targets](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#environments-and-build-targets)
- [Makefile](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#makefile)
    - [`.make/*.mk` includes](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#make-mk-includes)
    - [Shared variables: `.make/.env`](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#shared-variables-make-env)
        - [Manual modifications](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#manual-modifications)
    - [Enforce required parameters](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#enforce-required-parameters)
- [Make + Docker = <3](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#make-docker-3)
    - [Ensuring the build order](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#ensuring-the-build-order)
    - [Run commands in the docker containers](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#run-commands-in-the-docker-containers)
        - [Solving permission issues](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#solving-permission-issues)
- [PHP POC](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#php-poc)
- [Wrapping up](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#wrapping-up)

## Introduction

If you have read the previous tutorial [Structuring the Docker setup for PHP Projects](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/) you might encounter some significant changes. The tutorial was published over 2 years ago, Docker has evolved and I have learned more about it. Plus, I gathered practical experience (good and bad) with the previous setup. I would now consider most of the points under [Fundamentals on building the containers](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#fundamentals-on-building-the-containers) as either "not required" or simply "overengineered / too complex". To be concrete:

- [Setting the timezone](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#setting-the-timezone)
    - not required if the default is already UTC (which is almost always the case)
- [Synchronizing file and folder ownership on shared volumes](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#synchronizing-file-and-folder-ownership-on-shared-volumes)
    - this is only an issue if files need to be **modified** by containers and the host system - which is only really relevant for the PHP containers
    - in addition, I would recommend adding a completely new user (e.g. `application`) instead of re-using an existing one like `www-data` - this simplifies the whole user setup _a lot_
    - from now on we will be using `application` as the user name (`APP_USER_NAME`) and `10000` the user id (`APP_USER_ID`; following the best practice to [not use a UID below 10,000](https://github.com/hexops/dockerfile#do-not-use-a-uid-below-10000))
- [Modifying configuration files](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#modifying-configuration-files)
    - just use `sed` - no need for a dedicated script
- [Installing php extensions](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#installing-php-extensions)
    - see [PHP images](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#php-images) - will now be done via `apk add`
- [Installing common software](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#installing-common-software)
    - see [PHP images](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#php-images) - since there is only one base image there is no need for a dedicated script
- [Cleaning up](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#cleaning-up)
    - didn't really make sense because the "cleaned up files" were already part of a previous layer
    - we might "bring it back" later when we optimize the image size to speed up the pushing/pulling of the images to/from the registry
- [Providing host.docker.internal for linux host systems](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#providing-host-docker-internal-for-linux-host-systems)
    - can now be done via the [`host-gateway` magic reference](https://github.com/docker/for-linux/issues/264#issuecomment-965465879) `yaml services: myservice: extra_hosts: - host.docker.internal:host-gateway`
    - thus, no custom entrypoint is required any longer

## Local docker setup

The goal of this part is the introduction of a working local setup **without development tools**. In other words: We want the bare minimum to have something running locally.

The main components are:

- the `make` setup in the `Makefile` and in the `.make/` directory
- the docker setup in the `.docker/` directory
- some PHP files that act as a POC for the end2end functionality of the docker setup

Check out the code via

```
git checkout part-4-1-docker-from-scratch-for-php-applications-in-2022
```

initialize it via

```
make make-init
make docker-build
```

and run it via

```
make docker-up
```

Now you can access the web interface via [http://127.0.0.1](http://127.0.0.1/). The following diagram shows how the containers are connected

[![Docker container connections](https://www.pascallandau.com/img/docker-from-scratch-for-php-applications-in-2022/docker-containers.PNG "Docker container connections")](https://www.pascallandau.com/img/docker-from-scratch-for-php-applications-in-2022/docker-containers.PNG)

See also the [PHP POC](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#php-poc) for a full test of the setup.

The docker setup consists of

- an nginx container as a webserver
- a MySQL database container
- a Redis container that acts as a queue
- a php base image that is used by
    - a php worker container that spawns multiple PHP worker processes via `supervisor`
    - a php-fpm container as a backend for the nginx container
    - an application container that we use to run commands

[![Docker images](https://www.pascallandau.com/img/docker-from-scratch-for-php-applications-in-2022/docker-images.PNG "Docker images")](https://www.pascallandau.com/img/docker-from-scratch-for-php-applications-in-2022/docker-images.PNG)

We keep the [`.docker/` directory from the previous tutorial](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#the-docker-folder) , though it will be split into `docker-compose/` and `images/` like so:

```
.
└── .docker/
    ├── docker-compose/
    |   ├── docker-compose.yml
    |   └── <other docker compose files>
    ├── images/
    |   ├── nginx/
    |   |   ├── Dockerfile
    |   |   └── <other files for the nginx image>
    |   └── <other folders for docker images>
    ├── .env
    └── .env.example
```

### `docker compose`

All images are **build** via `docker compose` because the `docker-compose.yml` file(s) provide a nice abstraction layer for the build configuration. In addition, we can also use it to **orchestrate** the containers, i.e. control volumes, port mappings, networking, etc. - as well as start and stop them via `docker compose up` and `docker compose down`.

FYI: Even though it is _convenient_ to use `docker compose` for both things, I found it also to make the setup more complex than it needs to be when running things later in production (when we are _not_ using `docker compose` any longer). I believe the problem here is that some modifications are ONLY required for building while others are ONLY required for running - and combining both in the same file yields a certain amount of noise. But: It is what it is.

We use three separate `docker-compose.yml` files:

- `docker-compose.yml`
    - contains all information valid for all environments
- `docker-compose.local.yml`
    - contains information specific to the `local` environment, see [Environments and build targets](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#environments-and-build-targets)
- `docker-compose-php-base.yml`
    - contains information for building the php base image, see [PHP images](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#php-images)

#### `.docker/.env` file and required ENV variables

In our docker setup we basically have 3 different types of variables:

1. variables that **depend on the local setup** of an individual developer, e.g. the `NGINX_HOST_HTTP_PORT` on the host machine (because the default one might already be in use)
2. variables that **are used in multiple images**, e.g. the location of the codebase within a container's file system
3. variables that **hold information that is "likely to change"**, e.g. the exact version of a base image

Since - again - we strive to retain a single source of truth, we extract the information as variables and put them in a `.docker/.env` file. In a perfect world, I would like to separate these different types in different files - but `docker compose` only allows a single `.env` file, see e.g. [this comment](https://github.com/docker/compose/issues/6170#issuecomment-443523663). If the file does not exist, it is copied from `.docker/.env.example`.

The variables are then used in the `docker-compose.yml` file(s). I found it to be "the least painful" to always use the [`?` modifier on variables](https://docs.docker.com/compose/environment-variables/#substitute-environment-variables-in-compose-files) so that `docker compose` **fails immediately if the variable is missing**.

Note: Some variables are expected to be passed via environment variables when `docker compose` is invoked (i.e. they are required but not defined in the `.env` file; see also [Shared variables: `.make/.env`](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#shared-variables-make-env)

### Images

For **MySQL** and **redis** we do not use custom-built images but instead **use the official ones _directly_** and configure them through environment variables when starting the containers. In production, we won't use docker anyway for these services but instead rely on the managed versions, e.g.

- redis => [Memorystore for Redis (GCP)](https://cloud.google.com/memorystore/docs/redis) or [ElastiCache für Redis (AWS)](https://aws.amazon.com/de/elasticache/redis/)
- mysql => [Cloud SQL for MySQL (GCP)](https://cloud.google.com/sql/docs/mysql) or [RDS for MySQL (AWS)](https://aws.amazon.com/de/rds/mysql/)

The remaining containers are defined in their respective subdirectories in the `.docker/images/` directory, e.g. the image for the `nginx` container is build via the `Dockerfile` located in `.docker/images/nginx/Dockerfile`.

#### PHP images

We need 3 different PHP images (fpm, workers, application) and use a slightly different approach than in [Structuring the Docker setup for PHP Projects](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/):

Instead of using the [official PHP base images](https://hub.docker.com/_/php) (i.e. cli or fpm), we use a "plain" alpine base image and install PHP and the required extensions manually in it. This allows us to build a common base image for all PHP images. Benefits:

- a central place for shared tools and configuration (no more need for a `.shared/` directory)
- reduced image size when pushing the individual images (the base image is recognized as a layer and thus "already exists")
- installing extensions via `apk add` is **a lot** faster than via `docker-php-ext-install`

This new approach has two major downsides:

- we depend on the alpine release cycle of PHP (and PHP extensions)
- the image build process is more complex, because we must build the base image first before we can build the final images

Fortunately, both issues can be solved rather easily:

- [codecasts/php-alpine](https://github.com/codecasts/php-alpine) maintains an `apk` repository with the latest PHP versions for alpine
- we use a dedicated `make` target to build the images instead of invoking `docker compose` directly - this enables us to define a "build order" (base first, rest after) while still having to run only a single command as a developer (see [Ensuring the build order](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#ensuring-the-build-order))

##### ENV vs ARG

I've noticed that some build arguments are required in multiple PHP containers, e.g. the name of the application user defined in the `APP_USER_NAME` ENV variable. The username is needed

- in the base image to create the user
- in the fpm image to define the user that runs the fpm processes (see `php-fpm.d/www.conf`)
- in the worker image to define the user that runs the worker processes ( see `supervisor/supervisord.conf`)

Instead of passing the name to all images via build argument, i.e.

- define it explicitly under `services.*.build.args` in the `docker-compose.yml` file
- "retrieve" it in the Dockerfile via `ARG APP_USER_NAME`

I've opted to make the username available as an `ENV` variable in the base image via

```
ARG APP_USER_NAME
ENV APP_USER_NAME=${APP_USER_NAME}
```

and thus be able to access it in the child images directly, I can now write

```
RUN echo ${APP_USER_NAME}
```

instead of

```
ARG APP_USER_NAME
RUN echo ${APP_USER_NAME}
```

I'm not 100% certain that I like this approach as I'm more or less "abusing" ENV variables in ways that they are likely not intended ("Why would the username need to be stored as an ENV variable?") - but I also don't see any other practical downside yet.

#### Image naming convention

Defining a [fully qualified name for images](https://windsock.io/referencing-docker-images/) will make it much easier to reference the images later, e.g. when pushing them to the registry.

The naming convention for the images is `$(DOCKER_REGISTRY)/$(DOCKER_NAMESPACE)/$(DOCKER_SERVICE_NAME)-$(ENV)`, e.g.

```
                   docker.io/dofroscra/nginx-local
$(DOCKER_REGISTRY)---^          ^        ^     ^        docker.io
$(DOCKER_NAMESPACE)-------------^        ^     ^        dofroscra
$(DOCKER_SERVICE_NAME)-------------------^     ^        nginx
$(ENV)-----------------------------------------^        local
```

and it is used as value for `services.*.image`, e.g. for `nginx`

```
services:
  nginx:
    image: ${DOCKER_REGISTRY?}/${DOCKER_NAMESPACE?}/nginx-${ENV?}:${TAG?}
```

In case you are wondering: `dofroscra` stems from **Do**cker **Fro**m **Scra**tch

#### Environments and build targets

Our final goal is a setup that we can use for

- local development
- in a CI/CD pipeline
- in production

and even though we strive to for a [parity between those different environments](https://12factor.net/dev-prod-parity), there will be differences due to fundamentally different requirements. E.g.

- on _production_ I want a container **including the sourcecode without any test dependencies**
- on _CI_ I want a container **including the sourcecode WITH test dependencies**
- on _local_ I want a container **that mounts the sourcecode from my host (including dependencies)**

This is reflected through the `ENV` environment variable. We use it in two places:

1. as part of the image name as a suffix of the service name (see [Image naming convention](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#image-naming-convention))
2. to specify the [target build stage](https://docs.docker.com/engine/reference/commandline/build/#specifying-target-build-stage---target)

See the `docker-compose-php-base.yml` file for example:

```
services:
  php-base:
    image: ${DOCKER_REGISTRY?}/${DOCKER_NAMESPACE?}/php-base-${ENV?}:${TAG?}
    build:
      dockerfile: images/php/base/Dockerfile
      target: ${ENV?}
```

Using **multiple targets in the same Dockerfile** enables us to keep a **common base** but also include **environment specific instructions**. See the Dockerfile of the `php-base` image for example

```
ARG ALPINE_VERSION
FROM composer:${COMPOSER_VERSION} as composer
FROM alpine:${ALPINE_VERSION} as base

RUN apk add --update --no-cache \
        bash

WORKDIR $APP_CODE_PATH

FROM base as local

RUN apk add --no-cache --update \
        mysql-client \
```

- it first defines a `base` stage that includes software required in all environments
- and then defines a `local` stage that adds additionally a `mysql-client` that helps us to debug connectivity issues

After the build for `local` is finished, we end up with an image called `php-base-local` that used the `local` build stage as target build stage.

## Makefile

In the following section I will **introduce a couple of commands**, e.g. for building and running containers. And to be honest, I find it kinda challenging to keep them in mind without having to look up the exact options and arguments. I would **usually create a helper function** or an alias in my local `.bashrc` file in a situation like that - but that wouldn't be available to other members of the team then and it would be very specific to this one project.

Instead we'll use a **self-documenting Makefile** that [acts as the central entrypoint in the application](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#using-make-as-central-entry-point). Since Makefiles tend to grow over time, I've adopted some strategies to keep them "sane" via includes, shared variables and better error handling.

### `.make/*.mk` includes

Over time the `make` setup will grow substantially, thus we split it into multiple `.mk` files in the `.make/` directory. The individual files are prefixed with a number to ensure their order when we include them in the main `Makefile` via

```
include .make/*.mk
```

```
.
└── .make/
    ├── 01-00-application-setup.mk
    ├── 01-01-application-commands.mk
    └── 02-00-docker.mk
```

### Shared variables: `.make/.env`

We try to make **shared variables** available here, because we can then pass them on to individual commands as a prefix, e.g.

```
.PHONY: some-target
some-target: ## Run some target
    ENV_FOO=BAR some_command --baz
```

This will make the `ENV_FOO` available as environment variable to `some_command`.

Shared variables are used by different components, and we always try to maintain only a **single source of truth**. An example would be the `DOCKER_REGISTRY` variable that we need to define the [image names of our docker images](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#image-naming-convention) in the `docker-compose.yml` files but also when pushing/pulling/deploying images via make targets later. In this case, the variable is required by `make` as well as `docker compose` and the setup is explained in section [Make + Docker = <3](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#make-docker-3).

To have a clear separation between variables and "code", we use a `.env` file located at `. make/.env`. It can be initialized via

```
make make-init
```

by copying the `.make/.env.example` to `.make/.env`.

```
.
└── .make/
    ├── .make/.env.example
    └── .make/.env
```

The file is included in the main `Makefile` via

```
-include .make/.env
```

The `-` prefix ensures that make doesn't fail if the file does not exist (yet), see [GNU make: Including Other Makefiles](https://www.gnu.org/software/make/manual/html_node/Include.html).

**Note**

In a later part of this tutorial, we will introduce an additional files that holds variables:

  
- [`.make/variables.env`](https://www.pascallandau.com/blog/ci-pipeline-docker-php-gitlab-github/#initialize-the-shared-variables)

  

#### Manual modifications

You can always **modify the `.make/.env` file manually if required**. This might be the case when you run `docker` on Linux and need to match the `user id` of your host system with the `user id` of the docker container. It is common that your local user and group have the id `1000`. In this case you would add the entries manually to the `.make/.env` file.

```
APP_USER_ID=1000
APP_GROUP_ID=1000
```

See also section [Solving permission issues](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#solving-permission-issues).

### Enforce required parameters

We kinda "abuse" make for executing arbitrary commands (instead of building artifacts) and some of those commands require parameters that can be [passed as command arguments](https://stackoverflow.com/a/2826178/413531) in the form

```
make some-target FOO=bar
```

There is no way to "define" those parameters as we would in a method signature - but we can still ensure to fail as early as possible if a parameter is missing via

```
@$(if $(FOO),,$(error FOO is empty or undefined))
```

See also [SO: How to abort makefile if variable not set?](https://stackoverflow.com/a/10858332/413531)

We use this technique for example to ensure that all required variables are defined when we execute docker targets via the `validate-docker-variables` precondition target:

```
.PHONY: validate-docker-variables
validate-docker-variables: 
    @$(if $(TAG),,$(error TAG is undefined))
    @$(if $(ENV),,$(error ENV is undefined))
    @$(if $(DOCKER_REGISTRY),,$(error DOCKER_REGISTRY is undefined - Did you run make-init?))
    @$(if $(DOCKER_NAMESPACE),,$(error DOCKER_NAMESPACE is undefined - Did you run make-init?))
    @$(if $(APP_USER_NAME),,$(error APP_USER_NAME is undefined - Did you run make-init?))
    @$(if $(APP_USER_ID),,$(error APP_USER_ID is undefined - Did you run make-init?))
    @$(if $(APP_GROUP_ID),,$(error APP_GROUP_ID is undefined - Did you run make-init?))

.PHONY:docker-build-image
docker-build-image: validate-docker-variables
    $(DOCKER_COMPOSE) build $(DOCKER_SERVICE_NAME)
```

## Make + Docker = <3

We already introduced quite some complexity into our setup:

- "global" variables (shared between `make` and `docker`)
- multiple `docker-compose.yml` files
- build dependencies

Bringing it all together "manually" is quite an effort and prone to errors. But we can nicely tuck the complexity away in `.make/02-00-docker.mk` by defining the two variables `DOCKER_COMPOSE` and `DOCKER_COMPOSE_PHP_BASE`

```
DOCKER_DIR:=./.docker
DOCKER_ENV_FILE:=$(DOCKER_DIR)/.env
DOCKER_COMPOSE_DIR:=$(DOCKER_DIR)/docker-compose
DOCKER_COMPOSE_FILE:=$(DOCKER_COMPOSE_DIR)/docker-compose.yml
DOCKER_COMPOSE_FILE_LOCAL:=$(DOCKER_COMPOSE_DIR)/docker-compose.local.yml
DOCKER_COMPOSE_FILE_PHP_BASE:=$(DOCKER_COMPOSE_DIR)/docker-compose-php-base.yml
DOCKER_COMPOSE_PROJECT_NAME:=dofroscra_$(ENV)

DOCKER_COMPOSE_COMMAND:=ENV=$(ENV) \
 TAG=$(TAG) \
 DOCKER_REGISTRY=$(DOCKER_REGISTRY) \
 DOCKER_NAMESPACE=$(DOCKER_NAMESPACE) \
 APP_USER_ID=$(APP_USER_ID) \
 APP_GROUP_ID=$(APP_GROUP_ID) \
 APP_USER_NAME=$(APP_USER_NAME) \
 docker compose -p $(DOCKER_COMPOSE_PROJECT_NAME) --env-file $(DOCKER_ENV_FILE)

DOCKER_COMPOSE:=$(DOCKER_COMPOSE_COMMAND) -f $(DOCKER_COMPOSE_FILE) -f $(DOCKER_COMPOSE_FILE_LOCAL)
DOCKER_COMPOSE_PHP_BASE:=$(DOCKER_COMPOSE_COMMAND) -f $(DOCKER_COMPOSE_FILE_PHP_BASE)
```

- `DOCKER_COMPOSE` uses `docker-compose.yml` and extends it with `docker-compose.local.yml`
- `DOCKER_COMPOSE_PHP_BASE` uses only `docker-compose-php-base.yml`

The variables can then be used later in make recipes.

### Ensuring the build order

As mentioned under [PHP images](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#php-images), we **need to build images in a certain order** and use the following make targets:

```
.PHONY: docker-build-image
docker-build-image: ## Build all docker images OR a specific image by providing the service name via: make docker-build DOCKER_SERVICE_NAME=<service>
    $(DOCKER_COMPOSE) build $(DOCKER_SERVICE_NAME)

.PHONY: docker-build-php
docker-build-php: ## Build the php base image
    $(DOCKER_COMPOSE_PHP_BASE) build $(DOCKER_SERVICE_NAME_PHP_BASE)

.PHONY: docker-build
docker-build: docker-build-php docker-build-image ## Build the php image and then all other docker images
```

As a developer, I can now simply run `make docker-build` - which will first build the `php-base` image via `docker-build-php` and then build all the remaining images via `docker-build-image` (by not specifying the `DOCKER_SERVICE_NAME` variable, `docker compose` will build **all** services listed in the `docker-compose.yml` files).

I would argue that the **make recipes themselves are quite readable** and easy to understand but when we run them with the [`-n` option](https://www.gnu.org/software/make/manual/html_node/Options-Summary.html) to only "Print the recipe that would be executed, but not execute it", we get a feeling for the complexity:

```
$ make docker-build -n
ENV=local TAG=latest DOCKER_REGISTRY=docker.io DOCKER_NAMESPACE=dofroscra APP_USER_NAME=application APP_GROUP_NAME=application docker compose -p dofroscra_local --env-file ./.docker/.env -f ./.docker/docker-compose/docker-compose-php-base.yml build php-base
ENV=local TAG=latest DOCKER_REGISTRY=docker.io DOCKER_NAMESPACE=dofroscra APP_USER_NAME=application APP_GROUP_NAME=application docker compose -p dofroscra_local --env-file ./.docker/.env -f ./.docker/docker-compose/docker-compose.yml -f ./.docker/docker-compose/docker-compose.local.yml build
```

### Run commands in the docker containers

Tooling is an important part in the development workflow. This includes things like linters, static analyzers and testing tools but also "custom" tools geared towards your specific workflow. Those tools usually **require a PHP runtime**. For now, we only have a single "tool" defined in the file `setup.php`. It ensures that a table called `jobs` is created.

To run this tool, we must first start the docker setup via `make docker-up` and then execute the script in the `application` container. The corresponding target is defined in `.make/01-00-application-setup.mk`:

```
.PHONY: setup-db
setup-db: ## Setup the DB tables
    $(EXECUTE_IN_APPLICATION_CONTAINER) php setup.php $(ARGS);
```

which essentially translates to

```
docker compose exec -T --user application application php setup.php
```

if we are outside of a container and to

```
php setup.php
```

if we are inside a container. That's quite handy, because we can **run the tooling directly from the host system** without having to log into a container.

The "magic" happens in the `EXECUTE_IN_APPLICATION_CONTAINER` variable that is defined in `.make/02-00-docker.mk` as

```
EXECUTE_IN_WORKER_CONTAINER?=
EXECUTE_IN_APPLICATION_CONTAINER?=

EXECUTE_IN_CONTAINER?=
ifndef EXECUTE_IN_CONTAINER
    # check if 'make' is executed in a docker container, 
    # see https://stackoverflow.com/a/25518538/413531
    # `wildcard $file` checks if $file exists, 
    # see https://www.gnu.org/software/make/manual/html_node/Wildcard-Function.html
    # i.e. if the result is "empty" then $file does NOT exist 
    # => we are NOT in a container
    ifeq ("$(wildcard /.dockerenv)","")
        EXECUTE_IN_CONTAINER=true
    endif
endif
ifeq ($(EXECUTE_IN_CONTAINER),true)
    EXECUTE_IN_APPLICATION_CONTAINER:=$(DOCKER_COMPOSE) exec -T --user $(APP_USER_NAME) $(DOCKER_SERVICE_NAME_APPLICATION)
    EXECUTE_IN_WORKER_CONTAINER:=$(DOCKER_COMPOSE) exec -T --user $(APP_USER_NAME) $(DOCKER_SERVICE_NAME_PHP_WORKER)
endif
```

We can take a look via `-n` again to see the resolved recipe on the host system

```
pascal.landau:/c/_codebase/dofroscra# make setup-db ARGS=--drop -n
ENV=local TAG=latest DOCKER_REGISTRY=docker.io DOCKER_NAMESPACE=dofroscra APP_USER_NAME=application APP_GROUP_NAME=application docker compose -p dofroscra_local --env-file ./.docker/.env -f ./.docker/docker-compose/docker-compose.yml -f ./.docker/docker-compose/docker-compose.local.yml exec -T --user application application php setup.php --drop
```

Within a container it looks like this:

```
root:/var/www/app# make setup-db ARGS=--drop -n
php setup.php --drop;
```

#### Solving permission issues

If you are using Linux, you **might run into permission issues when modifying files** that are shared between the host system and the docker containers **when the user id is not the same** as explained in section [Synchronizing file and folder ownership on shared volumes](https://www.pascallandau.com/blog/structuring-the-docker-setup-for-php-projects/#synchronizing-file-and-folder-ownership-on-shared-volumes) of the previous tutorial.

In this case, you need to [modify the `.make/.env` manually](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#manual-modifications) and add the `APP_USER_ID` and `APP_GROUP_ID` variables according to your local setup. This **must be done _before_ building the images** to ensure that the correct `user id` is used in the images.

In very rare cases it can lead to problems, because **your local ids will _already exist_ in the docker containers**. I've personally never run into this problem, but you can read about it in more detail at [Docker and the host filesystem owner matching problem](https://www.joyfulbikeshedding.com/blog/2021-03-15-docker-and-the-host-filesystem-owner-matching-problem.html). The author [even proposes a general solution](https://twitter.com/honglilai/status/1509781424345432064) via [the Github project "FooBarWidget/matchhostfsowner"](https://github.com/FooBarWidget/matchhostfsowner).

## PHP POC

To ensure that everything works as expected, the repository contains a minimal PHP proof of concept. By default, port 80 from the host ist forwarded to port 80 of the `nginx` container.

FYI: I would also recommend to add the following entry [in the hosts file on the host machine](https://www.howtogeek.com/howto/27350/beginner-geek-how-to-edit-your-hosts-file/)

```
127.0.0.1 app.local
```

so that we can access the application via [http://app.local](http://app.local/) instead of [http://127.0.0.1](http://127.0.0.1/).

The files of the POC essentially ensure that the container connections outlined in [Local docker setup](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#local-docker-setup) work as expected:

[![Docker container connections](https://www.pascallandau.com/img/docker-from-scratch-for-php-applications-in-2022/docker-containers.PNG "Docker container connections")](https://www.pascallandau.com/img/docker-from-scratch-for-php-applications-in-2022/docker-containers.PNG)

- `dependencies.php`
    - returns configured `Redis` and `PDO` objects to talk to the queue and the database
- `setup.php`
    - => _ensures that `application` can talk to `mysql`_
- `public/index.php`
    - is the web root file that can be accessed via [http://app.local](http://app.local/)
        - => _ensures that `nginx` and `php-fpm` are working_
    - contains 3 different "routes":
        - [http://app.local?dispatch=some-job-id](http://app.local/?dispatch=some-job-id)
            - dispatches a new "job" with the id `some-job-id` on the queue to be picked up by a worker
                - => _ensures that `php-fpm` can talk to `redis`_
        - [http://app.local?queue](http://app.local/?queue)
            - shows the content of the queue
        - [http://app.local?db](http://app.local/?db)
            - shows the content of the database
                - => _ensures that `php-fpm` can talk to `mysql`_
- `worker.php`
    - is started as daemon process in the `php-worker` container
    - checks the redis datasbase `0` for the key `"queue"` every second
    - if a value is found it is stored in the `jobs` table of the database
        - => _ensures that `php-worker` can talk to `redis` and `mysql`_

A full test scenario is defined in `test.sh` and looks like this:

```
$ bash test.sh


  Building the docker setup


//...


  Starting the docker setup


//...


  Clearing DB


ENV=local TAG=latest DOCKER_REGISTRY=docker.io DOCKER_NAMESPACE=dofroscra APP_USER_NAME=application APP_GROUP_NAME=application docker compose -p dofroscra_local --env-file ./.docker/.env -f ./.docker/docker-compose/docker-compose.yml -f ./.docker/docker-compose/docker-compose.local.yml exec -T --user application application php setup.php --drop;
Dropping table 'jobs'
Done
Creating table 'jobs'
Done


  Stopping workers


ENV=local TAG=latest DOCKER_REGISTRY=docker.io DOCKER_NAMESPACE=dofroscra APP_USER_NAME=application APP_GROUP_NAME=application docker compose -p dofroscra_local --env-file ./.docker/.env -f ./.docker/docker-compose/docker-compose.yml -f ./.docker/docker-compose/docker-compose.local.yml exec -T --user application php-worker supervisorctl stop worker:*;
worker:worker_00: stopped
worker:worker_01: stopped
worker:worker_02: stopped
worker:worker_03: stopped


  Ensuring that queue and db are empty


Items in queue
array(0) {
}
Items in db
array(0) {
}


  Dispatching a job 'foo'


Adding item 'foo' to queue


  Asserting the job 'foo' is on the queue


Items in queue
array(1) {
  [0]=>
  string(3) "foo"
}


  Starting the workers


ENV=local TAG=latest DOCKER_REGISTRY=docker.io DOCKER_NAMESPACE=dofroscra APP_USER_NAME=application APP_GROUP_NAME=application docker compose -p dofroscra_local --env-file ./.docker/.env -f ./.docker/docker-compose/docker-compose.yml -f ./.docker/docker-compose/docker-compose.local.yml exec -T --user application php-worker supervisorctl start worker:*;
worker:worker_00: started
worker:worker_01: started
worker:worker_02: started
worker:worker_03: started


  Asserting the queue is now empty


Items in queue
array(0) {
}


  Asserting the db now contains the job 'foo'


Items in db
array(1) {
  [0]=>
  string(3) "foo"
}

```

## Wrapping up

Congratulations, you made it! If some things are not completely clear by now, don't hesitate to leave a comment. Apart from that, you should now have a running docker setup and the means to "control" it conveniently via `make`.

In the next part of this tutorial, we will [configure PhpStorm as our IDE to use the docker setup](https://www.pascallandau.com/blog/phpstorm-docker-xdebug-3-php-8-1-in-2022/).

Please subscribe to the [RSS feed](https://www.pascallandau.com/feed.xml) or [via email](https://www.pascallandau.com/blog/docker-from-scratch-for-php-applications-in-2022/#newsletter) to get automatic notifications when this next part comes out :)

---

### Wanna stay in touch?

Since you ended up on this blog, chances are pretty high that you're into Software Development (probably PHP, Laravel, Docker or Google Big Query) and I'm a big fan of feedback and networking.

So - if you'd like to stay in touch, feel free to shoot me an email with a couple of words about yourself and/or connect with me on [LinkedIn](https://www.linkedin.com/in/pascallandau) or [Twitter](https://twitter.com/PascalLandau) or simply subscribe to my [RSS feed](https://www.pascallandau.com/feed.xml) or go the crazy route and subscribe via mail and don't forget to leave a comment :)

#### Subscribe to posts via mail

![Waving bear](https://www.pascallandau.com/img/waving-bear.gif)

## Comments