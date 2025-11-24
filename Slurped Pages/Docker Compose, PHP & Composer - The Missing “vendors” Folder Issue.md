---
link: https://medium.com/@lukaspereyra8/docker-compose-php-composer-the-missing-vendors-folder-issue-66faa5475c59
byline: Lucas Pereyra
site: Medium
date: 2022-04-14T16:39
excerpt: "Docker Compose, PHP & Composer: The Missing “vendors” Folder Issue A couple of days ago, I had to work with a very simple PHP Slim web application setup that was totally containerized with …"
twitter: https://twitter.com/@Medium
slurped: 2025-11-14T11:02
title: "Docker Compose, PHP & Composer: The Missing “vendors” Folder Issue"
tags:
  - composer
  - docker
---

A couple of days ago, I had to work with a very simple _PHP Slim_ web application setup that was totally containerized with Docker. I managed it to make it work by means of a Nginx Docker container, a PHP-FPM container, and the Slim app container. All was working good when I started to struggle with a _docker compose build_ problem when attempting to install my PHP dependencies using Composer, inside of the Dockerfile’s container building steps (these are the steps executed when you run a _dockercompose build_). I decided to share with you this issue and a couple of workarounds you can apply when facing it, with a simple yet practical example.

## Basic project setup example

So, as a starting point, let’s consider the following project structure:

/php  
    composer.json  
    main.php  
    php.Dockerfile  
docker-compose.yml

A pretty basic and minimal structure for a small PHP project, that makes use of Composer to define its dependencies. Let’s take a look at the _docker-compose.yml_ file:

```
services:  
    php:  
      container_name: my_php_container__  
    build:  
      context: ./php  
      dockerfile: ./php.Dockerfile  
    volumes:  
      - ./php/:/usr/src/myapp

```

This only defines a PHP service, which uses a shared volume with the _/php_ folder and its _Dockerfile_ instructions are defined in the _php.Dockerfile_ file. Let’s see it:


```

FROM php:7.4-cli# INSTALL ZIP TO USE COMPOSER  
RUN apt-get update && apt-get install -y \  
    zlib1g-dev \  
    libzip-dev \  
    unzip  
RUN docker-php-ext-install zip# INSTALL AND UPDATE COMPOSER  
COPY --from=composer /usr/bin/composer /usr/bin/composer  
RUN composer self-updateWORKDIR /usr/src/myapp  
COPY . .# INSTALL YOUR DEPENDENCIES  
RUN composer installCMD php main.php

```
Here we start from a PHPv7.4 image; install and enable the “zip” library, needed to be able to use Composer; we install Composer by means of the _copy from_ Dockerfile feature; copy all the files from our project; install the PHP dependencies; and finally execute the _main.php_ file.

Let’s suppose our _main.php_ file is just as simple as the following:

```
  <?phprequire './vendor/autoload.php';  
use Carbon\Carbon;printf("Right now is %s\n", Carbon::now()->toDateTimeString());printf(  
    "Right now in Vancouver is %s\n",  
    Carbon::now('America/Vancouver')  
);printf("Tomorrow will be %s\n", Carbon::now()->addDay());

For this to work successfully, we’ll need to install the _Carbon_ PHP dependency, so our _composer.json_ file may look like:

{  
   "name": "test/app",  
   "description": "Simple PHP Composer app",  
   "type": "project",  
   "require": {  
      "nesbot/carbon": "^2.57"  
   }  
}

```

## The missing “vendors” folder issue

With the previous provided setup, we’d think nothing could go wrong if we run`docker compose up --build` . Well, let’s take a change and give it a try:
![[Pasted image 20251114110852.png]]
Press enter or click to view image in full size

As we can see, PHP cannot find the _vendors_ folder that should be generated when _composer install_ is executed. Note that the _composer.lock_ file doesn’t exist neither. So…, what’s happening here?

It took me a couple of hours to figure it out, but what happens results to be quite simple: when building the Docker PHP container, docker executes each of the building steps defined in the Dockerfile, hence, running the _composer install_ command and generating the /vendors folder and the composer.lock file as expected. At this point, these file exist only in the container filesystem. When _docker compose up_ is executed, Docker mounts the host machine’s (our machine) filesystem into the container’s filesystem, thus overwriting the container files. That’s when the /vendors directory and the composer.lock file are removed from the container’ filesystem. Since Docker doesn’t mounts the specified volumes during the _building_ stage, all the files stored inside of the container during this building stage will be erased and replaced with the files you’re storing in the _/php_ folder when you start the containers. Having this in mind, note that the _COPY . ._ instruction we defined in the _php.Dockerfile_ is useless, and we could remove it without producing an unexpected result.

