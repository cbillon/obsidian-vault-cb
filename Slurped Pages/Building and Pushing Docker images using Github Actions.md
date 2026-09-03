---
link: https://www.mikestreety.co.uk/blog/building-and-pushing-docker-images-using-github-actions/
byline: Mike Street
site: Mike Street
date: 2023-09-07T02:00
excerpt: How you build a Docker image with Github actions and push to the Github
  Package registry
slurped: 2026-08-22T22:22
title: Building and Pushing Docker images using Github Actions
---

### By Mike Street

2023-09-07T00:00:00.000Z Posted on **7th September 2023**. **2 mins** reading time

We needed to build a Docker image via Github Actions and push to the Package repository to create a public, shareable image.

Add the following in a `.github/workflows` folder with a `.yml` extension. You'll also need to add the following secrets to the respoitory:

- `DOCKER_USERNAME` - the username whose token you will be using
- `DOCKER_TOKEN` -an access token with `write:packages` permission

Once you have pushed the package, you will need to go to the **Packages** tab on either the organisation and associate it with the repository. Once that is done, the package appears on the right-hand side of the repository list.

## Action YAML Example

This code was copied (and adapted) from the official [Github Docs](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images) website.

I've added (but left commented out) and example of how to pass in a build argument too.

```
name: Create and publish a Docker image

on:
  push:
    branches: ['main']

env:
  REGISTRY: ghcr.io
  REPO_NAME: $
  IMAGE_TAG: latest

jobs:
  build-and-push-image:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write

    steps:
	  # Hack to add an env variable built up of env variables
      - name: Set docker image env var
        run: |
          echo "IMAGE_NAME=$/$:$" >> $GITHUB_ENV

      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: $
          password: $

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
           #build-args: |
             #"ARG_KEY=VALUE"
          push: true
          tags: $
```

[View this post on Github](https://github.com/mikestreety/mikestreety/tree/main/app/content/blog/2023/2023-09-07-building-and-pushing-docker-images-on-github.md "./app/content/blog/2023/2023-09-07-building-and-pushing-docker-images-on-github.md")

[](https://fed.brid.gy/)

## You might also enjoy…

- [A release process for our NPM and Composer packages](https://www.mikestreety.co.uk/blog/a-release-process-for-our-npm-and-composer-packages/ "A release process for our NPM and Composer packages")
    
    2023-10-01T00:00:00.000Z 1st October 2023
    
    ### [A release process for our NPM and Composer packages](https://www.mikestreety.co.uk/blog/a-release-process-for-our-npm-and-composer-packages/)
    
    A post outlining our process and workflow for releasing packages and extensions
    
- [PHP Ternary and null coalescing operators](https://www.mikestreety.co.uk/blog/php-ternary-and-null-coalescing-operators/ "PHP Ternary and null coalescing operators")
    
    2023-08-10T00:00:00.000Z 10th August 2023
    
    ### [PHP Ternary and null coalescing operators](https://www.mikestreety.co.uk/blog/php-ternary-and-null-coalescing-operators/)
    
    What's the difference between a ternary and null coalescing operators? Do they give different results?
    

![Mike Street](https://www.mikestreety.co.uk/assets/img/mike.jpg)

#### Written by

Mike is a CTO and Lead Developer from Brighton, UK. He spends his time writing, cycling and coding. You can find Mike on [Mastodon](https://hachyderm.io/@mikestreety).