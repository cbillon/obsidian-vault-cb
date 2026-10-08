---
tags:
  - map
  - docker_compose
---
[[Docker compose Environment variable]]
_.env_ is for variables that are parsed in to the _docker-compose.yml_ interpreter, not for the container
 For variables to be set in the container, you will need to specify a _.env_ file in your _docker-compose.yml_ using env_file:
 It's a confusing antipattern to use `env_file: .env` which works but basically uses the 
 variables both for the interpreter and the container
The order of precedence (highest to lowest) is as follows:

1. Set using [`docker compose run -e` in the CLI](https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/#set-environment-variables-with-docker-compose-run---env).
2. Set with either the `environment` or `env_file` attribute but with the value interpolated from your [shell](https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/#substitute-from-the-shell) or an environment file. (either your default [`.env` file](https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/#env-file), or with the [`--env-file` argument](https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/#substitute-with---env-file) in the CLI).
3. Set using just the [`environment` attribute](https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/#use-the-environment-attribute) in the Compose file.
4. Use of the [`env_file` attribute](https://docs.docker.com/compose/how-tos/environment-variables/set-environment-variables/#use-the-env_file-attribute) in the Compose file.
5. Set in a container image in the [ENV directive](https://docs.docker.com/reference/dockerfile/#env). Having any `ARG` or `ENV` setting in a `Dockerfile` evaluates only if there is no Docker Compose entry for `environment`, `env_file` or `run --env`.

[[7 Docker Compose Tricks to Level Up Your Development Workflow]]
1. Use Profiles to Toggle Services Conditionally
    Define a profile in your `docker-compose.yml` under a service’s `profiles` key. Then, use the `--profile` flag to activate it. Services without a profile run by default, but those with profiles only start when explicitly called.
2. Override Environment Variables with .env Files
3. Optimize Builds with Cache and Context
4. Manage Dependencies with Healthchecks
    Add a `healthcheck` to a service and use `depends_on` with a `condition: service_healthy` to control startup order. Docker waits until the healthcheck passes.
5. Simplify Multi-Container Logs with Custom Names
6. Use Named Volumes for Persistent Data
7. Extend Compose Files for Modularity
8. 
