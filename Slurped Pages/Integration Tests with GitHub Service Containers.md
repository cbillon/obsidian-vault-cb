---
link: https://packagemain.tech/p/integration-tests-with-github-service
byline: Alex Pliutau
site: packagemain.tech
date: 2024-12-27T14:16
excerpt: A tutorial on running integrations tests with dependencies in Github Actions workflows.
slurped: 2025-11-02T18:08
title: Integration Tests with GitHub Service Containers
tags:
  - go
  - test
---

Not so long ago we published an **[article](https://packagemain.tech/p/integration-tests-using-testcontainers)** about using **[Testcontainers](https://testcontainers.com/)** for emulating external dependencies such as a database and cache for the purpose of backend integration tests. The article also explains the different ways of running the integration tests, environment scaffolding and their pros and cons.

In this post we want to show another alternative in case you use GitHub Actions as your CI platform, which became the most popular CI/CD solution at the moment.

It’s called **[Service Containers](https://docs.github.com/en/actions/use-cases-and-examples/using-containerized-services/about-service-containers)**, and we realised that, unfortunately, not so many developers are aware of it.

In this hands-on tutorial we want to demonstrate how to create a GitHub Actions workflow for integration tests with external dependencies (MongoDB and Redis), using the [demo Go application](https://github.com/plutov/packagemain/tree/master/testcontainers-demo) we wrote previously, as well as review the pros and cons of GitHub Service Containers.

Service Containers are Docker containers that offer a simple and portable way to host dependencies like databases (MongoDB in our example), web services, or caching systems (Redis in our example) that your application needs within a workflow. This article focuses on integration tests, however there are many other possible applications. Service containers can also be used to run supporting tools required by your workflow, such as code analysis tools, linters, or security scanners.

Sounds similar to **services** in Docker Compose, right? Because it is.

However, while you could technically [use Docker Compose](https://github.com/marketplace/actions/docker-compose-action) within a GitHub Actions workflow by installing Docker Compose and running the **docker-compose up**, service containers provide a more integrated and streamlined approach specifically designed for the GitHub Actions environment.

Also, while they are similar, they solve different purpose.

- Docker Compose is good when you need to manage a multi-container application on your local machine or a single server. best suited for long-living environments.
    
- Service Containers are ephemeral and exist only for the duration of a workflow run, and defined directly within your GitHub Actions workflow file.
    

Also, the feature set of service containers (at least as of now) is more limited compared to Docker Compose, so be ready to discover some potential bottlenecks, we will cover some of them at the end of this article.

You can run GitHub jobs directly on a runner machine or in a Docker container (by specifying **container** property). The second option simplifies the access to your services by using labels you defined in the **services** section.

Run directly on a runner machine.

> _**.github/workflows/test.yaml**_

```
jobs:
  integration-tests:
    runs-on: ubuntu-24.04

    services:
      mongo:
        image: mongodb/mongodb-community-server:7.0-ubi8
        ports:
          - 27017:27017

    steps:
      - run: |
          echo "addr 127.0.0.1:27017"
```

Or run in a container ([Chainguard Go Image](https://images.chainguard.dev/directory/image/go/overview) in our case):

```
jobs:
  integration-tests:
    runs-on: ubuntu-24.04
    container: cgr.dev/chainguard/go:latest

    services:
      mongo:
        image: mongodb/mongodb-community-server:7.0-ubi8
        ports:
          - 27017:27017

    steps:
      - run: |
          echo "addr mongo:27017"
```

You can also omit the host port, so the container port will be randomly assigned to a free port on the host. You can then access the port using the variable.

```
jobs:
  integration-tests:
    runs-on: ubuntu-24.04
    container: cgr.dev/chainguard/go:1.23

    services:
      mongo:
        image: mongodb/mongodb-community-server:7.0-ubi8
        ports:
          - 27017/tcp

    steps:
      - run: |
          echo "addr mongo:${{ job.services.mongo.ports['27017'] }}"
```

Prior to running the job steps that connect to our provisioned containers we sometimes need to make sure that the services are ready. It’s possible to do so by specifying [docker create options](https://docs.docker.com/reference/cli/docker/container/create/#options) such as **health-cmd**.

This is very important, otherwise the services may not be ready when we start accessing them in the **steps** section.

In a case of MongoDB and Redis these will be:

```
    services:
      mongo:
        image: mongodb/mongodb-community-server:7.0-ubi8
        ports:
          - 27017/27017
        options: >-
          --health-cmd "echo 'db.runCommand("ping").ok' | mongosh mongodb://localhost:27017/test --quiet"
          --health-interval 5s
          --health-timeout 10s
          --health-retries 10

      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 5s
          --health-timeout 10s
          --health-retries 10
```

In our example we use public images from Dockerhub, however it’s possible to use private images as well and pass the credentials:

```
services:
  private_service:
    image: ghcr.io/org/service_repo
    credentials:
      username: ${{ secrets.registry_username }}
      password: ${{ secrets.registry_token }}
```

You can use volumes to share data between services or other steps in a job. You can specify named Docker volumes, anonymous Docker volumes, or bind mounts on the host. However, it’s not directly possible to mount the source code as a container volume. [Open discussion](https://github.com/orgs/community/discussions/42127)

```
volumes:
  - /src/dir:/dst/dir
```

Now as we can provision our external dependencies, let’s have a look at how to run our integration tests in Go. We will do it in the **steps** section of our workflow file.

We will run our tests in a container which uses [Chainguard Go image](https://images.chainguard.dev/directory/image/go/overview), which means we don’t have to install/setup Go. In case you want to run your tests directly on a runner machine, you need to use [setup-go](https://github.com/actions/setup-go) Action.

You can find the full source code with tests and this workflow [here](https://github.com/plutov/service-containers).

> _**.github/workflows/integration-tests.yaml**_

```
name: "integration-tests"

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  integration-tests:
    runs-on: ubuntu-24.04
    container: cgr.dev/chainguard/go:latest

    env:
      MONGO_URI: mongodb://mongo:27017
      REDIS_URI: redis://redis:6379

    services:
      mongo:
        image: mongodb/mongodb-community-server:7.0-ubi8
        ports:
          - 27017:27017
        options: >-
          --health-cmd "echo 'db.runCommand("ping").ok' | mongosh mongodb://localhost:27017/test --quiet"
          --health-interval 5s
          --health-timeout 10s
          --health-retries 10

      redis:
        image: redis:7
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 5s
          --health-timeout 10s
          --health-retries 10

    steps:
      - name: Check out repository code
        uses: actions/checkout@v4

      - name: Download dependencies
        run: go mod download

      - name: Run Integration Tests
        run: go test -tags=integration -timeout=120s -v ./...
```

To summarize:

1. We run our job in a container with Go (**container**)
    
2. We spin up 2 services: MongoDB and Redis (**services**)
    
3. We configure healthchecks to make sure our services are “Healthy” when we run the tests (**options**)
    
4. Standard code checkout
    
5. Run the Go tests
    

Once the Action is completed (it took **~1 min** for this example), all the services will be stopped and orphaned so we don’t need to worry about that.

[

![](https://substackcdn.com/image/fetch/$s_!_Y9r!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9af2df88-ec95-41e9-9bab-bb22ea2deb10_480x409.png)

](https://substackcdn.com/image/fetch/$s_!_Y9r!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F9af2df88-ec95-41e9-9bab-bb22ea2deb10_480x409.png)

We’ve been using service containers for running backend integration tests at [BINARLY](https://www.binarly.io/) for some time, and they work great. However, the initial workflow creation took some time and we encountered the following bottlenecks:

- It’s not possible to override or run custom command in an action service container (as you would do in Docker Compose using **command** property). [Open pull request](https://github.com/actions/runner/pull/1152)
    
- It’s not directly possible to mount the source code as a container volume. [Open discussion](https://github.com/orgs/community/discussions/42127)
    

GitHub service containers is a great option to scaffold the ephemeral testing environment by configuring it directly in your GitHub workflow. With configuration somehow similar to Docker Compose it’s easy to run any containerised application and to communication with it in your pipeline, making sure that GitHub runners take care of shutting everything down upon completion.

If you use Github Actions, this approach works extremely well as it is specifically designed for the GitHub Actions environment.

- [Source Code](https://github.com/plutov/service-containers)
    
- [GitHub Documentation](https://docs.github.com/en/actions/writing-workflows/workflow-syntax-for-github-actions#jobsjob_idservices)