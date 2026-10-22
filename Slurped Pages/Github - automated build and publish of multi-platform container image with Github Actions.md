---
link: https://fabianlee.org/2023/09/16/github-automated-build-and-publish-of-multi-platform-container-image-with-github-actions/
site: "Fabian Lee : Software Engineer"
excerpt: "Github Actions provide the ability to define a build workflow based on
  Github repository events.  The workflow steps are defined as yaml and can be
  triggered by various events, including a code push, branch, or tagging in the
  repository. In this article, I will show how to define workflow steps that
  build and push a ... Github: automated build and publish of multi-platform
  container image with Github Actions"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/continuous
  - slurp/deployment
  - slurp/docker
  - slurp/oci
  - slurp/pipeline
  - slurp/publish
  - slurp/workflow
slurped: 2026-10-20T08:54
title: "Github: automated build and publish of multi-platform container image
  with Github Actions"
---

[![](https://fabianlee.org/wp-content/uploads/2022/01/github-icon-logo-png-transparent.png)Github Actions](https://docs.github.com/en/actions) provide the ability to define a build workflow based on Github repository events.  The workflow steps are defined as yaml and can be [triggered by various events](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow), including a code push, branch, or tagging in the repository.

In this article, I will show how to define workflow steps that build and push a multi-platform image (amd64,arm64,arm32) with manifest index to the Github Container Registry.  This is enabled by using ‘docker buildx’ and QEMU software emulation for the various target architectures.

We will be creating a [Github Action workflow](https://docs.github.com/en/actions/using-workflows/about-workflows) by adding a yaml manifest file into the standard ‘.github/workflows’ directory.  Here is a link to my full [github-actions-buildOCI-image.yml](https://github.com/fabianlee/tiny-tools-multi-arch/blob/master/.github/workflows/github-actions-buildOCI.yml), with step-through below.

### Trigger

The workflow will kick off when any new semantic tag starting with a “v” (e.g. v1.0.1) is pushed to the repository.

on:
  push:
    tags: ['v*']

### Checkout Code

[Checkout](https://github.com/marketplace/actions/checkout) the Github repository.

    - name: Check out repository code
      uses: actions/checkout@v4

### Setup QEMU and Docker buildx

As described in the [Docker multi-platform with Github Actions](https://docs.docker.com/build/ci/github-actions/multi-platform/) page, add the following steps that enabled platform emulation and the Docker buildx/BuildKit that provide multi-platform builds.

      # QEMU for software emulation of multiple platforms
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      # Docker buildx/buildkit for multi-platform builds
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

That is all you need to do for this pipeline, but if you want a deeper dive into components see my other article on doing manual [multi-platform builds on your local server using QEMU/buildx](https://fabianlee.org/2023/09/16/docker-building-multi-platform-images-that-use-fat-manifest-list-index/).

### Login context to Container Registries

We do need to establish a login context to the Github Container registry to push the images, but these value are already provided as part of the standard pipeline context.

      # Github container registry credentials
      - name: Log in to the Github Container registry ${{ env.GH_REGISTRY }} as ${{ github.actor }}
        uses: docker/login-action@v2
        with:
          registry: ${{ env.GH_REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

The “[secrets.GITHUB_TOKEN](https://docs.github.com/en/actions/security-guides/automatic-token-authentication)” used for pushing to the Github Container Registry is a special variable already populated by Github.

### Create labels and tags for images

Create the proper image tags and version labels for the build.

      # tags and labels
      - name: Extract metadata (tags, labels) for image ${{ env.FULL_IMAGE_NAME }}
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: |
            ${{ env.GH_REGISTRY }}/${{ env.FULL_IMAGE_NAME }}
          tags: |
            type=semver,pattern={{version}}
            type=ref,event=pr
            type=ref,event=branch

### Generate custom build arguments

As a customization, we want to pass along the time of the build along with the short SHA of the latest git commit.  Placing these as key/value pairs on their own line will allow us to access them in later steps as ‘steps.get_buildargs.outputs.BUILD_TIME’ and ‘steps.get_buildargs.outputs.GITREF’

      - name: Get fresh build arguments
        shell: bash
        run: echo -e "BUILD_TIME=$(date -u '+%Y-%m-%d_%H:%M:%S')\nGITREF=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT
        id: get_buildargs

### Build and Publish OCI image

Finally, the multi-platform image is built using the [Dockerfile](https://github.com/fabianlee/tiny-tools-multi-arch/blob/master/Dockerfile) and then published to its [Github Container Registry](https://github.com/fabianlee/tiny-tools-multi-arch/pkgs/container/tiny-tools-multi-arch).  The ‘[platforms](https://docs.docker.com/build/ci/github-actions/multi-platform/)‘ attribute is a CSV list of the desired architectures.

      - name: build and push docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          platforms: linux/amd64,linux/arm64,linux/arm/v7
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          build-args: |
            MY_VERSION=${{ fromJSON(steps.meta.outputs.json).labels['org.opencontainers.image.version'] }}
            MY_BUILTBY=github-action
            BUILD_TIME=${{ steps.get_buildargs.outputs.BUILD_TIME }}
            GITREF=${{ steps.get_buildargs.outputs.GITREF }}

The ‘build-args’ specified above can be referenced in the [Dockerfile](https://github.com/fabianlee/tiny-tools-multi-arch/blob/master/Dockerfile), and per the snippet below are used to populate ‘/build.log’ inside the container.

# standard Docker arguments
ARG TARGETPLATFORM
ARG BUILDPLATFORM
# custom build arguments
ARG BUILD_TIME
ARG GITREF
# persist these build time arguments into container as debug
RUN echo "[$BUILD_TIME] [$GITREF] building on host that is $BUILDPLATFORM, for the target architecture $TARGETPLATFORM" > /build.log

### Trigger Github Workflow

This workflow is invoked by pushing a tag that looks like a semantic version.  Here are the git commands for creating a tag and pushing it remotely.

# check for any changes that might need to be checked in first
git status

# create new tag that triggers workflow, push tag
newtag=v1.0.0
git tag $newtag && git push origin $newtag

This will almost immediately create a workflow which you can view in real-time by going to your Github repository > [Actions tab](https://github.com/fabianlee/tiny-tools-multi-arch/actions).

![](https://fabianlee.org/wp-content/uploads/2023/09/github-tiny-tools-multi-arch-actions.png)

You can view the real-time progress by clicking into the task.  The icon will indicate when the workflow is complete.

### Validate multi-platform publish

From the Github web UI click on “Packages” > [tiny-tools-multi-arch](https://github.com/fabianlee/tiny-tools-multi-arch/pkgs/container/tiny-tools-multi-arch), and each image version will be listed.

![](https://fabianlee.org/wp-content/uploads/2023/09/github-tiny-tools-multi-arch-packages.png)

The OCI manifest index details can be seen by using ‘docker manifest index’.

docker manifest inspect ghcr.io/fabianlee/tiny-tools-multi-arch:2.0.0 | head

Test a pull and run of the image by running docker like below.

# will output architecture family of running image
docker run -it --rm ghcr.io/fabianlee/tiny-tools-multi-arch:2.0.0 uname -m

REFERENCES

[docker, example of multi-platform github pipeline yaml](https://github.com/marketplace/actions/checkout)

[fabianlee.org, article showing how to do multi-platform builds on local server using QEMU/buildx](https://fabianlee.org/2023/09/16/docker-building-multi-platform-images-that-use-fat-manifest-list-index/)

[stackoverflow, explains that @v{} syntax on github actions is determined by author and can indicate: branch, sha, tag](https://stackoverflow.com/questions/71558639/is-there-a-way-to-specify-a-dynamic-version-for-a-github-action)

[actions/checkout](https://github.com/marketplace/actions/checkout)

[actions/build-push-action](https://github.com/docker/build-push-action)

[github docker buildx plugin](https://docs.docker.com/engine/reference/commandline/buildx/)

[docker, buildx command reference](https://docs.docker.com/engine/reference/commandline/buildx/)

[github buildkit, configuration that can be specified via buildx ‘config’ flag](https://github.com/moby/buildkit/blob/master/docs/buildkitd.toml.md)