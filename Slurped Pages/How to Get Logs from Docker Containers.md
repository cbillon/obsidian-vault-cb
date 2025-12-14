---
link: https://last9.io/blog/how-to-get-logs-from-docker-containers/
byline: Preeti Dewani
site: Last9
date: 2025-07-09T13:38
excerpt: Learn how to access, filter, and monitor Docker container logs, plus tips for structured logging, rotation, and production-ready setups.
twitter: https://twitter.com/@last9io
slurped: 2025-12-07T10:46
title: How to Get Logs from Docker Containers
tags:
  - docker
  - log
---

When a container misbehaves, logs are the first place to look. Whether you're debugging a crash, tracking API errors, or verifying app behavior—`docker logs` gives you direct access to what's happening inside.

This blog covers the full workflow: how to retrieve logs, filter them by time or service, and set up logging for production environments.

## Quick Commands Reference

These essential commands handle most Docker logging scenarios you'll encounter:

```
# Basic log viewing
docker logs nginx-proxy
```

Displays all logs from the nginx-proxy container since it started. This shows everything the container has written to stdout and stderr.

```
# Follow logs in real-time
docker logs -f api-server
```

Continuously streams new log entries as they appear, similar to `tail -f`. Use this to monitor live application behavior. Press Ctrl+C to stop following.

```
# Show last 50 lines
docker logs --tail 50 postgres-db
```

Displays only the most recent 50 log entries, perfect for containers with extensive log history where you only care about recent activity.

```
# Show logs from last hour
docker logs --since 1h redis-cache
```

Filters logs to show only entries from the past hour. You can use `2h`, `30m`, `1d`, or specific timestamps like `2024-01-15T10:00:00`.

```
# Show logs with timestamps
docker logs -t web-app
```

Adds RFC3339 timestamps to each log entry, helping you correlate log events with specific times when debugging issues.

```
# Combine options for debugging
docker logs --since 1h --tail 100 -f api-server
```

Shows the last 100 log entries from the past hour, then continues following new logs. This combination is particularly useful when you want recent context before monitoring ongoing activity.

Now, let's understand some common scenarios you'll face and how to tackle them.

## Common Docker Logging Issues and How to Fix Them

### The container is not showing the expected logs

When `docker logs` returns empty output, or you don't see expected log entries, start with these diagnostic steps:

**Check if the container is running:**

```
docker ps
# If not running, check all containers
docker ps -a
```

The `docker ps` command shows only running containers. If your container isn't listed, it may have stopped or crashed. Adding `-a` shows all containers, including stopped ones, helping you identify if the container exited unexpectedly.

**Python applications not logging:**

```
# Python buffers stdout - force unbuffered output
docker run -e PYTHONUNBUFFERED=1 python:3.9 python app.py
```

Python buffers stdout by default, which means log messages might not appear immediately or at all in `docker logs`. The `PYTHONUNBUFFERED=1` environment variable forces Python to flush output immediately to stdout, making logs visible through Docker's logging system.

**Application writes to files instead of stdout:**

```
# Check what your app actually outputs
docker exec -it nginx-proxy cat /var/log/nginx/access.log
```

Some applications write logs to files instead of stdout/stderr. This command lets you access log files directly inside the container. However, these logs won't appear in `docker logs` and will be lost when the container is removed.

### Logs from stopped containers

Docker preserves logs from stopped containers, which is crucial for debugging crashed applications:

```
# Get logs from stopped container
docker logs crashed-api-server
```

Even after a container stops or crashes, Docker retains its logs until you explicitly remove the container. This lets you examine what happened before the container stopped, which is essential for debugging crashes or unexpected shutdowns.

```
# Remove container and lose logs forever
docker rm crashed-api-server
```

Once you remove a container with `docker rm`, all its logs are permanently deleted. If you need to preserve logs after container removal, you'll need to use external log drivers or copy logs to persistent storage first.

### Real-time log monitoring

For monitoring live application behavior, these commands stream logs as they're generated:

```
# Follow logs like tail -f
docker logs -f nginx-proxy
```

The `-f` flag continuously displays new log entries as they appear, similar to the Unix `tail -f` command. This is essential for monitoring live application behavior and debugging real-time issues.

