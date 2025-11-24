---
link: https://www.baeldung.com/ops/install-php-composer-docker-container#:~:text=Installing%20composer%20Using%20Shell%20Commands,the%20command%20line%20with%20php.&text=After%20referencing%20the%20base%20image%2C%20the%20Dockerfile%20installs%20wget.
site: Baeldung on Ops
date: 2024-09-24T12:14
excerpt: Learn how to install PHP Composer inside a Docker container using shell commands or copying an executable from an existing image.
twitter: https://twitter.com/@baeldung
slurped: 2025-11-14T16:52
title: Installing PHP Composer Inside a Docker Container | Baeldung on Ops
tags:
  - composer
  - Installation
---

## 1. Introduction

Composer is a PHP dependency management tool; put simply, it assesses our project’s PHP libraries and installs or updates those libraries as needed.

We can install PHP Composer in a Docker container using [shell](https://www.baeldung.com/linux/sh-vs-bash#what-is-a-shell) commands, but for readability and efficiency, we can simply copy a [_composer_](https://getcomposer.org/doc/01-basic-usage.md) executable from an existing image to our build. In this tutorial, we discuss how to install PHP Composer inside a Docker container.

## 2. Installing _composer_ Using Shell Commands

We can install _composer_ in a Docker container by downloading the installer with [_wget_](https://www.baeldung.com/linux/wget-examples) or _[curl](https://www.baeldung.com/linux/curl-guide)_ and executing it from the command line with [_php_](https://www.baeldung.com/linux/lamp-apache-mysql-php#install-php).

Let’s demonstrate how to download and install _composer_ using _wget_ and _php_ in our Dockerfile:

```
FROM php:8.1-apache-bullseye
RUN ["/bin/bash", "-c", "apt update && apt install wget -y \
&& wget -O- https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer"]
ENTRYPOINT composer --version
```

After referencing the base image, the Dockerfile installs _w__get_. Thereafter, _wget_ writes the content of the _composer_ installer to _[stdout](https://www.baeldung.com/linux/redirect-process-output#1-redirect-stdout)_ and said content is redirected to _php_ for execution.

Let’s build and run the Dockerfile to confirm the installation:

```
$ docker build . -t test -q && docker run test
...truncated...
Composer version 2.7.9 2024-09-04 14:43:28
PHP version 8.1.29 (/usr/local/bin/php)
Run the "diagnose" command to get more detailed diagnostics output.
```

As expected of the _ENTRYPOINT_ command in our Dockerfile, the container outputs the _composer_ version.

Now, let’s repeat the same process with _curl_ and _php_ in another Dockerfile:

```
FROM php:8.1-apache-bullseye
RUN ["/bin/bash", "-c", "curl https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer"]
ENTRYPOINT composer --version
```

_curl_ was preinstalled in our base image, so we didn’t have to add installation commands. That said, if we’ll be **performing other network operations in our Dockerfile besides a download,** **we’ll opt for _curl_ over _wget_, as _curl_ supports more protocols**.

Once again, we’ll build and run to confirm the installation:

```
$ docker build . -t test -q && docker run test
...truncated...
Composer version 2.7.9 2024-09-04 14:43:28
PHP version 8.1.29 (/usr/local/bin/php)
Run the "diagnose" command to get more detailed diagnostics output.
```

### 2.1. Installing a Specific Version of _composer_

The **installer script installs the latest version of** _**composer**._ However, if we want a specific version, we can get a [manual download link](https://getcomposer.org/download/) for our desired _composer.phar_ file and download the file to a _PATH_ directory. Here’s how we’d get _composer 2.4.4_ installed in our container:

```
FROM php:8.1-apache-bullseye
ARG path=/usr/local/bin/composer
RUN ["/bin/bash", "-c", "apt update && apt install wget -y \
&& wget -O $path https://getcomposer.org/download/2.4.4/composer.phar && chmod +x $path"]
ENTRYPOINT composer --version
```

The Dockerfile above installs _wget,_ downloads the _composer.phar_ executable version _2.4.4_ to _PATH_ with the filename _composer_, then makes the _composer_ file executable. Let’s confirm the installation with _[docker build](https://www.baeldung.com/ops/docker-buildx)_ and [_docker run_](https://docs.docker.com/reference/cli/docker/container/run/):

```
$ docker build . -t test -q && docker run test
...truncated...
Composer version 2.4.4 2022-10-27 14:39:29
```

We can achieve the same result with _curl_ instead of _wget_:

```
FROM php:8.1-apache-bullseye
ARG path=/usr/local/bin/composer
RUN ["/bin/bash", "-c", "curl -o $path https://getcomposer.org/download/2.4.4/composer.phar && chmod +x $path"]
ENTRYPOINT composer --version
```

Like _wget_, _curl_ downloads _composer.phar 2.4.4_ to _PATH,_ names it “_composer”_, and makes it executable.

## 3. Installing _composer_ Using an Existing Image

To install _composer_ from an existing image, we use the _[COPY –from](https://docs.docker.com/build/building/multi-stage/#use-an-external-image-as-a-stage)_ instruction in our Dockerfile:

```
FROM php:8.1-apache-bullseye
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer
ENTRYPOINT composer --version
```

In the Dockerfile above, the _COPY –from_ instruction simply copies the _composer_ executable from _/usr/bin/composer_ on the _composer:latest_ image to _/usr/local/bin/composer_ on our image. So, when we create a container from our image, the container will have _composer_:

```
$ docker build . -t test -q && docker run test
...truncated...
Composer version 2.7.9 2024-09-04 14:43:28
PHP version 8.1.29 (/usr/local/bin/php)
Run the "diagnose" command to get more detailed diagnostics output.
```

**To get a specific _composer_ version using this method, we’ll pass an image with our desired version to the _COPY –from_ instruction**. For example, if we wanted _composer 2.7.1_, we’d use the _composer:2.7.1_ image:

```
FROM php:8.1-apache-bullseye
COPY --from=composer:2.7.1 /usr/bin/composer /usr/local/bin/composer
ENTRYPOINT composer --version
```

Let’s build this image and run a container from it to confirm installation:

```
$ docker build . -t test -q && docker run test
...truncated...
Composer version 2.7.1 2024-02-09 15:26:28
```

## 4. Conclusion

In this article, we learned how to install PHP _composer_ in a Docker container using shell commands and by copying an executable from an existing image.