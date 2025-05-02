---
link: https://medium.com/@maheshwar.ramkrushna/chap-13-docker-networking-with-docker-compose-example-and-configuration-cc5a8b2bdb2b
byline: Ramkrushna Maheshwar
site: Medium
date: 2023-05-25T07:35
excerpt: Docker networking refers to the mechanisms and configurations used to enable communication between Docker containers and other network resources, both within and outside the Docker environment…
twitter: https://twitter.com/@Medium
slurped: 2025-03-08T07:32
title: "Chap-14: Docker Networking with Docker Compose: Example and Configuration"
tags:
  - docker-compose
  - network
---

Docker networking refers to the mechanisms and configurations used to enable communication between Docker containers and other network resources, both within and outside the Docker environment. Docker provides several networking options that allow containers to communicate with each other, with the host machine, and with external networks.

Types of networks in docker:

1. **Bridge Network:** The default network created by Docker is the bridge network. It allows containers on the same host to communicate with each other using IP addresses. Containers in a bridge network can be connected to the external network via NAT (Network Address Translation).
2. **Host Network:** When containers use the host network mode, they share the host’s network stack, bypassing Docker’s network isolation. This means containers have direct access to the host’s network interfaces, enabling maximum network performance but sacrificing container network isolation.
3. **Overlay Network:** Overlay networks are used in Docker Swarm mode for container communication across multiple Docker hosts. They provide a virtual network overlay on top of the physical network infrastructure, allowing containers on different hosts to communicate with each other as if they were on the same network.
4. **Custom Networks:** Docker allows you to create user-defined networks with specific configurations. Custom networks provide isolation and control over container communication. You can define the network type (bridge, overlay, etc.), subnet, gateway, and DNS settings according to your requirements.

Let’s consider we have two containers: a web server container and a database container. We want the web server container to be able to communicate with the database container.

First, you can create a user-defined network in Docker using the `docker network create` command. Let's call it "my-network":
```
  docker network create my-network
```

Next, you can start the database container and connect it to the “my-network” network. Let’s assume you are using a MySQL container:

```
  docker run -d --name database --network my-network -e 

```

MYSQL_ROOT_PASSWORD=password mysql:latest

The `--network my-network` option connects the container to the "my-network" network.

Now, you can start the web server container and also connect it to the “my-network” network. Let’s assume you are using an Nginx container:
```
  docker run -d --name webserver --network my-network -p 80:80 nginx:latest

```
The `-p 80:80` option maps port 80 of the host machine to port 80 of the web server container, allowing external access to the web server.

At this point, the web server container and the database container are connected to the same network, “my-network,” and they can communicate with each other using their container names.

Inside the web server container’s configuration, you can specify the database container’s hostname as “database.” The internal Docker DNS will resolve this hostname to the IP address of the database container.

For example, in the web server container’s Nginx configuration, you can have:

server {  
    ...  
    location / {  
        ...  
        proxy_pass http://database:3306;  
    }  
}

The `http://database:3306` address will be resolved to the IP address of the database container on the "my-network" network.

By using Docker networking, you have created a bridge network, “my-network,” and connected the web server and database containers to it. They can communicate with each other using their container names, and you have exposed the web server container’s port 80 to the host machine, allowing external access.

**WITH DOCKER COMPOSE:**

Here’s the same example using **Docker Compose**, which allows you to define and manage multi-container Docker applications:

Assuming you have the following directory structure:

my-app/  
├── docker-compose.yml  
├── webserver  
│   └── Dockerfile  
└── database  
    └── Dockerfile

First, create a `docker-compose.yml` file in the `my-app` directory with the following content:

```
  services:  
    webserver:  
      build: ./webserver  
      ports:  
        - 80:80  
      networks:  
        - my-network

    database:  
      build: ./database  
      networks:  
        - my-network

  networks:  
    my-network:

```

Next, create the `Dockerfile` for the web server in the `webserver` directory with the following content:

FROM nginx:latest  
COPY nginx.conf /etc/nginx/nginx.conf

Create the `nginx.conf` file in the same directory with the following content:

server {  
    ...  
    location / {  
        ...  
        proxy_pass http://database:3306;  
    }  
}

Similarly, create the `Dockerfile` for the database container in the `database` directory with the necessary configurations for your chosen database.

Now, navigate to the `my-app` directory in the terminal and run the following command:

docker-compose up -d

This command builds the Docker images and starts the containers defined in the `docker-compose.yml` file. The `-d` flag runs the containers in the background.

Docker Compose automatically creates a user-defined network named `my-app_default` (where `my-app` is the directory name) and connects the containers to this network.

The web server container can communicate with the database container using the hostname `database` since they are connected to the same network.

Additionally, the web server container’s port 80 is mapped to the host machine’s port 80 (`80:80` in the `docker-compose.yml` file), allowing external access to the web server.

> With Docker Compose, you can manage and orchestrate multiple containers with their networking configurations easily. It simplifies the process of defining and deploying multi-container applications while ensuring their proper network connectivity.