## 3 alternatives to prevent the missing “vendors” folder issue

To avoid having this issue, we must remove the `RUN composer install` instruction from the _php.Dockerfile_ file. Without this, we won’t have our PHP dependencies installed once the container is built, so we’ll have to run this step afterwards, at the moment of starting the container. Here I provide 3 alternatives for doing this, starting with the simplest ones:

### Manually installing the dependencies after each docker compose build

As this self-descriptive tittle suggests, we could just manually install the PHP dependencies we need each time we build up the container. This seems to work if we’re sure we won’t need to run _composer install_ more than just a couple of times for a project.

We can install the composer PHP dependencies via the _docker compose run_ command, like this:

docker compose run php composer install

Note that after running this command, we should have the _composer.lock_ and _/vendor_ files already existing in our initial project files structure.

### Installing the dependencies every time the container is started

The _php.Dockerfile_ we’re using executes the `CMD php main.php` command as the starting command for the PHP container. Here we could include the extra step of installing the PHP dependencies by modifying it to be:

CMD bash -c "composer install && php main.php"

With this approach, we won’t need to care about manually installing our PHP dependencies every time we build up the container. Whenever we execute _docker compose up_, this will first check for all the packages status, install the missing ones and run our application.

### Installing the dependencies once during docker compose build, by scoping volume bindings

There’s a way we can achieve the PHP dependencies installation only during _docker compose build_ execution. Since in our previous setup we’re binding the root directory with the container’s filesystem, all the files get overwritten when we start the container. We could just scope this binding to only have a bunch of our files linked to the container’s filesystem.

Let’s say we just want to link our .php files and the composer.json file with the container’s filesystem, hence, any change made to those files, both from our machine or from inside of the container, would be reflected in both directions. We’d need to modify our _docker-compose.yml_ file to explicitly tell it which files we want to bind/link:

```
  services:  
    php:  
        container_name: my_php_container__  
        build:  
            context: ./php  
            dockerfile: ./php.Dockerfile  
        volumes:  
            - ./php/main.php:/usr/src/myapp/main.php  
            - ./php/composer.json:/usr/src/myapp/composer.json

```

With that, our _php.Dockerfile_ can be responsible for installing the composer dependencies:
```
...  
WORKDIR /usr/src/myappCOPY *.json . # copy the composer.json file  
COPY *.php .  # copy the php source files# INSTALL YOUR DEPENDENCIES  
RUN composer installCMD php main.php

```


Every time we run _docker compose build_, our PHP dependencies will be installed, a /vendors folder will be generated as well as a composer.lock file, BUT ONLY INSIDE OF THE CONTAINER. Hence, we won’t be able to see these files in our host machine. Anyway, since all we need will be in the container itself, we’ll be able to run _docker compose up_ and see the expected results:

```
docker compose up  
[+] Running 1/0  
 - Container my_php_container__  Created                                                                                                                                                                                                 0.0s Attaching to my_php_container__  
my_php_container__  | Right now is 2022-04-14 14:31:04  
my_php_container__  | Right now in Vancouver is 2022-04-14 07:31:04  
my_php_container__  | Tomorrow will be 2022-04-15 14:31:04  
my_php_container__ exited with code 0
```

Moreover, if we install a new PHP dependency using Composer (_composer install XXX)_, that new requirement will be injected into our composer.json file, allowing us to always have all the required dependencies listed there, in a cool portable format (nothing bad, uh?). Something similar will happen with the .php files: we’ll be able to edit these files in our host machine, and changes should be reflected in the container’s version of the files. For that, you could even have a /src folder with all your .php files there and just bind that folder into the container.

Comments:
- 1
Please do not use `COPY --from=composer` and then self-update. Copy from specific version

Please do not rebuild local image every time you need one bit changed. Use volumes for local development

- 2
I had this problem and I found a trick that solves all the problems, simply add a command in the docker compose to build each time the vendor folder after a docker-compose and you can accumulate a docker volume with it.
Inside your docker-compose.yml add :
...
volumes:
- ./vendor:/var/www/html/vendor
command: >
sh -c "composer install && apache2-foreground"