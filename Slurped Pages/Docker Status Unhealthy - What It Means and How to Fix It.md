---
link: https://last9.io/blog/docker-status-unhealthy-how-to-fix-it/
byline: Faiz Shaikh
site: Last9
date: 2025-07-04T10:41
excerpt: What Docker’s “unhealthy” status means, why it happens, and how to
  debug failing containers with clarity and control.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T15:06
title: "Docker Status Unhealthy: What It Means and How to Fix It"
---

If your container shows `Status: unhealthy`, Docker's health check is failing. The container is still running, but something inside—usually your app—isn't responding as expected.

This doesn't always mean a crash. It just means Docker can't verify the app is working. The health check runs separately from the container's lifecycle, so even a running container can be marked unhealthy if it fails repeated checks.

Here's what triggers it: Docker runs a command inside your container at regular intervals (typically every 30 seconds). If that command fails multiple times in a row, the status flips to `unhealthy`. The failure could be a timeout, a connection refusal, or any non-zero exit code.

The most common causes: your app crashed, a dependency is down, resources are maxed out, or the health check itself is misconfigured. We'll walk through how to diagnose each one and get your container back to healthy.

## How Docker Health Status Works

Docker runs health checks separately from the container's lifecycle. Even if a container is running, Docker can still mark it as `unhealthy` if the health check command fails.

A health check runs inside the container at set intervals. It typically hits an endpoint or runs a command to check if your app is alive and responding.

There are three possible health states:

- `starting`: The container is still in its startup period.
- `healthy`: The last few health checks passed.
- `unhealthy`: Multiple health checks failed.

The container's health status depends on the exit code from the command:

- `0`: Healthy
- `1`: Unhealthy
- Anything else: Inconclusive, Docker leaves the status unchanged.

This separation matters because a container can be running but completely broken from a user's perspective. Docker gives you a way to detect that without manually checking every service.

## Quick Wins

Here's how to get an unhealthy container back on track quickly.

### 1. Check What Failed

Inspect health logs:

```
docker inspect --format "{{json .State.Health }}" container_name | jq
```

Look for recent `Output`, `ExitCode`, and error messages.

### 2. Test the Health Check Inside the Container

Match the container's environment:

```
docker exec -it container_name sh -c '<your health check command>'
```

This catches issues with missing ports, permissions, or dependencies.

### 3. Fix the Most Common Causes

- **App crash**: Logs show connection refused or stack traces
- **Missing dependency**: DB or API call fails inside the container
- **Slow startup or load**: Health check times out repeatedly
- **Wrong health check config**: Mismatched port or URL path

### 4. Adjust the Health Check Settings

Tweak the timing so Docker doesn't panic too early:

```
HEALTHCHECK --interval=30s --timeout=10s --start-period=45s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

### 5. Let Docker Restart It

To enable restarts:

- Use [autoheal](https://github.com/willfarrell/docker-autoheal)
- Or exit the process on failure to trigger the restart policy:

```
CMD curl -f http://localhost:8080/health || kill -s 15 1
```

**When DIY Docker Health Checks Aren't Enough**

The debugging steps above work great for:

- Local development
- Small deployments (< 10 containers)
- One-off issues you can reproduce immediately

But if you're dealing with:

- Containers that flap healthy/unhealthy, and you can't reproduce it
- Health failures that only happen in production under load
- Multiple containers failing across different hosts
- No clear correlation between health failures and application behavior

...then you need centralized health tracking with automatic correlation. That's what [Last9](https://last9.io/) does.

## Inspect Health Check Logs

### Check the Health Logs

Start by inspecting the container's health details. This helps you see exactly what Docker's health check is doing and why it's failing.

Use this command:

```
docker inspect --format "{{json .State.Health }}" container_name | jq
```

It returns structured output with:

- Current health status (`starting`, `healthy`, `unhealthy`)
- Failing streak count
- A log of recent health checks (with timestamps, exit codes, and output)

If `jq` isn't installed, drop it for raw JSON:

```
docker inspect --format "{{json .State.Health }}" container_name
```

Look for `ExitCode: 0` (success) or `ExitCode: 1` (failure). Any output from the failed checks can help narrow down the issue.

The output usually tells you immediately what's wrong. If you see "Connection refused", the app isn't listening. If you see "timeout", something's hanging. If the output is empty, the health check command itself might be broken.

### Debug Health Check Commands

Test your health check command directly inside the container to isolate the issue:

```
docker exec -it container_name curl -f http://localhost:8080/health
echo $?
```

This approach lets you see exactly what your health check encounters. You might discover that the health endpoint returns unexpected status codes, takes too long to respond, or that required tools like `curl` aren't available in your container.

For more complex health checks, examine the command that Docker runs. Look at your `Dockerfile` or `docker-compose.yml` to understand the exact test being performed:

```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost/api/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

