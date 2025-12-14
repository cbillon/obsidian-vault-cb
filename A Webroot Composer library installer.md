---
tags:
  - composer
---
[site](github.com/fancyguy/webroot-installer)


This is for PHP packages that support composer to configure in their `composer.json`. It will allow a root package to define a webroot directory and webroot package and magically install it in the correct location.

## Requirements

In order to use this package your project must use both components in the given version, otherwise you cannot use this plugin:

1. Composer v2
2. PHP 8.x

## Installation

This repository is a fork of [Fancyguy/Webroot-Installer](https://github.com/nopenopenope/webroot-installer), so its not commited to packagist and you cannot install it via an unmodified composer require.