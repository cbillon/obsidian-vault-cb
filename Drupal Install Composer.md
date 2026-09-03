---
tags:
  - drupal
  - composer
---
[Install dependencies with composer](https://www.drupal.org/docs/getting-started/installing-drupal/install-dependencies-with-composer)

If you do not have composer installed, see the official [composer installation instructions](https://getcomposer.org/download/).
In order to get a working codebase, you need to run `composer install --no-dev` from the top level of the repository. This will install Symfony and other packages required by Drupal in the `vendor/` directory.

If you skip this step, then you are likely to see an error message like this when you try to run the installer:

> `Warning: require(.../drupal/vendor/autoload.php): failed to open stream: No such file or directory in .../drupal/autoload.php on line 14`

It is important to include the `--no-dev` option when installing packages for your production server. Some packages, such as `phpunit/*`, are inherently insecure and should never be installed on the production server. If you are installing Drupal on a local or development server, and you want to install development packages, then leave off the `--no-dev` option.

You may also add the `-o` option, to generate optimized autoload files. For a complete list of options for the `composer install` subcommand, use `composer help install`.
## Install Drupal
# Drupal quick-start demo on your own hosting

Last [updated](https://www.drupal.org/node/2972473/discuss) on 

12 July 2026

The `quick-start` command is intended only for launching a local demo version of Drupal in your own bespoke setup. Consider the recommended routes for:

- [Downloading Drupal to start building a new site](https://new.drupal.org/download)
- [Getting started with local development via DDEV](https://www.drupal.org/docs/getting-started/installing-drupal/install-drupal-using-ddev-for-local-development)

The only requirement for the `quick-start` command is that PHP must be installed. Once that is complete, the `quick-start` command will install and run Drupal using PHP's built-in web server on your own computer.

If you need to install Drupal for production, see the instructions in the rest of this guide. If you need to install Drupal for development, go to the [install Drupal using DDEV](https://www.drupal.org/docs/getting-started/installing-drupal/install-drupal-using-ddev) in the [Local server setup guide](https://www.drupal.org/docs/develop/local-server-setup).

## [](https://www.drupal.org/docs/getting-started/installing-drupal/drupal-quick-start-demo-on-your-own-hosting#s-step-1-install-php "Permalink to this headline")Step 1 - Install PHP

Read the [PHP requirements](https://www.drupal.org/docs/system-requirements/php-requirements) to find the version of PHP required by currently supported versions of Drupal.

You can install PHP using the standard method for your operating system. Also, install the following extensions:

```php
php-cli php-curl php-gd php-mbstring php-sqlite3 php-xml
```

There are many on-line resources for installing PHP to assist with this step.

## [](https://www.drupal.org/docs/getting-started/installing-drupal/drupal-quick-start-demo-on-your-own-hosting#s-step-2-get-drupal "Permalink to this headline")Step 2 - Get Drupal

### [](https://www.drupal.org/docs/getting-started/installing-drupal/drupal-quick-start-demo-on-your-own-hosting#s-option-a-using-composer "Permalink to this headline")Option A - Using Composer

Paste the line below into your command line to download and extract the Drupal package with Composer:

```php
composer create-project drupal/recommended-project my-site && cd my-site
```

### [](https://www.drupal.org/docs/getting-started/installing-drupal/drupal-quick-start-demo-on-your-own-hosting#s-option-b-using-git-and-composer "Permalink to this headline")Option B - Using Git and Composer

Paste the line below into your command line to download and extract the Drupal package with Git and Composer:

```php
git clone https://git.drupalcode.org/project/drupal.git my-site && cd my-site && composer install
```

## [](https://www.drupal.org/docs/getting-started/installing-drupal/drupal-quick-start-demo-on-your-own-hosting#s-step-3-install-drupal-demo-using-quick-start "Permalink to this headline")