If the command works when you run it manually but fails in the health check, the difference is usually timing (it's running before your app is ready) or environment (missing env vars, wrong network settings).

## Set up Health Checks in Docker and Docker Compose

Health checks help Docker understand if your containerized app is working, not just running. Here's how to add them to your setup and make sure they reflect real application health.

### Add a Basic Health Check in Your Dockerfile

Use the `HEALTHCHECK` instruction to define how Docker monitors your container.

For a simple HTTP service:

```
HEALTHCHECK CMD curl --fail http://localhost:8080 || exit 1
```

This tells Docker to run that command inside the container. If `curl` gets a successful response, Docker marks the container as healthy. If not, it sets the status to `unhealthy`.

You can customize how often Docker runs the check and how sensitive it is:

```
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD curl --fail http://localhost:8080/health || exit 1
```

- `interval`: Time between checks (default: 30s)
- `timeout`: Max time Docker waits for the check to finish (default: 30s)
- `start-period`: Grace period after startup when failures are ignored
- `retries`: How many failures trigger an `unhealthy` state

### Use Custom Health Check Scripts

If your app needs more than a single HTTP check, write a script:

```
COPY healthcheck.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/healthcheck.sh
HEALTHCHECK CMD /usr/local/bin/healthcheck.sh
```

Example `healthcheck.sh`:

```
#!/bin/bash

# Check Postgres
pg_isready -h localhost -p 5432 -U myuser || exit 1

# Check API
curl -f http://localhost:8080/api/health || exit 1

# Check Redis
redis-cli ping || exit 1

exit 0
```

Keep the script fast and lightweight. Avoid expensive operations or long-running queries—health checks shouldn't add overhead.

When you have multiple dependencies, this approach gives you more control over what "healthy" actually means. A single failed dependency can take down your entire app, so checking them all makes sense.

### Write Checks That Reflect Real Application Health

A health check should confirm the app can serve requests, not just that the process exists.

Good examples:

- HTTP services: `curl -f http://localhost:8080/api/health`
- Databases: `pg_isready -h localhost -p 5432 -U myuser`
- Message brokers: `nc -z rabbitmq 5672`

Avoid checks that only confirm the process is alive (e.g., checking PID files or running `ps`). Your container can look "healthy" even if the app is broken.

The best health checks hit a real endpoint or connection that exercises the core functionality. If your app serves HTTP requests, check an HTTP endpoint. If it processes queue messages, check the queue connection. Don't check a sidecar process that has nothing to do with your actual service.

## Tune Your Health Check Parameters

Health checks are great, until they get too noisy or too slow. Most of that comes down to timing. Set the wrong values, and you'll either miss real issues or end up restarting healthy containers.

Here's how to get each setting right.

### `interval`: How Often to Run the Check

This controls how often Docker runs the health check.

- Set it too low, and your container spends half its time checking itself.
- Set it too high, and you won't catch failures quickly.

**Tip:** 10–30 seconds is a good starting point. Go shorter for critical services that need fast detection.

### `timeout`: How Long to Wait for a Response

If your app takes a while to respond—especially under load—short timeouts can cause false alarms.

**Tip:** Match this to your app's real response time. If most endpoints return in under a second, 2–5 seconds should be plenty.

### `retries`: How Many Fails Before Giving Up

Some apps stall occasionally—say, during GC or burst load. One failure doesn't always mean something's wrong.

**Tip:**

- Use higher retries (3–5) for apps that sometimes hiccup.
- Lower retries (1–2) are better if you want to fail fast and restart quickly.

### `start_period`: Grace Time After Startup

Some apps need a bit to get going—connecting to databases, loading config, etc. Without a grace period, the health check might fail before the app is even ready.

**Tip:** If your app takes 30 seconds to boot, set a `start_period` of 30–60 seconds. It'll save you from false starts and restart loops.

In production, tuning these values matters more than you'd think. We've seen containers flap healthy/unhealthy every 15 seconds simply because the timeout was too aggressive for the actual response time under load. If you're tracking these patterns over time, you start to see which services need more lenient settings and which ones are legitimately broken.

## Health Checks in docker-compose

Docker Compose lets you define health checks alongside service dependencies. Here's an example:

```
version: '3.8'

services:
  web:
    build: .
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:13
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5
```

Use `docker-compose ps` to see the health status of each container in your stack.

The `depends_on` with `condition: service_healthy` is powerful—it ensures your web service doesn't start until the database is actually ready, not just running. This prevents the classic startup race condition where your app crashes because it tries to connect before Postgres is accepting connections.

## What to Do When a Container Turns Unhealthy

By default, Docker won't restart a container just because it's marked `unhealthy`. That's intentional—health checks are diagnostic, not reactive.

If you're running standalone containers, you'll need to wire up your own restart logic. One common option is the [autoheal](https://github.com/willfarrell/docker-autoheal) container, which watches for unhealthy containers and restarts them.

Another approach: combine your health check with a manual process exit. That way, Docker's built-in restart policies can kick in.

```
HEALTHCHECK --interval=5m --timeout=2m --start-period=45s \
  CMD curl -f --retry 3 --max-time 5 http://localhost:8080/health || kill -s 15 1
```

This example uses `curl` to check an endpoint. If it fails, the container's main process (PID 1) is terminated, triggering a restart—assuming you've set a `--restart` policy.

In orchestrated setups (like [Docker Swarm](https://last9.io/blog/docker-vs-docker-swarm-key-differences-explained/) or [Kubernetes](https://last9.io/blog/kubernetes-vs-docker-swarm/)), this is usually handled for you. The scheduler watches health states and automatically replaces failing containers as part of service management.

### A Better Alternative: Unified Health + Auto-Remediation

Instead of stitching together autoheal + restart policies + custom scripts, [Last9](https://last9.io/docs/integrations/containers-and-k8s/docker/) gives you:

- **Built-in health tracking** across all environments (no extra containers to manage)
- **Customizable restart policies** that trigger based on health + context (e.g., "restart if unhealthy for 5 min AND memory usage < 80%")
- **Audit trail** of every auto-restart with full context (what was running, what caused the failure, what changed)

The difference is you're not managing a separate autoheal container or writing bash scripts to monitor Docker events. You get health-aware automation that understands the difference between "briefly unhealthy during deploy" and "actually broken and needs intervention."

💡

Now, fix Docker container health issues instantly, right from your IDE, with AI and [Last9 MCP](https://last9.io/mcp/). Bring real-time production context, logs, metrics, and traces, into your local environment to debug and fix failing health checks faster.  

## Common Causes of Unhealthy Containers

The container's up, but health checks are failing. Here's what to look for and what the symptoms often tell you.

### Crashing Applications

The most straightforward case: the app inside the container has crashed. Health checks that hit a port or check a process will fail because nothing is listening anymore.

**What you might see:**

In the health log:

```
Output: curl: (7) Failed to connect to localhost port 8080: Connection refused
```

**What to check:**

- Run `docker logs <container>` to check for stack traces or crash messages.
- Look for OOM kills (check `dmesg` if running locally, or container metrics if using a platform).
- If it's a language runtime crash (e.g., Node.js, Python), you'll usually find it in the logs.

### Broken Dependencies

Sometimes the app is fine, but a dependency isn't. Maybe your database isn't reachable. Maybe an API is timing out. From Docker's point of view, the container looks alive. But your app can't do its job.

**What you might see:**

```
Output: psql: could not connect to server: Connection refused
```

**What to check:**

- Is the dependent service running?
- Can you connect manually using `docker exec` + `curl` or `psql`?
- If the issue happens during boot, maybe the health check runs too early—add retries or a delay.

### Resource Exhaustion

The app hasn't crashed, but it's hanging. Not enough memory, high CPU, or a full disk can make the health check time out.

**What you might see:**

- Empty `Output` with a long delay
- Health check keeps failing with timeouts

**What to check:**

- Use `docker stats` to monitor memory and CPU usage in real-time.
- Run `df -h` or `free -m` inside the container to check the disk and memory.
- If the container is hitting limits, increase them or reduce the app load.

This is where centralized metrics become essential. Running `docker stats` on 40 containers across 8 hosts to find the one that's OOM-ing isn't practical. You need something that tracks resource usage alongside health status automatically.

### Misconfigured Health Checks

This one's easy to miss. The app is working, but the health check is pointing to the wrong place.

**What you might see:**

```
Output: curl: (7) Failed to connect to localhost port 8080: Connection refused
```

...when the app is running on port 3000.

**What to check:**

- Does the health check use the right port and path?
- Is the app listening on `localhost`, `0.0.0.0`, or something else?
- If it needs env vars to start up, make sure they're available during the health check.

## When DIY Health Monitoring Breaks Down

The commands above work great for debugging a single unhealthy container on your laptop. But here's where things get messy in production:

### The "Flapping Container" Problem

Your container toggles between healthy → unhealthy → healthy every few minutes. With `docker events`, you'll see the transitions, but:

- **No historical view** - You can't see patterns from yesterday or last week
- **No context** - Was it CPU spikes? Memory pressure? A downstream DB timeout?
- **No correlation** - Good luck connecting this to your application traces or error logs

You end up manually correlating timestamps across three different tools (logs, metrics, traces) to figure out what happened. By the time you've assembled the timeline, the container has flapped again, and you're back to square one.

### The "3 AM Debug" Problem

Your on-call gets paged. A container went unhealthy 4 times in the last hour. They run:

```
docker inspect --format "{{json .State.Health }}" container_name | jq
```

Great—they see `ExitCode: 1` and some timeout errors. But the health check has already passed again. The logs rolled over. The evidence is gone.

Now they're stuck trying to reproduce the issue, hoping it happens again while they're watching. This is the worst kind of debugging: reactive, evidence-free, and entirely dependent on luck.

### The "Which Container, Which Host?" Problem

You're running 47 containers across 12 hosts. One is flapping. Which one? You'll need to:

1. SSH into each host
2. Run `docker ps` to find the unhealthy one
3. Check its logs, inspect its health status
4. Correlate with your separate logging/metrics tools
5. Hope the issue reproduces while you're watching

This is where `docker events` stops being useful and becomes noise.

> Last9 helped us forget all the different observability tools and consolidate every dashboard into one single place.
> 
> Rahul Mahale, Principal DevOps Engineer, Circle

## How Last9 Makes Container Health Actionable

Last9 treats container health as a first-class signal, not an add-on. You get a clear history, automatic context, and predictable costs without juggling multiple tools.

**Historical Health You Can Actually Use**

Every health transition becomes a time series. You can spot patterns like:

- Fails during deployments
- Flaps at specific times
- Only breaks in one environment

Intermittent issues stop being guesswork.

**Automatic Context Around Failures**

When a container goes unhealthy, Last9 shows what was happening **right then**:

- Traces running inside the container
- Downstream timeouts (DB, cache, external API)
- Recent deploys, config changes, or traffic spikes

You get the full picture without hopping across SSH, dashboards, and logs.

**Faster Debugging From Your Terminal**

With [MCP](https://last9.io/mcp/), the same context lands in your terminal or IDE:

It returns:

- Last 50 health checks with output
- Correlated traces
- CPU/memory/I/O at failure time
- Recent deploys or config changes

No VPN, no multi-tab debugging.

**Smarter Alerts That Don’t Exhaust You**

Set rules that focus on real issues:

- Alert only if unhealthy for >5 minutes
- Alert if multiple containers fail together
- Suppress noise during deployment windows

Alerts become trustworthy instead of tiring.

**Built the Way Developers Expect**

Last9’s container health model is OTEL-native and built for scale. That means:

- **No vendor lock-in** — export data to Prometheus, Grafana, or warehouses
- **No trace sampling** — every failure is captured, even rare ones
- **Lower storage cost** — backend optimized for high-cardinality time series
- **Easy grouping** by service, version, environment, or custom labels

All environments—staging, production, DR—sit in one view, so you can tell whether a failure is code, config, or infra.

[Try it for free](https://app.last9.io/?utm_source=docker_status_unhealthy) today or [book sometime with us](https://last9.io/schedule-demo/?utm_source=docker_status_unhealthy) for a detailed walkthrough!

💡

Also, our [Discord community](https://discord.com/invite/W8gMppQC4b) is open. There's a dedicated channel where you can talk through details with other developers.

## FAQs

### Why is my container marked as unhealthy when it's still running?

Docker treats the container's `State` (running/stopped) separately from its health check results. If your health check fails repeatedly, Docker marks it `unhealthy`, even while the container stays up. Use `docker inspect` on `.State.Health` to see what's failing.

The reason for this separation is simple: a running process doesn't mean a working application. Your container might be running but completely unable to serve requests. Health checks give Docker (and you) a way to detect that difference.

### How do I see which health check command is breaking?

Check the full configuration with:

```
docker inspect container_name
```

Look for the `Healthcheck` section to see the exact `CMD` used. To test it manually, run it inside the container:

```
docker exec -it container_name sh -c '<healthcheck command>'
```

If the command works manually but fails in the health check, the issue is usually timing (runs too early) or environment (missing variables).

### Can I make Docker restart an unhealthy container automatically?

Not by default. You can either use tools like autoheal or modify your health check to force the container to exit, letting Docker's `--restart` policy take over. For example:

```
HEALTHCHECK … CMD curl -f … || kill -s 15 1
```

If you're in Docker Swarm or Kubernetes, the scheduler handles unhealthy containers automatically.

In production, this is where orchestration really pays off. Manually managing restart policies for dozens of containers gets messy fast. Orchestrators do this for you, and tools like Last9 show you when and why containers are being restarted.

### What's the difference between a health check timing out and failing?

A timeout means the command didn't complete within the defined timeout window. A failure means it finished but returned a non-zero exit code (usually `1`). Docker marks the check as failed in either case.

Timeouts usually indicate the app is hanging or overloaded. Failures usually indicate that the app returned an error. The distinction matters because the fix is different: timeouts need more resources or longer timeout settings, failures need code or config changes.

### How often should I run health checks?

It depends. A 30‑second `interval` is a good default. If rapid failure detection is important, go lower—but be cautious about adding too much load. Adjust based on how quickly or slowly your application usually responds.

For critical services that need fast failover, 10-15 seconds makes sense. For background workers or batch processors, 60 seconds is fine. The key is balancing detection speed with overhead.

### My health check passes manually, but fails in Docker—why?

The context is different. Docker runs the command inside the container, not from your host. That means different network settings, file paths, and environment variables. Always test with:

```
docker exec container_name sh -c '<healthcheck command>'
```

This is one of the most common gotchas. Your health check might assume environment variables that aren't set when Docker runs the check, or it might try to connect to `localhost` when the app is listening on a specific IP.

### Should every container have a health check?

Focus on containers where serving requests or data matters—like web apps, databases, message queues, etc. Not every short-lived or job container needs a health check. But for infrastructure services, yes, it's well worth it.

If a container failing would impact users or other services, add a health check. If it's just a one-off script that runs and exits, don't bother.

### How do I stop an unhealthy container automatically?

Docker won't shut it down on its own. You can either:

- Add logic to the health check itself to kill PID 1 on failure
- Run a small host-side script to restart all containers with `health=unhealthy`

For example:

```
docker ps -q -f health=unhealthy | xargs -r docker restart
```

This avoids giving containers full Docker socket access, which is a security risk. Run it as a cron job or systemd timer if you need automated restarts outside an orchestrator.