---
link: https://last9.io/blog/docker-container-lifecycle/
byline: Faiz Shaikh
site: Last9
date: 2025-05-28T15:21
excerpt: Explore the key stages of the Docker container lifecycle and learn best
  practices to manage containers efficiently and reliably.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T15:16
title: "Docker Container Lifecycle: Key States and Best Practices"
---

You’ve probably run a lot of Docker containers, but do you know what happens behind the scenes? The Docker container lifecycle is the path a container follows from being created to running, stopping, and finally getting removed.

Understanding these steps helps you figure out why a container might not start or when to restart it instead of creating a new one.

## What Is the Docker Container Lifecycle

The [Docker container](https://last9.io/blog/docker-container-performance-metrics/) lifecycle is the path a container follows from the moment it’s created until it’s removed. Unlike virtual machines, which can pause and resume, containers move through a simpler set of states.

Every container typically goes through these main stages: created, running, paused, stopped, and removed. Some containers skip steps—like you can’t pause one that never started—but the general flow stays the same.

Why does this matter? When a container misbehaves, knowing which stage it’s stuck in helps you decide what to do next. It also sheds light on how each state affects performance and helps you plan better ways to deploy and manage containers.

💡

For more on managing container logs and preventing them from consuming excessive disk space, check out our guide on clearing [Docker logs](https://last9.io/blog/docker-clear-logs/)!

## The Five Critical States in Docker Container Lifecycle

### The Created State

Your container starts life in the created state. This happens when you run `docker create` or `docker run` (which combines create and start operations).

```
docker create --name my-nginx nginx:latest
```

At this point, Docker has allocated resources and prepared the container, but it's not running yet. The container exists but isn't consuming CPU or memory beyond basic overhead.

You can see containers in the created state by listing all containers, even the ones not running:

```
docker ps -a
```

This command shows every container on your system, along with their current status, including those that are “Created” but not yet running.

### The Running State

Your container moves into the running state once you start it. This can happen when you run `docker start` on a container that was created earlier, or when `docker run` finishes setting everything up and starts the container.

```
docker start my-nginx
# or
docker run -d --name my-nginx nginx:latest
```

While running, the container’s main process (PID 1) is active and doing its job. It can handle network requests, write data to volumes, and communicate with other containers. This is the state you want for any workload in production.

A container stays running as long as its main process keeps going. If that process crashes or finishes, the container automatically switches to the stopped state. That’s why understanding this helps you figure out why some containers stop unexpectedly.

### The Paused State

When a container is paused, all its processes stop without actually ending. The container stays in memory but doesn’t use any CPU.

```
docker pause my-nginx
```

This works using Linux’s cgroups freezer feature. The container keeps using memory since the processes are still there, but it won’t use any CPU while paused. When you’re ready to continue, unpause the container, and everything picks up right where it left off.

```
docker unpause my-nginx
```

This state is useful if you need to inspect what’s going on inside a container without letting it continue running. It also helps free up CPU temporarily while keeping the container’s current state intact.

💡

If you want to understand how pausing fits into the container lifecycle and when to use it effectively, check out [this guide](https://last9.io/blog/pausing-docker-containers/).

### The Stopped State

Containers enter the stopped state when their main process finishes, either because it completed normally or crashed. You can also stop a container manually:

```
docker stop my-nginx
```

When you stop a container, Docker first sends a **SIGTERM** signal to the main process. This lets the process shut down cleanly. If it doesn’t stop within a grace period (usually 10 seconds), Docker sends a **SIGKILL** signal to force it to quit immediately.

- **SIGTERM:** Tells the process to wrap up and exit properly
- **SIGKILL:** Forces the process to stop right away

Stopped containers keep any changes made to their filesystems and can be restarted later. This is different from removed containers—stopped ones still exist and can be brought back to running with all their data intact.

### The Removed State

The final stage is when a container is removed and no longer exists on your system. You do this with:

```
docker rm my-nginx
```

Once removed, all changes made inside the container’s filesystem are lost—unless you used volumes or bind mounts to keep data outside the container. Docker also frees up the container’s metadata, logs, and any resources it was using.

You can stop and remove a running container at once by forcing removal:

```
docker rm -f my-nginx
```

This sends a **SIGKILL** to all processes and removes the container immediately.

## Docker Container Lifecycle Commands You Need to Know

Working with Docker containers gets easier once you know the right commands for each stage. Here are the key ones:

**Start a container and attach to see its output:**

```
docker start -a my-app
```

Useful for troubleshooting or monitoring because you see the container’s real-time output.

**Start a container that’s already created:**

```
docker start my-app
```

Starts a container that exists but isn’t running yet.

**Create and start at the same time:**

```
docker run -d --name my-app nginx:latest
```

This is the most common command—it creates and immediately starts the container in detached mode.

**Create without starting:**

```
docker create --name my-app nginx:latest
```

Use this when you want to prepare a container ahead of time or customize settings before running it.

Remember, `docker run` combines creation and starting in one step, which makes it quick and easy. Knowing the difference helps when you need more control over how and when containers start.

## How to Monitor and Inspect Docker Containers

Here are some key commands to check what’s going on with your containers:

- `"State.Status"` to see if the container is running, stopped, or paused
- `"State.ExitCode"` to check why a container stopped
- `"HostConfig.Memory"` to view memory limits
- `"Config.Env"` to see environment variables set inside the container

**Watch container events live:**

```
docker events --filter container=my-app
```

Shows real-time events like start, stop, pause, and unpause.

**Get detailed info about a container:**

```
docker inspect my-app
```

Returns a JSON output with detailed info. For example, you might look at:

**Show all containers, including stopped ones:**

```
docker ps -a
```

Lists every container on your system, regardless of state.

**Show running containers:**

```
docker ps
```

Lists all containers currently running.

**Example:** To check if a container stopped unexpectedly, run:

```
docker inspect my-app --format='{{.State.Status}} {{.State.ExitCode}}'
```

This will output something like:

```
exited 137
```

Meaning the container stopped with exit code 137 (often an out-of-memory kill).

## Pause, Unpause, and Check Container Status

**Check if a container is paused:**

```
docker ps --filter status=paused
```

Shows all containers currently paused.

**Resume a paused container:**

```
docker unpause my-app
```

This resumes the container from where it was paused.

**Pause all processes in a container:**

```
docker pause my-app
```

This freezes all running processes inside the container.

**Example:**  
If you want to pause a container to temporarily free up CPU but keep its state, run:

```
docker pause my-app
```

Later, when you want it running again:

```
docker unpause my-app
```

You can confirm the pause by running:

```
docker ps --filter status=paused
```

If the container appears here, it’s paused.

💡

For practical tips on tracking container resource usage, see [our guide](https://last9.io/blog/container-resource-monitoring-with-docker-stats/) on using `docker stats`!

## Best Practices for Stopping and Cleaning Up Containers

#### Graceful Stop: Ask the Container to Shut Down Cleanly

```
docker stop my-app
```

This sends a SIGTERM signal, letting the container finish ongoing work before shutting down. It waits up to 10 seconds by default.

_Example:_ Use this when you want to avoid data loss or corruption during shutdown.

#### Force Stop: Immediately Kill the Container

```
docker kill my-app
```

Sends a SIGKILL signal that stops the container right away without waiting.

_Example:_ Useful when a container isn’t responding to a graceful stop.

#### Stop with Custom Timeout: Control How Long to Wait

```
docker stop --time=30 my-app
```

Waits 30 seconds for the container to stop gracefully before forcing termination.

_Example:_ Increase the timeout if your app needs more time to shut down cleanly.

#### Remove a Stopped Container: Clean Up Old Containers

```
docker rm my-app
```

Deletes the container but leaves data in volumes intact.

_Example:_ Remove containers that you no longer need to free up resources.

#### Stop and Remove in One Step: Quick Cleanup

```
docker rm -f my-app
```

Forces a running container to stop and removes it immediately.

_Example:_ Use this to quickly get rid of a misbehaving container.

#### Remove All Stopped Containers: Bulk Cleanup

```
docker container prune
```

Deletes all stopped containers to free up disk space.

_Example:_ Run this regularly to keep your environment clean.

💡

To quickly find specific information in your container logs, see our guide on filtering [Docker logs with grep](https://last9.io/blog/how-to-filter-docker-logs-with-grep/)!

## Key Differences Between Docker Commands

#### Docker Create vs Docker Start vs Docker Run

**`docker run`** combines create and start in one step. It can also pull images if they aren’t on your system:

```
docker run -d --name my-app nginx:latest
```

**`docker start`** runs a container that already exists (either created or stopped):

```
docker start my-stopped-container
```

**`docker create`** prepares a container from an image but doesn’t start it. Use this if you want to set up several containers first and then start them all at once:

```
docker create --name web-server nginx:latest
docker create --name database postgres:13
# Start both containers together
docker start web-server database
```

### Docker Pause vs Docker Stop

**`docker stop`** sends SIGTERM to terminate the main process gracefully, followed by SIGKILL if needed. Memory is freed, and restarting means starting fresh:

```
docker stop my-app
docker start my-app
```

**`docker pause`** freezes all processes inside the container using SIGSTOP. Memory remains used, but the CPU stops. When unpaused, the container continues from the same spot:

```
docker pause my-app
docker unpause my-app
```

### Docker Remove vs Docker Kill

**`docker kill`** sends a signal to the main process but doesn’t remove the container:

```
docker kill my-app               # Sends SIGKILL
docker kill --signal=SIGTERM my-app  # Sends SIGTERM explicitly
```

**`docker rm`** deletes container metadata and filesystem (only works on stopped containers unless forced):

```
docker rm my-app          # Remove a stopped container
docker rm -f my-app       # Force remove a running container
```

### How Container States Differ from Container Health

It’s important not to mix up container states with container health. A container can be running but still be unhealthy if the app inside isn’t working properly.

Docker lets you set up health checks that monitor the app separately from the container’s state. For example:

```
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost/ || exit 1
```

This command tells Docker to regularly check if the app responds correctly. If it doesn’t, Docker marks the container as unhealthy, even if it’s still running.

Tracking both the container lifecycle and these health checks gives you a clearer view of how your system is really doing. For good observability, keep an eye on both.

💡

To explore how Docker logging drivers collect, route, and store container logs—and which one makes sense for your monitoring setup, check out our detailed guide: [How Docker Logging Drivers Work](https://last9.io/blog/docker-logging-drivers/).

## Docker Container Lifecycle Best Practices for Reliable Applications

### Handle Shutdown Signals to Avoid Data Loss

Your app should listen for **SIGTERM** signals and clean up resources before exiting. This helps containers stop gracefully and prevents corruption.

> For example, catch SIGTERM in your code to close database connections or flush caches before shutting down.

### Use Docker Features to Manage Container Lifecycle

Take advantage of Docker’s built-in options to handle common lifecycle needs:

- Add an init process to properly forward signals and manage child processes.
- Set a stop timeout to control how long Docker waits during shutdown.
- Define restart policies to automatically recover from failures.

### Make Container Startup and Shutdown Faster

Speed up state transitions by keeping your images small, adding health checks to track readiness, and caching dependencies. Proper signal handling also helps your app shut down quickly.

### Ensure Your App Cleans Up Resources on Exit

Write cleanup scripts and tell Docker which signal to use for stopping your app. This avoids leftover resources and keeps your environment clean.

💡

Fix production Docker issues instantly—right from your IDE, with [Last9 MCP](https://last9.io/mcp/). Get real-time production context—logs, metrics, and traces—directly in your local setup to speed up troubleshooting and code fixes.

## Troubleshooting Common Docker Container Lifecycle Problems

Dealing with containers that misbehave is part of the daily grind. Here’s how to approach the most common lifecycle issues, step by step.

### When Containers Won’t Start

If your container sits stuck in the created state and won’t move forward, start by checking its logs for any error messages:

```
docker logs my-app
```

Next, inspect the container’s configuration to spot misconfigurations or resource limits:

```
docker inspect my-app
```

Also, make sure the ports your container needs are free—conflicts here can silently block startup:

```
netstat -tlnp | grep :8080
```

Finally, verify the image and command syntax is correct by running a test command inside a fresh container:

```
docker run --rm nginx:latest nginx -t
```

Common causes for startup failure include incorrect commands, missing dependencies, port conflicts, or permission issues.

### Containers That Keep Restarting

If you see your container cycling through restarts quickly, this usually means the main process is crashing repeatedly. Check the restart policy to confirm how Docker is configured to handle failures:

```
docker inspect my-app | grep -i restart
```

At the same time, keep an eye on real-time container events:

```
docker events --filter container=my-app
```

And review the latest logs to catch error messages or exceptions from your app:

```
docker logs --tail=50 my-app
```

If your app crashes frequently, consider adjusting your restart policy to include backoff delays, which can prevent rapid crash loops from overwhelming your system.

### Containers That Won’t Stop

Sometimes `docker stop` doesn’t work as expected because the app inside isn’t responding properly to shutdown signals. If the stop command hangs, try forcing it:

```
docker kill my-app
```

You can also look inside the container to see what processes are still running:

```
docker exec my-app ps aux
```

This usually points to missing or incorrect signal handling in your application. Make sure your app listens for **SIGTERM** and cleans up before exiting.

### High Resource Usage

Even containers that aren’t running can sometimes hog resources due to zombie processes or leftover files. To monitor resource usage, start with real-time stats:

```
docker stats
```

Check disk usage for containers and images:

```
docker system df
```

Look specifically at any memory limits or resource constraints set on the container:

```
docker inspect my-app | grep -i memory
```

If resource usage looks off for stopped containers, investigate cleanup scripts or lingering processes that might not be exiting correctly.

💡

For a deeper understanding of capturing and analyzing Docker build logs, check out our guide on [Docker Build Logs](https://last9.io/blog/docker-build-logs/).

## Performance Implications of Docker Container Lifecycle Management

Every step in the Docker container lifecycle comes with its own performance cost.

**Container Creation** involves setting up the filesystem, allocating resources, and configuring the network. If you create containers ahead of time (leaving them in the created state), you can cut down startup delays later, but this also uses more memory since those containers are reserved but not yet running.

**Starting a Container** means launching processes and loading dependencies. To speed this up, use smaller base images and cache dependencies wherever possible. This reduces the time your app takes to become ready.

**Pausing a Container** freezes its processes, keeping memory allocated but stopping CPU use. Since the container isn’t doing any work, pausing should be used sparingly, mainly for debugging or temporarily freeing CPU resources without losing container state.

**Stopping a Container** triggers a graceful shutdown and resource cleanup. Your application should be designed to shut down quickly to avoid holding up deployments or causing delays during scaling.

When building deployment strategies or setting auto-scaling policies, weigh these performance trade-offs carefully. The right balance depends on your app’s needs and the limits of your infrastructure.

## Wrapping Up

Understanding every step in the Docker container lifecycle is key to keeping your apps reliable and efficient. But knowing what’s happening isn’t enough—monitoring needs to cover the whole lifecycle to help you catch problems early.

[Last9](https://last9.io/) is built to give you clear visibility into Docker container environments, where metrics, logs, and traces from many containers can quickly add up. By supporting OpenTelemetry and Prometheus, we help you collect and connect this telemetry data throughout every stage of the Docker container lifecycle.

[Get started for free](https://app.last9.io/) today, or if you'd like to know more about platform capabilities, [book a time with us](https://last9.io/schedule-demo/)!

## FAQs

**What happens to data when a container moves through different lifecycle states?**

Data in the container's filesystem layer persists through created, running, paused, and stopped states. Only removal permanently deletes this data. Data in mounted volumes or bind mounts persists through all states, including removal.

**Can I restart a container that's in the stopped state?**

Yes, you can restart stopped containers using `docker start`. The container will return to the running state with all its previous filesystem changes intact, but any in-memory state is lost.

**What's the difference between pausing and stopping a container?**

Pausing freezes all processes using SIGSTOP without terminating them—they resume exactly where they left off. Stopping terminates the main process with SIGTERM (then SIGKILL), requiring a fresh start when restarted.

**How long does Docker wait before force-killing a container?**

Docker waits 10 seconds by default after sending SIGTERM before sending SIGKILL. You can customize this with the `--time` flag: `docker stop --time=30 my-container`.

**Why would I use docker create instead of docker run?**

Use `docker create` when you want to prepare containers but not start them immediately. This is useful for batch operations, scheduled tasks, or when you need to modify container settings before starting.

**Do health checks affect the container lifecycle states?**

Health checks don't directly change container states, but they provide additional information about container wellness. Orchestrators like Kubernetes use health check results to make lifecycle decisions like restarting unhealthy containers.

**Can I change a container's lifecycle state while it's running?**

Yes, you can transition containers between states using Docker commands. A running container can be paused, stopped, or removed. A paused container can be unpause or stopped. However, you cannot directly move from stopped to paused, you must start the container first.

**What signals does Docker use during the container lifecycle?**

Docker primarily uses SIGTERM for graceful shutdowns and SIGKILL for forced termination. During pause operations, it uses the Linux cgroups freezer subsystem rather than traditional signals. Understanding these signals helps you implement proper shutdown handling in your applications.