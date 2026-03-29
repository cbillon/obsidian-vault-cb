---
link: https://last9.io/blog/how-to-configure-dockers-shared-memory-size-dev-shm/
byline: Faiz Shaikh
site: Last9
date: 2025-06-25T14:15
excerpt: Understand how to configure Docker’s /dev/shm size to avoid memory
  errors in Chrome, PostgreSQL, and other high-memory workloads.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T15:09
title: How to Configure Docker’s Shared Memory Size (/dev/shm)
---

Your Node.js app runs fine on your machine. But inside Docker? You start getting weird crashes—`ENOSPC: no space left on device`. Chrome headless tests fail out of nowhere. PostgreSQL throws shared memory errors under load.

The problem?

It’s probably `/dev/shm`, the shared memory volume Docker sets up by default. Most containers get just 64MB of space here. That’s not much, especially for apps that rely on multiple processes, big in-memory buffers, or things like Chrome or PostgreSQL that depend on fast, internal communication.

In this blog, we’ll look at what `/dev/shm` is how Docker handles it, and how to configure it properly so your containers don’t fall over when traffic spikes or tests run in parallel.

When multiple processes inside a container need to talk to each other or share data quickly, they often use shared memory. This lives at `/dev/shm`—a RAM-backed temporary filesystem that’s much faster than writing to disk. It’s commonly used for:

- Inter-process communication (IPC)
- In-memory caching between processes
- Buffer sharing in databases or browsers
- Fork-heavy workloads like test runners or worker pools

If your app uses Chrome in headless mode, runs PostgreSQL, or forks worker threads (like Node.js clusters or ML jobs), there’s a good chance it depends on `/dev/shm` without you realizing it.

### Docker’s Default Shared Memory Limit

Here’s where problems begin: Docker gives each container its own `/dev/shm` volume, but only 64MB by default.

That might be fine for basic apps or simple APIs. But it’s rarely enough for real-world workloads that run browsers, databases, or data-heavy tasks.

You’re likely to hit this limit if you're running:

- Headless browsers (Chrome, Chromium via Puppeteer or Selenium)
- PostgreSQL or other databases that use shared memory
- ML frameworks that exchange data between processes
- Applications with clustering, background jobs, or IPC

To check the available shared memory in a running container:

```
docker exec <container_id> df -h /dev/shm
```

And 64MB might be okay for small setups, but under real load, it’s usually the first thing to break.

## Three Practical Approaches to Configuring Docker SHM Size

Docker containers come with a default shared memory size of 64MB. That’s often not enough for applications like headless browsers, databases, or ML frameworks. Here’s how you can configure it properly.

#### 1. CLI: Use the [`--shm-size`](https://docs.docker.com/engine/reference/run/#shm-size-shared-memory-size) flag

You can increase the shared memory size at runtime using the `--shm-size` option:

```
docker run --shm-size=1g my-app
```

Units can be specified in bytes (`b`), kilobytes (`k`), megabytes (`m`), or gigabytes (`g`), depending on your needs.

#### 2. Docker Compose: Use `shm_size`

When working with Docker Compose, you can define the shared memory size directly in your service configuration:

```
version: '3.8'
services:
  web:
    image: my-app
    shm_size: 1gb
```

This is useful for applications that consistently require more shared memory under load.

#### 3. Dockerfile: Document the Requirement

You can’t set `shm_size` directly in a Dockerfile, but you can document it clearly for anyone running the image:

```
FROM node:18

# Requires --shm-size=1g for running headless Chrome tests
COPY . /app
WORKDIR /app
RUN npm install
```

This ensures developers and CI pipelines are aware of the requirement upfront.

Shared memory failures in Docker often manifest as misleading "no space left on device" errors. These aren’t related to disk space; they’re almost always caused by Docker’s default `/dev/shm` size being too small.

Here's how this shows up in common applications and how to address it.

#### Chrome and Puppeteer Crashes

Chrome's multi-process architecture depends heavily on shared memory. Inside Docker, if `/dev/shm` is under-provisioned, Chrome will fail to start. A typical error looks like:

```
Error: Failed to launch the browser process!
[0125/084815.775316:FATAL:shared_memory_posix.cc(157)]
Creating shared memory in /dev/shm/... failed: No space left on device
```

