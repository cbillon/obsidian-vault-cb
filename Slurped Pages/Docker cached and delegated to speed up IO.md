---
link: https://blog.oxyconit.com/docker-cached-and-delegated-to-speed-up-io/
byline: Kamil Dzierbicki
site: Oxycon Blog
date: 2024-01-29T17:57
excerpt: >-
  As developer, you have a broad understanding of Docker. However, some hidden
  gems can significantly improve your workflow. Docker's cached and delegated
  methods are two such valuable features. This article will walk you through
  their importance and use.



  Docker and Data Consistency


  Docker's primary purpose is to foster seamless collaboration between the host
  and the Docker container. It keeps data consistent between the two
  environments. This shared data interaction is facilitated by Docker vo
twitter: https://twitter.com/@KamilOxyconIt
slurped: 2024-11-18T09:40
title: Docker cached and delegated to speed up IO
---

As developer, you have a broad understanding of Docker. However, some hidden gems can significantly improve your workflow. Docker's `cached` and `delegated` methods are two such valuable features. This article will walk you through their importance and use.

### Docker and Data Consistency

Docker's primary purpose is to foster seamless collaboration between the host and the Docker container. It keeps data consistent between the two environments. This shared data interaction is facilitated by Docker volumes, which are essentially "parts" of the container. Volumes act as a storage mechanism for shared data. They exist independently of the container state, so if a container is removed, the volume persists. It ensures that backup data exists both in the container and on the host file system.

These Docker volumes can be viewed in three modes:  
- `default`: This mode provides full consistency. Updates are performed simultaneously on both the host and container sides  
- `delegated`: when the container performs changes a lot, the host is in 'read-only mode' (for example for databases when you don't expect changes immediately visible on your host machine by reducing file system calls to your host).  
- `cached`: when the host performs changes, the container is in 'read-only mode' (for example you are changing code in your host a lot).

An example use of a Docker volume is as follows:

```
- ./my_host_delegated_folder:/container/volume/path:delegated
- ./my_host_cached_folder:/container/volume/path:cached
- ./my_host_cached_readonly_folder:/container/volume/path:ro,cached
```

### Focus Example: Docker Compose File for Nginx Service

Here is a sample `docker-compose.yml` file which sets up a service using nginx and a read-only, cached volume.

```
version: '3.4'
  services:
    nginx_service:
      image: nginx:latest
      container_name: nginx_my_service
      restart: unless-stopped
      volumes:
        - ./data:/usr/share/nginx/html:ro,cached
      ports:
        - 8080:80
```

This `docker-compose` file sets up a service named `nginx_service` using the latest nginx image. The `./data` volume on the host is mounted on `/usr/share/nginx/html` on the container, and is read-only and cached.

### Summary

In conclusion, Docker's cached and delegated methods are incredibly versatile features that create data consistency between the host and the container. They are useful when one side is constantly making changes to the data, while the other side is set to read-only. Knowing when to use each method can greatly optimize your data flow.  
Happy coding!