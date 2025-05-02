---
tags:
  - docker-compose
  - profile
---
## 1. Overview

[Docker Compose](https://www.baeldung.com/ops/docker-compose) is a powerful tool for defining and running multi-container [Docker](https://www.baeldung.com/ops/docker-guide) applications. It allows developers to define their application’s services, networks, and volumes in a single YAML file, making it easy to deploy and manage complex applications. **Container management is easier with [Docker’s service profiles](https://docs.docker.com/compose/profiles/), which allow users to define containers’ configurations specifically.**

Sometimes, we may need to run certain services based on some configurations from our Docker Compose file.

In this tutorial, we’ll learn about service profiles and their importance in Docker.

## 2. Understanding the Docker Compose Services

Before we use service profiles to run Docker services with specific configurations, let’s first understand the _docker-compose_ services.

A Docker service is a logical grouping of containers that work together to provide a specific function. In Docker Compose, a service refers to a containerized application element that can be horizontally scaled. It allows multiple instances of the same service to run simultaneously.
In a _docker-compose.yml_ file, we can define various properties of services, such as container [image](https://www.baeldung.com/category/docker/tag/docker-image), network configuration, and environment variables.

To demonstrate, let’s create a _docker-compose.yml_ file with _web_ and _db_ services:

```bash
version: "3.9"
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: secretpassword
```

Here, the web service uses the [_nginx_](https://www.baeldung.com/nginx-forward-proxy) image with port _8080_ exposed from the host machine inside the container. Database services use the [_postgres_](https://www.baeldung.com/ops/postgresql-docker-setup) image and set the _POSTGRES_PASSWORD_ environment variable to a _secretpassword_.

Furthermore, we can use the _docker-compose up_ command to run only certain services from our Docker Compose file. To demonstrate, let’s check out the command to run only the _web_ service from the previous example:

```bash
$ docker-compose up web
```

Using the command above, only the _web_ service and any services that it depends on will be started. The _web_ service doesn’t depend on the database service, so that it won’t start. By separating with a space, we can specify more than one service. To illustrate, let’s run both the _web_ and _db_ service:

```bash
$ docker-compose up web db
```

This command will start with the web service, the database service, and any dependencies.

## 3. Service Profiles

Docker Compose files define specific configurations for containers that run in a Docker service based on service profiles. The service profile is an experimental feature introduced in Docker version _20.10_. Service profiles allow us to define configurations for specific containers that run as part of a service. As a result, they simplify container management and also maintain the correct configurations.

**Docker service profiles permit us to define specific configurations for containers inside a Docker service. Thus, it is easy to manage individual containers and manage the configurations dynamically.** We can only use service profiles in Docker Compose files under the [_profile_](https://docs.docker.com/compose/profiles/) section. To demonstrate, let’s look at the _docker-compose.yml_ with profiling of a service:

```bash
version: "3.9"
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD: secretpassword  
    profiles:
      - dev   
  mysqldb:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: testrootpassword
      MYSQL_DATABASE: test
      MYSQL_USER: test
      MYSQL_PASSWORD: testpassword
    profiles:
      - prod
    volumes:
      - db_data:/var/lib/mysql
volumes:
  db_data:
```

In Docker Compose, we’ve created three different services _web_, _db_, and _mysqldb._ The _db_ service has the _dev_ service profile, whereas _mysqldb_ has the _prod_ service profile. In our Docker Compose file, we can specify the service profile to use by passing the option _–profile_ followed by the profile’s name. To illustrate, let’s check out the command to run the _mysqldb_ service with the _“prod”_ profile:

```bash
$ docker-compose --profile=prod up
```

In the above case, _mysqldb_ and _web_ services will start running. The _mysqldb_ service will start due to the prod profile in the configuration. In addition, the _web_ service will also run as services without profiles are always enabled.

## 4. Conclusion

In this article, we explored how to use service profiles in Docker. First, we learned the basic concept of using a service in a _docker-compose_ file. After that, we introduced the service profiles with the _docker-compose_ file.