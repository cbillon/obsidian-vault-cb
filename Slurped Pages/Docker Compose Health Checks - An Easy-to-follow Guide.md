---
link: https://last9.io/blog/docker-compose-health-checks/
byline: Anjali Udasi
site: Last9
date: 2025-03-27T10:31
excerpt: Ensure your containers are truly ready, not just running. This guide
  covers Docker Compose health checks and how to use them effectively.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T18:39
title: "Docker Compose Health Checks: An Easy-to-follow Guide"
---

A container in a "running" state doesn’t guarantee the application inside is operational. Services can fail silently, crashing after startup (often due to [OOM kills](https://last9.io/blog/what-is-oom/)), waiting on unavailable dependencies, or serving incomplete responses, while still appearing healthy to orchestration tools.

These false-positive states often lead to extended debugging sessions during incidents. Internal data shows that each occurrence can cost teams an average of 3.2 hours to resolve.

Docker Compose health checks mitigate this by running explicit readiness probes inside the container. These checks execute user-defined commands at regular intervals to validate that the service is responsive and behaves as expected. This ensures downstream services only interact with fully ready components, reducing deployment failures and accelerating incident recovery.

**How do you add a health check to Docker Compose?** Add a `healthcheck` block to a service in your `compose.yaml` with `test`, `interval`, `timeout`, `retries`, and `start_period` fields. The `test` command runs inside the container — a zero exit code means healthy. Use `depends_on` with `condition: service_healthy` to hold dependent services until the check passes. Run `docker compose ps` to see live health status for all services.

## What Are Docker Compose Health Checks

Docker Compose health checks define commands that run inside a container to verify that the application is functioning as expected.

Unlike basic status checks that only confirm if a container is running, these checks validate service readiness, ensuring that the process inside the container is healthy, responsive, and capable of handling requests.

💡

Keeping logs in check is just as important as monitoring container health. Here's a [guide on clearing Docker logs](https://last9.io/blog/docker-clear-logs/) to help you avoid unnecessary clutter.

### The Problem Docker Compose Health Checks Solve

Containers often start before their dependencies are fully ready. A common failure case occurs when the API container initializes before the database has completed its startup, resulting in connection errors and failed requests.

Without a readiness check, Docker Compose treats the container as healthy, even when it's not operational. Health checks solve this by allowing containers to delay startup sequencing until critical dependencies are confirmed to be ready.

Here are some scenarios where health checks are essential:

- **Microservices with tight dependencies**  
    In distributed systems with many interdependent services, health checks help enforce startup order and ensure that each service begins only once its dependencies are confirmed healthy.
- **Database-backed applications**  
    Many services require the database to be fully initialized, not just running. Health checks can confirm the database is accepting connections or has applied required migrations before the application starts.
- **External service integration**  
    When a container relies on third-party APIs or services, health checks can validate connectivity or response status before marking the container as healthy.
- **Production and HA environments**  
    In orchestrated setups (e.g., Docker Swarm or Kubernetes), health checks inform the scheduler when to restart unhealthy containers or remove them from load balancers, ensuring only live instances receive traffic.

## Quick Start: A Common Health Check Pattern

If your service exposes an HTTP `/health` or `/status` endpoint, you can start with this simple health check. It works well for most HTTP-based applications:

```
# A widely used pattern for HTTP services
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
  interval: 30s
  timeout: 10s
  retries: 3
```

You can plug this directly into your `docker-compose.yml`. Just make sure the image includes `curl`. For Alpine-based images, that usually means adding this to your Dockerfile:

```
RUN apk add --no-cache curl
```

This gives you a fast, reliable way to catch startup issues without needing custom scripts.

## Set up Your First Docker Compose Health Check

### Prerequisites

Before running the example below, ensure each container has the necessary tools to run its health check.

For instance, the `web` and `api` services use `curl` to validate HTTP endpoints. If you're using minimal base images like Alpine, they won't include `curl` by default. You'll need to add it manually:

```
# Example Dockerfile for Alpine-based image
FROM nginx:alpine
RUN apk add --no-cache curl
```

Without this, the health check will fail silently with errors like `curl: not found`, even if the container builds and runs fine otherwise. Always verify that health check dependencies exist in your image.

_Here's how to setup your first health check:_

Defining a health check in your `docker-compose.yml` file is a direct way to enforce runtime sanity checks for your containers. Here’s a basic example:

```
version: '3.8'
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

In this setup, Docker executes the health check command (`curl -f http://localhost`) inside the container at the specified interval. The container’s health status can be one of the following:

- `starting`: Health checks are running, but failures are ignored during the `start_period`.
- `healthy`: The check passed with an exit code `0`.
- `unhealthy`: The check failed `retries` times consecutively.

### Health Check Options Explained

|Option|Purpose|Example Value|
|---|---|---|
|`test`|Command executed to check service health|`["CMD", "curl", "-f", "http://localhost"]`|
|`interval`|Time between successive health checks|`30s`|
|`timeout`|Maximum time to wait for a check to complete|`10s`|
|`retries`|Failures required before marking the container unhealthy|`3`|
|`start_period`|Grace period before counting failures|`40s`|

### What This Does

#### `test: ["CMD", "curl", "-f", "http://localhost"]`

**Why this works:**  
This command attempts to fetch the container’s root URL. The `-f` flag makes `curl` exit with a non-zero code if the response code is 400 or above.

**Common failure scenarios:**

- Web server hasn't started yet or is bound to a different interface.
- Service returns a 5xx error due to missing environment variables or bad config.
- Firewall rules or missing `EXPOSE` instructions block internal traffic.

**Resource impact:**  
Minimal. `curl` is lightweight and fast unless the endpoint hangs, in which case the timeout setting will cap the impact.

**Troubleshooting tips:**

- Shell into the container and manually run the command to verify it works:  
    `docker exec -it <container> curl -f http://localhost`
- Check the container logs to confirm that the app is listening on the expected port.
- If `curl` isn’t available in your base image (e.g., `alpine`), install it or switch to `wget` or `nc`.

## Health Check Commands for Common Services

Not all containers should be checked the same way. A simple `curl` might be fine for a web server, but a database isn't truly "healthy" until it's ready to handle real connections. Here are practical health check examples for widely used services, along with why they work, what can go wrong, and how to debug them effectively.

### Web Server (Nginx, Apache)

```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
```

This sends a basic HTTP request to the root endpoint. The `-f` flag ensures the health check fails on HTTP error responses (e.g., 404, 500).

**Why it works:**  
Confirms the web server is listening, responding on the right port, and returning a valid HTTP status.

**Common pitfalls:**

- The web server isn’t fully started
- Listening on a different interface (`0.0.0.0` vs `localhost`)
- `curl` not present in the container image

**Troubleshooting tip:**

```
docker-compose exec <service> curl -f http://localhost
```

### MySQL

```
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "${MYSQL_USER}", "-p${MYSQL_PASSWORD}"]
```

Uses `mysqladmin ping` to check server liveness, no SQL queries involved.

**Why it works:**  
Fast and lightweight. It checks the TCP handshake and verifies credentials without opening a full session.

**Common pitfalls:**

- Credentials are incorrect
- MySQL hasn’t finished initializing
- `mysqladmin` isn’t available in the image

**Troubleshooting tip:**

```
docker-compose exec <service> mysqladmin ping -h 127.0.0.1 -u root -p
```

If you hit socket errors, force TCP by using `127.0.0.1`.

### PostgreSQL

```
healthcheck:
  test: ["CMD", "pg_isready", "-U", "postgres"]
```

`pg_isready` reports whether the server is accepting new connections.

**Why it works:**  
Purpose-built for this exact need. It avoids full auth and provides a reliable readiness signal, even during recovery.

**Common pitfalls:**

- Postgres is still starting (e.g., applying WAL logs)
- `pg_isready` missing from the image
- Role mismatch or invalid permissions

**Troubleshooting tip:**

```
docker-compose exec <service> pg_isready -U postgres -h localhost
```

Check logs for startup states like `"database system is starting up"`.

### Redis

```
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
```

Sends the standard `PING` command and expects a `PONG` in response.

**Why it works:**  
Minimal and direct. If Redis is up and reachable, this succeeds instantly.

**Common pitfalls:**

- Redis hasn’t bound to the expected port or IP
- `redis-cli` isn’t in the container
- Auth is required and not supplied

**Troubleshooting tip:**

```
docker-compose exec <service> redis-cli ping
# For password-protected setups:
redis-cli -a $REDIS_PASSWORD ping
```

### MongoDB

```
healthcheck:
  test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
```

Runs a native command that confirms server liveness without reading from collections.

**Why it works:**  
`db.adminCommand('ping')` is as lightweight as it gets for MongoDB, confirming readiness at the command level.

**Common pitfalls:**

- `mongosh` missing from the image
- Server still configuring a replica set
- Auth not provided

**Troubleshooting tip:**

```
docker-compose exec <service> mongosh --eval "db.adminCommand('ping')"
# For secured deployments:
mongosh -u admin -p secret --authenticationDatabase admin --eval "db.adminCommand('ping')"
```

### RabbitMQ

```
healthcheck:
  test: ["CMD", "rabbitmq-diagnostics", "check_port_connectivity"]
```

Performs an internal diagnostic to verify critical ports are available.

**Why it works:**  
Goes beyond checking the process. It confirms that RabbitMQ is ready to route messages.

**Common pitfalls:**

- RabbitMQ still booting or initializing plugins
- Diagnostic tool missing or not on `$PATH`

**Troubleshooting tip:**

```
docker-compose exec <service> rabbitmq-diagnostics check_port_connectivity
```

Inspect logs for boot delays or permission issues.

💡

Identify and resolve container health check failures in real time, directly from your IDE. [Last9 MCP](https://last9.io/mcp/) brings live container status, logs, and metrics into your local setup, helping you troubleshoot issues faster with full production context.

### Coordinate Service Startup with Health Checks

Docker Compose’s `depends_on` with `condition: service_healthy` is a powerful way to enforce startup order between containers, not just at the process level, but based on actual service readiness.

**Example:**

```
version: '3.8'

services:
  db:
    image: postgres
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

  api:
    image: myapi
    depends_on:
      db:
        condition: service_healthy
```

In this configuration:

- The `db` service runs a health check using `pg_isready`.
- Docker Compose will only start the `api` container after the database has passed its health check.

This goes beyond the default `depends_on`, which simply checks whether a container has started. Without a health check, services like the API might start too early, before the database is fully initialized, causing connection errors or unexpected behavior.

By using `condition: service_healthy`, you ensure that:

- `pg_isready` must return success.
- The API only starts once the database is fully ready to accept connections.

This setup results in more predictable startup behavior and fewer race conditions during initialization.

💡

If you need better visibility into your containers, check out this [guide on Docker Compose logs](https://last9.io/blog/docker-compose-logs/) to track and troubleshoot issues efficiently.

## Handle Complex Service Dependencies in Docker Compose

Docker Compose gives you more than just a way to spin up services. With health checks and conditional dependencies, you can control _when_ services start, not just _that_ they start.

Let’s say your API needs a database to be fully ready before it starts. You can use `depends_on` with `condition: service_healthy` to make this happen:

```
version: '3.8'
services:
  db:
    image: postgres
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

  api:
    image: myapi
    depends_on:
      db:
        condition: service_healthy
```

In this setup, the API waits for the database’s health check to pass before it begins. That’s a big improvement over the default behavior, where `depends_on` only ensures the container has _started_, not that it’s _ready_.

### What Conditions Can You Use?

Docker Compose supports different types of startup conditions under `depends_on`. Here’s a quick reference:

|Condition|What it means|
|---|---|
|`service_started` (default)|Waits for the container to start, without checking internal readiness|
|`service_healthy`|Waits for the container’s health check to report success|
|`service_completed_successfully`|Waits for the container to exit with a 0 status (useful for init containers)|

You can use these conditions together to build startup flows that suit your architecture.

For example, let’s say you’re running database migrations with Flyway before starting your API. You can set up `db-init` to run only after the database is healthy, and the API to wait for both `db` and `db-init`:

```
services:
  db-init:
    image: flyway
    command: migrate
    depends_on:
      db:
        condition: service_healthy

  api:
    depends_on:
      db:
        condition: service_healthy
      db-init:
        condition: service_completed_successfully
```

This ensures the API doesn’t start until both the database is up and the migrations have finished.

### What About Circular Dependencies?

Sometimes services rely on each other. Maybe the database needs to contact the API for some bootstrap config, and the API also needs the DB to be up. Compose doesn’t have a built-in way to handle circular dependencies, but you can work around it.

Here’s one option: start services in a simplified mode using environment variables, then switch to full behavior once everything’s up.

```
services:
  api:
    environment:
      - STARTUP_MODE=standalone
    healthcheck:
      test: ["CMD", "sh", "-c", "if [ \"$STARTUP_MODE\" = \"standalone\" ]; then curl -f http://localhost/basic-health; else curl -f http://localhost/full-health; fi"]
```

This lets the API run basic checks early on, before all its dependencies are ready. Once the system stabilizes, you can switch to full checks or restart the service with different settings.

## Advanced Health Check Techniques

Once you've covered the basics, checking if your service is running and listening, there’s value in going further. These techniques help you build more reliable, context-aware health checks.

#### 1. Custom Health Endpoints That Reflect Application State

A common pattern is to expose a dedicated `/health` or `/status` endpoint that checks the state of key dependencies: your database, cache, file system, or upstream APIs. This lets your application define its readiness.

**Example in Node.js:**

```
app.get('/health', async (req, res) => {
  try {
    await db.query('SELECT 1');
    await redis.ping();
    const apiResponse = await fetch('https://api.example.com/ping');
    if (!apiResponse.ok) throw new Error('External API unavailable');

    const disk = await checkDiskSpace();
    if (disk.free < 100 * 1024 * 1024) throw new Error('Low disk space');

    res.status(200).json({ status: 'healthy' });
  } catch (err) {
    res.status(500).json({ status: 'unhealthy', error: err.message });
  }
});
```

Then wire it into your Compose config:

```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
  interval: 10s
  timeout: 5s
  retries: 3
```

This setup allows your service to self-report its operational state, rather than relying on superficial checks.

#### 2. Shell-Based Checks for Multi-Component Services

If your service relies on several subsystems, like background workers, queues, or file mounts, a shell script can coordinate the checks more flexibly.

**Compose example:**

```
healthcheck:
  test: ["CMD", "sh", "-c", "./health-check.sh"]
  interval: 15s
  timeout: 5s
  retries: 3
```

**`health-check.sh`:**

```
#!/bin/bash

# Check that the main process is alive
pgrep -f myservice > /dev/null || exit 1

# Confirm internal endpoints are responsive
curl -s http://localhost:8080/ping > /dev/null || exit 1

# Check dependency health
./check-db.sh || exit 1
./check-redis.sh || exit 1

# Optional: look for critical errors in logs
grep -q "FATAL" /var/log/myservice.log && exit 1

exit 0
```

This approach gives you full control over how liveness and readiness are defined for your application.

#### 3. Functional Health Checks That Simulate Usage

Sometimes, the best way to verify a system is healthy is to act like a user. A functional check creates and deletes a test resource to make sure critical paths work end-to-end.

**Example:**

```
#!/bin/bash

# Try creating a user
curl -s -X POST -d '{"username":"hc","password":"test"}' \
  -H "Content-Type: application/json" \
  http://localhost:8080/api/users > /dev/null || exit 1

# Authenticate and extract token
TOKEN=$(curl -s -X POST -d '{"username":"hc","password":"test"}' \
  -H "Content-Type: application/json" \
  http://localhost:8080/api/auth | jq -r .token)

[ -z "$TOKEN" ] || [ "$TOKEN" = "null" ] && exit 1

# Delete the user
curl -s -X DELETE -H "Authorization: Bearer $TOKEN" \
  http://localhost:8080/api/users/hc > /dev/null || exit 1

exit 0
```

Functional checks are especially useful in services with tight SLAs or frequent regressions, offering confidence that core workflows are intact.

### Health Checks in a Multi-Service Docker Compose Stack

Let’s walk through a working example of health checks in Docker Compose. This stack includes a PostgreSQL database, Redis cache, backend API, and a web frontend. Each service is configured with a health check that reflects its actual readiness, not just whether the container is running.

```
version: '3.8'

services:
  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: example
      POSTGRES_USER: app
      POSTGRES_DB: appdb
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "app"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

  redis:
    image: redis:6
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 3

  api:
    build: ./api
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 15s

  web:
    build: ./frontend
    depends_on:
      api:
        condition: service_healthy
    ports:
      - "80:80"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
```

**What This Setup Does**

- **PostgreSQL** is considered healthy once it accepts connections via `pg_isready`.
- **Redis** waits for a successful `PING` response before it's marked healthy.
- **API service** won’t start until both the database and Redis are marked healthy.
- **Frontend** only starts after the API has passed its own health check.
- Every service includes a tailored health check that defines what “ready” means in the context of that service, whether that’s SQL readiness, Redis responsiveness, or HTTP 200s.

This pattern prevents race conditions like the API trying to query a database that’s still booting up or the frontend hitting an uninitialized backend.

On modern machines or CI runners, this stack typically reaches a fully healthy state within 15–30 seconds after `docker-compose up`, depending on image size, network conditions, and health check intervals.

💡

If you're looking to understand and manage Docker logs effectively, check out this [guide on Docker logs](https://last9.io/blog/docker-logs/) for a clear breakdown.

## Troubleshooting Health Checks in Docker Compose

Health checks are powerful, but when they misfire, they can be frustrating. Here are a few common problems and how to debug them.

### The Container Keeps Restarting

If your container keeps restarting, the health check is probably failing. Start by checking the basics:

- **Is the command valid?** For example, if you’re using `curl`, make sure it’s installed in the container. Minimal base images like Alpine often don’t include it by default.
- **Is the service running?** The container might be up, but the application inside it may not have started yet.
- **Are you hitting the correct port or endpoint?** A mismatch here will cause false failures.

If a health check fails repeatedly, Docker marks the container as `unhealthy`, which can trigger restarts depending on your Compose configuration.

### The Health Check Passes, But the Service Isn’t Ready

Sometimes the check itself is too shallow. For example, a database might respond to a ping but still not be ready to accept connections. A basic TCP connection isn't always enough.

In such cases, strengthen the health check. Instead of checking if the process is up, check whether it's ready to serve:

```
healthcheck:
  test: ["CMD", "sh", "-c", "pg_isready -U postgres && psql -U postgres -c 'SELECT 1'"]
```

This ensures not just process availability, but actual readiness to handle queries.

### The Service Takes Too Long to Start

Some services need extra time to initialize, and premature health checks can mark them as unhealthy before they’re ready.

To account for this, use the `start_period` parameter:

```
healthcheck:
  start_period: 120s
```

This gives the container a 2-minute grace period before failures are counted toward the health status. It’s especially useful for large services like databases, search engines, or anything that runs heavy migrations during startup.

## FAQ

### What does `start_period` do in a Docker Compose health check?

`start_period` gives the container a grace period to initialize before failed checks count against the retry limit. During `start_period`, a failed check does not increment the failure counter — only a successful check ends the grace period early. Use it for services that take several seconds to boot, like databases or JVM apps.

### How do you disable a health check in Docker Compose?

Set `test: ["NONE"]` in the service's `healthcheck` block. This overrides any `HEALTHCHECK` instruction baked into the Docker image and marks the container as having no health check.

### What happens when a Docker Compose health check fails?

After `retries` consecutive failures, Docker marks the container as `unhealthy`. Any dependent services using `condition: service_healthy` in their `depends_on` block will not start. The container itself keeps running — Docker Compose does not automatically restart an unhealthy container unless you configure a restart policy.

### Can you use `depends_on` without health checks?

Yes. The default condition is `service_started`, which only waits for the container process to start — not for the application inside to be ready. This is often insufficient for databases or APIs. Use `condition: service_healthy` with a proper `healthcheck` block to wait for real readiness.

### How do you see health check output and history?

Run `docker inspect --format='{{json .State.Health}}' <container_name>` to see the last five health check results, including exit codes, timestamps, and stdout output from the test command. This is the fastest way to debug why a container is stuck in `starting` or `unhealthy` state.

## Conclusion

Docker Compose health checks do more than keep containers from crashing into each other. They help you build services that actually know when they're ready, cutting down on flaky startups, failed dependencies, and painful debug sessions.

But readiness in staging isn't the same as resilience in production. At [Last9](https://last9.io/), we go one step further, tracking how service health evolves over time, across environments. Whether it's a container stuck in `starting`, a database that’s "up" but not accepting connections, or a subtle delay causing cascading retries, we give you the metrics and traces to see _why_ something broke, not just that it did.

Health checks get you consistency at the container level. Last9 brings that same confidence to production.

💡

If you've any questions about implementing health checks in your specific stack Drop a comment or join our [Discord Community](https://discord.com/invite/W8gMppQC4b) – we're always down to chat about container health, dependencies, and everything around DevOps.