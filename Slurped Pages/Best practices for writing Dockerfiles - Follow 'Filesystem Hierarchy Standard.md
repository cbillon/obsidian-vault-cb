---
link: https://gagor.pro/2024/03/best-practices-for-writing-dockerfiles-follow-filesystem-hierarchy-standard/
byline: Tom
site: Tom's Blog
date: 2024-03-11T01:00
excerpt: Learn best practices for writing Dockerfiles by following the Filesystem Hierarchy Standard (FHS) to enhance organization and maintainability of your Docker images.
tags:
  - dockerfile
  - best_pratices
slurped: 2026-02-24T10:52
title: Best practices for writing Dockerfiles - Follow "Filesystem Hierarchy Standard"
---

When it comes to building Docker images, adhering to the “Filesystem Hierarchy Standard”[1](#fn:1)[2](#fn:2) can greatly enhance the organization and maintainability of your containers. Unfortunately, it’s not uncommon to encounter Docker images where files are haphazardly scattered across directories, leading to confusion and unnecessary complications. Let’s delve into some best practices to ensure your Dockerfiles follow the FHS guidelines, thus avoiding common pitfalls and streamlining your container development process.

Below you can find the most important directories, from the perspective of Docker images. `/dev` or `/root` rarely are useful here.

graph LR
    root["root /"] --> bin["/bin"]
    root --> sbin["/sbin"]
    root --> etc["/etc"]
    root --> home["/home"]
    root --> usr["/usr"]
    root --> opt["/opt"]
    root --> srv["/srv"]
    root --> tmp["/tmp"]
    root --> var["/var"]

    usr --> local["/local"]
    local --> local_bin["/bin"]
    local --> local_sbin["/sbin"]
    local --> share["/share"]
    share --> java["/java"]

    opt --> app["/your-app-name"]
    var --> var_lib["/lib"]
    var --> var_cache["/cache"]
    var --> var_log["/log"]
    var --> var_tmp["/tmp"]

## Organize scripts

One frequent oversight is the placement of scripts and binaries within Docker images. Instead of cluttering the root directory (`/`) or other unconventional locations, adhere to the FHS guidelines:

- **Scripts**: Store your scripts in `/usr/local/bin` or `/usr/local/sbin` for root-owned scripts. Placing scripts here eliminates the need to manipulate the `PATH` environment variable manually and ensures they’re easily accessible.
    
    ```
    COPY script.sh /usr/local/bin/
    ```
    
      
    

## Handle Java binaries properly

- **Binaries and Artifacts**: If your Docker image includes binary artifacts like Java `jar` packages or shared libraries, designate appropriate directories such as `/opt/your-app-name` or `/usr/local/share/java`. This practice maintains clarity and consistency within your image structure.
    
    ```
    COPY your-app.jar /opt/your-app-name/
    ```
    
      
    

## Handling Web Files for Web Servers

When configuring web servers like Nginx or Apache within Docker images, it’s crucial to adhere to FHS principles for storing web files. Here’s how you can effectively manage web content:

- **Nginx**: For Nginx, follow the convention of placing web files in `/usr/share/nginx/html`. This directory is the default location where Nginx looks for static web content. By adhering to this standard, you ensure seamless integration with Nginx configurations.
    
    ```
    COPY --chown=nginx:nginx index.html /usr/share/nginx/html/
    ```
    
      
    
- **Apache**: Similarly, Apache web server deployments should utilize `/var/www/html` for storing web files. This directory is Apache’s default document root, simplifying the setup and maintenance of your Apache-powered containers.
    
    ```
    COPY --chown=www-data:www-data index.html /var/www/html/
    ```
    
      
    

## Handling caches or temp files

Writing files inside of container is generally a bad idea. Layered file systems like `aufs` or `overlay` provide poor performance, so if you really need to write files inside container, write them to volumes. First setup any permissions, then make them volumes “and freeze” them in this state. Volumes are bind mounted to the local file system on hosts so they provide much better performance.

- **Temporary files**: There are two most common locations for temporary files.
    
    ```
    VOLUME ["/tmp", "/var/tmp"]
    ```
    
      
    
- **Cache**: Set permissions before volume creation, because later it’s not persisting. Remember that those files are stored on local file system, so set reasonable limits. When container will be terminated, volume will be left, so your cluster should have some cleanup job configured to purge them.
    
    ```
    RUN mkdir -p /var/cache/nginx && \
        chown -R nginx:nginx /var/cache/nginx
    VOLUME ["/var/cache/nginx"]
    ```
    
      
    

## Summary

By following these guidelines, you just make your life easier. Many things could just work out of the box, leading to smoother development and deployment workflows.