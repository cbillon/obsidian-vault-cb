---
link: https://www.mikestreety.co.uk/blog/how-i-improved-the-speed-of-docker-builds-in-gitlab-ci/
byline: Mike Street
site: Mike Street
date: 2022-06-08T02:00
excerpt: I reduced our Gitlab CI powered Docker build by 80% - taking it more
  than 15 minutes to around 3 minutes to build and push with these steps
slurped: 2026-08-22T22:36
title: How I improved the speed of Docker builds in Gitlab CI
---

2022-06-08T00:00:00.000Z Posted on **8th June 2022**. **5 mins** reading time

Our Gitlab CI powered Docker build is now 500% faster (I think?) - taking it more than 15 minutes to around 3 minutes to build and push to a registry.

Unfortunately, there isn't one thing to copy and paste - this blog post is more a collection of concepts and steps I took to drop the build time. 15 minutes wasn't the end of the world, but I wanted to see how far I could take it.

This blog post features some pseudo dockerfile & Gitlab CI yaml code - it might not be able to be copied but should give you some idea as to what was happening

## Application Background

The application is a website which runs on TYPO3 - a composer-based open-source CMS and the client provided a Docker environment to host the website. I'm a big fan of Gitlab CI, so took the challenge to completely compile and build a Docker image via Gitlab, rather than building assets and installing dependencies locally (which I had done in the past).

The site also features NPM dependencies - nothing over the top, but our SCSS and Webpack JS is compiled using Gulp, along with optimising images and [building a sprite](https://www.liquidlight.co.uk/blog/creating-svg-sprites-using-gulp-and-sass/).

**Note:** I'll be using "website" and "application" interchangeably - they ultimately mean the same thing.

## Requirements

The build needed to:

- Clone the repository
- `composer install` - to get the backend dependencies
- `npm ci` - this is "clean install"
- `gulp compile` - this generates all the assets
- Copy the compiled assets into an apache-ready Docker container
- Push the container to the registry

## Docker multi-stage build

The first attempt at building the application was to use a [multi-stage Docker build](https://www.mikestreety.co.uk/blog/creating-a-multi-stage-docker-build-to-make-your-images-smaller/) (this is actually the project that was written for). This used a custom image which had NPM and Composer preloaded to install and compile the dependencies and the final assets we passed to the Web image for pushing.

The Docker file looked something like

```
FROM custom-node-and-composer-image as build

# COPY package.json, composer.json & app files
composer i
npm ci
gulp compile

FROM custom-web-app

COPY --from=build /app/dist /var/www/
```

With the Gitlab CI file having a script block like:

```
script:
  - docker build -t registry:my-image .
  - docker push registry:my-image
```

This meant the Gitlab file was extremely lightweight, which was great as it meant you could compile the full Docker image locally, if required. It left Docker to do most of the heavy lifting which seemed to make the most sense.

This built the Docker image as expected, but took a whopping **15 minutes** from start to finish. Granted, it is all done on Gitlab so you can push and carry on with some other work if required, but it still needed you to go back 15 minutes later to test everything worked as expected.

## Installing with Gitlab CI

In a bid to try and speed up the process I took a blind punt and seeing what would happen if I took the `npm` and `composer` install steps and moved them _outside_ of the Docker build. Let Gitlab do the installing and then copy the resulting files to Docker.

**Docker:**

```
FROM custom-web-app

COPY /app/dist /var/www/
```

**Gitlab CI:**

```
image: custom-node-and-composer-image-with-docker
script:
  - composer i
  - npm ci
  - gulp compile
  - docker build -t registry:my-image .
  - docker push registry:my-image
```

The premise behind this is to run all our build steps outside the Docker file and then copy the resulting files in.

We use the base Docker image used in the original multi-stage build to run the Gitlab CI step - the only difference being that Docker is installed inside the Docker image so it can build the final… image (Docker and image were mentioned far too many times).

This obviously comes with the big disadvantage of suddenly not being able to build the Docker image locally - however we use ddev for local development so this wasn't an issue for us.

This dropped the build times from 15 to 7 minutes - halving the time it takes just by moving the build step out of Docker.

## Caching

With the time down to 7 minutes for a fresh build, it was time to start looking at caching. Using the next few steps, I was able to get the build down to around 3 minutes. However, if all the caches got invalidated then there is likelihood the next build would go back up to 7 minutes again.

## Docker Caching

Docker has caching build in to the process - that is why a fresh build always takes longer. Each step is made into a mini image that then gets "loaded" into the main build. More steps means more HDD space but a speedier build. Less steps means smaller space, but a bigger chance of one of the steps' cache being invalidated.

During this exercise I learnt a lot about Dockerfiles and optimising them to get the best performance. The big thing I learnt was:

**If anything changes in a step, it invalidates the cache for _every_ step after that**

Knowing that really helps order the copy steps - make sure you have the files which change the least being copied first (base config files etc.). The last `COPY` (or other step) should be the files that change the most - for example if you have FE assets with cache-busting file names.

## Gitlab CI Caching

The final piece to the puzzle was to use Gitlab CI caching. You can use the cache in the runner itself, however this isn't very reliable - especially if you have several runners.

Gitlab runners don't retain for projects or even jobs in a pipeline, so you can have a different runner picking up different stages - if you use the runner cache there is a high likelihood it would be out-of-date or (dangerously) wrong.

The most efficient (and safest) method is to use an external service like an Amazon S3 bucket or a self hosted solution, such as [Minio](https://min.io/) - this has S3 compatible APIs so can be used as drop in replacement.

Our Gitlab CI file features a global cache of the `node_modules` and `vendor` folders, which is invalidated if the respective lockfiles change

```
cache:
  - key:
      files:
        - composer.lock
    paths:
      - vendor/
  - key:
      files:
        - package-lock.json
    paths:
      - node_modules/
```

There is plenty of documentation about [Gitlab Caching on their website](https://docs.gitlab.com/ee/ci/caching/).

The last step was to change my `npm ci` command to just `npm i`. The `ci` command deletes the exsiting `node_modules` folder, which defeats the point in caching it!

## Conclusion

Removing the build from Docker, re-ordering my Dockerfile and leaning on caching are the main steps I took in dramatically dropping the build time of my Docker image using Gitlab CI.

At time of writing, the last deployment (including linting) took 3 minutes and 21 seconds - the build of that taking 2 minutes and 9 seconds.