This failure is deterministic under load or when launching multiple instances. The fix is to increase the shared memory allocation:

```
docker run --shm-size=1g my-puppeteer-app
```

Headless testing frameworks like Puppeteer and Selenium often require 512MB to 1GB of shared memory per container, depending on concurrency.

#### PostgreSQL: Misleading "No Space Left on Device" Errors

PostgreSQL uses shared memory for critical internals—shared buffers, locks, and inter-process coordination. When the container runs out of shared memory, you’ll see errors such as:

```
pq: could not resize shared memory segment: No space left on device
FATAL: could not create shared memory segment: No space left on device
```

These aren't disk space errors. They're signals that `/dev/shm` is full.

Update your `docker-compose.yml` to allocate more shared memory:

```
services:
  postgres:
    image: postgres:15
    shm_size: 1gb
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
```

Also, make sure PostgreSQL’s internal memory settings are tuned appropriately. Example `postgresql.conf` values:

```
# Shared buffer pool; 25-40% of total memory is a typical starting point
shared_buffers = 1GB

# Memory per sort/hash operation
work_mem = 4MB

# Used by VACUUM, CREATE INDEX, etc.
maintenance_work_mem = 64MB
```

Insufficient shared memory will prevent PostgreSQL from launching under certain configurations, especially with higher `shared_buffers`.

#### Node.js Clustered Applications

Node.js apps using `cluster` or `worker_threads` can also hit shared memory bottlenecks when spawning multiple processes. These errors are less obvious—workers simply fail to start or behave unpredictably under high concurrency.

Example pattern:

```
const cluster = require('cluster');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }
}
```

If shared memory is exhausted, forks may silently fail or crash during IPC. This is more likely to occur in apps doing large message passing or heavy inter-process communication.

Provisioning 512MB to 1GB of `/dev/shm` is often enough for Node.js workloads, but always test under expected concurrency levels.

Understanding how much shared memory your containers use is critical, especially when dealing with performance-sensitive applications like databases, browsers, or worker-heavy Node.js apps.

Here's how to monitor `/dev/shm` usage both at the system and application levels.

#### Container-Level Monitoring: Real-Time SHM Checks

Use these commands to inspect and monitor shared memory usage inside a running Docker container:

```
# Check overall shared memory usage
docker exec <container> df -h /dev/shm

# View shared memory segments (relevant for PostgreSQL)
docker exec <container> ipcs -m

# Live monitoring every 1 second
docker exec <container> watch -n 1 'df -h /dev/shm'
```

##### PostgreSQL-Specific Monitoring

If you're running PostgreSQL inside Docker, use SQL-level introspection for better insights:

```
# PostgreSQL 13+ shared memory allocations
docker exec <container> psql -U postgres -c "SELECT * FROM pg_shmem_allocations;"

# For PostgreSQL 12 and earlier
docker exec <container> psql -U postgres -c "SELECT buffers_checkpoint, buffers_clean, buffers_backend, buffers_alloc FROM pg_stat_bgwriter;"
```

These queries help you correlate in-Postgres memory behavior with shared memory limits at the container level.

### Application-Level Metrics and Observability

For production workloads, container-level metrics aren’t enough. You need observability across your infrastructure to catch early signs of memory pressure.

