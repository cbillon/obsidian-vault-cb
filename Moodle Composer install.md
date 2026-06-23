---
tags:
  - moodle
  - composer
  - install
---
# The Seed to grow a new Moodle site

[github](https://github.com/moodle/seed#adding-a-moodle-plugin)

This package is intended as a seed to start a new Moodle site using Composer.

## Features

By using the Composer-based approach you are able to:

- install Moodle plugins using Composer
- install Moodle PHP-based dependencies
## Usage

### Seeding a new Moodle instance

To create a new Moodle site using the seed project you can simply run:

```shell
composer create-project moodle/seed [yourlocation]
```

The Moodle scaffolding tool will then guide you through setting up your new Moodle seedling.

Within your new `yourlocation` directory you will find a number of files and folders, including:

- a `composer.json` and `composer.lock` file;
- a `vendor`; and
- a `moodle` directory, containing your Moodle site.

### Specifying a specific version of Moodle

Change directory into your composer root, and run:

```shell
composer require "moodle/moodle:~5.1.0"
```

For more information on writing version constraints see the [Composer documentation](https://getcomposer.org/doc/articles/versions.md#writing-version-constraints).

### Adding a Moodle plugin

You can require any correctly-configured Moodle plugin which exists in Packagist using Composer:

```shell
cd [yourlocation]
composer require fmcorz/moodle-block_xp
```