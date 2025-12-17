---
link: https://www.agiledrop.com/blog/how-effortlessly-install-moodle-locally-using-ddev
excerpt: In today's digital age, utilizing robust learning management systems like Moodle has become increasingly essential for educational institutions and corporate training. However, setting up such a platform can be a daunting task, especially if you're looking to install it locally for development or testing purposes. Fortunately, with DDEV, a Docker-based PHP development environment, you can have Moodle up and running in no time. This comprehensive guide will walk you through every step of the process, from Docker and DDEV installation to Moodle setup.
slurped: 2025-12-14T17:26
title: How to effortlessly install Moodle locally using DDEV
tags:
  - moodle
  - install
---

[source](https://www.agiledrop.com/blog/how-effortlessly-install-moodle-locally-using-ddev)
## Prerequisites

- Docker
- DDEV

## Moodle Installation on DDEV:

After setting up Docker and DDEV, you're now ready to install Moodle. The process is straightforward and can be accomplished with a series of command-line instructions.

Configure DDEV for your new project using the command:
```
ddev config --composer-root=public --create-docroot --docroot=public --webserver-type=apache-fpm --database=mariadb:10.6

```
Start DDEV by running:
```
ddev start

```

Create a new Moodle project with the command:
```
ddev composer create moodle/moodle -y

```
Install Moodle with the command:

```
ddev exec 'php public/admin/cli/install.php --non-interactive --agree-license --wwwroot=$DDEV_PRIMARY_URL --dbtype=mariadb --dbhost=db --dbname=db --dbuser=db --dbpass=db --fullname="DDEV Moodle Demo" --shortname=Demo --adminpass=password'

```
Launch Moodle by running:

```
ddev launch /login

```
Log into your Moodle account using the username 'admin' and the password 'password'.

By following these steps, you'll have a local Moodle environment ready for development or testing.

## Conclusion:

Setting up Moodle locally using DDEV is a straightforward process that can significantly streamline your workflow. Not only does it save you time, but it also provides a consistent and reliable environment for your Moodle development and testing. By following the steps outlined in this comprehensive guide, you'll have a local Moodle environment ready for development in no time. So, why wait? Experience the ease and convenience of local Moodle development with DDEV today.