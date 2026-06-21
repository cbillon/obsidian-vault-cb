---
link: https://konradreiche.com/blog/docker-bash-helpers/
byline: Konrad Reiche
excerpt: "As I have started to dive into the world of Docker I found myself using a new plethora of commands in my terminal. Thanks to a combination of my terminal multiplexer (tmux), history search (reverse-i-search) and fuzzy finder (fzf) it is relatively easy to retrieve previous commands. When you need to replay a combination of commands this starts to become inefficient. Here is a collection of Bash aliases and functions I use to utilize my daily work with Docker:"
slurped: 2026-06-21T10:11
title: Docker Bash Helpers
tags:
  - docker
  - linux
  - bash
---

May 13, 2016

As I have started to dive into the world of **Docker** I found myself using a new plethora of commands in my terminal. Thanks to a combination of my terminal multiplexer ([tmux](https://tmux.github.io/)), history search ([reverse-i-search](https://www.gnu.org/software/bash/manual/html_node/Commands-For-History.html)) and fuzzy finder ([fzf](https://github.com/junegunn/fzf)) it is relatively easy to retrieve previous commands. When you need to replay a combination of commands this starts to become inefficient. Here is a collection of Bash aliases and functions I use to utilize my daily work with Docker:

## Kill all running Docker container

```
alias docker-killall='docker kill $(docker ps -q)'
```

Here `docker ps` lists all running containers where the `-q` (quiet) option only displays the numeric identifier. With command substitution the results of one command can be passed to the next one. I use this command primarily when I want to make sure that there are no Docker containers left running in the background.

## Remove all Docker container

```
alias docker-rm='docker rm $(docker ps -a -q)'
```

The `-a` (all) option displays in addition to running containers also stopped containers.

## Remove all Docker images

```
alias docker-rmi='docker rmi $(docker images -q)'
```

Sometimes it is also necessary to get rid of all images to start fresh. Here `docker images` is used to list the images stored in the local Docker repository.

## Inspect a running Docker container

With the release of Docker 1.3.0 the command `docker exec` was added which allows to spawn a process inside a running Docker container and even though I live by the following principle

> If you have to ssh into one of your instances your automation has failed.

it is arguable invaluable for development and debugging purposes to create a new Bash session inside a Docker container which can be achieved with the following command:

```
$ docker exec -it CONTAINER bash
```

I just found it increasingly annoying to look up the container identifier each time with `docker ps` and copy-paste it into the command string. In order to automatize this I created a Bash function and alias it:

```
function find_docker_container() {
  docker ps | grep "$1 " | cut -d ' ' -f1
}

function inspect_docker_container() {
  docker exec -it `find_docker_container $1` bash
}

alias docker-inspect=inspect_docker_container
```

With `grep $1` the container with the name we are looking for is parsed and with `cut -d ' ' -f1` the first column (ID) is selected. This makes it nice and easy to quickly jump into one of your containers:

```
$ docker-inspect redis
root@29892c13c29d:/#
```

While it is surely fun to create your own aliases to automate tedious tasks it might become problematic when you actually have to use these commands on a remote instance where you cannot import your Bash profile so make sure you understand the underlying commands before seeking comfort.