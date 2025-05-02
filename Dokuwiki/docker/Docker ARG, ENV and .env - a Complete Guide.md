---
link: https://vsupalov.com/docker-arg-env-variable-guide/
site: vsupalov.com
date: 2017-07-13T00:00
excerpt: Stop struggling to build Docker images and configuring your dockerized apps. This is the complete guide to build-time arguments, environment variables and docker-compose templating.
slurped: 2024-06-11T09:03:37.266Z
title: Docker ARG, ENV and .env - a Complete Guide
tags:
  - docker
  - arg
  - env
---

```
Error: Docker image build failed.
```

![](https://vsupalov.com/images/docker-env-vars/search/1.png)

Building Docker images and configuring your dockerized apps doesn’t have to be a try-fail-repeat Google extravaganza. This article will help you work with Docker ARG, ENV, env_file and .env files with confidence.

> The only prerequisite: make sure that you’re comfortable with the [basics](https://vsupalov.com/6-docker-basics/) of Docker.

Read on and you will understand how to configure your Docker images and dockerized apps with ease - with the power of Docker build-time variables, environment variables and docker-compose templating.

## An Overview

Alright, let’s get started with the details. The guide is split up into the following topics:

- [Frequent Misconceptions](https://vsupalov.com/docker-arg-env-variable-guide/#frequent-misconceptions)
- [The Dot-Env File (.env)](https://vsupalov.com/docker-arg-env-variable-guide/#the-dot-env-file-env)
- [ARG and ENV](https://vsupalov.com/docker-arg-env-variable-guide/#arg-and-env)
- [Setting ARG Values](https://vsupalov.com/docker-arg-env-variable-guide/#setting-arg-values)
- [Setting ENV Values](https://vsupalov.com/docker-arg-env-variable-guide/#setting-env-values)
- [Overriding ENV Values](https://vsupalov.com/docker-arg-env-variable-guide/#overriding-env-values)

Feel free to jump right to the one you need right now. All of them together will help you get a good understanding of ENV and ARG in your Dockerfiles.

> Just starting out with Docker? Check out [this article about 6 Docker basics you should know](https://vsupalov.com/6-docker-basics/).

## Frequent Misconceptions

This is a long, in-depth read. Let’s start with something you can use right now, without having to read the whole thing!

Here’s a list of **quick takeaways**:

- Some tools look for a file called **.env** and try to read from it. `docker-compose` is one of these, but there are others. Not all of these tools use the entries to set environment variables though.
- `docker-compose` uses value from the .env file, to set values for a pre-processing step for docker-compose.yml files. Dollar-notation variables like $HI are substituted **in the docker-compose.yml** (or whatever file you point docker-compose to). This does not set environment variables directly.
- **ARG** is only available during the build of a Docker image (RUN etc), not after the image is created and containers are started from it (ENTRYPOINT, CMD). You can use ARG values to set ENV values to [work around](https://vsupalov.com/docker-build-time-env-values/) that.
- **ENV** values are available to containers, but also to the commands of your Dockerfile which run during an image build, starting with the line where they are introduced.
- If you set an environment variable in a **Dockerfile instruction** using bash (RUN export VARI=5 && …) it will **not persist**. The next command will not be able to access this value. (You can use [this approach](https://vsupalov.com/set-dynamic-environment-variable-during-docker-image-build/) to set dynamic environment variables if you really need to.)
- An **env_file**, is a term for a file containing lines about environment variables which are usable by the Docker CLI. It is a convenient way to pass many environment variables to a single command. The name sounds like a _.env_ file, but they are not the same.
- Setting ARG and ENV values leaves **traces** in the Docker image. Don’t use them for secrets which are not meant to stick around (well, you kinda can with [multi-stage builds](https://vsupalov.com/build-docker-image-clone-private-repo-ssh-key/)).

## The Dot-Env File (.env)

First of all, the `.env` file is just a file. However, _some_ tools look into it for various reasons by default, if it is in the current directory. `docker-compose` is one of these tools.

If you have a file named [.env](https://docs.docker.com/compose/environment-variables/#the-env-file) in your project, it’s only used to put values into the active `docker-compose.yml` file (or the ones you point towards) by `docker-compose`.

This mechanism has not set any ENV or ARG values in Docker by itself. It’s exclusively a docker-compose.yml thing.

The values in the `.env` file are written in the following notation:

```
VARIABLE_NAME=some value
OTHER_VARIABLE_NAME=some other value, like 5
```

> Notice how in a bash script, you would need to write something like `export VARIABLE_NAME=something`? Slightly different syntax here.

Those key-value pairs, are used to substitute dollar-notation variables in the `docker-compose.yml` file. It’s a pre-processing step, and the resulting temporary file is used by `docker-compose`. This is a nice way to avoid hard-coding values!

You can also use this to set the values for environment variables, _from within_ `docker-compose`. That does not happen automatically.

Here is an example docker-compose.yml file, which relies on values provided from a `.env` file to set environment variables of a future container:

```
version: '3'

services:
  example:
    image: ubuntu
    environment:
      - env_var_name=${VARIABLE_NAME} # here it is!
```

> Hint: When working with an .env file, you can debug your docker-compose.yml files with a single command. Type `docker-compose config`. This way you’ll see how the docker-compose.yml file content looks _after_ the substitution step, without running anything else.

One more thing about `docker-compose` and `.env` files! It is kind of a **gotcha** you should know about. Environment variables on your host **will override** the values specified in your .env file. Yes, you read that right. Read more [here](https://vsupalov.com/override-docker-compose-dot-env/), or check out the [docs on env var precedence for compose](https://docs.docker.com/compose/environment-variables/envvars-precedence/).

Now that `.env` is out of the way, let’s look at what’s going on in Dockerfiles.

## ARG and ENV

When using Docker, we distinguish between two different types of variables - ARG and ENV. They differ in when in an image-container lifecycle the values are available.

Here is a simplified overview of ARG and ENV availabilities. Starting with building a Docker image from a Dockerfile, until a container runs. ARG values are not usable from inside running containers.

We will go detail about each of these right afterwards.

 ![](https://vsupalov.com/images/docker-env-vars/docker_environment_build_args.png)An overview of ARG and ENV availability.

> Note: in the image, there is a rectangle around Dockerfile. This is a bit confusing. It should be around the “build image” step only. But that would be harder to read…

### ARG (build time)

Variables defined through **ARG** are also known as [build-time variables](https://docs.docker.com/engine/reference/builder/#arg). They are only available from the moment they are ‘announced’ in the Dockerfile with an ARG instruction in the Dockerfile.

Running containers **can’t** access the values of ARG variables. So anything you run via CMD and ENTRYPOINT instructions won’t see those values by default.

The benefit of ARG is, that Docker will expect to get values for those variables. At least, if you don’t specify a default value. If those values are not provided when running the `build` command, there will be an error message. Here is an example where Docker fill complain during build:

```
# no default value is specified!
ARG some_value
```

Even though ARG values are not available to the container, they can easily be inspected through the Docker CLI after an image is built. For example by running `docker history` on an image. **ARG and ENV are a poor choice for sensitive data** if untrusted users have access to your images.

### ENV (build time and run time)

**ENV** variables are available both during the build and to the future running container. In the Dockerfile, they are usable as soon as you introduce them with an ENV instruction.

Unlike ARG, ENV values **are** accessible by containers started from the final image. ENV values can be overridden when starting a container, more on that below.

## Setting ARG Values

By now, you understand the difference between ARG and ENV. Let’s look about how to set them, starting with ARG.

You have the choice to _just_ specify that an ARG value should exist, or to provide a default value for it as well. Here is a Dockerfile example, both without a default values and with one:

```
# no default value:
ARG some_variable_name

# WITH a hard-coded default value:
#ARG some_variable_name=some_default_value
# we will leave this commented out for the upcoming example

# let's use the value
RUN echo "Oh dang look at that $some_variable_name"
# NOTE: you can can also put curly brackets around
# the variable name: ${some_variable_name}
# I find this more readable
```

[relevant docs](https://docs.docker.com/engine/reference/builder/#arg)

When building a Docker image from the command line, you can set **ARG** values using _–build-arg_:

```
$ docker build --build-arg some_variable_name=a_value
```

Running that command, with the above Dockerfile, will result in the following line being printed (among others):

```
Oh dang look at that a_value
```

What if you build your images via `docker-compose`? sWhen using docker-compose, you can specify “args” to pass to the Dockerfile ARG entries:

(docker-compose.yml file)

```
version: '3'

services:
  example:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        some_variable_name: a_value
```

[relevant docs](https://docs.docker.com/compose/compose-file/#args)

When you try to set a variable which is not ARG mentioned in the Dockerfile, Docker will complain. If you don’t set a variable which is required, Docker will complain as well.

## Setting ENV Values

So, how to set ENV values? You can do so in multiple steps of the process! Each one overrides the previous ones.

- You can set ENV values during the build (via defaults or from an ARG value).
- You can provide values for environment variables in **various** ways when running your containers. We will get into this detail later.

First, here is a basic Dockerfile, using ENV with hard-coded default values:

```
# no default value would crash
#ENV hey

# ENV needs a default value
ENV foo /bar
# you can also write
ENV foo=/bar

# ENV values can be used during the build
COPY . $foo
# or COPY . ${foo}
# this resolves to: COPY . /bar
```

[relevant docs](https://docs.docker.com/engine/reference/builder/#env)

What if you want to set ENV variables dynamically during build? When building an image, the only thing you can provide are ARG values. You can’t provide values for ENV variables **directly**. However, both ARG and ENV can work together. You can use ARG to set the default values of ENV vars in your Dockerfile.

And here is a snippet for a Dockerfile, using dynamic on-build env values:

```
# expect a build-time variable
ARG A_VARIABLE
# use the value to set the ENV var default
ENV an_env_var=$A_VARIABLE
# Success! If not overridden, the value of an_env_var
# will be available to your containers!
```

Once the image is built, you can specify environment variables when launching your containers. All will override any values set via ENV in your Dockerfile.

You can provide values for environment variables in **three different ways**, both with the docker CLI or using docker-compose and a docker-compose.yml file. We look at both tools in details for each of those three ways.

> NOTE: You can pass environment variables which were not defined via ENV! Build-time ARG variables are more strict about needing to be defined.

### 1. Provide Values One-By-One

When operating `docker run` via the command line, use the -e flag:

```
$ docker run -e "env_var_name=another_value" alpine env
```

[relevant docs](https://docs.docker.com/engine/reference/builder/#environment-replacement)

From a docker-compose.yml file, this works out to the same thing:

```
version: '3'

services:
  example:
    image: ubuntu
    environment:
      - env_var_name=another_value
```

[Relevant docs](https://docs.docker.com/compose/compose-file/#environment)

### 2. Pass Environment Variable Values From Your Host

Similar to the above, but worth pointing out. You provide the variable name, but leave out the value. This will make Docker look up the current value in the host environment and pass it on to the container.

A good way to keep lengthy variables out of your CLI.

```
$ docker run -e env_var_name alpine env
```

For the docker-compose.yml file, just specifying the name of the environment variable has the same effect:

```
version: '3'

services:
  example:
    image: ubuntu
    environment:
      - env_var_name
```

### 3. Take Values From a File (sometimes called an env_file)

Instead of listing a lot of variables, we can specify a file where they are written down.

The contents of such a file look something like this:

```
env_var_name=another_value
env_var_name2=yet_another_value
```

Let’s play through an example. Imagine we create a file called “BOB” (name arbitrary), and put the above lines into it.

We will need to reference the filename like this:

```
$ docker run --env-file=BOB alpine env
```

[relevant docs](https://docs.docker.com/compose/compose-file/#env_file)

With docker-compose.yml files, we use the _env_file_ instruction:

```
version: '3'

services:
  example:
    image: ubuntu
    env_file: BOB
```

[Relvant docs](https://docs.docker.com/compose/compose-file/#env_file)

> Why “BOB” you may ask? Well, most docs and tutorials call that file “env_file” or something similar, and it sounds all important and stuff. But it’s just an arbitrary name. I chose to call it BOB to make a point.

Remember the image from above? I added examples of different ways to set ENV and ARG in different stages via the Docker CLI or in the Dockerfile as an overview.

 ![](https://vsupalov.com/images/docker-env-vars/docker_environment_build_args_overview.png)An overview of ARG and ENV availability.

Before we move on: time to mention **another frequent gotcha**. if you try to set the value of an environment variable from **inside a RUN statement** of your Dockerfile (like `RUN export VARI=5 && ...`), you **won’t have access to that value** in any of the following RUN statements. This is not the same as setting an ENV value!

The reason for this, is that the “setting” via RUN statement happens inside a temporary container. An image when the command has finished to run, but the environment of the container does not persist that way.

> You can use [this approach](https://vsupalov.com/set-dynamic-environment-variable-during-docker-image-build/) to set dynamic environment variables if you really need to.

### Sidenote: Inspecting Images

If you’re curious about an image, and would like to know if it provides default ENV variable values before the container is started, you can “inspect” it, and see which ENV entries are set by default:

```
# first, get the images on your system
$ docker images
# select an image id you are curious about and...
$ docker inspect image-id

# look out for an "Env" array in the output
```

## Overriding ENV Values

Imagine, you have an image built from a Dockerfile. That Dockerfile defines some ENV values.

How can you change those values when running a container? Or, if you prefer, how can you make sure that the ENV values stay intact?

ENV values from the Dockerfile will be **overridden** when providing single environment variables, or an _env_files_ with the right entry, via the CLI.

Also, the process running inside the container can change their environment for themselves. Here is a command, which overrides the CMD with a shell line that sets SOME_VAR before running a Python script:

```
$docker run myimage SOME_VAR=hi python app.py
```

This way of running the container will completely override any SOME_VAR you might have defined in your image. At least from the point of view of the app.py script. Even if you set a value with a -e flag before the final command.

The precedence is, from **stronger** to less-strong:

- stuff the containerized application sets,
- values from single environment entries,
- values from the env_file(s),
- Dockerfile ENV entries.

## In Conclusion

This was a really thorough look at all the ways you can set ARG and ENV variables when building Docker images and starting containers.

I hope it has helped you get a good overview build-time arguments, environment variables, env_files and how docker-compose does things.

All the best on your image building journey! If you want to read more, check out my other articles on [building Docker images](https://vsupalov.com/categories/building-docker-images/) and [getting the most out of Docker](https://vsupalov.com/categories/working-with-docker/).