```
# Follow with timestamps
docker logs -ft nginx-proxy
```

Combines following (`-f`) with timestamps (`-t`), showing exactly when each log entry was generated. This helps correlate log events with external events or issues.

```
# Follow starting from specific time
docker logs --since "2024-01-15T10:00:00" -f api-server
```

Starts displaying logs from a specific timestamp, then continues following new entries. This is useful when you want to see what happened after a known event time without scrolling through earlier logs.

## Filter Docker Logs by Time, Length, or Service

When debugging specific issues, you need to narrow down log output to find relevant information quickly:

```
# Logs from specific time window
docker logs --since "2024-01-15T10:00:00" --until "2024-01-15T12:00:00" api-server
```

Shows logs only from the specified time range. This is particularly useful when you know approximately when an issue occurred and want to focus on that specific period without scrolling through hours of logs.

```
# Last 2 hours
docker logs --since 2h postgres-db
```

Uses relative time syntax to show logs from the past 2 hours. You can use various formats like `30m` (30 minutes), `1d` (1 day), or `3h30m` (3 hours 30 minutes).

```
# Combine with grep for specific errors
docker logs api-server | grep "ERROR"
docker logs api-server | grep -i "failed\|error\|exception"
```

Pipes Docker logs through grep to filter for specific patterns. The first command finds lines containing "ERROR", while the second uses case-insensitive matching (`-i`) to find lines containing "failed", "error", or "exception". This helps you quickly identify error conditions without reading through all log output.

## View Logs from All Your Containers

When working with multi-container applications, Docker Compose provides convenient log access across your entire application stack:

```
# All services
docker-compose logs
```

Displays logs from all services defined in your docker-compose.yml file, with each line prefixed by the service name. This gives you a unified view of your entire application's logging output.

```
# Specific service
docker-compose logs web
```

Shows logs only from the "web" service defined in your compose file. This is useful when you want to focus on a specific component of your application.

```
# Follow multiple services
docker-compose logs -f web api db
```

Continuously streams logs from multiple specific services (web, api, and db in this example). This lets you monitor related services simultaneously while filtering out noise from other components.

```
# With timestamps
docker-compose logs -t web
```

Adds timestamps to each log line from the web service, helping you correlate events across different services or with external systems.

### Multiple standalone containers

For applications using standalone containers rather than Docker Compose:

```
# Quick check across containers
docker logs nginx-proxy && docker logs api-server && docker logs postgres-db
```

Runs `docker logs` commands sequentially for multiple containers. The `&&` operator ensures each command runs only if the previous one succeeds, giving you a quick overview of multiple containers' recent activity.

💡

