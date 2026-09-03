---
link: https://www.mikestreety.co.uk/blog/creating-a-multi-stage-docker-build-to-make-your-images-smaller/
byline: Mike Street
site: Mike Street
date: 2022-02-11T01:00
excerpt: During the build of a Docker image you may need to step through several
  stages. Using a multi-build helps keep the end result smaller in size.
slurped: 2026-08-22T22:37
title: Creating a multi-stage Docker build to make your images smaller
---

2022-02-11T00:00:00.000Z Posted on **11th February 2022**. **4 mins** reading time

It may be the case that you want to build a Docker image containing your compiled website or application, but you need to step through seveal stages to get there. Composer, NPM and other package managers need to download and compile assets before your application is good to go, along with build tools and scripts - most of which don't need to be in your end deliverable.

There are serveral ways of achiving this, you can write a bash script that compiles the assets then builds the Docker image or you can use several steps of a CI or similar.

Docker has recently introduced [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) which helps solve this problem.

For our websites [at work](https://www.liquidlight.co.uk/), we use Composer as the PHP dependency manager, NPM for front-end assets and Gulp for compilation. Some of our NPM scripts rely on assets stored in our Composer modules, but none of the assets in `node_modules` folder need to be in the final build image, nor do `node`, `npm` or `composer`.

The idea behind multi-stage builds is you lean on the strengths of different images to do the grunt work and building, then you cast them aside and copy your assets out.

What we do is:

1. Use the `shippingdocker/php-composer:7.4` image to run a `composer install`
2. Copy the downloaded files into a `node:14` image which then does an `npm install` and uses `gulp` to compile our front-end assets
3. We then copy the compiled assets & PHP code into a final Debain image with apache

The key thing about multi-stage builds is relying on the `as` keyword when using `FROM`, that way, you can `COPY` the results.

Before we dive into that example, let's have a look at a 2 stage multi-build:

```
###
# Node/NPM Dependencies
###
FROM node:14 as node

# Set a workdir other than `/`
WORKDIR /app

# Copy our package files
COPY package.json ./
COPY package-lock.json ./

# Run a production-ready, CI focused install
RUN npm ci

###
# Main Web App
###
FROM debian:bullseye

# Copy just the dist files
COPY --from=node /app/dist /var/www/html/
```

Note the `--from` which leans on the `as`. This allows you to step through and have several different images building different parts of the application and bring them all together at the end. For example:

```
COPY --from=backend /app/ /var/www/
COPY --from=frontend /app/ /var/www/
```

As mentioned, however, our app relies on a `composer install` being done before `npm` as some JavaScript assets are pulled in. Becuase of that, we copy from composer, to npm, to web.

This means image at the end only has the applications & code required to run a fully compiled app.

Our Dockerfile looks something like:

```
###
# Composer
###

# Use the PHP 7.3 with Composer image and call it `composer`
FROM shippingdocker/php-composer:7.3 as composer

# Set a workdir other than `/`
WORKDIR /app

# Copy src from your repository into a folder called src
COPY src src

# Copy both composer.json and composer.lock
COPY composer.* ./

# Run a production-ready composer install
RUN composer install --no-ansi --no-dev --no-interaction --no-scripts --no-progress --optimize-autoloader

###
# Node/NPM Dependencies
###
FROM node:14 as node

# Set a workdir for the new "node" app
WORKDIR /app

# Copy the app folder from the composer image above
COPY --from=composer /app/ /app/

# Copy the build folder from our original repository
COPY build build

# Copy our package & gulp files
COPY package.json ./
COPY package-lock.json ./
COPY gulpfile.js gulpfile.js

# Run a production-ready, CI focused install
RUN npm ci
# Run a "local" gulp compile (doesn't need global installation)
RUN ./node_modules/.bin/gulp compile

###
# Main Web App
###
FROM debian:bullseye

# Use the debian webroot
WORKDIR /var/www/

# Copy various assets from different images
COPY --from=composer /app/src/ src
COPY --from=composer /app/vendor/ vendor
COPY --from=node /app/html/ html
```

So you can see how powerful multi-stage builds can be. You can even use previous builds as the `FROM` of your next stage.