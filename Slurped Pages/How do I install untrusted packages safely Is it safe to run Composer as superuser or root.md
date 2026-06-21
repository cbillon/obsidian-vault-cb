---
link: https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md
excerpt: A Dependency Manager for PHP
slurped: 2026-06-20T08:45
title: How do I install untrusted packages safely? Is it safe to run Composer as superuser or root?
tags:
  - composer
---

- [Why am I seeing a "Do not run Composer as root/super user" warning/error?](https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md#why-am-i-seeing-a-do-not-run-composer-as-root-super-user-warning-error-)
- [Is it safe to run Composer as superuser or root?](https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md#is-it-safe-to-run-composer-as-superuser-or-root-)
- [Running Composer inside Docker/Podman containers](https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md#running-composer-inside-docker-podman-containers)

## Why am I seeing a "Do not run Composer as root/super user" warning/error?[#](https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md#why-am-i-seeing-a-do-not-run-composer-as-root-super-user-warning-error-)

It was always discouraged to run Composer as root for the reasons detailed below.

As of Composer 2.4.2, plugins are disabled automatically when running as root and there is no sign that the user is consciously doing this. There are two ways this user consent can be given:

- If you run interactively, Composer will prompt if you are sure that you want to continue running as root. If you run non-interactively, plugins will be disabled, unless..
- If you set the [COMPOSER_ALLOW_SUPERUSER](https://getcomposer.org/doc/03-cli.md#composer-allow-superuser) environment variable to `1`, this also indicates that you intended to run Composer as root and are accepting the risks of doing so.

## Is it safe to run Composer as superuser or root?[#](https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md#is-it-safe-to-run-composer-as-superuser-or-root-)

Certain Composer commands, including `exec`, `install`, and `update` allow third party code to execute on your system. This is from its "plugins" and "scripts" features. Plugins and scripts have full access to the user account which runs Composer. For this reason, it is strongly advised to **avoid running Composer as super-user/root**. All commands also dispatch events which can be caught by plugins so unless explicitly disabled installed plugins will be loaded/executed by **every** Composer command.

You can disable plugins and scripts during package installation or updates with the following syntax so only Composer's code, and no third party code, will execute:

```
php composer.phar install --no-plugins --no-scripts ...
php composer.phar update --no-plugins --no-scripts ...
```

Depending on the operating system we have seen cases where it is possible to trigger execution of files in the repository using specially crafted `composer.json`. So in general if you do want to install untrusted dependencies you should sandbox them completely in a container or equivalent.

Also note that the `exec` command will always run third party code as the user which runs `composer`.

See the [COMPOSER_ALLOW_SUPERUSER](https://getcomposer.org/doc/03-cli.md#composer-allow-superuser) environment variable for more info on how to disable the warnings.

## Running Composer inside Docker/Podman containers[#](https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md#running-composer-inside-docker-podman-containers)

Composer makes a best effort attempt to detect that it runs inside a container and if so it will allow running as root without any further issues. If that detection fails however you will see warnings and plugins will be disabled unless you set the [COMPOSER_ALLOW_SUPERUSER](https://getcomposer.org/doc/03-cli.md#composer-allow-superuser) environment variable.

Found a typo? Something is wrong in this documentation? [Fork and edit](https://github.com/composer/composer/edit/main/doc/faqs/how-to-install-untrusted-packages-safely.md "Go to the docs to fork and propose updates (external link)") it!