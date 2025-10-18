---
link: https://matthewsetter.com/composer-beyond-the-basics/
site: Software Engineer and Technical Author, specialising in PHP, Go, and DevOps
date: 2025-01-20T01:00
excerpt: Are you getting the most out of Composer? In this short tutorial, you’re going to learn about more of its functionality including removing and updating dependencies, and configuring autoload namespaces.
slurped: 2025-10-18T12:41
title: Composer. Beyond the Basics
tags:
  - composer
---

Are you getting the most out of Composer? In this short tutorial, you’re going to learn about more of its functionality including removing and updating dependencies, and configuring autoload namespaces.

In [the previous post on the blog](https://matthewsetter.com/getting-started-with-composer/), I stepped through the essentials of using Composer, PHP’s **excellent** package manager, by showing how you’d use it to simplify building a small web-based application, built around [the Slim framework](https://www.slimframework.com/).

If you haven’t read it, it shows how to:

- Add Composer support to a project
- Define all of the required metadata
- Configure autoload namespaces
- Add packages; and
- Add scripts to simplify development

I feel that it was a great introduction, but also that the post could have gone a lot further. This is because Composer is such a richly-featured project that one tutorial can’t cover everything you need to know.

So, in this tutorial I’m going to show you some of that deeper, more advanced functionality, including:

- Removing and updating dependencies
- Handling package conflicts
- Finding out which packages are installed
- Finding out why a given package was installed
- Finding out if/when a package has updates available; and
- How to use custom Composer repositories (in addition to [Packagist](https://packagist.org/))

Let’s, get into it!

## How to update dependencies

From time to time, no matter what package you use, unless it’s been abandoned by it’s maintainer(s), it’s going to get updates. This can happen for a number of reasons, but likely mostly because of security updates, adding or deprecating functionality, and to take advantage [of newer versions of PHP](https://www.twilio.com/en-us/blog/whats-new-php84).

Regardless of the reason – but especially if it’s a security update – you’ll likely want to upgrade your version of that package. To do that, use [Composer’s update command](https://getcomposer.org/doc/03-cli.md#update-u-upgrade), by running the command below.

The command will check if there are newer versions of any of the packages defined in your project’s _composer.json_ file, based on [the version constraints](https://getcomposer.org/doc/04-schema.md#package-links) that you’ve provided.

If there are, it will update them to the latest versions possible, and also update _composer.lock_, so that the next time your application is installed, it will install the newer versions instead of the previously installed versions.

You can pass a number of options to the update command, if you’d like (or need) to be a bit more precise about what is updated. Some notable options are:

- `--patch-only`: Only allow patch version updates for currently installed dependencies.
- `--prefer-stable`: Prefer stable versions of dependencies
- `-W` / `--with-all-dependencies`: Update also dependencies of packages in the argument list, including those which are root requirements.

Additionally, you can update individual dependencies if you want. To do that, pass them as arguments to the command, as in the following examples:

```
# Update Twig to the latest available version
composer update twig/twig

# Update Twig and psr/container to the latest available version
composer update twig/twig psr/container

# Update Twig to the latest available version and psr/container to version 2.0.2
composer update twig/twig psr/container:2.0.2
```

## How to remove dependencies

In addition to updating dependencies, there’ll be times when you need to _remove_ dependencies. As mentioned in the previous section, for various reasons packages can be abandoned, leaving you to look for an alternative. Other times, PRs aren’t implemented as quickly as you’d like or want, forcing you to look for an alternative to meet your project deadlines.

Whatever the reason, here’s how to remove one or more packages. Firstly, you can manually remove them from _composer.json_ and run `composer update`. However, I find that can get a little messy and time-consuming. So instead, use [the composer remove command](https://getcomposer.org/doc/03-cli.md#remove-rm-uninstall) which:

> Removes a package from the require or require-dev

Also known as the `rm` or `uninstall` command, all you need to do is to pass one or more packages to the command and Composer will:

- Remove them from _composer.json_ for you; and
- Remove them from the _vendor_ directory in your project (along with every dependency which only that dependency required), and update _composer.lock_

For example, if you wanted to migrate from [Pest](https://pestphp.com/) to [PHPUnit](https://phpunit.de/index.html) for writing tests in your application, you’d remove Pest with the following command:

```
composer remove pestphp/pest
```

What if you didn’t want to remove the package straight away, but wanted to know what would happen _if_ you were to remove it? For that, you’d use the command’s `--dry-run` option. I don’t have Pest installed, as I’m a PHPUnit fan, but I do have [Mini Mezzio](https://github.com/asgrim/mini-mezzio) installed on a package I’ve been working on lately. To do a dry run of removing it, I’d run the following command.

```
composer remove --dry-run asgrim/mini-mezzio
```

Here’s the command’s output:

```
./composer.json has been updated
Running composer update asgrim/mini-mezzio
Loading composer repositories with package information
Updating dependencies
Lock file operations: 0 installs, 0 updates, 6 removals
  - Removing asgrim/mini-mezzio (dev-add-php8.4-support 753577f)
  - Removing laminas/laminas-diactoros (3.5.0)
  - Removing laminas/laminas-escaper (2.16.0)
  - Removing laminas/laminas-httphandlerrunner (2.11.0)
  - Removing laminas/laminas-stratigility (3.13.0)
  - Removing mezzio/mezzio (3.20.1)
Installing dependencies from lock file (including require-dev)
Package operations: 0 installs, 0 updates, 6 removals
  - Removing mezzio/mezzio (3.20.1)
  - Removing laminas/laminas-stratigility (3.13.0)
  - Removing laminas/laminas-httphandlerrunner (2.11.0)
  - Removing laminas/laminas-escaper (2.16.0)
  - Removing laminas/laminas-diactoros (3.5.0)
  - Removing asgrim/mini-mezzio (dev-add-php8.4-support 753577f)
52 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
No security vulnerability advisories found.
```

You can see that Composer would update _composer.json_, then remove Mini Mezzio along with its dependencies, laminas/laminas-diactoros, laminas/laminas-escaper, laminas/laminas-httphandlerrunner, and laminas/laminas-stratigility.

The command supports a host of other options, such as `--unused` which removes all packages which are locked but not required by any other package, and `--dev` which removes a development dependency.

Check out [the command’s documentation](https://getcomposer.org/doc/03-cli.md#remove-rm-uninstall) for full details.

## How to handle package conflicts

Despite being one of the key problems that package manager’s solve, no package manager that I’ve yet used has been able to avoid package conflicts completely.

Sure, they reduce the likelihood no end, but there’s always the possibility that the combination of packages that you’re trying to install (or upgrade to) might result in a conflict.

Before I go any further, Composer can’t solve conflicts for you directly. However, it often does an excellent job of showing you what the conflicts are caused by, and gives you tooling to help you solve the conflicts for yourself.

Let’s start off with a simple example of a package conflict, so that you can see what happens. Let’s say that you want to build a small API using Mini Mezzio, specifically version 2.1.0, and attempt to add the package with the following command:

```
composer require "asgrim/mini-mezzio:2.1.0"
```

Because you’re running PHP 8.4, which that version of Mini Mezzio doesn’t support, you’re going to encounter a package conflict. Here’s what you’re going to see in the terminal:

```
./composer.json has been created
Running composer update asgrim/mini-mezzio
Loading composer repositories with package information
Updating dependencies
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - Root composer.json requires asgrim/mini-mezzio 2.1.0 -> satisfiable by asgrim/mini-mezzio[2.1.0].
    - asgrim/mini-mezzio 2.1.0 requires php 8.1.*|8.2.*|8.3.* -> your php version (8.4.4) does not satisfy that requirement.
```

Composer has created a minimalist _composer.json_ file with only a `require` section, and added `asgrim/mini-mezzio 2.1.0`, and was about to install it. However, as your installed PHP version is 8.4.4 and the package only allows PHP 8.1 - 8.3, you can’t install the package.

To be fair, you could ignore the PHP version requirement by passing [the `--ignore-platform-reqs` option](https://getcomposer.org/doc/03-cli.md#require-r), but I wouldn’t (at least not more than occasionally).

Anyway, to solve this problem, you’d need to use an earlier version of PHP (if that’s practical), or have Composer look for a later version of Mini Mezzio (which is available) using the `--with-all-dependencies` (or `-W`) option to `composer update`.

Using a different version of PHP or the package that you want might not be possible, because you may be working for a client/employer who uses earlier versions. Alternatively, a later version of the package might not be available, yet.

In my case, this was a fairly easy problem to solve. However, conflicts that you’ll encounter will likely not be quite so easy to solve. Let’s say that you’ve taken over an older PHP application, and want to update it to support the latest version of PHP. I had such an application available and decided to start updating the dependencies in _composer.json_ to the latest versions before running `composer update`, which lead to the following conflict output:

```
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - laminas/laminas-component-installer is locked to version 3.2.0 and an update of this package was not requested.
    - laminas/laminas-component-installer 3.2.0 requires php ~8.0.0 || ~8.1.0 || ~8.2.0 -> your php version (8.4.4) does not satisfy that requirement.
  Problem 2
    - laminas/laminas-config-aggregator is locked to version 1.13.0 and an update of this package was not requested.
    - laminas/laminas-config-aggregator 1.13.0 requires php ~8.0.0 || ~8.1.0 || ~8.2.0 -> your php version (8.4.4) does not satisfy that requirement.
  Problem 3
    - laminas/laminas-diactoros is locked to version 2.25.2 and an update of this package was not requested.
    - laminas/laminas-diactoros 2.25.2 requires php ~8.0.0 || ~8.1.0 || ~8.2.0 -> your php version (8.4.4) does not satisfy that requirement.
  Problem 4
    - Root composer.json requires laminas/laminas-servicemanager ^4.4.0 -> satisfiable by laminas/laminas-servicemanager[4.4.0].
    - laminas/laminas-servicemanager 4.4.0 requires laminas/laminas-stdlib ^3.19 -> found laminas/laminas-stdlib[3.19.0, 3.20.0] but the package is fixed to 3.16.1 (lock file version) by a partial update and that version does not match. Make sure you list it as an argument for the update command.
  Problem 5
    - laminas/laminas-stdlib is locked to version 3.16.1 and an update of this package was not requested.
    - laminas/laminas-stdlib 3.16.1 requires php ~8.0.0 || ~8.1.0 || ~8.2.0 -> your php version (8.4.4) does not satisfy that requirement.
  Problem 6
    - mezzio/mezzio is locked to version 3.15.0 and an update of this package was not requested.
    - mezzio/mezzio 3.15.0 requires php ~8.0.0 || ~8.1.0 || ~8.2.0 -> your php version (8.4.4) does not satisfy that requirement.
  Problem 7
    - mezzio/mezzio-fastroute is locked to version 3.8.0 and an update of this package was not requested.
    - mezzio/mezzio-fastroute 3.8.0 requires php ~8.0.0 || ~8.1.0 || ~8.2.0 -> your php version (8.4.4) does not satisfy that requirement.
  Problem 8
    - mezzio/mezzio-helpers is locked to version 5.13.1 and an update of this package was not requested.
    - mezzio/mezzio-helpers 5.13.1 requires php ~8.0.0 || ~8.1.0 || ~8.2.0 -> your php version (8.4.4) does not satisfy that requirement.
  Problem 9
    - laminas/laminas-development-mode is locked to version 3.10.0 and an update of this package was not requested.
    - laminas/laminas-development-mode 3.10.0 requires php ~8.0.0 || ~8.1.0 || ~8.2.0 -> your php version (8.4.4) does not satisfy that requirement.
  Problem 10
    - mezzio/mezzio-tooling is locked to version 2.8.0 and an update of this package was not requested.
    - mezzio/mezzio-tooling 2.8.0 requires php ~8.0.0 || ~8.1.0 || ~8.2.0 -> your php version (8.4.4) does not satisfy that requirement.
```

Gladly, this conflict output was pretty easy to parse. All I had to do was to keep updating the remaining packages to the latest versions and it would be resolved. But, sometimes it’s not that simple, as you end up in somewhat of a dependency circle where one depends on another, but you can update it until you’ve updated the first.

I wasn’t able to generate such a conflict to show you, but you’ll know it if you see it. That will take a bit of time and patience to resolve, but Composer’s output is pretty good at showing you what you need to do; within reason.

## How to find out which packages are installed

This might not be as interesting to you as it is to me. But, I know that, from time to time, I like to find out which packages are installed for a given application or library that I’m working on; not just the ones that I’ve directly listed in _composer.json_.

If that’s something that you want (or need) to know, then you’ll need [Composer’s show (or info) command](https://getcomposer.org/doc/03-cli.md#show-info). It lists all of the installed packages, both direct and transitive, in alphabetical order of package name.

Here’s an excerpt from an app I’ve been working on recently:

```
composer show
asgrim/mini-mezzio                             dev-add-php8.4-support 753577f Minimal Application Factory for Mezzio
brick/varexporter                              0.5.0                          A powerful alternative to var_export(), which can export closures and objects without __set_state()
dealerdirect/phpcodesniffer-composer-installer 1.0.0                          PHP_CodeSniffer Standards Composer Installer Plugin
dflydev/dot-access-data                        3.0.3                          Given a deep data structure, access data by dot notation.
fig/http-message-util                          1.1.5                          Utility classes and constants for use with PSR-7 (psr/http-message)
laminas/laminas-coding-standard                3.0.1                          Laminas Coding Standard
laminas/laminas-diactoros                      3.5.0                          PSR HTTP Message implementations
```

You can see the output shows the package’s name, currently installed version, and description.

If you want the path to the package as well, add the `-P` or `--path` option to the command. And, to sort the output by oldest to newest packages, use the `-A` or `--sort-by-age` options.

Now, you might want more information than that, such as each dependency’s dependencies, all the way down to the required PHP version. To do that, you need to use the command’s `-t` or `--tree` option. This lists the dependencies as a tree, and in so doing provides much more information.

Here’s an excerpt from the same package I’ve been working on:

```
├──mezzio/mezzio-router ^3.7
│  ├──fig/http-message-util ^1.1.5
│  │  └──php ^5.3 || ^7.0 || ^8.0
│  ├──php ~8.1.0 || ~8.2.0 || ~8.3.0 || ~8.4.0
│  ├──psr/container ^1.1.2 || ^2.0
│  │  └──php >=7.4.0
│  ├──psr/http-factory ^1.0.2
│  │  ├──php >=7.1
│  │  └──psr/http-message ^1.0 || ^2.0
│  │     └──php ^7.2 || ^8.0
│  ├──psr/http-message ^1.0.1 || ^2.0.0
│  │  └──php ^7.2 || ^8.0
│  ├──psr/http-server-middleware ^1.0.2
│  │  ├──php >=7.0
│  │  ├──psr/http-message ^1.0 || ^2.0
│  │  │  └──php ^7.2 || ^8.0
│  │  └──psr/http-server-handler ^1.0
│  │     ├──php >=7.0
│  │     └──psr/http-message ^1.0 || ^2.0
│  │        └──php ^7.2 || ^8.0
│  └──webmozart/assert ^1.11
│     ├──ext-ctype *
│     │  └──php >=7.2
│     └──php ^7.2 || ^8.0
```

You can see that _mezzio/mezzio-router_ requires _fig/http-message-util_, _psr/container_, _psr/http-factory_, and so on. If you’re familiar with tree output, you’ll know that the output above is part of a larger output snippet. So, to show details for just _mezzio/mezzio-router_, I’d pass that to the command, as in the example command below.

```
composer show --tree mezzio/mezzio-router
```

As with most Composer commands, there are a wealth of options you can use to filter and sort the output, so make sure you go through [the command’s documentation](https://getcomposer.org/doc/03-cli.md#show-info).

## How to find out why a given package was installed

Instead of wanting to know all of the packages that are installed, let’s say that as you’re working doing some [step-through debugging an error](https://matthewsetter.com/setup-step-debugging-php-xdebug3-docker/), you see a package and wonder why a given package is installed. To find that which package requires it, you’d use [Composer’s depend command](https://getcomposer.org/doc/03-cli.md#depends-why). This command:

> Shows which packages cause the given package to be installed

You call the command passing it the name of the package you want to know more about, as in the example below.

```
composer depends --locked mezzio/mezzio-router
```

The command will print output similar to the example below.

```
asgrim/mini-mezzio         dev-add-php8.4-support requires mezzio/mezzio-router (^3.16)
mezzio/mezzio              3.20.1                 requires mezzio/mezzio-router (^3.7)
mezzio/mezzio-fastroute    3.12.0                 requires mezzio/mezzio-router (^3.14)
mezzio/mezzio-helpers      5.17.0                 requires mezzio/mezzio-router (^3.0)
mezzio/mezzio-twigrenderer 2.17.0                 requires mezzio/mezzio-router (^3.7)
```

The command outputs the name of each package that depends on the named package, along with its currently installed version, and the minimum version of the nominated package that each package stipulates. For example, mezzio/mezzio 3.20.1 requires a minimum mezzio/mezzio-router version 3.7.

## How to find out if/when a package has updates available

Now, not that I use this all that often, but it can be handy to know if a package has updates available, rather than running `composer update` and updating all packages in a somewhat blunt way.

To do that, you can use [Composer’s show command](https://getcomposer.org/doc/03-cli.md#show-info) with the `-o` or `--outdated` option, or the `outdated` alias. This command:

> Shows a list of installed packages that have updates available, including their latest version.

Here’s an example of running it with no options and an excerpt of the output that it printed to the terminal, in a package that I’ve been working on recently.

```
composer outdated
Color legend:
- patch or minor release available - update recommended
- major release available - update possible

Direct dependencies required in composer.json:
phpunit/phpunit              11.5.8 12.0.4 The PHP Unit Testing framework.

Transitive dependencies not required in composer.json:
brick/varexporter            0.5.0  0.6.0  A powerful alternative to var_export(), which can export closures and objects...
laminas/laminas-stratigility 3.13.0 4.1.0  PSR-7 middleware foundation for building and dispatching middleware pipelines
phpstan/phpdoc-parser        1.33.0 2.1.0  PHPDoc parser with support for nullable, intersection and generic types
phpunit/php-code-coverage    11.0.8 12.0.3 Library that provides collection, processing, and rendering functionality for...
phpunit/php-file-iterator    5.1.0  6.0.0  FilterIterator implementation that filters files based on a list of suffixes.
phpunit/php-invoker          5.0.1  6.0.0  Invoke callables with a timeout
phpunit/php-text-template    4.0.1  5.0.0  Simple template engine.
phpunit/php-timer            7.0.1  8.0.0  Utility class for timing
sebastian/cli-parser         3.0.2  4.0.0  Library for parsing CLI options
sebastian/comparator         6.3.0  7.0.0  Provides the functionality to compare PHP values for equality
sebastian/complexity         4.0.1  5.0.0  Library for calculating the complexity of PHP code units
sebastian/diff               6.0.2  7.0.0  Diff implementation
```

You can see that the output’s split into two sections. The top section shows updates for packages that have been directly listed in the project’s _composer.json_ file, or **direct** dependencies.

Then, it shows updates for packages that were installed because of either a direct dependency or of a dependency of a dependency, referred to as a **transitive** dependency.

For each package, regardless of the section, you see its name, currently installed version, latest available version, and description. You can filter the output out on a range of criteria, including only considering direct dependencies, and packages that have major, minor, or patch updates available.

Here’s an example of searching for outdated packages that have a major version update available, and sorting them by age:

```
composer show --outdated --major-only --sort-by-age
```

Check out [the command’s documentation](https://getcomposer.org/doc/03-cli.md#show-info) for more filter options.

## How to use alternative repositories

This is one of the features that I most love about Composer – you can override which repositories it pulls packages from. This might not make a lot of sense on first hearing it, or you might wonder why you might want to. So here’s the scenario I use most often, if I need to: using forks of a given package.

Say, for example, there’s a package that you use, such as [laminas-validator](https://docs.laminas.dev/laminas-validator/), and you find a bug. After reporting the bug, you can either create a PR for it yourself if you’re so inclined and have the required technical knowledge, or wait for someone else to do so.

Regardless of what you do you’ll have to wait, and that bug’s going to be a blocker on your app’s development time.

Do you have the time to wait? Do you _want_ to wait? You’ll likely answer “no” to both questions.

So, let’s say that you’ve created a PR to fix the bug in your fork of the project repository. While you’re waiting for it to be accepted and merged, you can have Composer refer to your fork instead of [Packagist](https://packagist.org/) by using [the `repositories` key in _composer.json_](https://getcomposer.org/doc/05-repositories.md), and then tell Composer to use your development branch in your fork.

Let’s assume I’d done just that for laminas-validator. Having forked it on GitHub it would be available at [https://docs.laminas.dev/laminas-validator/](https://docs.laminas.dev/laminas-validator/). So, I’d add the following to a given project’s composer.json that used the package to have Composer use my fork instead of Packagist.

```
{
    "repositories": [
        {
            "type": "vcs",
            "url": "http://github.com/settermjd/laminas-validator"
        }
    ]
}
```

Then, I’d update the `require` section to refer to my development branch, as in the following example, where I’ve prefixed the branch “my-dev-branch” with `dev-` to indicate that it’s a branch, not a tag.

```
"require": {
    "laminas/laminas-validator": "dev-my-dev-branch"
},
```

You can see that in the `repositories` array, there’s one object. Its type is set to “vcs” indicating a version control system, and its URL is set to a GitHub repository; this could point to any Git hosting service, such as another commercial one like GitLab, or a private, self-hosted one such as using Gitea.

There are two other [types](https://getcomposer.org/doc/05-repositories.md#types): “composer” and “package”, as well as [self-hosted repositories](https://getcomposer.org/doc/05-repositories.md#hosting-your-own). For the sake of this tutorial, I’m sticking with “vcs”.

Regardless, now, when running `composer install` and `composer update`, for laminas-validator, Composer will use [http://github.com/settermjd/laminas-validator](http://github.com/settermjd/laminas-validator) instead of the official one. Then, after the PR is accepted and merged, I’d revert the changes in _composer.json_ and continue on as before.

## And that’s Composer, beyond the basics

I hope that you found it helpful, enlightening, and even interesting to learn more about Composer’s functionality. As I say regularly, Composer is an excellent package manager, one which has benefited PHP immeasurably. I hope that this tutorial will help you take greater advantage of it, and get the most out of it. Let me know your power tips and tricks in the comments below.