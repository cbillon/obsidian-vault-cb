---
tags:
  - docker_compose
  - tips
  - ressources
---
## Docker compose

[Nick Lanetkis depends on docker compose](https://nickjanetakis.com/blog/optional-depends-on-with-docker-compose-v2-20-2)
I’ve written about [Docker Compose profiles and optional dependent services](https://nickjanetakis.com/blog/docker-tip-94-docker-compose-v2-and-profiles-are-the-best-thing-ever) in the past but a few minor things have changed since then when Docker Compose v2 accidentally created a backwards incompatible change in v2.19.

### [#](https://nickjanetakis.com/blog/optional-depends-on-with-docker-compose-v2-20-2#how-did-we-arrive-here)How Did We Arrive Here?

Here’s a brief timeline:

- **April 2022**: Docker Compose v2 is considered GA (General Availability)
    - Optional `depends_on` was supported implicitly by accident
- **June 2023**: Docker Compose v2.19 is released
    - A breaking change requires all `depends_on` services to be set and available
- **July 2023**: Docker Compose v2.20.2 is released
    - A new `required` property is added to `depends_on` to make it explicitly optional

Back in September 2022 [I opened an issue](https://github.com/compose-spec/compose-spec/issues/274) to augment the Docker Compose specification because there was talk of this feature being “fixed” without an official workaround. That would have introduced a breaking change that folks have been using for a year.

This wasn’t supposed to be fixed until a solution was provided by Docker Compose but I think things got lost over time since that proposal was about a year old.

After bringing it up with the Docker Compose development team (one of the perks of being a [Docker Captain](https://www.docker.com/captains/nick-janetakis/) btw!), they were understanding of the problem and quickly worked on a solution. Not even a few weeks later they shipped v2.20.2 with the new feature.

I wish they made `required: false` be the default value so it’s a true backwards compatible enhancement because you do need to adjust your Docker Compose file to set `required: false` if you want the old behavior.

However, that property only exists for v2.20.2+ which means users must change their YAML configuration and be using at least this version. If you try to use this new property with an older version of Docker Compose it will fail.

It’s not the end of the world, I’m happy we arrived at a solution. Especially since all of this will blow over and be the new norm within a few months.

[Docker Desktop v4.22.0](https://docs.docker.com/desktop/release-notes/#4220) supports this new version of Docker Compose v2 and it’s been available without DD for a while now too. If you’re reading this post and can upgrade versions you can start using this feature no matter how you’re running Docker!

### [#](https://nickjanetakis.com/blog/optional-depends-on-with-docker-compose-v2-20-2#how-does-it-work)How Does It Work?

Let’s say you only want to run `postgres` and `redis` in development because you use managed services in production or perhaps you have `esbuild` and `tailwind` watchers running in development but not in production.

That’s 2 common use cases of wanting optional `depends_on`, so now let’s say you have `web` and `worker` services defined with `depends_on` attached to them:

##### Before v2.19 (with the “bug”):

```yml
depends_on:
  - "postgres"
  - "redis"
```

If you set `COMPOSE_PROFILES="web,worker"`, Docker Compose would silently allow your project to be upped without `postgres` and `redis`. This is essentially optional dependencies since it worked with or without them set to run.

##### After and including v2.19 (without the “bug”):

```yml
depends_on:
  - "postgres"
  - "redis"
```

If you set `COMPOSE_PROFILES="web,worker"`, Docker Compose would fail to start and say you need to run `postgres` and `redis`. If you want them to always be required you can leave things how they are and you’re good to go.

##### After and including v2.20.2 (without the “bug”):

```yml
  depends_on:
    postgres:
      condition: "service_started"
      required: false
    redis:
      condition: "service_started"
      required: false
```

Now you can set `required` to be `true` or `false` based on your preference. You can still use the shorthand array syntax if you want them to be required by default.


## Wait for

This is a CLI tool that waits for an event before continuing. Simples. But it does it cross platform and as a single dependency that can be downloaded into your container or environment.

Typically, you would use this to wait on another resource (such as an HTTP resource) to become available before continuing - or timeout and exit with an error.

At the moment, you can wait for a few different kinds of thing. They are:

- HTTP or HTTPS success response or any expected response following regular expressions
- TCP or GRPC connection
- DNS IP resolve address change

[Wait for dnnrly](https://github.com/dnnrly/wait-for)

## Wait for Nick Janetkis

[Nick Janetkis](https://github.com/nickjj/wait-until)

### Use Cases

[](https://github.com/nickjj/wait-until#use-cases)

I mainly use this in my CI pipelines to wait until a Dockerized database is ready to accept connections to a specific database as a specific user.

For example, you might have something like this in your CI set up:

```shell
docker-compose up -d

make db-reset
make lint
make test
```

The make commands don't really matter, but the idea is you can't really reset your database or run your tests that depend on your database being set up before your database is available.

When you first start the official PostgreSQL or MySQL Docker containers, it will take about 5 to 10 seconds for your initial database user / database to be created based on environment variables you pass into the container.

This isn't a problem in development because you'd probably run everything in the foreground and manually wait until your DB is ready before interacting with your project. But in a CI environment, it's a problem because it will blow up without waiting.

Technically you can use this script to wait for any command to run successfully. There is nothing about it that's limited to databases or CI environments. There are [usage examples](https://github.com/nickjj/wait-until#usage-examples) included in the README.

### Why not sleep X seconds?

[](https://github.com/nickjj/wait-until#why-not-sleep-x-seconds)

You could, but that's kind of brittle because depending on where you run the command, it might take a different amount of time to be ready since the spin up time of a command is based on the hardware specs of the machine.

If you put a long sleep like 30 seconds to be safe then you're waiting potentially tens of extra seconds on every CI run.

### Can't you do `docker-compose up -d && make db-reset`?

[](https://github.com/nickjj/wait-until#cant-you-do-docker-compose-up--d--make-db-reset)

Technically yes, but you're going to be at the mercy of race conditions. That isn't going to ensure that your database is "really" ready before control is passed over to the 2nd command in the chain.

I ran that 10 times manually and it failed 9 out of 10 times. That's not really suitable to have running in CI.

### What about the `wait-for-it` script?

[](https://github.com/nickjj/wait-until#what-about-the-wait-for-it-script)

The [wait-for-it](https://github.com/vishnubob/wait-for-it) script is popular, but in my opinion it's not suitable for the CI use case and it solves a different use case.

For starters, it waits for a TCP port to be available. Technically that port can be available shortly after your database service is up, but your DB is not really ready for connections because the DB user and database itself hasn't been created yet.

That could potentially cause your CI run to fail.

Another issue with that script is that it expects you to access a TCP port, which means you'll need to run it inside of your app's container or publish your database's port back to your Docker host if you wanted to use it for CI.

If you're using that script to make `depends_on` more robust, then yes you'll have that in your app's Docker image but personally I don't do that. IMO that problem should be solved at the web framework level. Once the DB itself is created with the proper user credentials your app's code should be robust enough to retry your database connection until it works or gives up.

As for publishing the port back to the host so you can run the script from within your CI server, that comes with 2 problems:

1. Now you're responsible for having to install the `psql` or `mysql` CLI directly within your CI environment.
2. You'll need to publish your database's port in your `docker-compose.yml` file to access your DB's TCP port.

Personally I don't like either of those things. The second one means allowing the public internet to attempt to login to your database so having that in my real compose file by default isn't happening and I didn't want to have to rig up some `sed` in-line replacement script to do the port publish specifically for CI (basically editing the file only for CI).

And that's why I decided to make this script.

## Installation

[](https://github.com/nickjj/wait-until#installation)

Typically you would install this script into a base Docker image that you use as your CI environment, but we'll cover a few installation methods here.

The snippets below are set to use the latest release. If you're living on the edge you can always replace the tag name with `master`, but I don't recommend that since it's not guaranteed to be stable and may change in the future.

### Installing it directly on your system

[](https://github.com/nickjj/wait-until#installing-it-directly-on-your-system)

This allows you to run `wait-until` from any directory. If this is on your personal dev box or something like that, feel free to adjust this to put it into a local `bin/` directory that's on your path for your user.

```shell
sudo curl \
  -L https://raw.githubusercontent.com/nickjj/wait-until/v0.3.0/wait-until \
  -o /usr/local/bin/wait-until && sudo chmod +x /usr/local/bin/wait-until
```

### Installing it remotely within a Docker image (useful for CI images)

[](https://github.com/nickjj/wait-until#installing-it-remotely-within-a-docker-image-useful-for-ci-images)

You would add these instructions somewhere near the bottom of your `Dockefile`.

```dockerfile
ADD https://raw.githubusercontent.com/nickjj/wait-until/v0.3.0/wait-until /usr/local/bin
RUN chmod +x /usr/local/bin/wait-until
```

This is one of those times where [using `ADD` instead of `COPY`](https://nickjanetakis.com/blog/docker-tip-2-the-difference-between-copy-and-add-in-a-dockerile) comes in handy.

### Installing it locally within a Docker image (useful for CI images)

[](https://github.com/nickjj/wait-until#installing-it-locally-within-a-docker-image-useful-for-ci-images)

If you're going for a super optimized base CI image and you already have your own scripts being copied to `/usr/local/bin` and you want to piggy back off your `COPY usr/local/bin /usr/local/bin` instruction you may want to just copy / paste this script directly into your image.

This 1 liner will download the latest release into the current directory of your system. Then you can put it anywhere you want.

```shell
curl \
  -L https://raw.githubusercontent.com/nickjj/wait-until/v0.3.0/wait-until \
  -o wait-until && chmod +x wait-until
```

## Usage Examples

[](https://github.com/nickjj/wait-until#usage-examples)

You can use this for any command but here's a couple of database related examples.

### A quick note about .env files and database passwords

[](https://github.com/nickjj/wait-until#a-quick-note-about-env-files-and-database-passwords)

In all examples, we're sourcing an `.env` file because typically that's where you would define your database environment variables that the official Docker images expect to be set.

Your `.env` file is usually ignored from version control which is a good idea. What I do is commit a safe `.env.example` file to version control and then `mv .env.example .env` as part of my CI pipeline. This file would have development credentials that work and are safe to commit as defaults.

But if you're connecting to a remote database with sensitive credentials, then you may want to skip that source step and read those environment variables directly in from your CI provider's secure environment variable settings.

### Waiting for PostgreSQL

[](https://github.com/nickjj/wait-until#waiting-for-postgresql)

This expects that you've named your Docker Compose PostgreSQL service `postgres` if you plan to copy / paste what's below.

```shell
docker-compose up -d

source .env
wait-until "docker-compose exec -T -e PGPASSWORD=${POSTGRES_PASSWORD} postgres psql -U ${POSTGRES_USER} ${POSTGRES_USER} -c 'select 1'"

# TODO: Run your DB reset commands, test suite, etc.
```

### Waiting for MySQL

[](https://github.com/nickjj/wait-until#waiting-for-mysql)

This expects that you've named your Docker Compose MySQL service `mysql` if you plan to copy / paste what's below.

```shell
docker-compose up -d

source .env
wait-until "docker-compose exec -T -e MYSQL_PWD=${MYSQL_ROOT_PASSWORD} mysql mysql -D ${MYSQL_DATABASE} -e 'select 1'"

# TODO: Run your DB reset commands, test suite, etc.
```

#### What's with the `-T` flag?

[](https://github.com/nickjj/wait-until#whats-with-the--t-flag)

By default `docker-compose exec` configures a TTY which is fine and dandy in development since you're running a terminal emulator. This has nice advantages like being able to have an interactive prompt and see colors.

Within most (maybe all?) CI environments you'll get a `the input device is not a TTY` error without having `-T` set when using this script. That flag disables pseudo-tty allocation.

#### The failed connection output is noisy, make it stop!

[](https://github.com/nickjj/wait-until#the-failed-connection-output-is-noisy-make-it-stop)

You can always adjust your command to redirect everything to `/dev/null`. For example, at the end of your command (outside of the quotes) you can add `> /dev/null 2>&1` to eliminate all output (both STDOUT and STDERR).

Personally I like seeing all of the output in CI. It's especially handy for being able to debug the command you're waiting on. For example I initially redirected everything to `/dev/null` by default but it made debugging that `-T` issue a pain in the butt.

## Configuration Options

[](https://github.com/nickjj/wait-until#configuration-options)

You can add an optional second argument to customize the timeout in seconds. By default it's set to 60 seconds.

For example you can run `wait-until "grep" 3` to try it out. The `grep` command will fail to run since it requires at least 1 argument. As configured `wait-until` will try for 3 seconds until it gives up.

## Wait-for-it

[Wait-for-it](https://github.com/vishnubob/wait-for-it)

`wait-for-it.sh` is a pure bash script that will wait on the availability of a host and TCP port. It is useful for synchronizing the spin-up of interdependent services, such as linked docker containers. Since it is a pure bash script, it does not have any external dependencies.

## Usage

[](https://github.com/vishnubob/wait-for-it#usage)

```
wait-for-it.sh host:port [-s] [-t timeout] [-- command args]
-h HOST | --host=HOST       Host or IP under test
-p PORT | --port=PORT       TCP port under test
                            Alternatively, you specify the host and port as host:port
-s | --strict               Only execute subcommand if the test succeeds
-q | --quiet                Don't output any status messages
-t TIMEOUT | --timeout=TIMEOUT
                            Timeout in seconds, zero for no timeout
-- COMMAND ARGS             Execute command with args after the test finishes
```

## Examples

[](https://github.com/vishnubob/wait-for-it#examples)

For example, let's test to see if we can access port 80 on `www.google.com`, and if it is available, echo the message `google is up`.

```
$ ./wait-for-it.sh www.google.com:80 -- echo "google is up"
wait-for-it.sh: waiting 15 seconds for www.google.com:80
wait-for-it.sh: www.google.com:80 is available after 0 seconds
google is up
```

You can set your own timeout with the `-t` or `--timeout=` option. Setting the timeout value to 0 will disable the timeout:

```
$ ./wait-for-it.sh -t 0 www.google.com:80 -- echo "google is up"
wait-for-it.sh: waiting for www.google.com:80 without a timeout
wait-for-it.sh: www.google.com:80 is available after 0 seconds
google is up
```

The subcommand will be executed regardless if the service is up or not. If you wish to execute the subcommand only if the service is up, add the `--strict` argument. In this example, we will test port 81 on `www.google.com` which will fail:

```
$ ./wait-for-it.sh www.google.com:81 --timeout=1 --strict -- echo "google is up"
wait-for-it.sh: waiting 1 seconds for www.google.com:81
wait-for-it.sh: timeout occurred after waiting 1 seconds for www.google.com:81
wait-for-it.sh: strict mode, refusing to execute subprocess
```

If you don't want to execute a subcommand, leave off the `--` argument. This way, you can test the exit condition of `wait-for-it.sh` in your own scripts, and determine how to proceed:

```
$ ./wait-for-it.sh www.google.com:80
wait-for-it.sh: waiting 15 seconds for www.google.com:80
wait-for-it.sh: www.google.com:80 is available after 0 seconds
$ echo $?
0
$ ./wait-for-it.sh www.google.com:81
wait-for-it.sh: waiting 15 seconds for www.google.com:81
wait-for-it.sh: timeout occurred after waiting 15 seconds for www.google.com:81
$ echo $?
124
```