[Last9](https://last9.io/) is built to handle high-cardinality observability from the ground up. It connects memory metrics across containers, logs, and traces using native OpenTelemetry and Prometheus support.

To track shared memory inside your app, add metrics like:

```
const fs = require('fs');

function getShmUsage() {
  try {
    const statfs = fs.statSync('/dev/shm');
    return {
      total: statfs.size,
      used: statfs.size - statfs.free
    };
  } catch (err) {
    console.error('Unable to read /dev/shm stats:', err);
  }
}
```

You can export this to Prometheus, StatsD, or any custom telemetry backend.

### Security Considerations in Multi-Tenant Containers

While each container gets its own `/dev/shm` namespace, shared memory is still RAM-backed. In multi-tenant systems or CI runners, it’s worth considering:

- Avoid storing sensitive data in shared memory
- Monitor `/dev/shm` to detect overuse or leaks
- Right-size allocations to prevent abuse or overcommitment

### Alternatives to Default `/dev/shm`

If the default Docker behavior doesn’t fit your use case, you have two main alternatives:

#### 1. Mount Host `/dev/shm` for Full Access

```
docker run -v /dev/shm:/dev/shm my-app
```

This bypasses the container’s memory limit by giving access to the host’s shared memory. It’s useful in dev/test setups, but it reduces container isolation.

#### 2. Use `tmpfs` Mounts for Custom In-Memory Storage

```
docker run --tmpfs /app/temp:rw,noexec,nosuid,size=1g my-app
```

This creates a 1GB in-memory `tmpfs` mount at `/app/temp`, without affecting `/dev/shm`. Useful when you need isolated in-memory storage for caching, temporary files, or IPC that doesn’t depend on the system’s shared memory segment.

## Set SHM Size Across Docker, Kubernetes, and the Host

Once your container workloads move beyond local testing, you’ll likely need tighter control over shared memory, not just inside containers, but across orchestration layers and the host itself.

### Setting Shared Memory Size in Kubernetes

Unlike Docker, Kubernetes doesn’t support the `--shm-size` flag. Instead, shared memory is configured using an `emptyDir` volume backed by RAM.

Here’s how to mount it correctly:

```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: my-app
    volumeMounts:
    - name: shm
      mountPath: /dev/shm
  volumes:
  - name: shm
    emptyDir:
      medium: Memory
      sizeLimit: 1Gi
```

This replaces the default `/dev/shm` with a RAM-backed 1Gi volume, ensuring compatibility with apps like PostgreSQL, Chrome, and any workload that depends on IPC.

### How Much Shared Memory Is Enough?

There’s no one-size-fits-all answer, but these baseline recommendations can help:

|Workload Type|Suggested `/dev/shm` Size|
|---|---|
|Basic web applications|64MB (default is usually fine)|
|Chrome / Puppeteer|1GB minimum; 2GB for parallel tests|
|PostgreSQL - small|256MB|
|PostgreSQL - production|1–2GB depending on cache and connection pool sizes|
|ML pipelines / workers|1GB+ based on model and batch size|

If you're allocating 2GB of shared memory and still seeing failures, the problem might not be size—it might be memory leaks, inefficient IPC, or runaway forks.

### System-Level SHM Configuration on the Host

Even with the right container or pod configs, your host’s kernel settings can become a bottleneck.

Check current limits:

```
sysctl -a | grep kernel.shm
```

Sample output:

```
kernel.shmmax = 68719476736   # Max size per shared memory segment
kernel.shmall = 4294967296    # Total shared memory pages allowed
kernel.shmmni = 4096          # Number of SHM segments
```

If you’re seeing errors or silent failures even with `shm_size` configured, increase these limits in `/etc/sysctl.conf`:

```
kernel.shmmax = 68719476736   # 64GB
kernel.shmall = 4294967296    # ~16TB across all segments
```

Apply changes:

```
sudo sysctl -p
```

This is especially relevant on Kubernetes nodes or CI runners where multiple pods may compete for shared memory.

## Docker SHM Size: Troubleshooting and Practical Fixes

Shared memory issues in Docker don’t always show up as clean error messages. Sometimes they surface as performance degradation, random crashes, or subtle instability under load.

Here’s how to systematically debug and resolve them.

Start with these steps when investigating shared memory problems:

- **Check container logs** for errors like `No space left on device`
- **Review application patterns**—does it fork processes, use shared memory IPC, or run memory-intensive operations?
- **Test incrementally** by increasing `--shm-size` (e.g., 256MB → 512MB → 1GB)
- **Rethink architecture** if shared memory demands are unusually high or unpredictable (e.g., offload to external caches or IPC systems)

**Inspect current `/dev/shm` usage** during peak load:

```
docker exec <container> df -h /dev/shm
```

### Common Error Patterns You Should Know

#### PostgreSQL

When PostgreSQL runs out of shared memory, it fails to allocate required segments:

```
FATAL: could not create shared memory segment: No space left on device
DETAIL: Failed system call was shmget(key=..., size=..., 03600).
```

This is often triggered by higher `shared_buffers` or concurrent connections exceeding what Docker’s `/dev/shm` can support.

#### Chrome / Puppeteer

Headless Chrome inside Docker crashes when it can't allocate shared memory:

```
[ERROR:shared_memory_posix.cc(157)] Creating shared memory in /dev/shm/... failed: No space left on device
```

This usually means the container needs more than the default 64MB to 1GB is a good starting point.

#### Node.js (Cluster or Child Processes)

When Node.js apps attempt to fork processes under tight memory conditions:

```
Error: spawn ENOSPC
    at ChildProcess.spawn (internal/child_process.js:394:11)
```

This often happens when using `cluster` or `child_process.fork()` in high-concurrency scenarios. Shared memory pressure causes system calls to fail silently or throw low-level `ENOSPC` errors.

💡

Now, fix Docker shared memory issues instantly, right from your IDE, with AI and [Last9 MCP](https://last9.io/mcp/). Bring real-time production context, logs, metrics, and traces into your local environment to troubleshoot and fix `/dev/shm` problems faster.

## Performance Symptoms of SHM Constraints

Even without explicit crashes, low shared memory can lead to degraded performance:

- **Slower database queries** due to reduced caching
- **Increased disk I/O** when buffer pools can’t use shared memory effectively
- **Higher CPU usage** from more frequent memory copying or IPC overhead
- **Random crashes** under load or when concurrency spikes

If your app feels unstable only at scale or fails tests intermittently in CI, shared memory may be the bottleneck.

## Wrapping Up

Shared memory issues tend to show up under production load—when tests run in parallel, queries spike, or processes compete for limited RAM. Docker’s default 64MB `/dev/shm` size often isn’t enough for Chrome-based testing, PostgreSQL, or ML workloads using multiprocessing.

Fixing it usually means increasing `shm_size` and monitoring how that memory is used. You can use tools like `df -h` or `ipcs` inside the container.

For production environments, [Last9](https://last9.io/) helps track shared memory usage over time, alongside CPU, memory, and I/O metrics, so you can identify pressure points with data instead of assumptions.

💡

And if you’re still working through a tricky shared memory issue, jump into our [Discord](https://discord.com/invite/W8gMppQC4b), we’ve a channel where you can talk to engineers about their share setups, configs, and what’s worked for their workloads.

## FAQs

**Q: What is `/dev/shm` used for in Docker containers?  
A:** `/dev/shm` is a RAM-backed temporary filesystem used for inter-process communication (IPC) and shared memory. Applications like Chrome, PostgreSQL, and multiprocessing libraries rely on it for fast, in-memory data sharing.

**Q: How do I monitor `/dev/shm` usage in real time inside a container?  
A:** Run:

```
docker exec <container> watch -n 1 'df -h /dev/shm'
```

This shows usage and available space. For PostgreSQL, you can use SQL queries to monitor memory allocation patterns.

**Q: Does increasing `shm_size` improve performance?  
A:** It can, but only if the application is constrained by the current shared memory limit. Examples include improved stability for headless browsers, reduced disk I/O for databases, and smoother worker coordination in parallel workloads.

**Q: Can multiple containers share the same `/dev/shm`?  
A:** Not by default. Each container gets an isolated `/dev/shm` mount. However, you can mount the host’s shared memory into multiple containers using:

```
-v /dev/shm:/dev/shm
```

This breaks isolation and is generally discouraged unless necessary for specific IPC use cases.

**Q: How do I set `shm_size` in Docker Compose?  
A:** Use the `shm_size` key under the service definition:

```
services:
  app:
    image: my-app
    shm_size: 1gb
```

**Q: What's the Kubernetes equivalent of `shm_size`?  
A:** Use an `emptyDir` volume with `medium: Memory`, mounted to `/dev/shm`:

```
volumes:
  - name: shm
    emptyDir:
      medium: Memory
      sizeLimit: 1Gi
```

**Q: Is `/dev/shm` cleared when a container stops?  
A:** Yes. Because it’s a temporary in-memory filesystem scoped to the container, its contents are removed when the container stops or is deleted.

**Q: Can I use `tmpfs` as a replacement for `/dev/shm`?  
A:** Yes, for specific use cases. A `tmpfs` mount provides an in-memory filesystem at any path:

```
docker run --tmpfs /my/tmp:rw,size=1g my-app
```

However, this doesn't affect `/dev/shm` unless explicitly mounted there.