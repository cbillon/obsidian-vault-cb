---
link: https://packagemain.tech/p/docker-build-cache
byline: Julien Singler
site: packagemain.tech
date: 2024-08-01T09:01
excerpt: How to store and share Docker build cache across teams using Docker registry.
slurped: 2026-02-19T15:25
title: "Optimising Docker Builds: Leveraging Build Cache for Efficient Development"
tags:
  - docker
  - image
  - cache
---

Nowadays, Docker is a go-to tool for building, shipping, and running containerized applications. One of the challenges a developer may face is build times, especially for large and complex codebases.

Docker build cache can offer a powerful solution to this problem by allowing you to reuse previously built layers.

In this article we will explore how to create and store build cache for different stages, such as the builder stage, and how to share this cache with your team using Docker Registry.

Docker build cache is a mechanism that allows Docker to reuse layers from previous builds. Each instruction in a Dockerfile creates a new layer, and Docker caches these layers to avoid redundant work. When you rebuild an image, Docker checks if it can reuse any of the cached layers, which can drastically reduce build times.

Multi-stage builds are a powerful feature in Docker that allows you to use multiple FROM statements in your Dockerfile. This enables you to create intermediate stages, such as a builder stage, which can be cached and reused independently. By caching these stages, you can further optimise your build process.

First, create a Dockerfile with multiple stages. Here's an example:

```
# Stage 1: Builder
FROM node:14 AS builder
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
```

Locally, you might have to use _**containerd**_ to be able to use the cache feature.

In the settings of your **Docker Desktop**, in **General** tab, you have to enable “**Use containerd for pulling and storing images**.”

[

![](https://substackcdn.com/image/fetch/$s_!YPpR!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F10d2661d-0c75-4b75-9674-ceb32a819d59_1544x348.png)

](https://substackcdn.com/image/fetch/$s_!YPpR!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F10d2661d-0c75-4b75-9674-ceb32a819d59_1544x348.png)

I also recommend you, to add this environment variable: **DOCKER_BUILDKIT=1**

To build the image and push the cache to your Docker Registry, use the following commands:

```
docker build --target builder \
  --cache-to=type=registry,ref=myregistry.com/myapp:cache-builder \
  -t myapp:builder-latest .
```

So, you built but didn’t push, still something was written to our registry. The cache layers were written to your registry.

We can tweak a little bit more our cache options to compress more those layers so it takes less space to store with: _**compression=zstd,mode=max**_

```
docker build --target builder \
  --cache-to=type=registry,ref=myregistry.com/myapp:cache-builder,compression=zstd,mode=max \
  -t myapp:builder-latest .
```

```
docker build --cache-from=type=registry,ref=myregistry.com/myapp:cache-builder -t myapp:latest .

# Push the final image
docker push myregistry.com/myapp:latest
```

You can also use **—cache-from** and **—cache-to** with Github Actions: [build-push-action](https://github.com/docker/build-push-action?tab=readme-ov-file#inputs)

For your team to benefit from this cache if they use docker-compose, you simply have to add in the build:

```
version: '3.5'

services:
    myapp:
        build:
            context: .
            args:
                - DOCKER_BUILDKIT=1
            cache_from:
                - type=registry,ref=myregistry.com/myapp:cache-builder
...
```

By storing the build cache in a Docker Registry, you can easily share it with your team. This ensures that everyone benefits from the cached layers, reducing build times and improving productivity.

1. **Use Multi-Stage Builds**: Break down your Dockerfile into multiple stages to optimize caching.
    
2. **Minimize Layer Changes**: Group frequently changing instructions together to minimize cache invalidation.
    
3. **Use .dockerignore**: Exclude unnecessary files from the build context to speed up the build process.
    
4. **Regularly Update Cache**: Periodically rebuild and push the cache to ensure it stays up-to-date with the latest changes.
    

Leveraging Docker build cache can significantly improve your development workflow by reducing build times and making the process more efficient. By storing and sharing the cache in a Docker Registry, you can ensure that all team members benefit from these optimizations. Implementing these practices will lead to faster builds, increased productivity, and a smoother development experience.

- [Docker Documentation: Build Cache](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#leverage-build-cache)
    
- [Multi-Stage Builds with Docker](https://docs.docker.com/develop/develop-images/multistage-build/)
    
- [Setting Up a Docker Registry](https://docs.docker.com/registry/deploying/)