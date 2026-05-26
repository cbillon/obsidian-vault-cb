---
link: https://cylab.be/blog/258/automatic-docker-reverse-proxy-with-traefik
excerpt: Traefik is an open-source reverse proxy and load balancer designed specifically for container platforms like Kubernetes and Docker. In these environment, Traefik can automatically discover services, perform load balancing and configure SSL termination. In this post, we’ll show how to deploy and use Traefik on Docker.
slurped: 2026-05-19T08:41
title: Automatic Docker reverse-proxy with Traefik
tags:
  - traefik
---

Traefik is an open-source reverse proxy and load balancer designed specifically for container platforms like Kubernetes and Docker. In these environment, Traefik can automatically discover services, perform load balancing and configure SSL termination. In this post, we’ll show how to deploy and use Traefik on Docker.

In this setup, I will deploy **a single instance of Traefik to serve multiple services** running on the same Docker host.

## Prerequisites[](https://cylab.be/blog/258/automatic-docker-reverse-proxy-with-traefik#prerequisites)

For this, you will need

- a VM with a public IP address,
- Docker installed on the VM and
- DNS entries (or a wildcard entry) that point to the IP of your VM.

For example, I’ll use the following wildcard entry:

```
*.traefik.cylab.be IN A 193.190.205.217
```

## Installation[](https://cylab.be/blog/258/automatic-docker-reverse-proxy-with-traefik#installation)

The easiest way to deploy Traefik is using Docker compose…

First create a **docker-compose.yaml** with the following content:

```
services:
  traefik:
    # the official Traefik docker image
    # 3.2 is currently the latest version
    image: traefik:v3.2

    # host network mode allows Traefik to reverse proxy for
    # containers on multiple Docker networks and multiple projects
    network_mode: "host"

    volumes:
      # so Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock

      # so we can tweak configuration as needed (see below)
      - ./traefik.yml:/etc/traefik/traefik.yml

      # keep certificates information outside of the container
      - ./acme.json:/acme.json
    restart: "unless-stopped"

```

Create the configuration file **traefik.yml** with the following content, and **don’t forget to update the email parameter at the end**:

```
################################################################
# Global configuration
################################################################

global:
  checkNewVersion: true
  sendAnonymousUsage: true

################################################################
# EntryPoints configuration
################################################################

entryPoints:
  web:
    address: :80

  websecure:
    address: :443

################################################################
# Traefik logs configuration
################################################################

log:
  # Log level
  # Default: "ERROR"
  #
  level: INFO

  # Sets the filepath for the traefik log. If not specified, stdout will be used.
  # Intermediate directories are created if necessary.
  # Default: os.Stdout
  #
#  filePath: log/traefik.log

  # Format is either "json" or "common".
  # Default: "common"
  #
#  format: json

################################################################
# Access logs configuration
################################################################

#accessLog:
  # Sets the file path for the access log. If not specified, stdout will be used.
  # Intermediate directories are created if necessary.
  # Default: os.Stdout
  #
#  filePath: /path/to/log/log.txt

  # Format is either "json" or "common".
  # Default: "common"
  #
#  format: json

################################################################
# API and dashboard configuration
################################################################

#api:
  # Enable the API in insecure mode
  # Default: false
  #
#  insecure: true

  # Enabled Dashboard
  # Default: true
  #
#  dashboard: false

################################################################
# Docker configuration backend
################################################################

providers:
  docker:
    endpoint: unix:///var/run/docker.sock

    # Default host rule.
    # Default: "Host(`{{ normalize .Name }}`)"
    #
#    defaultRule: Host(`{{ normalize .Name }}.docker.localhost`)

    # Expose containers by default in traefik
    # Default: true
    #
    exposedByDefault: false

################################################################
# TLS configuration
################################################################

certificatesResolvers:
  # Use Let's Encrypt to automatically get certificates
  #
  letsencrypt:
    acme:
      # Email address used for registration.
      # Required
      #
      email: your-email@example.com

      # File used for certificates storage.
      # Required
      #
      storage: acme.json
      
      # Use HTTP-01 challenge
      #
      httpChallenge:
        # Entrypoint used during the challenge
        #
        entryPoint: web

```

Finally, create an empty file **acme.json** :

```
touch acme.json
chmod 600 acme.json
sudo chown root:root acme.json
```

You can now start Traefik with:

```
docker compose up
```

[![traefik.png](https://cylab.be/storage/blog/258/files/GKPDyl5yoxFghnwv/traefik.png)](https://cylab.be/storage/blog/258/files/GKPDyl5yoxFghnwv/traefik.png)

## Services[](https://cylab.be/blog/258/automatic-docker-reverse-proxy-with-traefik#services)

You can now deploy your services. Traefik will automatically discover services using attached labels.

But there are 2 caveats:

1. each service must use a unique **service name, which is part of the label name** (not value)
2. the domain name must use backticks (**`**)

Here is a complete template:

```
services:
  web:
    image: your-image
    ports:
      # port must be exposed for Traefik to forward correctly
      - 80
    labels:
      - traefik.enable=true
      # specify hostname
      - traefik.http.routers.<service-name>.rule=Host(`your.domain.name`)
      # enable https
      - traefik.http.routers.<service-name>.tls=true
      - traefik.http.routers.<service-name>.tls.certresolver=letsencrypt
```

Here is a simple example, for a service and router that I will call **hello1**.

```
mkdir hello1
cd hello1
```

Then create **docker-compose.yml** :

```
services:
  apache:
    image: cylab/hello-message
    environment:
      - MESSAGE=<h1>Hello 1!</h1>
    ports:
      # port must be exposed for Traefik to forward correctly
      - 80
    labels:
      - traefik.enable=true
      # specify hostname
      - traefik.http.routers.hello1.rule=Host(`hello1.traefik.cylab.be`)
      # enable https
      - traefik.http.routers.hello1.tls=true
      - traefik.http.routers.hello1.tls.certresolver=letsencrypt
```

Once this service is started with `docker compose up`, it will be available at [https://hello1.traefik.cylab.be](https://hello1.traefik.cylab.be/)

[![traefik-hello.png](https://cylab.be/storage/blog/258/files/TwSEPYu0vtrHEQoc/traefik-hello.png)](https://cylab.be/storage/blog/258/files/TwSEPYu0vtrHEQoc/traefik-hello.png)

⚠ **Warning:** once https is enabled, the service will NOT be reachable using unsafe http

## Tips & tricks[](https://cylab.be/blog/258/automatic-docker-reverse-proxy-with-traefik#tips-tricks)

### Disable traefik for a single service

If traefik is enabled by default (see configuration above), but you need to disable traefik for a single service, you can simply add the `traefik.enable=false` label on the service definition:

```
services:
  apache:
    image: cylab/hello-message
    labels:
      - traefik.enable=false
```
