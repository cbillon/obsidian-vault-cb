---
tags:
  - docker-compose
  - variable
  - environment
---

_.env_ is for variables that are parsed in to the _docker-compose.yml_ interpreter, not for the container. For variables to be set in the container, you will need to specify a _.env_ file in your _docker-compose.yml_ using `env_file: .env.vars` or whatever your file name is that contains the variables. It's a confusing antipattern to use `env_file: .env` which works but basically uses the variables both for the interpreter and the container

The latest Docker Compose allows you to access environment variables from your compose file. So you can source your environment variables, then run Compose like so:

```yaml
set -a
source .my-env
docker-compose up -d
```

For example, assume we have the following `.my-env` file:

```yaml
POSTGRES_VERSION=14
```

(or pass them via command-line arguments when calling `docker-compose`, like so: `POSTGRES_VERSION=14 docker-compose up -d`)

Then you can reference the variables in `docker-compose.yml` using a `${VARIABLE}` syntax, like so:

```yaml
db:
  image: "postgres:${POSTGRES_VERSION}"
```

