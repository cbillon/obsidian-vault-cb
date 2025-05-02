---
tags:
  - docker
  - restart
---
## 1. Overview

In this tutorial, we’ll learn how to use the restart policy with [Docker Compose](https://www.baeldung.com/ops/docker-compose).

First, we’ll cover how to restart Docker containers with restart policies. Then we’ll cover how Docker Compose defines restart policies in normal mode and swarm mode as a configuration for multi-container Docker applications.

## 2. Docker Restart Policy

**Restart policies are strategies we can use to restart Docker containers automatically and manage their lifecycles.**

Given that containers can fail unexpectedly, **Docker has safeguards to prevent services from running into a restart loop**. In case of a failure, restart policies don’t take effect unless the container runs successfully for at least 10 seconds.

We can also assume that manually stopping a container will make Docker restart the service automatically when a restart policy is provided. However, these policies are dismissed in this case to prevent containers from restarting after being stopped arbitrarily.

To use restart policies, Docker provides the following options:

- _no_: Containers won’t restart automatically.
- _on-failure[:max-retries]_: Restart the container if it exits with a non-zero exit code, and provide a maximum number of attempts for the Docker daemon to restart the container.
- _always_**:** Always restart the container if it stops.
- _unless-stopped_: Always restart the container unless it was stopped arbitrarily, or by the Docker daemon.

Now let’s look at an example of how to set a restart policy using the Docker CLI for a single container:

```plaintext
docker run --restart always my-service
```

**From the example above, _my-service_ will _always_ restart if the container stops running.** However, **if we explicitly stop the container, the restart policy will only take effect when the Docker daemon restarts, or when we use the [_restart_ command](https://www.baeldung.com/ops/docker-compose-restart-container).**

The previous example demonstrates how the _restart_ flag configures the strategy to restart a single container automatically. However, **Docker Compose allows us to configure restart policies to manage multiple containers by using the _restart_ or _restart_policy_ properties in normal mode or swarm mode.**

## 3. Setup

Before diving into the restart policies implementation with Docker Compose, let’s set up a working environment.

We must have a running Docker container to test the restart policies we’ll specify. We’ll use a project from a previous article, [_spring-cloud-docker_](https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-docker), which is a dockerized Spring Boot application. This project has two Docker services that we’ll use to implement different restart strategies with Docker Compose.

First, we must confirm that we can run both containers by running the following command from the project root:

```plaintext
docker-compose up --detach --build
```

Now we should be able to see both services running by executing _docker-compose ps_:

```plaintext
$ docker ps
     Name                   Command              State            Ports         
--------------------------------------------------------------------------------
message-server   java -jar /message-server.jar   Up      0.0.0.0:18888->8888/tcp
product-server   java -jar /product-server.jar   Up      0.0.0.0:19999->9999/tcp
```

Alternatively, we could go to _localhost:18888_ or _localhost:19999_ in our browser and verify that we see the messages displayed by the application services.

## 4. Restart Policy in Docker Compose

Like the _restart_ Docker command, **Docker Compose includes the _restart_ property to restart containers automatically.**

We can also define restart policies in Docker Compose by providing the _restart_ property to the service in the _docker-compose.yml_ file. **Docker Compose uses the same values provided by the Docker CLI _restart_ command to define a restart strategy.**

Now let’s create a restart policy for our containers. In the _spring-cloud-docker_ project, we must change the _docker-compose.yml_ configuration file by adding the restart policy property:

```yaml
message-server:
    container_name: message-server
    build:
        context: docker-message-server
        dockerfile: Dockerfile
    image: message-server:latest
    ports:
        - 18888:8888
    networks:
        - spring-cloud-network
    restart: no
product-server:
    container_name: product-server
    build:
        context: docker-product-server
        dockerfile: Dockerfile
    image: product-server:latest
    ports:
        - 19999:9999
    networks:
        - spring-cloud-network
    restart: on-failure


Notice how we added the _restart_ property to both services. In this case, the _message-server_ will never restart automatically. The _product-server_ will only restart if it exits with a non-zero code as specified by the _on-failure_ value.

Next, let’s see how the same policies are declared using Docker Compose in swarm mode.

## 5. Restart Policy in Docker Compose Swarm Mode

**Docker Compose in swarm mode provides a larger set of options when specifying how containers will restart automatically. However, the following implementation only works in Docker Compose v3, which introduces the _deploy_ key-value pair in the configuration**.

Below we can find the different properties to further expand the configuration for restart policies in swarm mode:

- _condition_**:** _none_, _on-failure,_ or _any_ (default)
- _delay_**:** Duration between restart attempts
- _max_attempts_**:** Maximum number of attempts outside of the restart _window_
- _window_**:** Duration for determining if a restart is successful

Now let’s define our restart policies. First, we must make sure we’re using Docker Compose v3 by changing the _version_ property:

```yaml
version: '3'
```

Once we change the version, we can add the _restart_policy_ property to our services. Similar to the previous section, our _message-server_ container will always restart automatically by providing the _any_ value in the _condition_:

```yaml
deploy:
    restart_policy:
        condition: any
        delay: 5s
        max_attempts: 3
        window: 120s
```

Similarly, we’ll add an _on-failure_ restart policy to the _product-server_:

```yaml
deploy:
    restart_policy:
        condition: on-failure
        delay: 3s
        max_attempts: 5
        window: 60s
```

**Notice how the _restart_policy_ property is within the _deploy_ key, which indicates that restart policies are a deployment configuration we provide to manage a cluster of containers in swarm mode.**

Also, the restart policies in both services include the additional configuration metadata that makes the policies a more robust restart strategy for the containers.

## 6. Conclusion

In this article, we learned how to define restart policies with Docker Compose. After introducing Docker restart policies, we used a previous Baeldung project to demonstrate two ways to configure a restart strategy for containers.

First, we used the Docker Compose _restart_ property configuration, which includes the same options as the native _restart_ command in the Docker CLI. Then we used the _restart_policy_ property, only available in swarm mode and Docker Compose version 3, along with other configuration values that define restart policies.

As always, all the sample code used in this article is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-docker).