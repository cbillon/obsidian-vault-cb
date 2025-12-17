---
link: https://wpackagist.org/
excerpt: |-
  "repositories":[
          {
              "name": "wpackagist",
              "type": "composer",
              "url": "https://wpackagist.org",
              "only": [
                  "wpackagist-plugin/*",
                  "wpackagist-theme/*"
              ]
          }
      ],
slurped: 2025-12-14T17:13
title: Manage your WordPress&reg; plugins and themes with Composer
tags:
  - composer
---

### How do I use it?

1. Add the repository to your `composer.json`
2. Add the desired plugins and themes to your requirements using `wpackagist-plugin` or `wpackagist-theme` as the vendor name.
3. Run `$ composer.phar update`
4. Packages are [installed](https://github.com/composer/installers) to `wp-content/plugins/` or `wp-content/themes/` (unless otherwise specified by `installer-paths`)

### Example

{
    "name": "acme/brilliant-wordpress-site",
    "description": "My brilliant WordPress site",

    "repositories":[
        {
            "name": "wpackagist",
            "type": "composer",
            "url": "https://wpackagist.org",
            "only": [
                "wpackagist-plugin/*",
                "wpackagist-theme/*"
            ]
        }
    ],

    "require": {
        "aws/aws-sdk-php":"*",

        "wpackagist-plugin/akismet":"dev-trunk",
        "wpackagist-plugin/wordpress-seo":">=7.0.2",
        "wpackagist-theme/hueman":"*"

    },
    "autoload": {
        "psr-0": {
            "Acme": "src/"
        }
    },
    "extra": {
        "installer-paths": {

            "wp-content/mu-plugins/{$name}/": [
                "wpackagist-plugin/akismet"
            ],

            "wp-content/plugins/{$name}/": [
                "type:wordpress-plugin"
            ]
        }
    }
}

This example `composer.json` file adds the Wpackagist repository and includes the latest version of Akismet (installed as a must-use plugin), at least version 7.0.2 of Wordpress SEO, and the latest Hueman theme along with the Amazon Web Services SDK from the main [Packagist](https://packagist.org/) repository.

Find out more about [using Composer](https://getcomposer.org/doc/01-basic-usage.md) including [custom install paths](https://github.com/composer/installers#custom-install-paths).

The old vendor prefix `wpackagist` is now **removed** in favour of `wpackagist-plugin`.

### Why use Composer?

> “Composer is a tool for dependency management in PHP. It allows you to declare the dependent libraries your project needs and it will install them in your project for you.”

- Avoid committing plugins and themes into source control.
- Avoid having to use git submodules.
- Manage WordPress® and non-WordPress® project libraries with the same tools.
- Could eventually be used to manage dependencies between plugins.

### How does the repository work?

- Scans the WordPress® Subversion repository every hour for [plugins](https://plugins.svn.wordpress.org/) and [themes](https://themes.svn.wordpress.org/). Search and click to make any newer versions available.
- Fetches the `tags` for each updated package and maps those to versions.
- For plugins, adds `trunk` as a dev version.
- Rebuilds the composer package JSON files.

### Known issues

- Requires Composer 1.0.0-alpha7 or more recent
- Version strings which Composer cannot parse are ignored. All plugins have at least the trunk build available.
- Themes do not have a trunk version. It is recommended to use `"*"` as the required version.
- Even when packages are present on SVN, they won’t be available if they are not published on [wordpress.org](https://wordpress.org/plugins/). Try [searching](https://wpackagist.org/search) for your plugin before reporting a [bug](https://github.com/outlandishideas/wpackagist/issues/new?title=[Bug]%20).
- You can also check for [open issues](https://github.com/outlandishideas/wpackagist/issues).

### WordPress® Core

See `[fancyguy/webroot-installer](https://github.com/fancyguy/webroot-installer)` or `[roots/wordpress](https://github.com/roots/wordpress)` for installing WordPress® itself using Composer.

### Contribute or get support

Please visit our [GitHub page](https://github.com/outlandishideas/wpackagist).