---
link: https://laurent-bel.medium.com/running-go-on-docker-comparing-debian-vs-alpine-vs-distroless-vs-busybox-vs-scratch-18b8c835d9b8
byline: Laurent Bel
site: Medium
date: 2022-08-15T12:35
excerpt: Running Go on Docker — Comparing Debian vs Alpine vs Distroless vs BusyBox vs Scratch In this article, we will analyze different options to run Go on Docker, analyzing the size of the generated …
twitter: https://twitter.com/@Medium
slurped: 2026-02-16T17:55
title: Running Go on Docker — Comparing Debian vs Alpine vs Distroless vs BusyBox vs Scratch
tags:
  - golang
  - docker
  - image
---
muta
In this article, we will analyze different options to run Go on Docker, analyzing the size of the generated image.

Press enter or click to view image in full size

We will perform the analysis over 5 candidates :

- Debian 11 (Bullseye)
- Alpine 3.16
- Distroless
- BusyBox
- Scratch

> Note: The full source code is available [in the following GitHub repository.](https://github.com/laurentbel/go-docker)

## Basic Hello World

A basic “hello world” program is used, consisting in two files.

go.mod

module example/hello  
go 1.19

hello.go

package main  
import "fmt"  
func main() {  
    fmt.Println("Hello, World!")  
}

Let’s now look at how to deploy this using different Docker base image.

## Debian 11 (Bullseye)

This option offers the comfort of Debian. We are using the official go image that comes with Debian’s flavor.

FROM **golang:1.19-bullseye**WORKDIR /app  
COPY src/go.mod src/hello.go ./  
RUN go build .  
RUN rm go.mod hello.goCMD [ "/app/hello" ]

**Final image size: 994 MB**

## Alpine 3.16

The official go image can also come with Alpine’s flavor. Generating a significantly smaller image.

FROM **golang:1.19-alpine3.16**WORKDIR /app  
COPY src/go.mod src/hello.go ./  
RUN go build .  
RUN rm go.mod hello.goCMD [ "/app/hello" ]

**Final image size: 354 MB**

## Distroless

[Distroless images](https://github.com/GoogleContainerTools/distroless) are provided by Google and contains only your applications and its runtime dependencies. They do not contain package manager, shell or any other program you would expect to find in a standard Linux distribution, making it a optimized and safe choice.

FROM **golang:1.19-alpine as build**  
WORKDIR /app  
COPY src/go.mod src/hello.go ./  
RUN go buildFROM **gcr.io/distroless/static-debian11**  
COPY --from=build /app/hello /app/  
CMD [ "/app/hello" ]

Note that we are using a multi-stage build, so that the build is performed using the official go image. Alpine is chosen because it has CGO disabled by default, which is required to run on distroless images. If you prefer to build on debian, you will need to explicitly disable CGO during build phase:

RUN **CGO_ENABLED=0** go build

**Final image size: 4.17 MB**

## BusyBox

BusyBox combines tiny versions of many common UNIX utilities into a single small executable. It is not considered as a Linux distribution but more like a set of tools that can be used by Linux distributions (it is for instance an essential component of the famous Alpine distribution).

Again, we are using a multi-stage build:

FROM **golang:1.19-alpine as build**  
WORKDIR /app  
COPY src/go.mod src/hello.go ./  
RUN go buildFROM **busybox**  
COPY --from=build /app/hello /app/  
CMD [ "/app/hello" ]

**Final image size: 3.06 MB**

## Scratch

The scratch image is **the smallest possible image for docker**. It does not contain any folders or files. It is possible to run statically compiled and self contained binary files.

Here is how to use it, with again a multi-stage build:

FROM **golang:1.19-alpine as build**  
WORKDIR /app  
COPY src/go.mod src/hello.go ./  
RUN go buildFROM **scratch**  
COPY --from=build /app/hello /app/  
CMD [ "/app/hello" ]

**Final image size: 1.81 MB**

## Scratch advanced

Hold on… it is not that easy. The previous image is actually missing two important things to be fully functional:

- Timezone info: to be able to manipulates timezone (GMT, UTC, …).
- CA certificates: to validate certificates signed by valid Certificate Authority (CA).

To get that on our scratch image, we need to change the image used during the build stage and eventually copy a few files from that build image into the scratch image. The full result is the following:

FROM **golang:1.19-bullseye** as build  
WORKDIR /app  
COPY src/go.mod src/hello.go ./  
RUN **CGO_ENABLED=0** go buildFROM scratch  
**COPY --from=build /usr/share/zoneinfo /usr/share/zoneinfo  
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/**  
COPY --from=build /app/hello /app/  
CMD [ "/app/hello" ]

**Final image size: 3.77 MB**

## Summary

Here is the final ranking :

Press enter or click to view image in full size

- It is fair to exclude the basic “scratch” image as it does not contain timezone and CA certificates, making it an unfair comparison.
- BusyBox, Scratch-advanced and Distroless are very close, sitting between 3.06 MB and 4.17 MB. A safe choice especially if you don’t intend to connect into the container and install other tools (e.g.: for debugging purpose).
- Debian is, without a lot of surprise, the biggest image and still remains a good choice for development purpose and testing, or if you want to deploy other tools inside your container.
- Alpine offers a good compromise between the 3 smallest images and the Debian.

## Conclusion

We have been focusing on image size, but they are certainly other considerations that you may want to add before selecting your solution :

- Are you going to **install other programs** on the image, and are you going to do so with a package manager (apt, apk, …) ?
- What are your **security constraints** ? If you want to reduce the surface of exposition, a minimal image would be best. I will try to write shortly an article around that specific aspect.
- You may have a **preferred Linux distribution** that you are most familiar with.