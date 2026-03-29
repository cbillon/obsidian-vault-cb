---
link: https://last9.io/blog/docker-compose-restart/
byline: Preeti Dewani
site: Last9
date: 2025-12-02T11:38
excerpt: Master docker compose restart with examples for restarting single services, all containers, and applying config changes. Includes restart vs down vs recreate comparison and troubleshooting tips.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T14:45
title: "docker compose restart: Commands, Options & Common Fixes"
tags:
  - docker_compose
---

When you're managing containerized applications with Docker Compose, there will be times when you need to restart services.

Maybe you're applying changes, fixing errors, or just refreshing containers. Whatever the reason, the `docker compose restart` command is a handy tool to have in your toolkit.

In this guide, we'll talk about everything you need to know about this command, including its uses, options, and best practices.

**What is docker compose restart?**  
The `docker compose restart` command is designed to stop and start the services defined in your `docker-compose.yml` file.

It’s commonly used when you want to restart the services running in your Docker Compose setup without shutting down the entire Docker environment. This command allows you to refresh or apply updates to your containers with minimal effort.

[

A Complete Guide to Using the Grok Debugger | Last9

Learn how to use the Grok Debugger effectively for log parsing, with practical tips, debugging techniques, and pattern optimization.

![](https://last9.ghost.io/content/images/icon/icon-192x192-200.png)Last9

![](https://last9.ghost.io/content/images/thumbnail/The-Complete-Guide-to-Grok-Debugger_-Understanding--Implementation--and-Optimization.webp)

](https://last9.io/blog/grok-debugger/)

### **Why Should You Use docker compose restart?**

You might wonder why you need a command like a docker compose restart. After all, you can stop and start containers individually. However, docker compose restart provides several key advantages:

1. **Time-Saving**: Instead of manually stopping and restarting each container, the docker compose restart command does both tasks in one step.
2. **Dependency Management**: Docker Compose ensures that services restart in the proper order, respecting the dependencies defined in your configuration. This is especially important when some services depend on others to function correctly.
3. **Simplified Maintenance**: Restarting services is a quick way to apply changes, clear errors, or refresh services after troubleshooting.

### Basic Usage of `docker compose restart`

The most basic usage `docker compose restart` is simple:

```
docker compose restart
```

This command restarts all services defined in your `docker-compose.yml` file. It stops the running containers and starts them again in the correct order, based on their dependencies.

### Restarting a Specific Service

In some cases, you may want to restart a single service rather than all services. For example, if you made changes to your database service, you may only need to restart that service.

To restart a specific service, specify the service name:

```
docker compose restart <service_name>
```

For instance, if you have a web service named `web`, use the following command to restart just the web service:

```
docker compose restart web
```

This command will stop and start the `web` service without affecting the other services in the Docker Compose setup.

[

Crontab Logs: Track, Debug, and Optimize Your Cron Jobs | Last9

Crontab logs help you keep your cron jobs in check. Learn how to track, debug, and optimize your cron jobs with crontab logs.

![](https://last9.ghost.io/content/images/icon/icon-192x192-201.png)Last9Last9

![](https://last9.ghost.io/content/images/thumbnail/Crontab_logs.webp)

](https://last9.io/blog/crontab-logs/)

## Advanced Options with `docker compose restart`

Docker Compose offers a few options that can modify the behavior of the `restart` command. Here are the most useful ones:

#### 1. `--timeout`

By default, Docker Compose waits 10 seconds before forcefully stopping a service when using the `restart` command. If you need more time for your services to gracefully shut down, you can adjust the timeout duration.

For example, to set a timeout of 30 seconds:

```
docker compose restart --timeout 30
```

This is useful when you have services that take longer to shut down or when you're working with databases that need extra time for cleanup.

#### 2. `--no-deps`

The `--no-deps` option allows you to restart a service without restarting its dependencies. This can save time if you only need to restart one service and don't want to disturb other services.

For example, if you want to restart only the `web` service without restarting its dependencies like the `database` service, you can use:

```
docker compose restart --no-deps web
```

This is especially helpful in larger applications where certain services are relatively independent, and you don’t want to restart everything.

[

Morgan npm and Its Role in Node.js | Last9

Morgan npm simplifies HTTP request logging in Node.js, making it easier to monitor and debug your applications with customizable formats.

![](https://last9.ghost.io/content/images/icon/icon-192x192-202.png)Last9Last9

![](https://last9.ghost.io/content/images/thumbnail/Introduction-to-Morgan-and-Its-Role-in-Node.js-Logging.webp)

](https://last9.io/blog/morgan-npm-and-its-role-in-node-js/)

#### 3. `-t` (Timeout as shorthand)

The `-t` flag is shorthand for the `--timeout` option, allowing you to set the timeout for stopping a service. For instance, to set a timeout of 15 seconds:

```
docker compose restart -t 15
```

This provides the same functionality as `--timeout` but with a shorter syntax.

## When Should You Use `docker compose restart`?

Here are some common scenarios where `docker compose restart` is useful:

### 1. **After Updating Configuration Files**

If you've updated your `docker-compose.yml` file (for instance, changing environment variables or modifying volumes), restarting the services is often necessary for the changes to take effect. Instead of manually stopping and starting each container, use:

```
docker compose restart
```

This ensures the changes are applied seamlessly.

### 2. **Fixing Resource Usage or Errors**

If your containers are running into issues like high CPU or memory usage, or if a service stops responding, a restart can often fix the problem. This is a quick way to reset the state of your services without needing to stop and start individual containers.

### 3. **Applying Changes to Docker Images**

If you've built new Docker images for your services, you might want to restart the services to ensure the new images are used. If the services are already running, you’ll need to restart them to pull in the updated image.

[

Logging Errors in Go with ZeroLog: A Simple Guide | Last9

Learn how to log errors efficiently in Go using ZeroLog with best practices like structured logging, context-rich messages, and error-level filtering.

![](https://last9.ghost.io/content/images/icon/icon-192x192-203.png)Last9Last9

![](https://last9.ghost.io/content/images/thumbnail/Golang-Logging-8.webp)

](https://last9.io/blog/logging-errors-in-go-with-zerolog/)

### 4. **Refreshing Services After Troubleshooting**

After identifying and fixing issues in your containers, such as bugs or network issues, restarting the services ensures that the problem is resolved and the containers are running fresh.

### 5. **Clearing Temporary Errors**

Sometimes, services in Docker Compose can get into a weird state due to networking issues or temporary problems. Restarting the services clears out those errors and resets the containers, giving you a fresh start.

## Best Practices for Using `docker compose restart`

Here are a few tips for using `docker compose restart` effectively:

- **Use `docker compose restart` for Minor Changes**: If you’ve made small changes like configuration tweaks or environment variable adjustments, `docker compose restart` is the way to go. It’s faster and less disruptive than tearing down and recreating containers.
- **Restart Dependent Services Together**: If services depend on each other, consider restarting all of them together to avoid potential issues. For instance, if your `web` service relies on the `database` service, restart both to ensure they’re in sync.
- **Automate Restarting with `docker-compose` Commands**: If you’re dealing with frequent changes or regular maintenance, you can automate restarts with cron jobs or scripts. For example, you could schedule a restart of your containers at off-peak hours to ensure uptime without downtime during busy periods.
- **Check Logs After Restarting**: After performing a restart, use `docker compose logs` to ensure that the services are running smoothly. This can help you verify that everything is functioning properly post-restart.

[

The Only Kubectl Commands Cheat Sheet You’ll Ever Need | Last9

Here’s your go-to kubectl commands cheat sheet! Jump into Kubernetes management with these handy commands and make your life easier.

![](https://last9.ghost.io/content/images/icon/icon-192x192-204.png)Last9Last9

![](https://last9.ghost.io/content/images/thumbnail/Kubectl-commands-cheatsheet-1.webp)

](https://last9.io/blog/kubectl-commands-cheatsheet/)

## Troubleshooting `docker compose restart`

If `docker compose restart` isn’t working as expected, here are a few things to check:

- **Service State**: Make sure the services are actually running before trying to restart them. If a service is already stopped, `docker compose restart` won’t do anything.
- **Configuration Issues**: If there’s an error in your `docker-compose.yml` file, such as an invalid environment variable or incorrect syntax, Docker Compose may fail to restart the services. Always check for errors in your configuration before restarting.
- **Docker Daemon Health**: If Docker itself is encountering issues, you may need to restart the Docker daemon. Check the status of Docker with:

```
systemctl status docker
```

Restart Docker if necessary:

```
sudo systemctl restart docker
```

## **Health Checks and Monitoring**

### 1. Health Check Configuration

```
version: '3.8'
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]  # Command to check service health
      interval: 30s    # How often to perform the health check
      timeout: 10s     # Maximum time to wait for the check to complete
      retries: 3       # Number of consecutive failures needed to mark as unhealthy
      start_period: 40s # Initial grace period for service startup
```

**This configuration:**

- Uses `curl` to make an HTTP request to `localhost`.
- Checks every 30 seconds.
- Allows 10 seconds for each check to complete.
- Requires 3 failures to mark as unhealthy.
- Provides a 40-second startup grace period.

### 2. Restart Policies

```
services:
  web:
    restart: always           # Container will always restart
    # or
    restart: unless-stopped   # Restart unless explicitly stopped
    # or
    restart: on-failure      # Only restart if container exits with error
```

**Key differences:**

- `always`: Restarts in all scenarios, even after manual stop.
- `unless-stopped`: Doesn't restart if manually stopped.
- `on-failure`: Only restarts on non-zero exit codes.

[

Understanding Kubernetes Metrics Server: Your Go-to Guide | Last9

Learn how the Kubernetes Metrics Server helps monitor resource usage like CPU and memory, ensuring smooth cluster performance and scalability.

![](https://last9.ghost.io/content/images/icon/icon-192x192-205.png)Last9

![](https://last9.ghost.io/content/images/thumbnail/Understanding-Kubernetes-Metrics-Server_-Your-Go-to-Guide-3.webp)

](https://last9.io/blog/kubernetes-metrics-server/)

### 3. Network Inspection Commands

```
# Check network connectivity
docker network inspect bridge
# Shows detailed information about the bridge network

# List networks
docker network ls
# Lists all Docker networks

# Check container networking
docker inspect <container_id> -f '{{json .NetworkSettings}}'
# Shows container's network configuration
```

**These commands help troubleshoot network issues:**

- `network inspect`: Shows network details and connected containers.
- `network ls`: Lists available networks.
- `inspect`: Shows container-specific network settings.

### 4. Volume Management

```
services:
  db:
    volumes:
      - db_data:/var/lib/postgresql/data  # Named volume mapping
    restart: always
volumes:
  db_data:
    driver: local  # Uses local storage driver
```

**This configuration:**

- Creates a named volume `db_data`.
- Mounts it to PostgreSQL's data directory.
- Persists data across container restarts.
- Uses local storage driver for volume management.

### 5. CI/CD Testing Configuration

```
version: '3.8'
services:
  test:
    image: test-runner
    command: ["./test-restarts.sh"]
    depends_on:
      web:
        condition: service_healthy
```

**This setup:**

- Uses a `test-runner` container.
- Executes `test-restarts.sh` script.
- Waits for the `web` service to be healthy.
- Ensures dependencies are ready before testing.

[

kubectl exec: Commands, Examples, and Best Practices | Last9

Learn essential commands, examples, and best practices for using kubectl exec to troubleshoot and manage your Kubernetes applications.

![](https://last9.ghost.io/content/images/icon/icon-192x192-206.png)Last9Last9

![](https://last9.ghost.io/content/images/thumbnail/kubectl-exec_-Commands--Examples--and-Best-Practices.webp)

](https://last9.io/blog/kubectl-exec-commands-examples-and-best-practices/)

### 6. Environment-Specific Configuration

```
version: '3.8'
services:
  web:
    environment:
      - ENVIRONMENT=${ENV:-development}  # Default to development if ENV not set
    restart: ${RESTART_POLICY:-unless-stopped}  # Default restart policy
```

**This configuration:**

- Uses environment variables with defaults.
- `${ENV:-development}`: Uses 'development' if `ENV` not set.
- `${RESTART_POLICY:-unless-stopped}`: Default restart policy.
- Allows runtime configuration changes.

### 7. Scaling Configuration

```
version: '3.8'
services:
  web:
    deploy:
      replicas: 3  # Number of container instances
      restart_policy:
        condition: any  # Restart under any condition
```

**This defines:**

- Service scaling with 3 replicas.
- Automatic restart for any condition.
- Load distribution across replicas.
- Container redundancy.

### 8. Recovery Script

```
#!/bin/bash
if docker compose ps | grep -q "Exit"; then
    docker compose restart
    echo "Services restarted due to failure"
fi
```

**This script:**

- Checks for containers in 'Exit' state.
- Automatically restarts failed services.
- Logs restart actions.
- Can be scheduled via cron.

### **9. Advanced Health Check**

```
version: '3.8'
services:
  api:
    healthcheck:
      test: >
        wget --no-verbose --tries=1 --spider http://localhost:8080/health ||
        exit 1
      interval: 20s
      timeout: 5s
      retries: 5
      start_period: 60s
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

**This configuration:**

- Uses `wget` for health checking.
- Sets specific timing parameters.
- Defines restart behavior.
- Implements failure handling.

[

PromQL Cheat Sheet: Must-Know PromQL Queries | Last9

This cheat sheet provides practical guidance for diagnosing issues and understanding trends.

![](https://last9.ghost.io/content/images/icon/icon-192x192-207.png)Last9

![](https://last9.ghost.io/content/images/thumbnail/PromQL-Cheatsheet.webp)

](https://last9.io/blog/promql-cheat-sheet/)

### 10. Multi-Service Dependency

```
version: '3.8'
services:
  web:
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
  
  db:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
  
  cache:
    restart: always
```

**This setup:**

- Defines service dependencies.
- Implements different health checks.
- Coordinates service startup.
- Manages restart policies.

## Conclusion

The `docker compose restart` command is a powerful tool for managing your containerized applications. It makes it easier to restart services, apply changes, and troubleshoot issues without hassle.

Mastering how to use `docker compose restart` will boost the efficiency of your Docker workflows and help keep your services running smoothly.

Don’t forget to explore options like `--timeout`, `--no-deps`, and service-specific commands to customize the command for your needs.

Happy Docker Compose managing!

🤝

If you’d like to explore this further, join our [Discord community](https://discord.com/invite/W8gMppQC4b)! We have a dedicated channel where developers discuss specific use cases.

## FAQs

**What does docker compose restart do?**  
`docker compose restart` stops and then starts all the services defined in your docker-compose.yml file. It’s a quick way to refresh your containers and apply changes without manually stopping and starting each service.

**How do I restart a specific service in Docker Compose?**  
To restart a specific service, use the following command:

```
docker compose restart <service_name>
```

For example, to restart the web service, run:

```
docker compose restart web
```

This will only restart the specified service, not the entire setup.

**What’s the difference between docker compose restart and docker compose up?**  
`docker compose restart` simply stops and starts the running containers.  
`docker compose up` (with the --build option) not only restarts containers but also rebuilds them if there are any changes to the Dockerfile or the docker-compose.yml file. It can also start services that aren't running.

**Can I restart my Docker Compose services without affecting their dependencies?**  
Yes! You can use the `--no-deps` flag to restart a specific service without restarting its dependencies:

```
docker compose restart --no-deps <service_name>
```

This is helpful if you only want to restart one service and not impact others that are linked to it.

**How can I adjust the timeout for stopping a service during a restart?**  
By default, Docker Compose waits 10 seconds before forcefully stopping a service. You can adjust this timeout by using the `--timeout` or `-t` option. For example, to set a timeout of 30 seconds:

```
docker compose restart --timeout 30
```

This gives the services more time to gracefully shut down.

**Why does docker compose restart sometimes fail to restart services?**  
Common reasons for failure include:

- **Errors in docker-compose.yml**: Double-check your configuration file for syntax errors or invalid settings.
- **Service dependencies**: If a service fails to start due to an issue with a dependency, restarting might not work. Ensure that all dependencies are properly configured.
- **Docker Daemon Issues**: If the Docker daemon is encountering problems, the restart might not go through. Check Docker’s status using `systemctl status docker` and restart it if needed.

**Can I restart Docker Compose services automatically on a schedule?**  
Yes, you can automate restarts by using cron jobs or task schedulers. For example, you could set a cron job to run `docker compose restart` at specific intervals to ensure your services are refreshed regularly.

**Is docker compose restart the same as restarting Docker itself?**  
No, `docker compose restart` only restarts the containers defined in your docker-compose.yml. If you need to restart Docker’s entire daemon, you would need to run:

```
sudo systemctl restart docker
```

This is useful if Docker is experiencing issues that affect the entire system, not just specific containers.

**What happens to my data when I restart containers with docker compose restart?**  
If your services use volumes to persist data, the data will not be lost when you restart the containers. However, if your services are not using persistent storage, any unsaved data may be lost during the restart. Always ensure that important data is stored in volumes or external databases.