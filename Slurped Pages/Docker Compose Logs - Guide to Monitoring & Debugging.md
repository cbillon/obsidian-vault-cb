---
link: https://spacelift.io/blog/docker-compose-logs
byline: Written by
site: Spacelift
excerpt: "Master Docker Compose logs for multi-container applications: how to
  get, view, and troubleshoot using the docker-compose logs command."
twitter: https://twitter.com/@handle
slurped: 2025-02-08T17:02
title: Docker Compose Logs - Guide to Monitoring & Debugging
---

### Table of contents

Docker Compose simplifies the process of building and running multi-container applications. Compose lets you use a single command to start a stack of containers that are defined in a declarative config file. This is quicker and more repeatable than manually bringing each container up using standalone Docker commands.

Compose also improves the Docker container logs experience. A dedicated command (`docker compose logs`) displays the unified log stream for all the containers in your stack, within a single terminal window. Regular log monitoring during development allows you to detect errors, test fixes, and learn about activity in your containers, so learning Docker Compose’s logging options is an easy way to improve your workflow.

In this article, we’ll walk through the `docker compose logs` command and explain how to use its main features. We’ll then wrap up with some best practices that’ll help you get the most out of your log monitoring sessions.

What we will cover:

1. [What is docker compose logs?](https://spacelift.io/blog/docker-compose-logs#what-is-the-docker-compose-logs-command)
2. [How to use Docker Compose logs?](https://spacelift.io/blog/docker-compose-logs#how-to-use-docker-compose-logs)
3. [Docker compose logs and Docker Swarm](https://spacelift.io/blog/docker-compose-logs#docker-compose-logs-and-docker-swarm)
4. [Docker compose logs: best practices](https://spacelift.io/blog/docker-compose-logs#docker-compose-logs-best-practices)

## What is the docker compose logs command?[](https://spacelift.io/blog/docker-compose-logs#what-is-the-docker-compose-logs-command)

The `docker compose logs` command displays the log output from the containers in your Docker Compose stacks. It works similarly to the Docker CLI’s [`docker logs`](https://docs.docker.com/reference/cli/docker/container/logs) command but allows you to view a stack’s combined output in a single terminal window.

`docker compose logs` offers several benefits for container developers and operators:

- Easily correlate log output from multiple running containers, facilitating detailed investigation of issues that cause knock-on effects in several services.
- When troubleshooting Docker Compose container issues, view the logs and identify any error messages that might provide insights into misconfigurations occurring on both the containers and the host machine.
- Support compliance audits and other container governance initiatives by providing evidence of correct operations.
- Simplify your development experience by live streaming logs straight to your terminal.
- No need to install extra tools to get started—logs are automatically collected and stored by Docker, so you can use a single log management workflow across all your hosts and apps.

Running `docker compose logs` only surfaces the logs emitted by the apps within your containers — specifically, the output from each container’s [foreground process](https://spacelift.io/blog/docker-entrypoint-vs-cmd). It _doesn’t_ surface internal Docker events, such as those recorded for container creations, restarts, and health checks. To retrieve those logs, you must use the separate [`docker compose events` command](https://docs.docker.com/reference/cli/docker/compose/events).

### Where are the Docker Compose log files stored?

Docker defaults to storing container log files within the `/var/lib/docker/containers` system directory. Each container has its own subdirectory, named with the container’s full ID. 

You can use the [`docker ps` command](https://spacelift.io/blog/docker-ps) with the `--no-trunc` to retrieve the correct ID for a container:

```
$ docker ps -a --no-trunc
CONTAINER ID                                                       IMAGE          COMMAND              CREATED         STATUS                     PORTS     NAMES
0047e0d8e4500efe8394f03981ee0e8338149fe86f2ae8a66a6ffc828c732528   httpd:alpine   "httpd-foreground"   9 minutes ago   Exited (0) 9 minutes ago             agitated_boyd
```

For the example above, the container’s logs would be located in the directory `/var/lib/docker/containers/0047e0d...`. Within this directory, you’ll find a JSON file named as the container’s ID appended with `-json.log`. This file will contain all the log entries collected from the container as JSON objects, each on an individual line. 

You can either write your own tooling to consume this file’s contents or use `docker compose logs` to conveniently inspect it in your terminal.

Like other `docker compose` commands, it’s best to run `logs` within the directory containing the docker-compose.yml file for your project. Alternatively, you can use the CLI’s [`--project-directory` flag](https://docs.docker.com/compose/reference) to reference a file stored within a specific working directory.

### 1. Start a demo project

To follow along with this tutorial, clone Docker’s [`awesome-compose` repository](https://github.com/docker/awesome-compose) of example Compose stacks:

```
$ git clone https://github.com/docker/awesome-compose.git
```

We’ll use the [nginx-nodejs-redis stack](https://github.com/docker/awesome-compose/tree/master/nginx-nodejs-redis) to bring up a Node.js app that uses an NGINX proxy service and a Redis database:

```
$ cd awesome-compose/nginx-nodejs-redis

$ docker compose up -d
[+] Running 5/5
 ✔ Network nginx-nodejs-redis_default    Created                                                                                                                                                                                                                                     0.1s 
 ✔ Container nginx-nodejs-redis-web1-1   Started                                                                                                                                                                                                                                     1.1s 
 ✔ Container nginx-nodejs-redis-web2-1   Started                                                                                                                                                                                                                                     1.1s 
 ✔ Container nginx-nodejs-redis-redis-1  Started                                                                                                                                                                                                                                     1.1s 
 ✔ Container nginx-nodejs-redis-nginx-1  Started   
```

### 2. View the demo stack’s logs

Now you can run `docker compose logs` to view the logs that have been emitted by the containers in the stack:

![docker compose view logs](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2024%2F10%2Fdocker-compose-view-logs.png&w=3840&q=75)

Logs from all the stack’s containers are intermingled based on the chronological order in which each line was recorded. The identity of the container to which each line belongs is displayed to the left of the screen, with different containers also visually distinguished by color.

### 3. Get a specific service’s logs

To get the logs from a particular service in the stack, specify the service’s name as the first argument to the `docker compose logs` command:

![docker compose logs tail](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2024%2F10%2Fdocker-compose-logs-tail.png&w=3840&q=75)

In this example, only log lines originating from the `redis` service are displayed. This technique can help you debug problems noted in specific services more efficiently.

### 4. View Docker Compose logs in real-time (–follow)

Compose usually displays the entire contents of the collected logs before exiting and returning you to your terminal. But during long development and debugging sessions, it’s often preferable to continually stream and view logs live in your terminal as new lines are recorded.

The `-f` or `--follow` flag activates this behavior. Compose will output the existing logs as usual and then continue to stream new logs in real-time as they’re emitted by your stack’s containers.

```
$ docker compose logs --follow
```

Press Ctrl+C to stop following the logs and return to your terminal.

### 5. Filter Docker Compose logs by timestamp (–since and –until)

If you know you’re looking for events that happened within a particular timeframe, you can use the `--since` and `--until` flags to filter the docker compose logs output that’s displayed. Each flag accepts either a precise timestamp (such as `2024-08-01T12:00:00Z`) or a convenient relative reference, such as `1h`, to mean one hour.

Here are some useful examples:

- Get all logs emitted in the past hour

```
$ docker compose logs --since 1h
```

- Get logs that were emitted longer than 5 minutes ago

```
$ docker compose logs --until 5m
```

- Get logs that were emitted 1-2 hours ago

```
$ docker compose logs --since 2h --until 1h
```

- Get logs emitted since a specific date

```
$ docker compose logs --since 2024-08-01
```

- Get logs emitted before a specific timestamp

```
$ docker compose logs --until 2024-08-01T12:00:00Z
```

### 6. Get a specific number of lines from Docker Compose logs (–tail)

Compose provides an even simpler alternative to `--since` for occasions when you just want to see a snapshot of the most recent logs emitted by your stack. The `docker compose logs` command with the `--tail` flag lets you control the number of lines displayed, starting from the end of the log.

```
# Get the 10 most recent logs
$ docker compose logs --tail 10
```

The flag accepts 0 as a value, meaning no existing logs will be emitted. This is ideal when used in combination with `--follow`. Compose will start streaming new logs to your terminal without displaying any undesirable older output first.

### 7. Include timestamps with Docker Compose logs (–timestamps)

By default, Compose does not show timestamps next to log messages — it’s assumed your containerized apps will write timestamps next to log entries that require them. However, not all apps do this, so the `--timestamps` flag lets you opt into Compose prepending the log storage time to the start of each line.

![docker compose logs timestamp](https://spacelift.io/_next/image?url=https%3A%2F%2Fspaceliftio.wpcomstaging.com%2Fwp-content%2Fuploads%2F2024%2F10%2Fdocker-compose-logs-timestamp.png&w=3840&q=75)

The screenshot above shows how Compose adds the precise timestamp to each displayed log entry. This is helpful in the case of the web services — which aren’t writing their own timestamps — but it causes duplication in the logs for redis and nginx, where timestamps provided by the app can be observed immediately following Compose’s default insertion.

## Docker Compose logs and Docker Swarm[](https://spacelift.io/blog/docker-compose-logs#docker-compose-logs-and-docker-swarm)

Docker Compose allows you to run multiple replicas of your stack’s containers to improve redundancy. This is normally only used when [Docker’s distributed Swarm mode](https://spacelift.io/blog/docker-swarm-vs-kubernetes) is enabled. 

Using `docker compose logs` with a replicated service will normally display the combined log output from all of the replicas. However, you can instruct Compose to show only the logs from a specific replica by setting the `--index` flag. The value given to the flag should be the numerical index of the container replica to display the logs from, where 1 is the first replica.

```
# Get the logs from the third replica of the "web" service
$ docker compose logs web --index 3
```

## Docker Compose logs: Best practices[](https://spacelift.io/blog/docker-compose-logs#docker-compose-logs-best-practices)

The following best practices will help you effectively utilize Docker Compose logs to monitor your containerized apps:

- **Emit your app’s logs to** **`stdout`** **and** **`stderr`****:** Docker can only collect logs that containers emit to their standard output (`stdout`) and error (`stderr`) streams. Make sure your app writes its logs to these streams so they show up in the `docker compose logs` command’s output.
- **Use** **`--since`****,** **`--until`****, and** **`--tail`** **to exclude irrelevant logs:** Running `docker compose logs` on a large busy stack can produce very noisy output. Applying the `--since`, `--until`, or `--tail` flags helps to filter out irrelevant messages, making it easier to focus on the event timeframe you’re interested in.
- **Pipe logs to other commands to apply more complex filtering:** Docker Compose doesn’t provide any options to filter or sort the _content_ of collected logs, but you can pipe log output to standard Unix commands like [`grep`](https://man7.org/linux/man-pages/man1/grep.1.html) and [`sort`](https://man7.org/linux/man-pages/man1/sort.1.html) to achieve these use cases.
- **Set a specific log driver for long-term centralized log retention:** Docker Compose allows you to set the [Docker log driver](https://docs.docker.com/config/containers/logging/configure) to use on a per-service basis. The [`logging` key](https://docs.docker.com/compose/compose-file/05-services/#logging) in your `docker-compose.yaml` file can be used to specify a particular driver, including external storage options. Docker ships with [drivers](https://docs.docker.com/config/containers/logging/configure/#supported-logging-drivers) for Logstash, Fluentd, Splunk, AWS CloudWatch, and GCP Logging, enabling you to centrally store logs in your chosen cloud platform for long-term retention.

These tips will ensure you can effectively consume your Docker logs within your DevOps process. Nonetheless, the most crucial point is to ensure that logs are monitored regularly — otherwise, bugs, errors, and warnings that occur in your environments may persist unnoticed.

## Key points[](https://spacelift.io/blog/docker-compose-logs#key-points)

We’ve taken a tour through the `docker compose logs` command, a mechanism for browsing the log output from your Docker Compose stacks. The command displays the logs that containers have written to their `stdout` and `stderr` streams, enabling you to investigate the causes of errors and explore container activity efficiently.

Because Docker Compose simply builds upon Docker’s logging system, you can use Docker’s [driver settings](https://docs.docker.com/config/containers/logging/configure) to configure how logs are stored and delivered. This is an important factor when using Docker Compose in production because directing logs to an external service makes them easier to search, back up, and share with other team members.



