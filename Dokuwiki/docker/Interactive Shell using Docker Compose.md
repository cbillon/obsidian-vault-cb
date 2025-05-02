---
tags:
  - docker-compose
  - shell
  - interactive
---
## 1. Overview

In this tutorial, we’ll learn how to run multiple [Docker](https://www.baeldung.com/ops/docker-guide) containers with an interactive shell. First, we’ll run a Docker container using the simple [_docker run_](https://www.baeldung.com/ops/running-docker-containers-indefinitely) command. Later, we’ll run the same Docker container with the [_docker-compose_](https://www.baeldung.com/ops/docker-compose) command.

## 2. Docker and Docker Compose

Docker containers allow developers to package applications that work seamlessly across different environments. In fact, a typical deployment of a web application in a production environment may require several services:

- database server
- load balancing
- webserver

In such situations, Docker Compose is a very handy tool.

**Docker Compose is mainly used to run multiple containers as a single service while maintaining a smooth connection between the containers.**

## 3. Understanding Docker Compose

To run a Docker container using the _docker-compose_ command, we need to add all the configurations to the single _docker-compose.yml_ configuration file. Importantly, **one of the key benefits of using _docker-compose_ over the normal _docker run_ command is the configuration consolidation in a single file, which both machines and humans can read.**

Let’s create a simple _docker-compose.yml_ to show how to run Docker containers using the [_docker-compose up_](https://docs.docker.com/engine/reference/commandline/compose_up/) command:

```bash
version: "3"
services:
 server:
   image: tomcat:jre11-openjdk
   ports:
     - 8080:8080
```

Here, we used [_tomcat_](https://www.baeldung.com/tomcat) as the base image and exposed port _8080_ on the host machine. To see it in action, let’s build and run this image using the _docker-compose up_ command:

```bash
$ docker-compose up
Pulling server (tomcat:jre11-openjdk)...
jre11-openjdk: Pulling from library/tomcat
001c52e26ad5: Pull complete
...
704b1ae41f0e: Pull complete
Digest: sha256:85bfe38b723bc864ed594973a63c04b112e20d6d33eee57cd5303610d8e3dc77
Status: Downloaded newer image for tomcat:jre11-openjdk
Creating dockercontainers_server_1 ... done
Attaching to dockercontainers_server_1
server_1  | NOTE: Picked up JDK_JAVA_OPTIONS:  --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
server_1  | 03-Aug-2022 06:22:17.259 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/10.0.23
```

Critically, we should run the above command from the directory containing the _docker-compose.yml_ file.

In the above output, we can see that the _dockercontainers_server_1_ is up and running. However, **an issue with this approach is that once we exit the above shell, the container will also stop**.

To run the Docker container for the long run, we need to run it with an interactive shell.

## 4. Interactive Shell in Docker

**The interactive mode in Docker allows us to execute commands while the container is in a _running_ state.** To run the Docker container in interactive mode, we use the _-it_ option. Further, we attach both the [STDIN and STDOUT](https://www.baeldung.com/linux/stream-redirections) channels to our terminal with the _-it_ flags.

Docker Compose uses a single-host deployment that has multiple benefits:

- quick and easy to configure
- enables rapid deployment
- reduces the amount of time needed to complete multiple tasks
- all containers run independently, which reduces the risk of a breach

Let’s now run the _tomcat_ container from earlier using _docker-compose_ with an interactive shell:

```bash
version: "3"
services:
 server:
   image: tomcat:jre11-openjdk
   ports:
     - 8080:8080
   stdin_open: true 
   tty: true
```

In this case, **we a****dded the _stdin_open_ and _tty_ options in the _docker-compose.yml_ file so that we can have an interactive shell with the _docker-compose_ setup**.

Of course, to access the Docker container, we need first to run the container using the below command:

```bash
$ docker-compose up --d
```

Now, we can get an interactive shell of the running _docker-compose_ service:

```bash
$ docker-compose exec server bash
```

**Note how we use the service name and not the container name**.

Finally, we successfully log into the container with the above command.

## 5. Conclusion

In this article, we demonstrated how to get an interactive shell using the _docker-compose_ command. First, we learned how to run a Docker container using _docker-compose_. After that, we explored the same with an interactive shell using the _docker exec_ command and the _docker-compose_ YAML configuration.