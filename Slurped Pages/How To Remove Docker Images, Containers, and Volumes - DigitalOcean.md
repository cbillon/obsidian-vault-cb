---
link: https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes
excerpt: Free up disk space and optimize your Docker environment by learning how to remove unused images, containers, and volumes.
slurped: 2024-12-14T10:56
title: How To Remove Docker Images, Containers, and Volumes | DigitalOcean
tags:
  - docker
  - volumes
  - Moodle-in-English/-The-rate-of-upgrading-Moodle-and-the-significance-of-the-LTS-Moodle-org
---

### [Introduction](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#introduction)

[Docker](https://www.docker.com/) makes it easy to wrap your applications and services in containers so you can run them anywhere. However, as you work with Docker, it’s also easy to accumulate an excessive number of unused images, containers, and data volumes that clutter the output and consume disk space.

Docker gives you all the tools you need to clean up your system from the command line. This cheat sheet-style guide provides a quick reference to commands that are useful for freeing disk space and keeping your system organized by removing unused Docker images, containers, and volumes.

**How to Use This Guide:**

- This guide is in cheat sheet format with self-contained command-line snippets.
- Jump to any section that is relevant to the task you are trying to complete.

**Note:** The command substitution syntax, `==command== $(==command==)`, used in the commands is available in many popular shells, such as [bash](https://www.digitalocean.com/community/tutorial-series/an-introduction-to-shell-scripting), [zsh](https://www.zsh.org/), and [Windows Powershell](https://docs.microsoft.com/en-us/powershell/).

If you want a 1-click way to deploy a Docker application to a live server, take a look at [DigitalOcean App Platform](https://www.digitalocean.com/products/app-platform).

## [Purging All Unused or Dangling Images, Containers, Volumes, and Networks](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#purging-all-unused-or-dangling-images-containers-volumes-and-networks)

Docker provides a single command that will clean up any resources — images, containers, volumes, and networks — that are _dangling_ (not tagged or associated with a container):

```
docker system prune
```

To additionally remove any stopped containers and all unused images (not just dangling images), add the `-a` flag to the command:

```
docker system prune -a
```

## [Removing Docker Images](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#removing-docker-images)

### [Remove one or more specific images](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-one-or-more-specific-images)

Use the `docker images` command with the `-a` flag to locate the ID of the images you want to remove. This will show you every image, including intermediate image layers. When you’ve located the images you want to delete, you can pass their ID or tag to `docker rmi`:

**List:**

```
docker images -a
```

**Remove:**

```
docker rmi Image Image
```

**Note:** The `-a` or `--all` flag in the `docker images` command displays all the Docker images, including intermediate ones that are not referenced by any tags. By default, `docker images` shows only the images with at least one tag. However, there may be some images without any tags that are still taking up disk space on the system. The `-a` flag can be helpful in identifying images that can be pruned to save disk space. When used with the `docker rmi` command, the `-f` or `--force` flag can also be used to remove images with no tags.

### [Remove Dangling Docker Images](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-dangling-docker-images)

Docker images consist of multiple layers. Dangling images are layers that have no relationship to any tagged images. They no longer serve a purpose and consume disk space. They can be located by adding the filter flag `-f` with a value of `dangling=true` to the `docker images` command. When you’re sure you want to delete them, you can use the `docker image prune` command:

**Note:** If you build an image without tagging it, the image will appear on the list of dangling images because it has no association with a tagged image. You can avoid this situation by [providing a tag](https://docs.docker.com/engine/reference/commandline/build/#/tag-image--t) when you build, and you can retroactively tag an image with the [`docker tag`](https://docs.docker.com/engine/reference/commandline/tag/) command.

**List:**

```
docker images -f dangling=true
```

**Remove:**

```
docker image prune
```

### [Removing images according to a pattern](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#removing-images-according-to-a-pattern)

You can find all the images that match a pattern using a combination of `docker images` and [`grep`](https://www.digitalocean.com/community/tutorials/using-grep-regular-expressions-to-search-for-text-patterns-in-linux). Once you’re satisfied, you can delete them by using [`awk`](https://www.digitalocean.com/community/tutorials/how-to-use-the-awk-language-to-manipulate-text-in-linux) to pass the IDs to `docker rmi`. Note that these utilities are not supplied by Docker and are not necessarily available on all systems:

**List:**

```
docker images -a |  grep "pattern"
```

**Remove:**

```
docker images -a | grep "pattern" | awk '{print $1":"$2}' | xargs docker rmi
```

### [Remove all images](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-all-images)

All the Docker images on a system can be listed by adding `-a` to the `docker images` command. Once you’re sure you want to delete them all, you can add the `-q` flag to pass the image ID to `docker rmi`:

**List:**

```
docker images -a
```

**Remove:**

```
docker rmi $(docker images -a -q)
```

## [Removing Containers](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#removing-containers)

### [Remove one or more specific containers](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-one-or-more-specific-containers)

Use the `docker ps` command with the `-a` flag to locate the name or ID of the containers you want to remove:

**List:**

```
docker ps -a
```

**Remove:**

```
docker rm ID_or_Name ID_or_Name
```

### [Remove a container upon exiting](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-a-container-upon-exiting)

If you know when you’re creating a container that you won’t want to keep it around once you’re done, you can run `docker run --rm` to automatically delete it when it exits:

**Run and Remove:**

```
docker run --rm image_name
```

### [Remove all exited containers](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-all-exited-containers)

You can locate containers using `docker ps -a` and filter them by their status: `created`, `restarting`, `running`, `paused`, or `exited`. To review the list of `exited` containers, use the `-f` flag to filter based on status. When you’ve verified you want to remove those containers, use `-q` to pass the IDs to the `docker rm` command:

**List:**

```
docker ps -a -f status=exited
```

**Remove:**

```
docker rm $(docker ps -a -f status=exited -q)
```

### [Remove containers using more than one filter](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-containers-using-more-than-one-filter)

Docker filters can be combined by repeating the filter flag with an additional value. This results in a list of containers that meet either condition. For example, if you want to delete all containers marked as either `created` (a state which can result when you run a container with an invalid command) or `exited`, you can use two filters:

**List:**

```
docker ps -a -f status=exited -f status=created
```

**Remove:**

```
docker rm $(docker ps -a -f status=exited -f status=created -q)
```

### [Remove containers according to a pattern](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-containers-according-to-a-pattern)

You can find all the containers that match a pattern using a combination of `docker ps` and [`grep`](https://www.digitalocean.com/community/tutorials/using-grep-regular-expressions-to-search-for-text-patterns-in-linux). When you’re satisfied that you have the list you want to delete, you can use [`awk`](https://www.digitalocean.com/community/tutorials/how-to-use-the-awk-language-to-manipulate-text-in-linux) and `xargs` to supply the ID to `docker rm`. Note that these utilities are not supplied by Docker and are not necessarily available on all systems:

**List:**

```
docker ps -a |  grep "pattern”
```

**Remove:**

```
docker ps -a | grep "pattern" | awk '{print $1}' | xargs docker rm
```

### [Stop and remove all containers](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#stop-and-remove-all-containers)

You can review the containers on your system with `docker ps`. Adding the `-a` flag will show all containers. When you’re sure you want to delete them, you can add the `-q` flag to supply the IDs to the `docker stop` and `docker rm` commands:

**List:**

```
docker ps -a
```

**Remove:**

```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

## [Removing Volumes](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#removing-volumes)

### [Remove one or more specific volumes](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-one-or-more-specific-volumes)

Use the `docker volume ls` command to locate the volume name or names you wish to delete. Then you can remove one or more volumes with the `docker volume rm` command:

**List:**

```
docker volume ls
```

**Remove:**

```
docker volume rm volume_name volume_name
```

### [Remove dangling volumes](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-dangling-volumes)

Since the point of volumes is to exist independent from containers, when a container is removed, a volume is not automatically removed at the same time. When a volume exists and is no longer connected to any containers, it’s called a _dangling volume_. To locate them to confirm you want to remove them, you can use the `docker volume ls` command with a filter to limit the results to dangling volumes. When you’re satisfied with the list, you can remove them all with `docker volume prune`:

**List:**

```
docker volume ls -f dangling=true
```

**Remove:**

```
docker volume prune
```

### [Remove a container and its volume](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#remove-a-container-and-its-volume)

If you create an unnamed volume, it can be deleted at the same time as the container with the `-v` flag. Note that this only works with _unnamed_ volumes. When the container is successfully removed, its ID is displayed. Note that no reference is made to the removal of the volume. If it is unnamed, it is silently removed from the system. If it is named, it silently stays present.

**Remove:**

```
docker rm -v container_name
```

## [Conclusion](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes#conclusion)

This guide covers some of the common commands used to remove images, containers, and volumes with Docker. There are many other combinations and flags that can be used with each. For a comprehensive guide to what’s available, see the Docker documentation for [`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/), [`docker rmi`](https://docs.docker.com/engine/reference/commandline/rmi/), [`docker rm`](https://docs.docker.com/engine/reference/commandline/rm/), and [`docker volume rm`](https://docs.docker.com/engine/reference/commandline/volume_rm/). If there are common cleanup tasks you’d like to see in the guide, please ask or make suggestions in the comments.

For a detailed look at the different components of a Docker container, check out [The Docker Ecosystem: An Introduction to Common Components](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-an-introduction-to-common-components).