For a closer look at how different logging drivers work and when to use them, see our guide on [Docker logging drivers](https://last9.io/blog/docker-logging-drivers/).

## Docker Log Drivers and Storage Options

Docker supports different log drivers that fundamentally change how logs are stored and accessed. Understanding these drivers is crucial for production deployments:

```
# Check container's log driver
docker inspect nginx-proxy | grep LogDriver
```

Shows which log driver the container is using. The log driver determines where logs are stored and whether they're accessible via the `docker logs` command.

### Default json-file driver

Most containers use the `json-file` driver, which stores logs as JSON files on the Docker host:

```
# Configure log rotation to prevent disk issues
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx:latest
```

Runs a container with explicit log rotation settings. The `max-size=10m` option limits each log file to 10MB before rotation occurs, while `max-file=3` keeps only the 3 most recent log files. This prevents logs from consuming unlimited disk space, which is critical for long-running production containers.

### Alternative drivers

Some log drivers don't support the `docker logs` command but offer integration with external systems:

```
# These won't work with docker logs command:
# --log-driver none
# --log-driver syslog
# --log-driver journald
# --log-driver gelf
# --log-driver fluentd
```

When using these drivers, the `docker logs` command won't show any output because logs are sent directly to external systems instead of being stored locally. You'll need to use the respective system's tools to access logs.

For these drivers, use system-specific tools to access logs:

```
# journald driver
journalctl CONTAINER_NAME=nginx-proxy
```

When using the journald log driver, logs are sent to the systemd journal. Use `journalctl` to query logs with the container name as a filter.

```
# syslog driver
tail -f /var/log/syslog | grep nginx-proxy
```

When using the syslog driver, logs are sent to the system's syslog daemon. Use standard syslog tools to view logs, filtering by container name or other identifiers.

## Troubleshooting When Logs Don't Appear

You ran `docker logs`, and it returned... nothing. No output, no errors. Here's a quick decision flow to figure out what’s going wrong.

#### Decision Flow: Why Are Logs Missing?

1. **Does the app write to log files instead of stdout/stderr?**  
    Check the app’s logging configuration. Anything written to a file (like `/tmp/app.log`) won’t show up in `docker logs`.
2. **Are you using `docker exec` to run commands?**  
    Output from `docker exec` isn’t captured by Docker's logging system.

**Is a custom log driver in use?**  
Check with:

```
docker inspect <container> | grep LogDriver
```

Some drivers (like `syslog`, `journald`, `gelf`) don’t work with `docker logs`.

**Is this a Python app?**  
Python buffers stdout by default. Add this when running your container:

```
-e PYTHONUNBUFFERED=1
```

**Is the container running?**  
Check with:

```
docker ps
```

If it’s not running, use `docker ps -a` and fetch logs from the stopped container.

### Common Fixes

#### Python: Flush Logs Immediately

Python buffers output unless you explicitly disable it. To get logs right away:

```
docker run -e PYTHONUNBUFFERED=1 python:3.9 python -u app.py
```

This ensures logs go directly to stdout/stderr where Docker can capture them.

#### Main Process vs `docker exec`

Docker only captures logs from the container’s main process (PID 1).

This will appear in logs:

```
docker run busybox echo "hello"
```

This won’t:

```
docker exec busybox echo "hello"
```

Stick to the main process if you want logs to show up in `docker logs`.

### Access Log Files Directly (as a Last Resort)

If the container is using the `json-file` driver and logs still aren’t showing; you can read the raw files directly:

**Find the log file path:**

```
docker inspect nginx-proxy | grep LogPath
```

**Read the log file:**

```
sudo cat /var/lib/docker/containers/<container_id>/<container_id>-json.log
```

**Pretty-print the JSON output:**

```
sudo cat /var/lib/docker/containers/<container_id>/<container_id>-json.log | jq -r '.log'
```

Use this approach only when you're debugging the Docker daemon or diagnosing something low-level. For most apps, logs should always come through stdout/stderr and be routed properly using structured logging and a reliable log driver.

💡

Now, fix Docker container log issues instantly, right from your IDE, with AI and [Last9 MCP](https://last9.io/mcp/). Bring real-time production context, logs, metrics, and traces into your local environment to auto-fix code faster.

## What Changes When You're Logging in Production

In local dev, it’s okay if logs pile up or stream endlessly in your terminal. But in production, log volume grows fast, and unmanaged logging can eat up disk space, slow down your app, or leave you with gaps when you need the data.

Here’s what you should set up differently when your containers move to prod.

### 1. Manage Disk Usage and Set Up Log Rotation

Production containers run longer and generate more logs. You don’t want these logs filling up your disk.

**Check how much space Docker logs are using:**

```
docker system df
```

This shows disk usage across containers, images, and logs—useful for spotting containers that are writing too much.

**Clean up unused containers (and their logs):**

```
docker system prune
```

This frees up space by removing stopped containers, unused networks, and dangling images. But be careful—this deletes data permanently.

**Set up automatic log rotation:**

```
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=5 \
  nginx:latest
```

This keeps log files to a max of 10MB each and stores only the 5 most recent files. Older logs are automatically removed, helping you avoid disk issues over time.

### 2. Use Structured Logging for Better Search and Analysis

In production, plain text logs become hard to work with. Switching to structured JSON logs gives you more control.

**Example of a structured log entry:**

```
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "ERROR",
  "service": "api-server",
  "message": "Database connection failed",
  "error": "connection timeout",
  "request_id": "abc123"
}
```

Why it helps:

- `timestamp`: Enables time-based filtering
- `level`: Helps filter by severity
- `service`: Makes it easier to trace logs in multi-service systems
- `request_id`: Links logs across services for the same user request

Use a structured format early; it makes downstream processing and debugging a lot easier.

### 3. Centralize Logs Across Services

If you’re running multiple containers across nodes or clusters, reading logs one container at a time doesn’t scale.

**Send logs to Fluentd (or similar tools):**

```
docker run -d \
  --log-driver fluentd \
  --log-opt fluentd-address=localhost:24224 \
  --log-opt tag="app.{{.Name}}" \
  nginx:latest
```

This pushes container logs to a Fluentd instance, where you can parse, filter, enrich, and forward them to something like Elasticsearch, S3, or a commercial logging service.

Other options include Logstash, Vector, or using the OpenTelemetry Collector to process and export logs.

If you’re already sending metrics or traces through OpenTelemetry, you can route logs the same way and forward them to [Last9](https://last9.io/), which supports native correlation across logs, metrics, and traces. That way, you’re not just collecting logs, you’re connecting them to actual system behavior.

## Advanced Monitoring Setup

At some point, `docker logs` won’t cut it. Especially when logs start carrying high-cardinality data, user IDs, request paths, and dynamic labels. You’ll need a setup that’s designed to scale.

This is where structured logging and aggregation patterns help you. Last9 handles high-cardinality telemetry natively, across logs, metrics, and traces. If you're already using OpenTelemetry or Prometheus, your logs can follow the same pipeline. No format juggling. No extra glue code.

### Log Aggregation Patterns That Scale

Here’s one way to forward logs from containers into a centralized system using GELF (used by the ELK stack):

```
docker run -d \
  --log-driver gelf \
  --log-opt gelf-address=udp://localhost:12201 \
  api-server:latest
```

This pushes logs to a Logstash endpoint listening on UDP, where they can be parsed and routed to Elasticsearch. Kibana handles the search and visualization side.

You can also use the OpenTelemetry Collector to bring log data into [Last9](https://last9.io/) alongside your metrics and traces. The benefit? Everything shares a common timeline, trace context, and resource metadata, so you're not stitching systems together manually.

## Your Go-To Docker Logging Reference

```
docker logs <container>               # Basic logs  
docker logs -f <container>            # Stream logs in real time  
docker logs --tail 50 <container>     # Show last 50 lines  
docker logs --since 1h <container>    # Show logs from the past hour  
docker logs -t <container>            # Include timestamps
```

### Troubleshooting Checklist

- **Is the container running?**  
    `docker ps`
- **Is this a Python app?**  
    Add `-e PYTHONUNBUFFERED=1` to flush logs
- **Custom log driver in use?**  
    `docker inspect <container> | grep LogDriver`
- **App writing to a file instead of stdout/stderr?**  
    Check logging config
- **Using `docker exec`?**  
    Won’t show up in `docker logs`—only the main process output is captured

### Production Logging Essentials

- Use `--log-opt max-size=10m` and `--log-opt max-file=5` to rotate logs
- Output logs in structured JSON for better parsing and analysis
- Centralize logs using Fluentd, Logstash, or the OpenTelemetry Collector

Monitor disk usage with:

```
docker system df
```

## FAQs

**Q: Why don't I see logs from commands I run with docker exec?**

A: Docker logs only capture output from the main process (PID 1). Commands run via `docker exec` are separate processes.

**Q: My Python app uses print(), capture**, **but no logs appear. What's wrong?**

A: Python buffers output. Use `PYTHONUNBUFFERED=1` environment variable or run with `python -u`.

**Q: Can I get logs from a removed container?**

A: No, `docker rm` deletes logs. Use external log drivers if you need log persistence.

**Q: How do I save logs to a file?**

A: Use shell redirection: `docker logs nginx-proxy > logs.txt` or `docker logs --since=1h api-server > /path/to/logs.txt`

**Q: Why are my logs not showing up in real-time with -f?**

A: Some applications buffer output. Run containers with `-t` (pseudo-TTY) or configure applications to flush immediately.

**Q: Can I search through Docker logs?**

A: Pipe to grep: `docker logs api-server | grep "ERROR"` or use a log management system for advanced search.

**Q: How much disk space do logs use?**

A: By default, Docker doesn't limit log size. Use `--log-opt max-size=10m --log-opt max-file=3` to prevent disk issues.