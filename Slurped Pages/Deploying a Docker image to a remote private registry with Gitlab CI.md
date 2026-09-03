---
link: https://www.mikestreety.co.uk/blog/deploying-a-docker-image-to-a-remote-private-registry-with-gitlab-ci/
byline: Mike Street
site: Mike Street
date: 2023-03-01T01:00
excerpt: Deploying an image to a remote Docker registry can be achieved with Gitlab CI
slurped: 2026-08-22T22:30
title: Deploying a Docker image to a remote private registry with Gitlab CI
---

2023-03-01T00:00:00.000Z Posted on **1st March 2023**. **4 mins** reading time

I am a huge fan of [Gitlab](https://www.mikestreety.co.uk/category/gitlab/). What you are able to get on the free tier is incredible (although I do wish their Premium tier was slightly cheaper, but I digress) and the processes and optimisations it unlocks is infinite. The CI/Pipelines are such a powerful tool (but are by no means easy to get started with). The world of Gitlab CI is mind boggling, the syntax is confusing and debugging is near impossible, however every now and then you get it right and tingle of success flutters through you.

Hopefully, this post will help someone get to the completion point sooner, as it took me a lot of Googling and fake commits and getting things wrong before I got it right. I am fortunate to have my Gitlab runners self-hosted otherwise I would have quickly burnt through the 400 free minutes.

**The scenario is this**: We wanted our Gitlab CI to build our application, package it up in a Docker image and push that image to a private registry (which would then deploy to "the cloud").

The first step is to set up the variables which will be used during the process. It's good practice to use variables instead of committing your usernames and passwords directly in the `.gitlab-ci.yml` file.

In the repository on Gitlab, navigate to **Settings** -> **CI/CD** -> **Variables** and add the following (feel free to change the variable names, don't forget to update the YAML)

- `DOCKER_REGISTRY_PASS` - The password to the private registry
- `DOCKER_REGISTRY_USER` - The username for the private registry
- `DOCKER_REGISTRY` - The URL to the registry (without any protocol - e.g. `docker.io`)

Before proceeding, make sure the image you are using has the Docker process available. If you are using a custom image, this can be added by using the script from `docker.com`:

```
# Docker
RUN curl -fsSL https://get.docker.com -o get-docker.sh
RUN sh get-docker.sh
```

Next, create your (or edit) your `.gitlab-ci.yml` and add a build stage.

```
push_image:
  stage:
    - deploy
  before_script:
    - echo "$DOCKER_REGISTRY_PASS" | docker login $DOCKER_REGISTRY --username $DOCKER_REGISTRY_USER --password-stdin
  script:
    - composer install --no-ansi --no-dev --no-interaction --no-scripts --no-progress --optimize-autoloader
    - npm i
    - npm run build
    - docker build -t $PRODUCTION_DOCKER_REGISTRY/production-image-name:$CI_PIPELINE_IID .
    - docker push $PRODUCTION_DOCKER_REGISTRY/production-image-name:$CI_PIPELINE_IID
  only:
    - main
```

The example above has `composer` and `npm` install stages, should you need them. The `only` key is for the branch on which the task runs. The `$CI_PIPELINE_IID` is a [predefined variable](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) from Gitlab - it allows you to re-run the pipeline and overwrite the same image. It also ties it to the pipeline, not the date or time or any other variable of which you could accidentally generate a duplicate. It also allows you to republish previous pipelines if something goes amiss.

## Bonus enhancement

Our pipelines feature an extra step - which is to build the image and push it to the Gitlab instance before pulling this image and pushing it elsewhere. This means if the pipeline fails or if you need to redeploy the same image, you don't have to rebuild all the assets. It means if the pipeline fails upon deployment, you still have the image locally to re-push it.

Our `image_build` step looks like the above, except the `DOCKER` credentials for the [Gitlab registry](https://docs.gitlab.com/ee/user/packages/container_registry/). The image build step also had the push location updated to use the current repositories Docker registry:

```
script:
  - docker build -t $CI_REGISTRY_IMAGE:$CI_PIPELINE_IID .
  - docker push $CI_REGISTRY_IMAGE:$CI_PIPELINE_IID
```

We then have a second step which pulls from Gitlab and pushes to the remote registry:

```
.deploy:
  image: registry.gitlab.lldev.co.uk/devops/containers/development:bullseye-php7.4-composer2-node14
  allow_failure: false
  variables:
    GIT_STRATEGY: none
  before_script:
    - export VERSION=$(date +"%Y%m%d-%H%M")
    # Login to our registry
    - echo "$DOCKER_REGISTRY_PASS" | docker login $DOCKER_REGISTRY --username $DOCKER_REGISTRY_USER --password-stdin
    # Login to their registry
    - echo "$PRODUCTION_DOCKER_REGISTRY_PASS" | docker login $PRODUCTION_DOCKER_REGISTRY --username $PRODUCTION_DOCKER_REGISTRY_USER --password-stdin
  script:
	# Pull from our registry
    - docker pull $CI_REGISTRY_IMAGE:$CI_PIPELINE_IID
	# Re-tag
    - docker tag $CI_REGISTRY_IMAGE:$CI_PIPELINE_IID $PRODUCTION_DOCKER_REGISTRY/production-image-name:$CI_PIPELINE_IID
    - docker push $PRODUCTION_DOCKER_REGISTRY/production-image-name:$CI_PIPELINE_IID
```