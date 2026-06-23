---
link: https://github.com/moodle/seed
site: GitHub
excerpt: The seed to grow a Moodle site from. Contribute to moodle/seed development by creating an account on GitHub.
twitter: https://twitter.com/@github
slurped: 2026-06-23T19:20
title: "GitHub - moodle/seed: The seed to grow a Moodle site from"
tags:
  - moodle
  - install
  - composer
---

This package is intended as a seed to start a new Moodle site using Composer.

## Features

[](https://github.com/moodle/seed#features)

By using the Composer-based approach you are able to:

- install Moodle plugins using Composer
- install Moodle PHP-based dependencies

## Usage

[](https://github.com/moodle/seed#usage)

### Seeding a new Moodle instance

[](https://github.com/moodle/seed#seeding-a-new-moodle-instance)

To create a new Moodle site using the seed project you can simply run:

composer create-project moodle/seed [yourlocation]

The Moodle scaffolding tool will then guide you through setting up your new Moodle seedling.

Within your new `yourlocation` directory you will find a number of files and folders, including:

- a `composer.json` and `composer.lock` file;
- a `vendor`; and
- a `moodle` directory, containing your Moodle site.

### Specifying a specific version of Moodle

[](https://github.com/moodle/seed#specifying-a-specific-version-of-moodle)

Change directory into your composer root, and run:

composer require "moodle/moodle:~5.1.0"

For more information on writing version constraints see the [Composer documentation](https://getcomposer.org/doc/articles/versions.md#writing-version-constraints).

### Adding a Moodle plugin

[](https://github.com/moodle/seed#adding-a-moodle-plugin)

You can require any correctly-configured Moodle plugin which exists in Packagist using Composer:

cd [yourlocation]
composer require fmcorz/moodle-block_xp