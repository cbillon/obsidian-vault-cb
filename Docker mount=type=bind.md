---
link: https://medium.com/@lina.cloud/docker-run-mount-type-bind-1b4e1b5db443
tags:
  - docker
  - volumes
---
# Dockerfile RUN — mount=type=bind

With Dockerfile v1.2 (that was back in 2020. Currently 1.4 is the latest version) was the option — mount add to RUN  supporting the types bind cache secrets and ssh. In this case, I’m interested in the bind option.

## So what does it?

Simply put it’s just like the bind mount you can do with --volume at runtime.

## So what can it be used for?

Sadly not every application you may need comes as docker container or is in the Repo of your distor. For example you have only an .deb, .rpm or tar.gz. In this case you a have to copy it into the container and install it or extract it. This results into an layer only containing the installation artifact. By using RUN — mount=type=bind it is possible to mount the artifact into the build container so on installation artifact is left behind only the mount path.

## Example

```
# syntax=docker/dockerfile:1.2
FROM debian:10-slim

# Installing SampleApplication form SampleApplication.tar into /opt
RUN --mount=type=bind,source=./mount,target=/tmp/build-context-mount \
    tar -xf /tmp/build-context-mount/SampleApplication.tar -C /opt
CMD bash /opt/SampleApplication/sample.sh
```

With `source` the source of the mount can be defined either from the build context or from the root of an build stage defined via `from`. With the option `readwrite` or `rw` the mount can be made writeable, the changes will be discarded.  
Repository containing an example with and without mount:  
[gitlab.com/linalinn/dockerfile-run](https://gitlab.com/linalinn/dockerfile-run)

## **Compatibilty**

Docker:  
To build dockerfile v1.2 you need to enable buildkit or use buildx  
Podman:  
Podman is able to build dockerfile v1.2 in the latest version without configuration.  
