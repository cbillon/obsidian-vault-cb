---
link: https://www.mikestreety.co.uk/blog/build-different-images-with-multi-stage-docker-builds/
byline: Mike Street
site: Mike Street
date: 2023-08-06T02:00
excerpt: Using multi-stage Docker builds you can create incremental images
  allowing you to keep the ones you need small
slurped: 2026-08-22T22:23
title: Build different images with multi-stage Docker builds
---

2023-08-06T00:00:00.000Z Posted on **6th August 2023**. **4 mins** reading time

I've written before about [multi-stage Docker builds](https://www.mikestreety.co.uk/blog/creating-a-multi-stage-docker-build-to-make-your-images-smaller/) to help you make smaller images, however today I discovered you can build images out of each of the stages.

The advantage for this is being able to create two, or more, images with incremental features allowing you to used the leanest image for the function.

The use-case I had was for our Gitlab CI and the images we use for deployment. Our base image needed PHP, Composer and Node, however we needed an additional image which also included Docker - so we could build a Docker image inside the CI.

Rather than bloat all the base images or use two separate `Dockerfile` files, I heave lent on multi-stage Docker builds to make all the images I need.

## Dockerfile

Let's start with the Dockerfile, this below creates a minimal Linux (Alpine) based image with PHP (Version passed in via a CLI argument), NPM and tools to use image optimisation tools (such as `gulp-imagemin`)

```
###
# Global Arguments
###
ARG PHP_VERSION

###
# Set global component images
###
FROM composer:2 as COMPOSER

FROM php:$PHP_VERSION-cli-alpine3.16

# Copy artifacts from component images
COPY --from=COMPOSER /usr/bin/composer /usr/bin/composer

# Install dependencies
RUN apk add \
	--update \
	--no-cache \
	# Deployment
	bash \
	git \
	openssh \
	rsync \
	# Front-end tools
	nodejs \
	npm \
	# Tools for imagemin
	autoconf \
	automake \
	g++ \
	gcc \
	jpeg \
	libc6-compat \
	libjpeg-turbo-dev \
	libpng-dev \
	libtool \
	make \
	musl-dev \
	nasm \
	tiff \
	zlib \
	zlib-dev

# Create SSH config
RUN mkdir /root/.ssh \
	&& touch /root/.ssh/id_ed25519 \
	&& chmod 700 /root/.ssh; \
	chmod 600 /root/.ssh/id_ed25519;

ENTRYPOINT ["/bin/sh", "-c"]
```

We can build this, and specify the PHP version when doing so:

```
 docker build \
	--tag deployment:php8.1 \
	--build-arg PHP_VERSION=8.1 \
	.
```

If we wanted to create a second image with more applications than the first, we can do this with staged builds.

By giving each stage a name, you can use the `--target` argument to stop the build at the end of that stage.

Update the `FROM` to include a name by using the `as` keyword:

```
FROM php:$PHP_VERSION-cli-alpine3.16 AS image_baseline
```

Next, add another stage at the end of the file and use the `image_baseline` as the image and give it a name (in this example `image_dind` {Docker in Docker})

```
FROM image_baseline AS image_dind
# Install dependencies
RUN apk add \
	--update \
	--no-cache \
	# Deployment
	docker
```

We can now build a Docker image from both the `image_baseline` and `image_dind` stages:

```
# Build a image_baseline image
docker build \
	--target image_baseline
	--tag deployment:php8.1 \
	--build-arg PHP_VERSION=8.1 \
	.

# Build an image from image_baseline with Docker
docker build \
	--target image_dind
	--tag deployment/docker:php8.1 \
	--build-arg PHP_VERSION=8.1 \
	.
```

From there, you can build on the `image_baseline` again or even the `image_dind` stage. You get all the benefits of a tidy filesystem along with each stage being cached. You also get to keep your images as small as they need to be - it's a win, win, win.

## Bonus Gitlab CI

I use Gitlab CI to generate these Docker files, creating one for each PHP version from 7.4 - 8.2. Rather than repeat lots of code, I utilise the `extends` keyword in CI.

This is the `.gitlab-ci.yaml`` file I use to build 8 different Docker files:

```
image: docker:20.10.24

stages:
  - build

services:
    - docker:20.10.24-dind

.build:
  stage: build
  interruptible: true
  variables:
    COMPOSER_VERSION: "2"
    DOCKER_TLS_CERTDIR: "/certs"
    DOCKER_BUILDKIT: 1
  before_script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY --username $CI_REGISTRY_USER --password-stdin
  script:
    # Build baseline image
    - >
      docker build
      --target image_baseline
      --tag $CI_REGISTRY_IMAGE:php${PHP_VERSION}
      --build-arg PHP_VERSION=${PHP_VERSION}
      .
    - docker push $CI_REGISTRY_IMAGE:php${PHP_VERSION}
    # Build dind image: Baseline with docker installed
    - >
      docker build
      --target image_dind
      --tag $CI_REGISTRY_IMAGE/docker:php${PHP_VERSION}
      --build-arg PHP_VERSION=${PHP_VERSION}
      .
    - docker push $CI_REGISTRY_IMAGE/docker:php${PHP_VERSION}

build:7.4:
  extends:
    - .build
  variables:
    PHP_VERSION: "7.4"

build:8.0:
  extends:
    - .build
  variables:
    PHP_VERSION: "8.0"

build:8.1:
  extends:
    - .build
  variables:
    PHP_VERSION: "8.1"

build:8.2:
  extends:
    - .build
  variables:
    PHP_VERSION: "8.2"
```