---
link: https://cylab.be/blog/434/simplify-your-docker-management-with-portainer
excerpt: Docker and containers have revolutionized the way we build, ship, and run applications! However, managing multiple containers and services can become increasingly complex. That’s where Portainer comes in! This powerful and intuitive container management web application makes it easy to deploy, manage, and monitor your Docker environments.
slurped: 2026-05-19T08:30
title: Simplify Your Docker Management with Portainer
tags:
  - docker
---

Docker and containers have revolutionized the way we build, ship, and run applications! However, managing multiple containers and services can become increasingly complex. That’s where Portainer comes in! This powerful and intuitive container management web application makes it easy to deploy, manage, and monitor your Docker environments.

In this blog post, we will install Portainer using docker-compose, and explore its key features.

[![portainer-dashboard.png](https://cylab.be/storage/blog/434/files/GkMDDxl6E2PQqxmR/portainer-dashboard.png)](https://cylab.be/storage/blog/434/files/GkMDDxl6E2PQqxmR/portainer-dashboard.png)

## Installation[](https://cylab.be/blog/434/simplify-your-docker-management-with-portainer#installation)

For this post, we will install Portainer using Docker and Docker compose…

### Docker and docker compose

Import the repository key:

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

Add the repository to Apt sources:

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Install Docker and docker-compose plugin:

```
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

### Portainer

Create a **docker-compose.yaml** file with the following content:

```
services:
  portainer:
    # 2.33 is the current LTS version
    image: portainer/portainer-ce:2.33.0
    command: -H unix:///var/run/docker.sock
    restart: "unless-stopped"
    ports:
      # http web ui
      - 9000:9000
      # https web ui
      - 9443:9443
      # TCP tunnel
      #- "8000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer:/data

volumes:
  portainer:
```

Start the container with:

```
sudo docker compose up
```

And that’s it…

## First steps[](https://cylab.be/blog/434/simplify-your-docker-management-with-portainer#first-steps)

After a few seconds, the web interface of Portainer will be available at [https://ip.of.your.server:9443](https://ip.of.your.server:9443/)

⚠ Beware that this is a self-signed certificate hence your browser will show you a warning…

[![portainer-certificate.png](https://cylab.be/storage/blog/434/files/bXD499qO5AKzehCX/portainer-certificate.png)](https://cylab.be/storage/blog/434/files/bXD499qO5AKzehCX/portainer-certificate.png)

Then you’ll be presented with a wizard that will allow you to create a first administrator account.

[![portainer-wizard.png](https://cylab.be/storage/blog/434/files/wUelBW3PdERyNNqR/portainer-wizard.png)](https://cylab.be/storage/blog/434/files/wUelBW3PdERyNNqR/portainer-wizard.png)

Finally you’ll arrive on Portainer dashboard. **A single Portainer instance can actually manage multiple Docker hosts (called _Environments_)**, so they will be listed here…

[![portainer-dashboard.png](https://cylab.be/storage/blog/434/files/GkMDDxl6E2PQqxmR/portainer-dashboard.png)](https://cylab.be/storage/blog/434/files/GkMDDxl6E2PQqxmR/portainer-dashboard.png)

## Stacks and containers[](https://cylab.be/blog/434/simplify-your-docker-management-with-portainer#stacks-and-containers)

Once you select an environment (in this case the Docker host on which Portainer itself is running), you can easily manage all components like stacks, individual containers, images, networks and volumes.

[![portainer-stacks.png](https://cylab.be/storage/blog/434/files/XnKt5bJzDlvlZPOo/portainer-stacks.png)](https://cylab.be/storage/blog/434/files/XnKt5bJzDlvlZPOo/portainer-stacks.png)

Stacks deployed with docker compose (like Portainer itself) will appear with a “Limited” control indication. Indeed, Portainer provides additional features when stacks are deployed with Portainer itself, like webhook and access control.

The interface to create a new stack allows to copy-paste, upload or fetch from github the content of a `docker-compose.yaml` file:

[![portainer-new-stack.png](https://cylab.be/storage/blog/434/files/SEW2UieeySPyjugT/portainer-new-stack.png)](https://cylab.be/storage/blog/434/files/SEW2UieeySPyjugT/portainer-new-stack.png)

You can test it with a simple image:

```
services:
  web:
    image: cylab/hello
    ports:
      - 8080:80
```

And after a few seconds the container will be running the web page will be available at [http://ip.of.your.server:8080](http://ip.of.your.server:8080/)

[![portainer-stack.png](https://cylab.be/storage/blog/434/files/2TQPjsAbSKzaAWYF/portainer-stack.png)](https://cylab.be/storage/blog/434/files/2TQPjsAbSKzaAWYF/portainer-stack.png)

[![portainer-cylab-hello.png](https://cylab.be/storage/blog/434/files/RBQEeitHtDpqo3bU/portainer-cylab-hello.png)](https://cylab.be/storage/blog/434/files/RBQEeitHtDpqo3bU/portainer-cylab-hello.png)

### Stack webhook

A stack webhook is a very interesting feature which creates a private URL that allows to automatically pull and redeploy the stack, for example when a new version of the images is available.

[![portainer-webhook.png](https://cylab.be/storage/blog/434/files/hdengPI2qf7uf0T2/portainer-webhook.png)](https://cylab.be/storage/blog/434/files/hdengPI2qf7uf0T2/portainer-webhook.png)

This feature **requires the business edition** of Portainer, which is free to manage up to 3 nodes…

### Stack environment variables

When deploying a new stack, Portainer allows to define environment variables.

[![portainer-envfile.png](https://cylab.be/storage/blog/434/files/Fs8PPM5MW0Ddys6q/portainer-envfile.png)](https://cylab.be/storage/blog/434/files/Fs8PPM5MW0Ddys6q/portainer-envfile.png)

These variables are actually written to a file with name **env.stack**. Hence you can use them in your containers by adding the `env_file` definition to your stack services:

```
services:
  web:
    image: cylab/hello-message
    ports:
      - 8081:80
    env_file:
      - stack.env
```

### Containers

For each individual container, Portainer allows to check the logs:

[![portainer-logs.png](https://cylab.be/storage/blog/434/files/u5bdIMZiOL2389Pb/portainer-logs.png)](https://cylab.be/storage/blog/434/files/u5bdIMZiOL2389Pb/portainer-logs.png)

You can also monitor resource usage statistics:

[![portainer-statistics.png](https://cylab.be/storage/blog/434/files/NVvxFTOm7ejtfbY5/portainer-statistics.png)](https://cylab.be/storage/blog/434/files/NVvxFTOm7ejtfbY5/portainer-statistics.png)

And even open a console directly from the web interface:

[![portainer-console.png](https://cylab.be/storage/blog/434/files/NdIoRy0iJP85ltr4/portainer-console.png)](https://cylab.be/storage/blog/434/files/NdIoRy0iJP85ltr4/portainer-console.png)

## Environments[](https://cylab.be/blog/434/simplify-your-docker-management-with-portainer#environments)

From the left menu, you can manage and add environments.

[![portainer-environments.png](https://cylab.be/storage/blog/434/files/baWVXowzhAfYmMpf/portainer-environments.png)](https://cylab.be/storage/blog/434/files/baWVXowzhAfYmMpf/portainer-environments.png)

A single Portainer instance can manage container managers of various types:

- Docker standalone nodes
- Docker Swarm
- Podman nodes
- Kubernetes cluster
- Azure Container Instances (ACI)

For most environments, you can even choose between different ways to connect. For example, Portainer can manage a Docker Standalone node using:

- a socket or a URL
- a light container Agent running on the node or
- a slightly heavier Edge Agent that also allows to manage the host itself

[![portainer-docker-environment.png](https://cylab.be/storage/blog/434/files/CxGf07epa0tVb08J/portainer-docker-environment.png)](https://cylab.be/storage/blog/434/files/CxGf07epa0tVb08J/portainer-docker-environment.png)

## Business edition[](https://cylab.be/blog/434/simplify-your-docker-management-with-portainer#business-edition)

You can request a Business Edition key at [https://www.portainer.io/take-3](https://www.portainer.io/take-3) which is free to manage up to 3 nodes…

[![portainer-3nodes.png](https://cylab.be/storage/blog/434/files/YGy40Ncun6mri1WN/portainer-3nodes.png)](https://cylab.be/storage/blog/434/files/YGy40Ncun6mri1WN/portainer-3nodes.png)

After a few seconds you’ll receive the licence key by email…

## Going further[](https://cylab.be/blog/434/simplify-your-docker-management-with-portainer#going-further)

Portainer does not manage **https certificates**, but it’s easy to [deploy Traefik reverse-proxy](https://cylab.be/blog/258/automatic-docker-reverse-proxy-with-traefik) alongside Portainer to provide automatic SSL termination.

Also, if you plan to deploy a lot of stacks on your Docker host, it’s a good idea to [**change Docker’s default subnet IP range**](https://cylab.be/blog/277/changing-dockers-default-subnet-ip-range)

This blog post is licensed under [CC BY-SA 4.0 ![creative commons](https://cylab.be/images/cc.svg) ![attribution](https://cylab.be/images/by.svg) ![share-alike](https://cylab.be/images/sa.svg)](http://creativecommons.org/licenses/by-sa/4.0/?ref=chooser-v1)