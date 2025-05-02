---
link: https://medium.com/javarevisited/docker-volume-and-storage-how-internal-things-works-080b837ea178
byline: WhatInDev
site: Javarevisited
date: 2025-02-13T13:52
excerpt: We often misunderstood storage and volume management aspect in Docker. In this article, we will take an in-depth look at Docker volumes and storage, covering not only how they work but also some…
twitter: https://twitter.com/@javarevisited
slurped: 2025-04-11T06:41
title: "Docker Volume and Storage: How internal things works"
tags:
  - docker
  - volumes
---

## 1. Understanding Docker Storage: The Basics

Before diving into Docker volumes, it’s essential to understand how Docker handles storage at a high level. When you run a container, its filesystem is ephemeral — meaning that once the container stops or is removed, any data written inside the container is lost.

Docker offers several ways to persist data:

1. **Bind mounts**
2. **Volumes**
3. **tmpfs mounts**
4. **Named vs. Anonymous Volumes**

_Comparison of different storage types_

## 1. Bind Mounts

Bind mounts allow you to mount a directory from the host machine into a container. These are commonly used in development environments where you want to edit files on the host and see the changes reflected in the container.

**Example:**

docker run -d --name my_container -v /host/path:/container/path nginx

**Key Insight**: Bind mounts are not managed by Docker, meaning they depend on the host filesystem structure and may cause compatibility issues if moved across different environments.

## 2. Volumes

Unlike bind mounts, Docker **manages volumes** independently from the host filesystem. They are stored in `/var/lib/docker/volumes/` by default, ensuring better portability and stability.

_how Docker volumes are stored and managed separately from containers_

Example:

docker volume create my_volume  
docker run -d --name my_container -v my_volume:/data nginx

**Key Insight**: Volumes are the preferred method for production environments due to their reliability and compatibility across different host systems.

## 3. tmpfs Mounts

Tmpfs mounts store data in memory rather than disk, making them useful for sensitive data that shouldn’t persist between container restarts.

## Example:
```bash
 docker run -d --tmpfs /app:rw,size=100m nginx
```
## 2. Docker Volumes: How they work under the hood

Volumes provide a way to persist data beyond the lifecycle of a container. They are stored outside the container’s filesystem and are managed by Docker itself.

## Creating a Volume
```bash
docker volume create my_volume
```

## Inspecting a Volume
```bash
docker volume inspect my_volume
```
## Removing a Volume
```bash
docker volume rm my_volume
```

**Key Insight**: Be careful when removing volumes, as **Docker does not warn you if data is inside**. Use `docker volume prune` with caution!

## 4. Advanced Volume Features

## Named vs. Anonymous Volumes

Named vs. Anonymous Volumes

- **Named volumes** are explicitly created and referenced by name.
- **Anonymous volumes** are created dynamically by Docker when a volume is mounted without a name.

## Example of an anonymous volume:
```bash
docker run -d -v /data nginx
```
## Volume Drivers

Docker supports external storage solutions via volume drivers.

Example of an NFS volume:
```bash
docker volume create --driver local --opt type=nfs --opt o=addr=192.168.1.100,rw --opt device=:/data my_nfs_volume
```
## 5. Troubleshooting Common Volume Issues

## Volume Not Persisting Data

**Problem:** Data disappears after a container restart.

**Solution:** Ensure you’re using a **named volume** instead of an anonymous one.

## Permission Issues

**Problem:** The container cannot write to a mounted volume.

**Solution:** Adjust permissions using:
```bash
docker run -d --user $(id -u):$(id -g) -v my_volume:/data nginx
```