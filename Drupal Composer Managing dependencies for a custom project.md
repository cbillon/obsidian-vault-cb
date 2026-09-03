---
tags:
  - drupal
  - composer
---
[Managing dependencies for a custom project](https://www.drupal.org/docs/develop/using-composer/managing-dependencies-for-a-custom-project)

You can use Composer to manage dependencies for your custom modules. To do this, your Drupal site's composer.json (located in the repo root) must have a way to read your custom project's composer.json file. If your custom project is not hosted on Packagist or Drupal.org, you may use a Composer path repository to accomplish this.

Find the repositories section of your root composer.json file, and modify it to include your custom module or modules as path repositories:

```php
"repositories": [
    {
        "type": "composer",
        "url": "https://packages.drupal.org/8"
    },
    {
        "type": "path",
        "url": "docroot/modules/custom/example"
    }
]
```

If your repositories section looks different, keep every entry already listed there, and add only the last example path repository. Adjust the path as necessary to point to the installation location of your custom module.

Next, require your module in your project:

```php
composer require org/example
```

Note that the organization that you should use to require your custom project is defined in your module's composer.json file. It can be whatever you want; usually, this matches whatever the organization is in the project's repository on GitHub or GitLab, et. al.

## [](https://www.drupal.org/docs/develop/using-composer/managing-dependencies-for-a-custom-project#include-project-dependencies "Permalink to this headline")Include project dependencies in root composer.json

Alternately, if you wish to avoid including your custom module's composer.json file, you can follow this process to get the packages working in your local development environment.

Simply take the dependencies from your module in development, and add them to the root composer.json file. For example, the last line here is a new addition which is available via Packagist:

```php
"require": {
  "composer/installers": "^1.2",
  "cweagans/composer-patches": "^1.6",
  "drupal-composer/drupal-scaffold": "^2.2",
  "drupal/admin_toolbar": "1.x-dev",
  "drupal/console": "~1.0",
  "drupal/core": "~8.0",
  "drupal/devel": "1.x",
  "drupal/toolbar_themes": "^1.0@alpha",
  "drush/drush": "~8.0",
  "webflo/drupal-finder": "^0.2.1",
  "webmozart/path-util": "^2.3",

  "mf2/mf2": "dev-master"
}
```

After you save the composer.json file, run `composer update` from the same directory as the composer.json file.

Your dependencies will be added to the root /vendor directory and will be detected by the autoloader as expected.

Be sure to also run `drush cache-rebuild` or otherwise clear caches for the best results.

## Composer/installers

The [composer/installers](https://github.com/composer/installers) plugin is similar to this plugin in that it allows dependencies to be installed in locations other than the `vendor` directory. However, Composer and the `composer/installers` plugin have a limitation that one project cannot be moved inside of another project. Therefore, if you use `composer/installers` to place Drupal modules inside the directory `web/modules/contrib`, then you cannot also use `composer/installers` to place files such as `index.php` and `robots.txt` into the `web` directory. The drupal-scaffold plugin was created to work around this limitation.