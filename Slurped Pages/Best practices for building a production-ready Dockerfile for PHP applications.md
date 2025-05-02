---
link: https://snyk.io/fr/blog/building-production-ready-dockerfile-php/
byline: James Walker
site: Snyk
date: 2023-08-22T05:00
excerpt: In this tutorial, you'll learn how to write a dependable Dockerfile for PHP applications.
slurped: 2024-11-16T18:37
title: Best practices for building a production-ready Dockerfile for PHP applications
tags:
  - docker
  - dockerfile
aliases:
  - "#docker"
---

[Docker](https://www.docker.com/) is a containerization platform for bundling your code, dependencies, and runtime environment into self-contained units that run identically in different environments.

Dockerizing a PHP application simplifies deployment by packaging the PHP runtime, a web server, and your source code and composer dependencies into a container. Getting started with Docker is easy. However, there are a few pitfalls you need to avoid before you can safely use it in production. Choosing an appropriate base image, selecting the correct PHP runtime for your workload, and hardening your image for security are all essential to prevent problems. In this tutorial, you'll learn how to write a dependable Dockerfile for PHP applications.

## Building a production-ready Dockerfile for PHP

If you're reading this, you've probably written your PHP app, and now you want to Dockerize it for deployment. Following are ten best practices that maximize performance, security, reliability, and ease of development:

### 1. Use a specific base image

The Docker Community publishes [an official Docker image](https://hub.docker.com/_/php) for all supported PHP versions. Several image variants are available, covering a matrix of base operating systems, language releases, and web server integrations (such as Apache or [FastCGI Process Manager (FPM)](https://www.php.net/manual/en/install.fpm.php)).

It's important to select the specific image that best matches your requirements. Avoid using generic tags, such as `php:latest`, because these will expose you to potentially unwanted, code-breaking changes and unnecessary libraries required only for a full-blown operating system. For example, the `php:apache` image currently includes PHP 8.2, but it will immediately switch to PHP 9.0 upon that version's release. Selecting `php:8.2-apache` ensures your image will always run PHP 8.2, even when it's no longer the current channel:

```
1FROM php:8.2-apache
2WORKDIR /var/www/html
3
4COPY index.php .
```

The same principle applies to any other images you may depend on, such as [composer](https://hub.docker.com/_/composer) for dependency installation. Select the image tag that matches the major composer version you're using on your machine — `composer:2` instead of `composer:latest`.

### 2. Use environment variables for configuration

Docker containers are designed to be stateless. It's best to configure your application using environment variables that you set when the container starts. This is more favorable than relying on a configuration file, which would require you to always set up persistent container storage [using a Docker volume](https://docs.docker.com/storage/volumes).

Environment variables are set with the `-e` flag when you start a container using `docker run`:

```
$ docker run -d -e DATABASE_HOST=172.26.0.2 php-app:latest
```

Your PHP code can retrieve the variable's value by calling the [getenv() function](https://www.php.net/manual/en/function.getenv.php):

```
// 172.26.0.2
echo getenv("DATABASE_HOST");
```

These variables are runtime variables set on individual containers. Sometimes you might want to set a variable within your Dockerfile so it's supplied to containers automatically. To achieve this, use the `ENV` instruction:

```
1FROM php:8.2-apache
2WORKDIR /var/www/html
3
4ENV DATABASE_DRIVER=sqlite
5
6COPY index.php .
```

Environment variables can also be populated at build time and referenced within your Dockerfile's other instructions using the `ARG` keyword:

```
1FROM php:8.2-apache
2WORKDIR /var/www/html
3
4ARG DEFAULT_DATABASE_DRIVER
5
6RUN echo '{"db":"$DEFAULT_DATABASE_DRIVER"}' > config.json
7COPY index.php .
```

To set the value of the argument, use the `--build-arg` flag when you run `docker build`:

```
$ docker build --build-arg DEFAULT_DATABASE_DRIVER=sqlite -t php-app:latest .
```

### 3. Consider PHP FPM vs. Nginx/Apache

[FPM](https://www.php.net/manual/en/install.fpm.php) is an alternative PHP FastCGI implementation for integrating PHP with web servers, including Apache and Nginx. An alternative to FPM is to use the Apache web server with `mod_php`. Compared to the Apache alternative, FPM offers advanced non-blocking process management, the ability to create different pools of worker processes, and more graceful error handling. Using FPM often provides [a significant performance boost](https://www.cloudways.com/blog/php-fpm-on-cloud) for large apps.

PHP's [Docker base image](https://hub.docker.com/_/php) is available in both FPM and `mod_php` variants:

- `**php:8.2-fpm**`**:** This image and similar tags expose an FPM connection.
    
- `**php:8.2-apache**`**:** These images bundle an Apache installation and use `mod_php` to handle PHP requests.
    

If you're using Apache and are satisfied with `mod_php`, select one of the `:apache` base images. You can add your PHP code to the `/var/www/html` directory and start running it immediately.

To use FPM, you'll need a separate container that runs a server like Nginx. This should be configured as a [reverse proxy, which forwards](https://www.digitalocean.com/community/tutorials/php-fpm-nginx) PHP requests to the port exposed by the FPM process in your PHP container. The port number defaults to `9000`.

Use FPM when you want maximum performance and aren't worried about managing two containers: an FPM container and a web server in front of it. Apache-based images are a great starting point for getting up and running quickly without configuring additional containers or fine-tuning FPM worker pool configs.

### 4. Disable unnecessary debug logs/error reporting in production

Production applications shouldn't publicly disclose any information that an attacker could find useful. Disable PHP's [display_errors](https://www.php.net/manual/en/errorfunc.configuration.php#ini.display-errors) and [display_startup_errors](https://www.php.net/manual/en/errorfunc.configuration.php#ini.display-startup-errors) settings to prevent unhandled exceptions from being displayed on your website.

To change these values, modify the `php.ini` file within your container.

Start by creating your modified file:

```
1display_errors = Off
2display_startup_errors = Off
```

Then adjust your `Dockerfile` to copy `php.ini` into the correct place in the container's file system:

```
1FROM php:8.2-apache
2WORKDIR /var/www/html
3
4COPY php.ini /usr/local/etc/php/php.ini
5COPY index.php .
```

PHP's [error_reporting level](https://www.php.net/manual/en/errorfunc.configuration.php#ini.error-reporting) might need to be altered for production use. Low-level errors, such as notices and deprecation warnings, can quickly fill logs with unactionable information. You can set this value dynamically at the start of your script based on an environment variable:

```
1if (getenv("IS_PRODUCTION")) {
2    // Disable error reports which are not an immediate issue
3    error_reporting(E_ALL & ~E_NOTICE & ~E_STRICT & ~E_DEPRECATED);
4}
```

### 5. Run your PHP container as a non-root user

Docker defaults to running containers as `root`. The process running inside the container has the same privileges as the `root` on your host. Compromising the container and breaking out of its isolation allows malicious processes to run arbitrary commands on your host.

Running your container as a dedicated non-root user helps to mitigate these risks. Use the `USER` instruction in your Dockerfile to switch to a different user from that point forward:

```
1FROM php:8.2-apache
2WORKDIR /var/www/html
3USER appuser
4
5COPY index.php .
```

### 6. Set up PHP container health checks

Docker has a built in health check mechanism to detect when containers fail. You can configure Docker to automatically run a health check command every thirty seconds. If the command exits with code `0`, the container is considered healthy. However, health checks that exit with code `1` indicate a failed application.

You can set up health checks for your applications using the [HEALTHCHECK instruction](https://docs.docker.com/engine/reference/builder/#healthcheck) in your Dockerfile. For a PHP service, you can use `curl` to make a network request to one of your app's endpoints. If `curl` exits with `0`, it means that it received a response with an HTTP status code in the `2xx` (success) range:

```
1FROM php:8.2-apache
2WORKDIR /var/www/html
3USER appuser
4
5HEALTHCHECK CMD curl --fail http://localhost/index.php || exit 1
6COPY index.php .
```

The healthiness of your containers is displayed in the `docker ps` command's output:

```
1$ docker ps
2CONTAINER ID	IMAGE  		COMMAND              		CREATED    	STATUS
3335889ed4698	php-demo-app		"docker-php-entrypoi…"   	2 mins ago 	Up 2 mins (healthy)
```

After a health check fails, the container's status will show as `unhealthy`. Container orchestrators, such as [Kubernetes](https://kubernetes.io/), use this signal to automatically restart or replace containers after they fail.

### 7. Restrict the number of ports you use

You should avoid exposing unnecessary ports on your containers. Only ports that need to be reached by external processes should be exposed. These are the ports that other containers and public users will connect to.

For PHP applications, this is often port `80` when using an image with a bundled web server, such as `php:8.2-apache`. FPM-based images should expose port `9000` instead, ready for a separate web server container to connect to. The FPM port should not be open publicly.

Similarly, be conscious of the information you're exposing through environmental variables. Remove unused variables to reduce the risk of disclosure. Tools such as `docker inspect` and any malicious software inside the container can always access the full list of variables you've set.

### 8. Regularly update dependencies

All the components in your stack should be regularly updated to apply new bug fixes and security updates. These include the following:

- Updates to the base operating system in your Docker image.
    
- Updates to the PHP version and extensions that you're using.
    
- Composer packages that you've installed as dependencies in your project.
    

This is crucial so that you stay protected from new vulnerabilities. You should rebuild your image with `docker build --pull` to get updates each time you build. This will pull the latest version of your base image, including any updated operating system packages and PHP releases.

You should also run `composer update` in your project periodically to add the latest patches for your dependencies. You can use tools such as [Snyk](https://docs.snyk.io/scan-application-code/snyk-open-source/snyk-open-source-supported-languages-and-package-managers/snyk-for-php) to find dependencies that are outdated or contain vulnerabilities, with suggestions for how to update to a fixed version. After you’ve applied your package updates with Composer, rebuild your Docker image so production deployments use the new versions.

### 9. Restrict container capabilities to the minimum needed

Linux [kernel capabilities](https://man7.org/linux/man-pages/man7/capabilities.7.html) restrict the potentially sensitive actions that processes can perform. Docker already runs containers [with an unprivileged set](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) of capabilities, but these are still excessive for typical PHP workloads. For instance, the defaults allow container processes to change file permissions, kill processes, and manipulate user IDs.

You can add and remove container capabilities with the `--cap-add` and `--cap-drop` flags for `docker run`. It's most secure to drop every capability using `--cap-drop=all` and then add back the specific ones your app requires.

The following example gives the container only the `CHOWN` capability. The container can apply ownership changes to files and folders but will be prevented from performing any other privileged actions:

```
$ docker run -d --cap-drop=all --cap-add=CHOWN php-app:latest
```

### 10. Use multistage builds for build/runtime separation

[Multistage builds](https://docs.docker.com/build/building/multi-stage) are a mechanism that makes it easier to write and maintain more complex build pipelines. They let you involve multiple base images in the build process while keeping the output small and precise.

The multistage pattern is ideal for PHP Dockerfiles because you'll normally want to install composer dependencies during the build. PHP's official Docker images don't come with the composer — the project publishes its images instead. Using a multistage build, you can conveniently reference the composer binary within your Dockerfile:

```
1FROM php:8.2-apache
2WORKDIR /var/www/html
3
4COPY --from=composer:2 /usr/bin/composer /usr/bin/composer
5COPY composer.json .
6COPY composer.lock .
7RUN composer install --no-dev
8RUN rm /usr/bin/composer
9
10COPY index.php .
```

The `composer:2` image will be discarded when the build completes. This approach maintains good separation between build and runtime without increasing the size of your final image.

## Identifying and resolving vulnerabilities with Snyk

Using Docker can improve security by providing a degree of isolation between your PHP applications and their host environments. However, container images often contain security vulnerabilities due to outdated operating system packages and [composer dependencies](https://snyk.io/fr/blog/security-alert-php-pdf-library-dompdf-rce/):

![blog-build-dockerfile-snyk](https://snyk.io/_next/image/?url=https%3A%2F%2Fres.cloudinary.com%2Fsnyk%2Fimage%2Fupload%2Fv1692730669%2Fblog-build-dockerfile-snyk.jpg&w=2560&q=75)

Scanning your image for vulnerabilities is a critical step before you deploy it to production. You can use [Snyk](https://snyk.io/fr/) to scan your PHP Docker image and identify and resolve vulnerabilities. The [Snyk Vulnerability Database](https://security.snyk.io/vuln) includes records for all popular operating systems and dependencies, including [PHP packages](https://security.snyk.io/vuln/composer) published to [Packagist](https://packagist.org/).

To begin identifying vulnerabilities with Snyk, you need to create a simple PHP file and save it to `index.php`:

```
<?php

echo "Hello World";

?>
```

Next, write a Dockerfile for your application:

```
FROM php:8.2-apache
WORKDIR /var/www/html

COPY index.php
```

Build the image using Docker:

```
$ docker build -t php-app:latest .
```

Then head to the Snyk website and click **Start free** to create your account. Follow the steps to get set up and then install the Snyk CLI. You'll also need [Node.js and npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) installed:

Next, run the `snyk auth` command. This will open a new browser tab where you can log into Snyk and authenticate the CLI. Afterward, return to your terminal.

To scan your image, run the following command:

```
$ snyk container test php-app:latest
```

The scan may take a few moments to complete. The results will then be displayed in your terminal. You'll see a list of all the detected vulnerabilities, including the package where they're present, the vulnerable versions, and a link to get more information:

![blog-build-dockerfile-110-issues](https://snyk.io/_next/image/?url=https%3A%2F%2Fres.cloudinary.com%2Fsnyk%2Fimage%2Fupload%2Fv1692730669%2Fblog-build-dockerfile-110-issues.jpg&w=2560&q=75)

The summary at the end displays the total count of problems found and suggests alternative base images with fewer vulnerabilities.

### Fixing detected vulnerabilities

If vulnerabilities are found, you should assess each one to determine whether you need to take action. Not every vulnerability will necessarily be a problem for your use case. Focus on resolving the critical-, high-, and medium-severity problems first.

The most effective starting point is often to rebuild your image using an updated base. Popular community images, such as `php:8.2-fpm`, are regularly republished with new versions that include vulnerability fixes. Repeat the scan after you've rebuilt your image to check how it affects your score.

The report might list a large number of vulnerabilities in low-level operating system components. Consider using a smaller, more minimal base image to reduce your overall attack surface. [Alpine Linux](https://www.alpinelinux.org/) variants, such as `php:fpm-alpine`, include far fewer packages, which often means fewer vulnerabilities.

At the time of writing, the `php:8.2-fpm` image contains ninety-eight detected vulnerabilities, for example:

![blog-build-dockerfile-98-issues](https://snyk.io/_next/image/?url=https%3A%2F%2Fres.cloudinary.com%2Fsnyk%2Fimage%2Fupload%2Fv1692730669%2Fblog-build-dockerfile-98-issues.jpg&w=1240&q=75)

The `php:8.2-fpm-alpine` variant only has ten because it contains far fewer operating system packages:

![blog-build-dockerfile-10-issues](https://snyk.io/_next/image/?url=https%3A%2F%2Fres.cloudinary.com%2Fsnyk%2Fimage%2Fupload%2Fv1692730670%2Fblog-build-dockerfile-10-issues.jpg&w=1240&q=75)

Some vulnerabilities might not be resolvable in your project. For instance, issues in operating system packages may not affect your workload, or a patch may not be available. You can dismiss a vulnerability from a Snyk report by passing its ID to the `snyk ignore` command. You can find the ID by looking at the last part of the vulnerability's URL, such as `SNYK-ALPINE317-CURL-3320725` in `https://security.snyk.io/vuln/SNYK-ALPINE317-CURL-3320725`:

```
$ snyk ignore --id=SNYK-ALPINE317-CURL-3320725
```

Finally, remember that container security is the sum of multiple components. You need a secure base image, an updated and supported version of the PHP runtime, and patched versions of your dependencies, toolchain components, [and application code](https://snyk.io/fr/blog/prevent-php-code-injection/). Regularly scanning your image for vulnerabilities will keep you informed of your security position.

## Summary

After reading this article, you should have a production-ready Docker image for running your PHP application. This guide has covered some key best practices for writing your Dockerfile and using the images you build, including steps such as selecting a specific base image, using multistage builds to install dependencies, and scanning for security vulnerabilities to ensure your workloads are suitable for deployment to production.

If you'd like more recommendations for Docker images and PHP, check out [Docker's documentation](https://docs.docker.com/develop/develop-images/dockerfile_best-practices) and the [other articles available on the Snyk blog](https://snyk.io/fr/blog/securing-php-containers/).

[Snyk](https://snyk.io/fr/) is a developer-friendly security platform for anyone responsible for securing code. It can scan your Docker images to find vulnerabilities in your dependencies, operating system packages, [and PHP code](https://snyk.io/fr/blog/php-security-support-snyk-code/). Snyk also offers an [IDE plugin](https://snyk.io/fr/blog/php-security-support-snyk-code/) that performs static analysis to detect vulnerabilities as they appear. [Sign up now](https://app.snyk.io/sign-up) to automatically find and fix vulnerabilities in your PHP container images.