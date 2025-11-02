---
link: https://medium.com/@kittipat_1413/optimizing-multi-stage-builds-with-dockerfile-in-golang-a2ee8ed37ec6
byline: Kittipat.Po
site: Medium
date: 2023-07-16T08:45
excerpt: Optimizing Multi-Stage Builds with Dockerfile in GoLang Introduction Building efficient Docker images is one of those things that pays off every single day — smaller images mean faster deployments …
twitter: https://twitter.com/@Medium
slurped: 2025-11-02T19:36
title: Optimizing Multi-Stage Builds with Dockerfile in GoLang
tags:
  - go
  - dockerfile
---

## Introduction

Building efficient Docker images is one of those things that pays off every single day — smaller images mean **faster deployments, quicker CI/CD pipelines, and fewer headaches in production**.

In this post, we’ll walk through how to use **multi-stage builds** in Go to create lean and secure Docker images. We’ll start with why multi-stage builds matter, then dive into an optimized Dockerfile for a real-world Go service.

### What are Multi-Stage Builds?

Multi-stage builds let you define **multiple stages inside one Dockerfile**.

- One stage contains the full build environment (compilers, Git, Go toolchain).
- Another stage contains only the **final binary and runtime dependencies**.

This means you can throw away everything that isn’t needed to _run_ your app — leaving behind a **minimal, secure, and portable image**.

### Benefits of Multi-Stage Builds in GoLang:

1. **Smaller Docker Images**: By leveraging multi-stage builds, we can create a minimalistic final image that only contains the necessary runtime dependencies for our GoLang application. This reduces the image size, leading to faster and more efficient deployments.
2. **Improved Security**: In multi-stage builds, the final stage can exclude all the build-time tools and dependencies, leaving only the runtime components required to run the application. This separation significantly reduces the attack surface and potential vulnerabilities.
3. **Faster Builds**: By splitting the build process into multiple stages, Docker can leverage its caching mechanism effectively. Docker only rebuilds the stages that have changed, while reusing the previously built stages. This can dramatically speed up the build process, especially for iterative development.

### Optimizing Multi-Stage Builds in GoLang

To demonstrate the optimization techniques for multi-stage builds in GoLang, let’s create a sample Dockerfile for a simple Go application.

### Example Dockerfile:

# ---------- Stage 1: Build ----------  
FROM golang:1.25-alpine AS builder # 👈 change 1.25 to your Go version ‼️

# Set the working directory  
WORKDIR /app

# Ensure a portable, static-ish binary  
ENV CGO_ENABLED=0 GOOS=linux GOARCH=amd64

# Copy and download dependencies  
COPY go.mod go.sum ./  
RUN go mod download

# Copy the source code  
COPY . .

# Build the Go application (strip debug info for smaller size)  
RUN go build -trimpath -ldflags="-s -w" -o myapp .

# ---------- Stage 2: Final ----------  
FROM alpine:latest   # 👈 consider pinning (e.g., alpine:3.20) ‼️

# Set the working directory  
WORKDIR /app

# Install runtime dependencies you actually need  
RUN apk add --no-cache ca-certificates tzdata

# Create non-root user for security  
RUN addgroup -S appuser \  
 && adduser -S -G appuser -H -s /sbin/nologin appuser

# Copy the binary and set ownership  
COPY --from=builder --chown=appuser:appuser /app/myapp /app/myapp

# Run as non-root user  
USER appuser

# Set the entrypoint command  
ENTRYPOINT ["/app/myapp"]

## Explanation:

The Dockerfile is split into two stages: a **build stage** and a **final stage**.

### **Build** Stage

- `**FROM golang:1.25-alpine AS builder**`: Uses the official Go image with Alpine Linux as the base. ( ⚠️ Update 1.25 to match the Go version your project uses. )
- `**WORKDIR /app**`: Sets the working directory inside the container to `/app`.
- `**ENV CGO_ENABLED=0 GOOS=linux GOARCH=amd64**`: Ensures the binary is built for Linux with CGO disabled, resulting in a more portable, static-ish binary that doesn’t depend on system libraries.
- `**COPY go.mod go.sum ./ + RUN go mod download**`: Copies only the dependency files first, so Docker can cache module downloads and avoid re-downloading them every time the source code changes.
- `**COPY . .**`: Copies the entire application source code into the container.
- `**RUN go build -trimpath -ldflags=”-s -w” -o myapp .**`: Compiles the Go application and produces a binary named `myapp`.  
    - `trimpath` removes file system paths from the compiled binary.  
    - `ldflags="-s -w"` strips debugging info, making the binary smaller.

### Final Stage

- `**FROM alpine:latest**`: Starts a fresh, minimal Alpine Linux image.  
    ( ⚠️ For reproducible builds, consider pinning to a specific version like `alpine:3.20`. )
- `**WORKDIR /app**`: Sets the working directory again inside the new, clean image.
- `**RUN apk add — no-cache ca-certificates tzdata**`: Installs only the runtime dependencies needed for many Go apps:  
    - `ca-certificates` → enables secure HTTPS requests.  
    - `tzdata` → provides accurate timezone data.
- `**RUN addgroup -S appuser && adduser -S -G appuser -H -s /sbin/nologin appuser**`: Creates a dedicated non-root user (`appuser`) for running the app. This is a security best practice, reducing the risk if the container is compromised.
- `**COPY — from=builder — chown=appuser:appuser /app/myapp /app/myapp**`: Copies the built binary from the builder stage into the final image, assigning ownership to the non-root user.
- `**USER appuser**`: Ensures the app runs as a non-root user inside the container.
- `**ENTRYPOINT [“/app/myapp”]**`: Defines the binary as the container’s entrypoint — this is the command that runs automatically when the container starts.

By following this Dockerfile, you can create an optimized and minimal Docker image for your GoLang application, reducing its size and improving efficiency.

> Remember, the key is to strike the right balance between reducing image size and maintaining the necessary dependencies for your application to function properly. Happy optimizing!
