---
link: https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f
byline: Timo Schinkel
site: DEV Community
date: 2021-07-28T13:58
excerpt: In Good practices when working with Composer I discussed some basics
  about Composer. Something that...
twitter: https://twitter.com/@timoschinkel
tags:
  - slurp/php
  - slurp/composer
  - slurp/software
  - slurp/coding
  - slurp/development
  - slurp/engineering
  - slurp/inclusive
  - slurp/community
slurped: 2026-08-22T08:43
title: Keeping dependencies up-to-date in Composer
---

In [Good practices when working with Composer](https://dev.to/timoschinkel/good-practices-when-working-with-composer-5a1c) I discussed some basics about [Composer](https://getcomposer.org/). Something that is not discussed in that article is the importance of keeping your dependencies up-to-date. This goes for both libraries and applications. In this article I will focus on applications - codebases that also maintain a `composer.lock` file in source control. The majority of the tips and tricks however can also be used for libraries.

## [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#why-should-i-keep-my-dependencies-uptodate)Why should I keep my dependencies up-to-date?

Not updating your dependencies has its benefits - you are guaranteed that the interface of the dependency will not change for example. It has some downsides as well however; You will miss out on security updates, new features and improvements.

Another reason to keep your dependencies up-to-date is to keep your migrations small. It is - usually - simpler to migrate a minor version than to migrate a major version[1](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#fn1) as minor upgrades usually contain fewer changes and _should_ not introduce breaking changes.

## [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#composer-outdated)composer outdated

The simplest way to find out if you have dependencies that are not up-to-date is to run [`composer outdated`](https://getcomposer.org/doc/03-cli.md#outdated). This command will output a list of all dependencies - both direct dependencies and indirect dependencies[2](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#fn2) -, their current version in your `composer.lock` and their most recent version. Dependencies that are up-to-date are by default **not** shown. If you do want to show these add the flag `-a` or `--all`. This is an excerpt of the output of `composer outdated --all` I ran on one of my codebases:  

```
phpunit/phpunit                       9.5.6     9.5.6     The PHP Unit Testing framework.
psr/container                         1.1.0     2.0.1     Common Container Interface (PHP FIG PSR-11)
psr/log                               1.1.3     1.1.4     Common interface for logging libraries
```

 

It is good to know that `composer outdated` does not take any version constraints into account when compiling the list. The version constraint for `psr/container` is set to `^1.1`, but `composer outdated` shows `2.0.1` as the most recent version.

Apart from the `--all` flag a few other useful flags are available for `composer outdated`. The documentation of Composer explains these flags very well.

## [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#updating-packages)Updating packages

So what if one or more dependencies is outdated? You update them! To update dependencies two commands can be used: `composer update` and `composer require`. The difference between these two commands is that `composer update` will try to update a dependency based on the current constraints in `composer.json` and will only update `composer.lock`. With `composerrequire` however Composer will try to install the latest version - keeping existing dependencies and platform constraints in mind - and will update both `composer.json` _and_ `composer.lock`. This difference also means that `composer update` typically will not update a package to a new major version.

### [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#composer-update)composer update

Taking the output from `composer outdated` from before we can run `composer update`. This will try to update all dependencies:  

```
> composer update
Loading composer repositories with package information
Updating dependencies
Lock file operations: 0 installs, 2 updates, 0 removals
  - Upgrading psr/container (1.1.0 => 1.1.1)
  - Upgrading psr/log (1.1.3 => 1.1.4)
Writing lock file
```

 

When you're working on a large codebase with a lot of dependencies, updating all of them might result in a lot of new packages. This can be undesirable as you might not know how and where all these dependencies are used. If that is the case you can limit the dependencies to be updated by specifying them:  

```
composer update psr/container psr/log
```

 

The side effect of specifying what dependencies should be updated is that Composer will _not_ update any other packages. This can be a problem when you are trying to update to a new minor version, but one of the indirect dependencies needs a newer version as well. Assume that I have installed [`timoschinkel/codeowners-cli`](https://packagist.org/packages/timoschinkel/codeowners-cli) with the constraint `^1.0` and I have currently installed `1.0.0`. This library depends on `timoschinkel/codeowners:^1.0` and version `1.0.0` is therefore installed. A new version of `timoschinkel/codeowners-cli` is released, tagged `1.1.0` and now it requires `timoschinkel/codeowners:^1.1`. Running `composer update timoschinkel/codeowners-cli` will result in Composer not updating the dependency:  

```
> composer update timoschinkel/codeowners-cli
Loading composer repositories with package information
Updating dependencies
Nothing to modify in lock file
Writing lock file
Installing dependencies from lock file (including require-dev)
Nothing to install, update or remove
```

 

Composer will see that the constraint `timoschinkel/codeowners:^1.1` does not match the already installed version `1.0.0`. We can specify a specific version to be installed. Maybe Composer just needs some help:  

```
> composer update timoschinkel/codeowners-cli:1.1.0
Loading composer repositories with package information
Updating dependencies
Your requirements could not be resolved to an installable set of packages.

  Problem 1
    - Root composer.json requires timoschinkel/codeowners-cli ^1.0, 1.1.0 -> satisfiable by timoschinkel/codeowners-cli[1.1.0].
    - timoschinkel/codeowners-cli 1.1.0 requires timoschinkel/codeowners ^1.1.0 -> found timoschinkel/codeowners[1.1.0] but the package is fixed to 1.0.0 (lock file version) by a partial update and that version does not match. Make sure you list it as an argument for the update command.

Use the option --with-all-dependencies (-W) to allow upgrades, downgrades and removals for packages currently locked to specific versions.
```

 

One option to solve this would be to add `timoschinkel/codeowners` to the list of packages to update as well. This could however lead to just another dependency that is causing the same problem. The solution is to use either `-W` or `-w` with the `composer update` command - as Composer tells us already in the output. What this flag will do is also update dependencies of the dependencies that you are updating. The difference between `-w` (lowercase) and `-W` (uppercase) is that the latter will also update any direct dependencies.  

```
> composer update timoschinkel/codeowners-cli -w
Loading composer repositories with package information
Updating dependencies
Lock file operations: 0 installs, 2 updates, 0 removals
  - Upgrading timoschinkel/codeowners (1.0.0 => 1.1.0)
  - Upgrading timoschinkel/codeowners-cli (1.0.0 => 1.1.0)
Writing lock file
```

 

### [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#composer-require)composer require

As mentioned before, running `composer update` will seek newer versions of your dependencies _while taking the version constraints into account_. That means that a new major version is not likely to be installed. I found that the easiest way to upgrade a dependency to a new major version is to use `composer require`. This will work in the same way as when you would require a dependency that is not already present in your `composer.json`; It will find the most recent version - taking any existing constraints into account - and updates the `composer.json` with the closest minor version:  

```
> composer require psr/log
Using version ^3.0 for psr/log
./composer.json has been updated
Running composer update psr/log
Loading composer repositories with package 
Lock file operations: 0 installs, 1 update, 0 removals
  - Upgrading psr/log (1.1.4 => 3.0.0)
Writing lock file
```

 

Similar to `composer update` you can specify a specific version, require multiple dependencies, and you can use `-w` and `-W` if needed. Nice side effect is that Composer will verify if any indirect dependencies are now no longer needed and will remove them from your `composer.lock`.

**NB** When using `composer require` on an existing development dependency don't forget to use the `--dev` flag, otherwise Composer will mark the dependency a production dependency.

### [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#when-to-update-a-dependency-constraint)When to update a dependency constraint

Let's assume your codebase uses PHPUnit via the constraint `^9.4`. You are currently using version `9.4.4` and you are updating to version `9.5.7`. Should you update the constraint in `composer.json` to `^9.5`?

Your constraint should reflect the versions that your codebase supports. Does your codebase require a feature introduced in `9.5`? In that case you should definitely update the constraint to `^9.5` and thus update via `composer require --dev phpunit/phpunit:^9.5 -w`. Are you updating just the dependency with the purpose of running the most recent version? In that case there is no need to update the constraint and thus update via `composer update phpunit/phpunit -w`.

## [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#debugging-updates)Debugging updates

Composer offers two commands that are instrumental when maintaining larger codebases; [`composer why`](https://getcomposer.org/doc/03-cli.md#depends-why-) and [`composer why-not`](https://getcomposer.org/doc/03-cli.md#prohibits-why-not-)[3](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#fn3). Just as the names imply can you use these commands to ask Composer why a certain dependency is available or why a certain dependency cannot be installed.

The main use case for `composer why` is to find out why an indirect dependency is present in your `composer.lock`:  

```
> composer why paragonie/random_compat
ramsey/uuid  3.9.3  requires  paragonie/random_compat (^1 | ^2 | 9.99.99)
```

 

`composer why-not` can tell you the exact opposite. I have found this to be very useful when I find myself in a situation where I'm unable to update an indirect dependency to a specific version:  

```
> composer why-not paragonie/random_compat:9.99.100
ramsey/uuid  3.9.3  requires  paragonie/random_compat (^1 | ^2 | 9.99.99)
```

 

### [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#hidden-power)Hidden power

`composer why-not` has a "hidden power"; It also works for requirements like PHP version. This realisation was a very big help in preparing our codebases for the migration to PHP 8:  

```
> composer why-not php:8.0.0
paragonie/random_compat  v9.99.99  requires  php (^7)
```

 

## [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#automation)Automation

I've used different approaches trying to keep dependencies up-to-date. For a while I blocked an afternoon every month to update outdated dependencies. This works great as long as you have a limited number of codebases and dependencies to keep up-to-date. As these grow a few hours every month is no longer enough. Luckily this can be automated to a certain point.

The most known tool for this is [Dependabot](https://dependabot.com/). Dependabot integrates seemlessly into Github and is able to create pull requests for outdated dependencies. If you have set up automated tests on your codebase all you have to do is merge the pull request created by Dependabot. It does not get any easier.

If you're not using Github or you don't want to use Dependabot, there's also the possibility to build something yourself. `composer oudated` can be "configured" to help you with this:  

```
composer outdated --format json --strict
```

 

The flag `--strict` will make that the Composer executable will return a non-zero status code if any dependency is outdated, where `--format json` switches the output from text to json. The latter is much easier to parse. With the output of this call you can create a pull request, an issue or an alert to whatever communication channel you prefer. All you need to do is run this periodically.

## [](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#tldr)tl/dr;

There are multiple reasons why it's a good idea to keep your dependencies as up-to-date as possible, ranging from preventing vulnerabilities to making future migrations easier. Composer is well equipped to detect outdated dependencies and to update them without you needing to open an editor to change `composer.json`. If you want to update to a version supported by the constraints you can use `composer update`. If you need - or want - to update the constraints as well you can use `composer require`.

If Composer gives you an error trying to update a dependency you can find the culprit by running `composer why-not`. A good way pf preventing blocking dependencies is to use either `-w` or `-W` when running `composer update` or `composer require`.

---

1. See [Semantic Versioning](https://semver.org/) for the difference between minor and major versions. [↩](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#fnref1)
    
2. Direct dependencies are dependencies that are specified in the `composer.json` of your project [↩](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#fnref2)
    
3. Officially these commands are called `depends` and `prohibits`, but I have a personal preference for using `why` and `why-not` as it almost makes your command into a grammatically correct sentences. [↩](https://dev.to/timoschinkel/keeping-dependencies-up-to-date-in-composer-5g4f#fnref3)