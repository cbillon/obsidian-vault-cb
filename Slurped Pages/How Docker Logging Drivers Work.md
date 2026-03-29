---
link: https://last9.io/blog/docker-logging-drivers/
byline: Anjali Udasi
site: Last9
date: 2025-05-05T11:09
excerpt: Learn how Docker logging drivers collect, route, and store container
  logs—and which one makes sense for your monitoring setup.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T15:21
title: How Docker Logging Drivers Work
---

Troubleshooting containerized applications can quickly become complex when logs are scattered across multiple systems. Most DevOps teams face this challenge daily—what starts as a simple container deployment often evolves into a complex logging puzzle.

This guide explores Docker logging drivers in depth, covering configuration options, best practices, and practical solutions.

## What Are Docker Logging Drivers?

Docker logging drivers are built-in mechanisms that determine how container logs are collected, stored, and accessed. They act as the middlemen between your containers and whatever system you're using to store or analyze logs.

By default, Docker uses the `json-file` driver, which saves logs as JSON files on the host machine. But that's just the beginning—Docker supports multiple logging drivers that can send your logs to various destinations.

💡

For a closer look at how to capture and manage logs during image builds, check out this guide on [Docker build logs](https://last9.io/blog/docker-build-logs/).

## Why Docker Logging Drivers Matter

Proper logging is your lifeline when things go wrong. Here's why you should care about logging drivers:

- **Troubleshooting**: Find and fix issues faster by having logs in the right place
- **Monitoring**: Keep an eye on container health and performance
- **Compliance**: Meet regulatory requirements with proper log retention
- **Resource Management**: Prevent logs from eating up your disk space

## Available Docker Logging Drivers

Docker comes with several logging drivers out of the box. Let's break down the most useful ones:

|Driver|Best For|Key Features|
|---|---|---|
|json-file|Local development|Simple setup, easy access with `docker logs`|
|local|Production environments|Block I/O for better performance|
|syslog|Unix/Linux environments|Integration with system logging|
|journald|SystemD-based distros|Structured logging with metadata|
|fluentd|Distributed logging|Unified logging layer|
|awslogs|AWS environments|Direct integration with CloudWatch|
|splunk|Enterprise monitoring|Advanced search capabilities|
|gelf|Graylog integration|Compressed log messages|

## How to Set Up the Default Logging Driver

You can configure the default logging driver for all containers on your Docker daemon. Here's how to do it:

1. Edit the Docker daemon configuration file:

```
{
  "log-driver": "local",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

2. Restart the Docker daemon:

```
sudo systemctl restart docker
```

This configuration sets the `local` driver as default and limits log files to 10MB with a maximum of 3 files per container.

## Choose a Logging Driver for Individual Containers

Need different logging settings for specific containers? No problem. You can override the default driver when running a container:

```
docker run --log-driver=fluentd --log-opt fluentd-address=localhost:24224 nginx
```

For Docker Compose, add the logging configuration to your `docker-compose.yml` file:

```
services:
  web:
    image: nginx
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: web.{{.Name}}
```

## Step-by-Step Guide to JSON-File Logging Driver

The `json-file` driver is Docker's default choice—it's simple and works right out of the box. Let's see how to make the most of it:

### Configure Log Rotation

Without log rotation, your container logs can grow indefinitely and fill up your disk. Add these options to manage log size:

```
docker run --log-opt max-size=10m --log-opt max-file=3 nginx
```

### Access JSON-File Logs

The beauty of the `json-file` driver is how easy it is to access logs:

```
# View all logs
docker logs container_name

# Follow log output
docker logs -f container_name

# Show only the last 100 lines
docker logs --tail 100 container_name

# Include timestamps
docker logs -t container_name
```

## Use the Local Logging Driver for Better Performance

The `local` driver is similar to `json-file` but uses a more efficient storage format. It's perfect for production environments:

```
docker run --log-driver=local --log-opt max-size=10m nginx
```

Key benefits of the `local` driver:

- Better performance
- Lower disk usage
- Still accessible through the `docker logs` command

## How to Ship Logs to Remote Systems

For centralized logging, you'll want to send logs to a remote system. Here are a few options:

### Set Up Fluentd Logging

Fluentd is a popular open-source data collector that works great with Docker:

1. Start a Fluentd container:

```
docker run -d -p 24224:24224 fluent/fluentd
```

2. Configure your container to use the Fluentd driver:

```
docker run --log-driver=fluentd --log-opt fluentd-address=localhost:24224 nginx
```

### Use Syslog for Legacy Systems

Many organizations already have syslog infrastructure in place:

```
docker run --log-driver=syslog --log-opt syslog-address=udp://192.168.0.42:514 nginx
```

### Integrate with Last9 for Advanced Observability

If you're looking for a managed observability solution that's budget-friendly yet powerful, [Last9](https://last9.io/) is worth checking out.

Last9 integrates seamlessly with Docker logging drivers, bringing together metrics, logs, and traces in one platform. We've handled logging for some of the biggest live-streaming events around, so they know a thing or two about scale.

The platform plays nicely with OpenTelemetry and Prometheus, which means you can keep your existing logging setup while gaining better insights.

To get started, do checkout [our docs](https://last9.io/docs/)!

## Common Docker Logging Driver Issues and Fixes

### Logs Taking Up Too Much Space

**Problem**: Your disk is filling up with container logs.

**Solution**: Implement log rotation with the `max-size` and `max-file` options:

```
docker run --log-opt max-size=10m --log-opt max-file=3 nginx
```

### Can't Access Logs with Docker Logs Command

**Problem**: The `docker logs` the command fails with certain logging drivers.

**Solution**: Only the `json-file` and `local` drivers support the `docker logs` command. If you're using another driver but still want to use this command, you can enable dual logging:

```
docker run --log-driver=fluentd --log-opt fluentd-address=localhost:24224 --log-opt tag=nginx --log-opt env=prod --log-opt env-regex=^(NGINX|DOCKER)_ nginx
```

### Missing Log Messages

**Problem**: Some log messages aren't showing up where expected.

**Solution**: This often happens due to buffering. Configure your logging driver to flush more frequently:

```
docker run --log-driver=fluentd --log-opt fluentd-address=localhost:24224 --log-opt fluentd-async-connect=false nginx
```

### High CPU Usage During Logging

**Problem**: Logging is causing high CPU usage.

**Solution**: The `blocking` mode can reduce CPU usage at the cost of potentially slowing down applications:

```
docker run --log-driver=json-file --log-opt mode=blocking nginx
```

💡

Now, fix production Docker log issues instantly—right from your IDE, with AI and [Last9 MCP](https://last9.io/mcp/). Bring real-time context—logs, metrics, and traces—into your local setup to debug faster and ship with confidence.

## Advanced Logging Techniques You Need to Know

### Use Labels for Better Log Organization

Add metadata to your logs with container labels:

```
docker run --label environment=production --label service=api nginx
```

These labels will be included in the log metadata, making it easier to filter and search logs later.

### Parse Multiline Logs

Many applications produce multiline logs (like stack traces). Configure your logging driver to handle them properly:

```
docker run --log-driver=fluentd --log-opt fluentd-address=localhost:24224 --log-opt tag=app --log-opt fluentd-sub-second-precision=true app-image
```

Then configure Fluentd with a multiline parser plugin.

### Implement Log Sampling

For high-volume logs, you might want to sample rather than collect everything:

```
docker run --log-driver=syslog --log-opt syslog-address=udp://192.168.0.42:514 --log-opt syslog-format=rfc5424 --log-opt tag=production nginx
```

Then configure your syslog server to sample logs based on the tag.

## Logging Best Practices for Docker Environments

1. **Be Intentional About What You Log**: Not everything needs to be logged. Focus on actionable information.
2. **Standardize Log Formats**: Use consistent formats across all containers to make parsing and analysis easier.
3. **Use Tags and Labels**: Add metadata to help with filtering and categorization.
4. **Monitor Your Logging Infrastructure**: Your logging system needs monitoring too.
5. **Implement Log Rotation Everywhere**: Never let logs grow unbounded.
6. **Set Up Alerting Based on Logs**: Use logs to trigger alerts for critical issues.
7. **Secure Your Logs**: Logs often contain sensitive data—make sure they're protected.

💡

When working with Docker logs, filtering them using `grep` can significantly streamline the debugging process. For a practical guide on using `grep` with Docker logs, including examples and tips, check out this blog: [How to Filter Docker Logs with Grep](https://last9.io/blog/how-to-filter-docker-logs-with-grep/).

## Conclusion

Docker logging drivers give you the flexibility to handle container logs in a way that fits your specific needs. Whether you're just starting with containers or running a complex microservices architecture, proper log management is crucial.

What logging challenges are you facing with your Docker containers? Join our [Discord Community](https://discord.com/invite/W8gMppQC4b) to continue the conversation!

## FAQs

### What's the default Docker logging driver?

The default logging driver is `json-file`, which saves container logs as JSON files on the host machine.

### Can I use multiple logging drivers simultaneously?

Docker doesn't support multiple logging drivers for a single container natively. However, you can use tools like Fluentd to forward logs to multiple destinations.

### How do I check which logging driver a container is using?

```
docker inspect --format '{{.HostConfig.LogConfig.Type}}' container_id
```

### Will changing the logging driver affect running containers?

No, changing the default logging driver only affects containers created after the change. Existing containers will continue using their original driver.

### Can I access container logs if I'm not using the json-file driver?

The `docker logs` command only works with the `json-file` and `local` drivers. For other drivers, you'll need to access logs through the destination system.

### How do I handle log rotation in production environments?

For production, it's best to use the `local` driver with `max-size` and `max-file` options, or send logs to a centralized logging system that handles rotation for you.

### Are Docker logging drivers compatible with Kubernetes?

Kubernetes has its own logging architecture, but Docker logging drivers still work for containers running in Kubernetes pods. However, Kubernetes-specific approaches like the sidecar pattern are often preferred.