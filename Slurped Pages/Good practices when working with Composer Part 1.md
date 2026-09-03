---
link: https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c
byline: Timo Schinkel
site: DEV Community
date: 2020-09-11T15:25
excerpt: Dependency management in PHP has become a lot simpler with the introduction of Composer. The upcoming...
twitter: https://twitter.com/@timoschinkel
tags:
  - composer
slurped: 2026-08-22T08:37
title: Good practices when working with Composer
---

[Part 2](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f)
Dependency management in PHP has become a lot simpler with the introduction of [Composer](https://www.getcomposer.org/). The upcoming Composer 2.0 release is as good a reason as any to share some good practices I've learned in the years I have been using Composer.

## [](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c#what-is-composer)What is Composer

Composer is a dependency manager for PHP, similar to [NPM](https://www.npmjs.com/) or [Yarn](https://yarnpkg.com/) for Javascript and [pip](https://pypi.org/project/pip/) for Python. It simplifies the management of packages that your application depends on. As an additional "service" Composer adds the possibility for packages to register an autoloader. And that combination makes Composer one of the most critical utilities of the PHP ecosystem in my opinion.

The most used package repository for Composer is [Packagist](https://packagist.org/). This package repository only accepts open source packages. If you want to benefit from all the features Composer offers you can use [Private Packagist](https://packagist.com/). A paid version of Packagist build by the creators of Composer that allows closed source packages including granular access control.

I am not going to explain how to install and set up Composer in here. The maintainers of Composer explain that extensively in the [Getting Started](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos) section of the Composer website. For any examples where I use the Composer CLI I will assume Composer is installed globally and is called via `composer`.

## [](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c#split-production-and-development-dependencies)Split production and development dependencies

Composer makes a distinction between two types of dependencies; regular dependencies and development dependencies. The regular dependencies are the dependencies your code _always_ needs, independent of the environment the code is executed. Development dependencies are dependencies that are only needed when doing development on the code, eg. testing frameworks like PHPUnit, static code analysers like Psalm and code style analysers like PHP Code Sniffer. These are not necessary when running the code on a production environment. When adding a dependency via [`composer require [package]`](https://getcomposer.org/doc/03-cli.md#require) it will be marked a "production dependency". By using the flag `--dev` the package will be marked a "development dependency".

When running [`composer install`](https://getcomposer.org/doc/03-cli.md#install-i) both the production and the development dependencies of your application are downloaded and installed. In order to download and install only the production dependencies you add the flag `--no-dev`. This is typically what you want in your _deployment pipeline_ as you don't need the development dependencies there.

**NB** `composer install` will only download and install the development dependencies of _your application or package_ and not the development dependencies of dependencies. So if you have a dependency on "Package A" and "Package A" has a development dependency on "Package B" only "Package A" will be downloaded and installed in your project.

The development dependencies can acculumate to reasonable list of dependencies and subsequently a good amount of files. All those files are autoloadable via the autoloader. It can save both storage space, transfer time (during deployment) and execution time to minimize the amount of production dependencies in your application. The savings in storage space and transfer time might be neglectible looking at small scale, but as soon as your application will grow in size or maintainers this can amount to more substantial numbers. Another benefit of marking development dependencies is that they will not be included in the autoloader Composer constructs when running `composer install --no-dev`.

### [](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c#autoloading)Autoloading

As mentioned earlier an additional feature of Composer is that is offers utilities for packages to hook into PHP's [autoloading](https://www.php.net/manual/en/language.oop5.autoload.php). With this functionality packages can specify how Composer can match a class name with a filename. This frees developers from littering code with [`require_once`](https://www.php.net/manual/en/function.require-once.php) and [`require`](https://www.php.net/manual/en/function.require.php) calls. All you need to do is include `vendor/autoload.php`.

As with dependencies Composer offers two kinds of autoloaders; regular autoload directives and development autoload directives. Packages can utilize this to reduce their package size and their autoloader size, but applications can benefit from this as well. Assuming a well tested project the amount of files that are used just for testing can be a substantial part of the codebase. Similar to any development utilities like command line tooling. You don't need those in production. By marking the autoloader for those files a development autoloader these will not be considered when running `composer install --no-dev`, which can improve the performance of your application. The distinction between regular and development autoloading is made in the `composer.json` file:  

```
{
    "autoload": {
        "psr-4": {
            "MyApp\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "MyApp\\Tests": "tests/"
        }
    }
}
```

 

Composer offers some additional tools to improve performance of autoloading functions and classes via [Autoloader Optimization](https://getcomposer.org/doc/articles/autoloader-optimization.md).

## [](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c#version-constraints)Version constraints

Before Composer a developer would download a specific version of a dependency and put that in version control, since Composer we can step away from a specific version of a dependency and have Composer decide exactly what version of a dependency is to be installed. To determine this Composer relies heavily on the concept of [semantic versioning](https://semver.org/). Composer not only does this for your application, but extends this functionality to dependencies. This allows for packages to have their own dependencies, that can have their own dependencies and so on. This leads to more but smaller packages. Those smaller packages can focus on one functionality and thereby allowing centralization of this functionality.

Compared with for example NPM Composer allows only one version of a package per application. This means fewer packages need to be installed, but it does pose an additional risk of version collisions. That's a good reason to not use a specific version of a dependency, but request a _range of versions_. This is typically done via the caret operator: `^1.0`. This means that every version >= `1.0.0` and < `2.0.0` is accepted. Given semantic versioning this means that you are guaranteed of the required functionalities and no backwards compatibility breaking changes are introduced.

By specifying a range instead of a specific version you prevent version collisions when another dependency uses the same package. Let's assume the following situation:

The `composer.json` of your application:  

```
{
  "require": {
    "vendor/package": "^1.4",
    "another_vendor/package": "^1.0"
  }
}
```

 

The `composer.json` of `vendor/package`:  

```
{
  "require": {
    "another_vendor/package": "^1.2"
  }
}
```

 

When installing the dependencies Composer will detect that multiple requests for `another_vendor/package` are present, but because both are range constraints Composer can determine that it should _not_ install version `1.1.5`, but instead should install `1.2.8`.

Composer has multiple [constraint operators](https://getcomposer.org/doc/articles/versions.md#summary) and even allows combining operators. This means that a much used constraint `"php": "^7.2"` will _not_ include PHP 8. In order to indicate that both PHP 7.2 and higher _and_ PHP 8 are supported two operators can be combined: `"php": "^7.2 || ^8.0"`.

_NB_ Another option would be to take the approach [Symfony](https://github.com/symfony/symfony/pull/36876) has taken and opt for `"php": >= 7.2"`. I personally do not favor this approach as it states that the code will also work with PHP 9 and up. As there is no way of telling what backwards compatibility breaking changes will be introduced in PHP 9 my preference would be to not make such promises and that's the reason I'm planning on using `"php": "^7.2 || ^8.0"` on my repositories.

## [](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c#to-root-or-not-to-root)To root or not to root

Looking at a `composer.json` file a distinction can be made between dependencies required directly from your project - _root package_ in the Composer terminology - and dependencies that are required indirectly as a dependency of _root dependency_.

Let's take this bare `composer.json` file:  

```
{
    "require": {
        "laminas/laminas-diactoros": "^2.0"
    }
}
```

 

Running `composer install` on this file would show that not just the `laminas/laminas-diactoros` package is downloaded, but also `laminas/laminas-zendframework-bridge`, `psr/http-factory` and `psr-http-message`. In this example `laminas/laminas-diactoros` is a _root requirement_.

The difference between a root requirement and a derivative requirement lies in the amount of direct dependencies in your project. In general fewer root requirements means fewer packages and fewer conflicts when upgrading packages. Fewer packages because a new version of a package might have different dependencies and that will automatically be maintained by Composer. And fewer conflicts because there are fewer constraints in your dependency tree.

But what makes a package eligible to be a root requirement? My rule of thumb is that when a class - or interface - from a package is used in my domain code it should be a root requirement. Let's look at our example again:  

```
{
    "require": {
        "laminas/laminas-diactoros": "^2.0"
    }
}
```

 

Looking at the example where we rely only on `laminas/laminas-diactoros`. Diactoros offers an implementation of the interfaces specified in PSR-7 and PSR-15. If in my code I only refer to objects from the `laminas/laminas-diactoros` this would be a valid Composer configuration. But if I refer to the interfaces defined in PSR-7 and PSR-15 - which would be a good practice in my opinion - then that would be a reason to make `psr-http-message` and `psr/http-factory` a root requirement as well.  

```
{
    "require": {
        "laminas/laminas-diactoros": "^2.0",
        "psr-http-message": "^1.0",
        "psr/http-factory": "^1.0"
    }
}
```

 

This rule of thumbs means that a package performing actions based on PSR-7 objects only needs to mark `psr-http-message` as dependency, making it agnostic to the actual implementation of the interface. This is in essence the idea of the PSRs.

## [](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c#specifying-your-environment)Specifying your environment

You build your application or package on a given PHP version. And you only can only guarantee the correct functioning of your code on that version of PHP. Maybe your code requires a PHP extension to be installed. That's why it's a good practice to specify what version(s) of PHP are required or supported and what extensions are required.

When you're developing a package PHP versions and PHP extensions can be used as if these are regular packages:  

```
{
  "require": {
    "php": "^7.2",
    "ext-curl": "*"
  }
}
```

 

The reason we can use the wildcard `*` for the cURL extension is that the extension is tied to PHP and therefore specifying the version is not needed.

You can also use this approach when you're building an application, additionally you can specify these restraints as platform configuration:  

```
{
  "config": {
    "platform": {
      "php": "7.4.4",
      "ext-curl": "*"
    }
  }
}
```

 

The benefit of this is that Composer will use PHP version 7.4.4 as PHP version even if a different version is installed on the system where the Composer commands are run. With the trend of using virtualization solutions like Docker and Vagrant this allows users to run Composer commands locally and run the application within the virtualized environment. This also prevents that one colleague that runs a bleeding edge PHP version to install versions of dependencies that are not supported by the production environment.

## [](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c#working-with-source-control)Working with source control

Within your project Composer will have three entities on your filesystem; two files called `composer.json` and `composer.lock` and a folder called `vendor`*. But not all three need to be included in your source control.

The file `composer.json` contains all packages that are required with version constraints - see section "Version constraints" for more information. This file should _always_ be added to version control. Without this file a user that wants to use your code has no clue as to what packages are required.

The file `composer.lock` contains the exact versions, including checksums, of _all_ packages. If this file is present when running `composer install` it will install the exact same versions as specified in `composer.lock`. This feature makes that you can ensure that the dependencies on all environments are the same. When building an application you _do_ want to add this file to version control. When building a library you _do not_ want to add this file to version control. Main reason for this is that applications (or other packages) that use your package will not take this file into account, but your (local) development environment does. This can lead to you developing or testing against outdated versions of your dependencies, where consumers of your package do use the latest versions.

The folder `vendor` contains the code of the packages. The contents of this folder should not be added to source control. This folder is populated when running `composer install`.

* You _can_ change the name and location of this folder via the configuration option [`vendor-dir`](https://getcomposer.org/doc/06-config.md#vendor-dir), but for this article I assume the default.

### [](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c#making-changes-in-dependencies)Making changes in dependencies

Let's assume you want to update a dependency of your application. Or maybe you want to update multiple. You run a number of `composer update` commands and you see `composer.lock` change. You create a commit. Another batch of updates and another commit. Your attention shifts and a few days later you pick up where you left off. You're working with a busy repository so before you continue you rebase to ensure your changes are compatible with the latest version of the code. Merge conflict! A merge conflict in `composer.json` can be solved like any other merge conflict, but a merge conflict in `composer.lock` is harder to resolve.

The way to solve merge conflicts in a `composer.lock` file is to accept the incoming changes - these are the latest changes - and run your Composer commands that you ran to make the changes in the first place again. This will ensure that your changes will be applied on the most recent version of the codebase. Commit the new `composer.lock` and continue the rebase. If you did some more Composer work in a next commit you have to go through these steps again. My solution to avoid this:

- Changes in dependencies in a single commit
- List the Composer commands that were run
- Reapply these commands when `composer.lock` has a merge conflict

## [](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c#conclusion)Conclusion

Composer has become a vital part of PHP development. As with any package manager it might need some guidance. I have been using Composer daily in multiple active repositories and in that period I have been using some practices that I see as good practices:

- Split your dependencies in production and development dependencies to decrease the application size.
- Split your autoloader in production and development autoloader to decrease autoloader size and increase performance.
- Don't depend on fixed versions of dependencies, but rather on a range to make upgrading of packages less painful.
- Only directly depend on dependencies you actually call inside your application to make upgrading or switching to an alternative simpler.
- _Always_ put `composer.json` in source control, but only put `composer.lock` in source control for applications to prevent users of your package being unable to upgrade other packages.
- Try to have a maximum of one commit per pull request/merge request to contain changes to `composer.json` and `composer.lock` to minimize merge conflicts in these files.