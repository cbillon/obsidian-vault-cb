---
link: https://docs.docker.com/compose/environment-variables/envvars-precedence/
site: Docker Documentation
date: 2024-02-09T10:46
excerpt: Scenario overview illustrating how environment variables are resolved in Compose
slurped: 2024-06-04T13:59:40.914Z
title: Environment variables precedence in Docker Compose
tags:
  - docker-compose
  - variable
  - precedence
---

When the same environment variable is set in multiple sources, Docker Compose follows a precedence rule to determine the value for that variable.

This page contains information on the level of precedence each method of setting environmental variables takes.

The order of precedence (highest to lowest) is as follows:

1. Set using [`docker compose run -e` in the CLI](https://docs.docker.com/compose/environment-variables/set-environment-variables/#set-environment-variables-with-docker-compose-run---env)
2. Substituted from your [shell](https://docs.docker.com/compose/environment-variables/set-environment-variables/#substitute-from-the-shell)
3. Set using just the [`environment` attribute in the Compose file](https://docs.docker.com/compose/environment-variables/set-environment-variables/#use-the-environment-attribute)
4. Use of the [`--env-file` argument](https://docs.docker.com/compose/environment-variables/set-environment-variables/#substitute-with---env-file) in the CLI
5. Use of the [`env_file` attribute](https://docs.docker.com/compose/environment-variables/set-environment-variables/#use-the-env_file-attribute) in the Compose file
6. Set using an [`.env` file](https://docs.docker.com/compose/environment-variables/set-environment-variables/#substitute-with-an-env-file) placed at base of your project directory
7. Set in a container image in the [ENV directive](https://docs.docker.com/reference/dockerfile/#env). Having any `ARG` or `ENV` setting in a `Dockerfile` evaluates only if there is no Docker Compose entry for `environment`, `env_file` or `run --env`.

In the example below, we set a different value for the same environment variable in an `.env` file and with the `environment` attribute in the Compose file:

The environment variable defined with the `environment` attribute takes precedence.

> Hard coding variables in container scripts
> 
> Executing a command within the container that unconditionally sets a variable value overrules any setting in your `compose.yml` file.
> 
> For example, in a NodeJS project, if you have a `package.json` entry for `scripts.start`, such as `NODE_ENV=test` `node server.js`, any value set for `NODE_ENV` in your Compose file, is ignored when running `npm run start` within the container.

The following table uses `TAG`, an environment variable defining the version for an image, as an example.
```
cat ./Docker/api/api.env
NODE_ENV=test

cat compose.yml
services:
  api:
    image: 'node:6-alpine'
    env_file:
     - ./Docker/api/api.env
    environment:
     - NODE_ENV=production
```
### [How the table works](#how-the-table-works)

Each column represents a context from where you can set a value, or substitute in a value for `TAG`.

The columns `Host OS environment` and `.env file` is listed only as an illustration lookup. In reality, they don't result in a variable in the container by itself.

Each row represents a combination of contexts where `TAG` is set, substituted, or both. The **Result** column indicates the final value for `TAG` in each scenario.

|#|`docker compose run --env`|`environment` attribute|`env_file` attribute|Image `ENV`|`Host OS` environment|`.env` file||Result|
|---|---|---|---|---|---|---|---|---|
|1|-|-|-|-|`TAG=1.4`|`TAG=1.3`||-|
|2|-|-|-|`TAG=1.5`|`TAG=1.4`|`TAG=1.3`||**`TAG=1.5`**|
|3|`TAG`|-|-|`TAG=1.5`|`TAG=1.4`|`TAG=1.3`||**`TAG=1.4`**|
|4|-|-|`TAG`|`TAG=1.5`|-|`TAG=1.3`||**`TAG=1.3`**|
|5|`TAG`|-|-|`TAG=1.5`|-|`TAG=1.3`||**`TAG=1.3`**|
|6|`TAG=1.8`|-|-|`TAG=1.5`|`TAG=1.4`|`TAG=1.3`||**`TAG=1.8`**|
|7|-|`TAG`|-|`TAG=1.5`|`TAG=1.4`|`TAG=1.3`||**`TAG=1.4`**|
|8|`TAG`|`TAG=1.7`|-|`TAG=1.5`|`TAG=1.4`|`TAG=1.3`||**`TAG=1.4`**|
|9|`TAG=1.8`|`TAG=1.7`|-|`TAG=1.5`|`TAG=1.4`|`TAG=1.3`||**`TAG=1.8`**|
|10|`TAG=1.8`|-|`TAG=1.6`|`TAG=1.5`|`TAG=1.4`|`TAG=1.3`||**`TAG=1.8`**|
|11|`TAG=1.8`|`TAG=1.7`|`TAG=1.6`|`TAG=1.5`|`TAG=1.4`|`TAG=1.3`||**`TAG=1.8`**|
|12|-|-|`TAG=1.6`|`TAG=1.5`|`TAG=1.4`|-||**`TAG=1.6`**|
|13|-|`TAG=1.7`|-|`TAG=1.5`|`TAG=1.4`|-||**`TAG=1.7`**|