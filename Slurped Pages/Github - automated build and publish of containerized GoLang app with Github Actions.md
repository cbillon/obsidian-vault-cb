---
link: https://fabianlee.org/2022/11/21/github-automated-build-and-publish-of-containerized-golang-app-with-github-actions/
site: "Fabian Lee : Software Engineer"
excerpt: "Github Actions provide the ability to define a build workflow based on
  Github repository events.  The workflow steps are defined as yaml and can be
  triggered by various events, including a code push, branch, or tagging in the
  repository. In this article I will detail the steps of creating a
  statically-linked GoLang binary that when ... Github: automated build and
  publish of containerized GoLang app with Github Actions"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/actions
  - slurp/binary
  - slurp/continuous
  - slurp/deployment
  - slurp/docker
  - slurp/golang
  - slurp/oci
  - slurp/publish
  - slurp/workflow
slurped: 2026-09-18T08:30
title: "Github: automated build and publish of containerized GoLang app with
  Github Actions"
---

[![](https://fabianlee.org/wp-content/uploads/2022/01/github-icon-logo-png-transparent.png)Github Actions](https://docs.github.com/en/actions) provide the ability to define a build workflow based on Github repository events.  The workflow steps are defined as yaml and can be [triggered by various events](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow), including a code push, branch, or tagging in the repository.

In this article I will detail the steps of creating a statically-linked GoLang binary that when tagged with a semantic value (e.g. ‘v1.0.1’), will package it into an OCI-compatible (Docker) image that is published to both [Docker Hub](https://hub.docker.com/repository/docker/fabianlee/golang-github-action-example) and [Github Container Registry](https://github.com/fabianlee/golang-github-action-example/pkgs/container/golang-github-action-example).

### Prerequisites

We need to start with a Github repository that has a Go Language program proven out locally.  For this example, you can use my ‘[golang-github-action-example](https://github.com/fabianlee/golang-github-action-example)‘ repository.

sudo apt install make git -y

# get my example project
git clone https://github.com/fabianlee/golang-github-action-example.git
cd golang-github-action-example

### Local Docker build (optional)

If you have [Docker locally installed](https://fabianlee.org/2021/12/10/docker-installing-docker-ce-on-ubuntu-focal-20-04/), you can test a build of the image.  But we will automatically generating with a Github workflow in a later step, so this is optional.

# create image with local Docker
make docker-build

# run image, which is web server on localhost 8080
make docker-test-fg

### Github Action for OCI Image build and publish

We will be adding a [Github Action workflow](https://docs.github.com/en/actions/using-workflows/about-workflows) by adding a yaml manifest file into the “[.github/workflows](https://github.com/fabianlee/golang-github-action-example/tree/main/.github/workflows)” directory of our repository.

Our workflow will kick off when any new semantic tag starting with a “v” (e.g. v1.0.1) is pushed to the repository.  I will go over key areas below, but download the full version of [github-actions-buildOCI-image.yml](https://github.com/fabianlee/golang-github-action-example/blob/main/.github/workflows/github-actions-buildOCI.yml) from my github.

**Trigger**

on:
  push:
    tags: ['v*']

**Checkout Code**

[Checkout](https://github.com/marketplace/actions/checkout) and fetch all the history from the repo, because we will need this later when determining the release notes.

    - uses: actions/checkout@v3
      with:
        fetch-depth: 0 # get all tags, needed to get git log
        ref: main

**Login context to Container Registries**

In order to push the image into its final destination in Docker Hub and Github Container registry, we have to setup the login context.

      # docker hub credentials
      - name: login to docker hub
        uses: docker/login-action@v2
        with:
          #registry: docker.io
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      # Github container registry credentials
      - name: Log in to the Github Container registry ${{ env.GH_REGISTRY }} as ${{ github.actor }}
        uses: docker/login-action@v2
        with:
          registry: ${{ env.GH_REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

You may notice that the Docker Hub push uses a “secrets.DOCKER_USERNAME” and “secrets.DOCKER_TOKEN”.  These are your Docker Hub username and personal access token that need to be added as secrets in this Github repository.   The “secrets.GITHUB_TOKEN” used for pushing to the Github Container Registry is a special variable already populated by Github.

Create a Docker Hub personal access token using the [documentation provided here](https://docs.docker.com/docker-hub/access-tokens/).   Then you can either go to “Settings” > “Secrets” > “Actions” in your Github web UI and press “New Repository Secret”.  Or you can use the [Github CLI](https://fabianlee.org/2022/04/21/github-cli-tool-for-repository-operations/) to create repository secrets.

# prepend a space ' ' to the commands, avoids sensitive info going to history
 echo <yourDockerId> | gh secret set DOCKER_USERNAME
 echo <yourDockerPAT> | gh secret set DOCKER_TOKEN

### Create labels and tags for images

Create the proper image tags and version labels for the build.

      # tags and labels
      - name: Extract metadata (tags, labels) for image ${{ env.FULL_IMAGE_NAME }}
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: |
            ${{ env.FULL_IMAGE_NAME }}
            ${{ env.GH_REGISTRY }}/${{ env.FULL_IMAGE_NAME }}
          tags: |
            type=semver,pattern={{version}}
            type=ref,event=pr
            type=ref,event=branch

### Build and Publish OCI image

Finally, the image is built using the [Dockerfile](https://github.com/fabianlee/golang-github-action-example/blob/main/Dockerfile) and then published to both [Docker Hub](https://hub.docker.com/repository/docker/fabianlee/golang-github-action-example) and [Github Container Registry](https://github.com/fabianlee/golang-github-action-example/pkgs/container/golang-github-action-example).

      - name: build and push docker image
        uses: docker/build-push-action@v3.2.0
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          build-args: |
            MY_VERSION=${{ fromJSON(steps.meta.outputs.json).labels['org.opencontainers.image.version'] }}
            MY_BUILTBY=github-action

### Trigger Github Workflow

As mentioned earlier, this workflow is invoked by pushing a tag that looks like a semantic version.  Here are the git commands for creating a tag and pushing it remotely.

# check for any changes that might need to be checked in first
git status

# create new tag that triggers workflow, push tag
newtag=v1.0.0
git tag $newtag && git push origin $newtag

This will almost immediately create a workflow which you can view in real-time by going to your Github repository > Actions tab.

You can view the real-time progress by clicking into the task.  The icon will indicate when the workflow is complete.

### Validate Release

The Github Release can be reached from the Github web UI.  Click on “Packages” > [golang-github-action-example](https://github.com/fabianlee/golang-github-action-example/pkgs/container/golang-github-action-example), and each image version will be listed.

![](https://fabianlee.org/wp-content/uploads/2022/11/golang-github-action-example-packages-page.png)

REFERENCES

[actions/checkout](https://github.com/marketplace/actions/checkout)

[stackoverflow, github action invoking python](https://stackoverflow.com/questions/64049306/github-actions-how-to-trim-a-string-in-a-condition)

[github source, slackapi/slack-github-action](https://github.com/slackapi/slack-github-action)

[github cli, create repo](https://cli.github.com/manual/gh_repo_create)

[github cli, create secret](https://cli.github.com/manual/gh_secret_set)