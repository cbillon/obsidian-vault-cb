---
link: https://gagor.pro/2025/12/squashing-docker-images-with-buildx/
byline: Tom
site: Tom's Blog
date: 2025-12-24T01:00
excerpt: Learn how to squash Docker images using Buildx to create smaller, more efficient images by consolidating layers.
tags:
  - docker
  - build
  - images
slurped: 2026-02-24T10:34
title: Squashing Docker images with Buildx
---

One of interesting experimental feature of (now) [“legacy” Docker build system](https://docs.docker.com/reference/cli/docker/build-legacy/#squash) is ability to squash image layers into single one. It helps to reduce final image size, especially when there are many layers with temporary files that are not needed in final image. Of course, the good practice here is to use commands in a way that would allow to do everything you need in as little layers as possible, but it’s sometimes tricky to achieve.

If you [enable experimental features](https://docs.docker.com/reference/cli/dockerd/) in Docker daemon, you can use `--squash` flag with `docker build` command. This hack allows to use as many layers as you want during the build, and at the end system autmatically stash them to just one layer. It’s also smart enough to not touch the base image, so whatever you produces is squashed and is not recompressing the whole thing.

graph TD
    A[Base Image] --> B[Layer 1] -- Squashing --> F[Temporary Files]
    B --> C[Layer 2]
    C --> D[Layer 3]
    D --> E[Final Image]
    E --> F[Squashed Layer]

I really like this approach for 2 reasons:

1. It produce smaller base images effortlessly.
2. More directives allow to keep Dockerfile easier to read and maintain, in contrast to combining everything into single `RUN` command with `&&` operators.

However, this feature is not available when using Buildx, which is now the recommended way to build Docker images 😟

Thankfully the advanced capabilities of Buildx allow to achive similar results with a bit of workaround. The idea is to build the image in two stages: first, create the full image with all layers, and then create a new image based on the first one, copying only the final filesystem state.

```
# this image builds the application
FROM ubuntu:24.04 AS builder

RUN apt-get update && \
    apt-get install -y build-essential curl

RUN useradd -m -d /opt/app app && \
    mkdir -p /opt/app && \
    chown appuser:appuser /opt/app

RUN curl -o /tmp/app.tar.gz https://example.com/app.tar.gz && \
    tar -xzf /tmp/app.tar.gz -C /opt/app && \
    cd /opt/app && \
    make install && \
    rm /tmp/app.tar.gz

FROM scratch

# this image squashes the previous one
COPY --from=builder / /

USER app
CMD ["/opt/app/bin/start-app"]
```

Let me explain what happen here:

- `FROM scratch` creates a new empty image.
- `COPY --from=builder / /` copies all files from the `builder` stage to the new image.
- The final image contains files from both base image and all layers you built. Intermediate layers are gone.

This allows quite similar effect to `--squash` flag, producing a single-layer image with only the final filesystem state. And that’s actually a difference, because `--squash` keeps the base image layers intact, while this method creates a new image based on `scratch` compressing all previous layers to a single one. On one side it can produce more optimal image, on the other it doesn’t allow to use base image caching.

It’s possible to achieve similar effect by copying parts of filesystem selectively:

```
FROM ubuntu:24.04 AS builder
# ... we build stuff here ...

FROM ubuntu:24.04

RUN useradd -m -d /opt/app app && \
    mkdir -p /opt/app && \
    chown appuser:appuser /opt/app
COPY --from=builder /opt/app /opt/app

USER app
CMD ["/opt/app/bin/start-app"]
```

In this example we copy only the `/opt/app` directory, but we have to recreate the user as this part is not copied. It’s a typical “builder pattern” approach. It’s not that optimal as we created more layers than with `--squash`, but it’s the best we can do with Buildx.

You might also need to “redo” some of base image configuration like exporting `PATH` environment variable or `CMD`, or `ENTRYPOINT` directives, as those are not copied automatically. You can check base image with:

```
docker image inspect ubuntu:24.04 | jq '.[].Config'
{
  "Env": [
    "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
  ],
  "Cmd": [
    "/bin/bash"
  ],
  "Labels": {
    "org.opencontainers.image.ref.name": "ubuntu",
    "org.opencontainers.image.version": "24.04"
  }
}
```

To be honest, I find this approach a bit clunky compared to the simplicity of `--squash` flag. I can’t imagine too many developers going this path just to reduce image size, and it’s not helping to clarify of what you’re doing. I hope, they will eventually add `--squash` support to Buildx as well.

---

Enjoyed this post? [![Buy Me a Coffee at ko-fi.com](https://storage.ko-fi.com/cdn/kofi5.png?v=3)](https://ko-fi.com/tgagor)