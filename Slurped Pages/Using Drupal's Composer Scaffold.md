---
link: https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold
excerpt:
slurped: 2026-08-22T15:10
title: Using Drupal(s Composer Scaffold
---

# Using Drupal's Composer Scaffold

This documentation is for the [Drupal Core Scaffolding Composer plugin](https://packagist.org/packages/drupal/core-composer-scaffold) that has been available for Drupal core as of the 8.8.x branch. It was considered pre-release before the Drupal core 8.8.0 release. See the [change record](https://www.drupal.org/node/3041017).

## [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#s-drupal-composer-scaffold "Permalink to this headline")Drupal Composer Scaffold

The [Drupal Composer Scaffold](https://github.com/drupal/core-composer-scaffold/tree/10.1.x) project provides a Composer plugin for placing scaffold files (like `index.php`, `update.php`, …) from the `drupal/core` project into their desired location inside the web root. Only individual files may be scaffolded with this plugin.

The purpose of scaffolding files is to allow Drupal sites to be fully managed by Composer, and still allow individual asset files to be placed in arbitrary locations. The goal of doing this is to enable a properly configured composer template to produce a file layout that exactly matches the file layout of a Drupal 8.7.x and earlier tarball distribution. Other file layouts are also  possible; for example, a project layout very similar to the now deprecated [drupal-composer/drupal-project](https://github.com/drupal-composer/drupal-scaffold) template is provided as part of the [Drupal Recommended Project](https://www.drupal.org/docs/develop/using-composer/starting-a-site-using-drupal-composer-project-templates) composer template. When one of these projects is used, the user should be able to use `composer require` and `composer update` on a Drupal site immediately after untarring the downloaded archive.

Note that the dependencies of a Drupal site are only able to scaffold files if explicitly granted that right in the top-level composer.json file. See [allowed packages](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_2), below.

If you want Composer to check out your local copy or issue fork instead of using the official version, see [Tricks for using Composer in local development](https://www.drupal.org/docs/develop/using-composer/tricks-for-using-composer-in-local-development).

## [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_1 "Permalink to this headline")Usage

Composer-scaffold is used by requiring `drupal/core-composer-scaffold` in your project, and providing configuration settings in the `extra` section of your project's composer.json file. Additional configuration from the composer.json file of your project's dependencies is also consulted in order to scaffold the files a project needs. Additional information may be added to the beginning or end of scaffold files, as is commonly done to `.htaccess` and `robots.txt` files. See [altering scaffold files](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_4) for more information.

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_2 "Permalink to this headline")Allowed packages

Scaffold files are stored inside of projects that are required from the main project's composer.json file as usual. The scaffolding operation happens after `composer install`, and involves copying or symlinking the desired assets to their destination location. In order to prevent arbitrary dependencies from copying files via the scaffold mechanism, only those projects that are specifically permitted by the top-level project will be used to scaffold files.

The `drupal/core` and `drupal/legacy-scaffold-assets` packages are implicitly allowed to scaffold files. You do not need to explicitly allow these packages as long as they are dependencies in the top-level composer.json file.

Example: Permit scaffolding from the project `my/project-assets`

```php
  "name": "my/project",
  ...
  "extra": {
    "drupal-scaffold": {
      "allowed-packages": [
        "my/project-assets"
      ],
      ...
    }
  }
```

Allowing a package to scaffold files also permits it to delegate permission to scaffold to any project that it requires itself. This allows a package to organize its scaffold assets as it sees fit.

It is possible for a project to obtain scaffold files from multiple projects. For example, a Drupal project using a distribution, and installing on a specific web hosting service provider might take its scaffold files from:

- Drupal core
- Its distribution
- A project provided by the hosting provider
- The project itself

Each project allowed to scaffold by the top-level project will be used in turn, with projects declared later in the `allowed-packages` list taking precedence over the projects named before. The top-level composer.json itself is always implicitly allowed to scaffold files, and its scaffold files have highest priority.

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_3 "Permalink to this headline")Defining project locations

The top-level project in turn must define where the web root is located. It does so via the `locations` mapping, as shown below:

```php
  "name": "my/project",
  ...
  "extra": {
    "drupal-scaffold": {
      "locations": {
        "web-root": "./docroot"
      },
      ...
    }
  }
```

This makes it possible to configure a project with different file layouts; for example, either the `drupal/drupal` file layout or the `drupal-composer/drupal-project` file layout could be used to set up a project.

If a web-root is not explicitly defined, then it will default to `./`.

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_4 "Permalink to this headline")Altering scaffold files

Sometimes, a project might wish to use a scaffold file provided by a dependency, but alter it in some way. Two forms of alteration are supported: appending and patching.

The original files are located into `core/assets/scaffold/files/` and the initial settings are defined into `core/composer.json` (and could be changed by [Allowed Packages](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_2))

The example below shows a project that appends additional entries onto the end of the `robots.txt` file provided by `drupal/core`:

```php
"extra": {
    "drupal-scaffold": {
        "locations": {
            "web-root": "web/"
        },
        "file-mapping": {
            "[web-root]/robots.txt": {
                "append": "assets/my-robots-additions.txt"
            }
        }
    },
...
```

Under `"web-root": "web/",` adjust `web/` to match your environment. `[web-root]` is a token, and should not be adjusted.

It is also possible to prepend to a scaffold file instead of, or in addition to appending by including a "prepend" entry that provides the relative path to the file to prepend to the scaffold file.

Note: In case `composer install` has no effect, renaming or deleting the `robots.txt` file may help force an update.

The example below demonstrates the use of the `post-drupal-scaffold-cmd` hook to patch the `.htaccess` file using a patch. Place it in a "scripts" section at the root level of the `composer.json` (not within the "extras" section):

```php
"name": "my/project",
...
"scripts": {
    "post-drupal-scaffold-cmd": [
        "cd docroot && patch -p1 <../patches/htaccess-ssl.patch"
    ]
}
```

For details, see [Customise scaffold files the right way](https://www.computerminds.co.uk/articles/customise-scaffold-files-right-way).

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_5 "Permalink to this headline")Defining scaffold files

The placement of scaffold assets is under the control of the project that provides them, but the location is always relative to some directory defined by the root project -- usually the web root. For example, the scaffold file `robots.txt` is copied from its source location, `assets/robots.txt` into the web root in the snippet below.

```php
{
  "name": "drupal/assets",
  ...
  "extra": {
    "drupal-scaffold": {
      "file-mapping": {
        "[web-root]/robots.txt": "assets/robots.txt"
      },
      ...
    }
  }
}
```

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_6 "Permalink to this headline")Excluding scaffold files

Sometimes, a project might prefer to entirely replace a scaffold file provided by a dependency, and receive no further updates for it. This can be done by setting the value for the scaffold file to exclude to `false`:

```php
  "name": "my/project",
  ...
  "extra": {
    "drupal-scaffold": {
      "file-mapping": {
        "[web-root]/robots.txt": false
      },
      ...
    }
  }
```

If possible, use the `append` and `prepend` directives as explained in [altering scaffold files](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_4), above. Excluding a file means that your project will not get any bug fixes or other updates to files that are modified locally.

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_7 "Permalink to this headline")Overwrite

By default, scaffold files overwrite whatever content exists at the target location. Sometimes a project may wish to provide the initial contents for a file that will not be changed in subsequent updates. This can be done by setting the `overwrite` flag to `false`, as shown in the example below:

```php
{
  "name": "service-provider/d8-scaffold-files",
  "extra": {
    "drupal-scaffold": {
      "file-mapping": {
        "[web-root]/sites/default/settings.php": {
          "mode": "replace",
          "path": "assets/sites/default/settings.php",
          "overwrite": false
        }
      },
      ...
    }
  }
}
```

Note that the `overwrite` directive is intended to be used by starter kits, service providers, and so on. Individual Drupal sites should exclude the file by setting its value to false instead.

## [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_8 "Permalink to this headline")Specifications

Reference section for the configuration directives for the "drupal-scaffold" section of the "extra" section of a `composer.json` file appear below.

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_9 "Permalink to this headline")allowed-packages

The `allowed-packages` configuration setting contains an ordered list of package names that will be used during the scaffolding phase.

The `drupal/core` and `drupal/legacy-scaffold-assets` packages are implicitly allowed to scaffold files. You do not need to explicitly allow these packages as long as they are dependencies in the top-level composer.json file.

```php
"allowed-packages": [
  "project/assets"
],
```

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_10 "Permalink to this headline")file-mapping

The `file-mapping` configuration setting consists of a map from the destination path of the file to scaffold to a set of properties that control how the file should be scaffolded.

The available properties are as follows:

- mode: One of "replace", "append" or "skip".
- path: The path to the source file to write over the destination file.
- prepend: The path to the source file to prepend to the destination file, which must always be a scaffold file provided by some other project.
- append: Like `prepend`, but appends content rather than prepends.
- overwrite: If `false`, prevents a `replace` from happening if the destination already exists.

The mode may be inferred from the other properties. If the mode is not specified, then the following defaults will be supplied:

- replace: Selected if a `path` property is present, or if the entry's value is a string rather than a property set.
- append: Selected if a `prepend` or `append` property is present.
- skip: Selected if the entry's value is a boolean `false`.

Examples:

```php
"file-mapping": {
  "[web-root]/sites/default/default.settings.php": {
    "mode": "replace",
    "path": "assets/sites/default/default.settings.php",
    "overwrite": true
  },
  "[web-root]/sites/default/settings.php": {
    "mode": "replace",
    "path": "assets/sites/default/settings.php",
    "overwrite": false
  },
  "[web-root]/robots.txt": {
    "mode": "append",
    "prepend": "assets/robots-prequel.txt",
    "append": "assets/robots-append.txt"
  },
  "[web-root]/.htaccess": {
    "mode": "skip"
  }
}
```

The short-form of the above example would be:

```php
"file-mapping": {
  "[web-root]/sites/default/default.settings.php": "assets/sites/default/default.settings.php",
  "[web-root]/sites/default/settings.php": {
    "path": "assets/sites/default/settings.php",
    "overwrite": false
  },
  "[web-root]/robots.txt": {
    "prepend": "assets/robots-prequel.txt",
    "append": "assets/robots-append.txt"
  },
  "[web-root]/.htaccess": false
}
```

Note that there is no distinct "prepend" mode; "append" mode is used to both append and prepend to scaffold files. The reason for this is that scaffold file entries are identified in the file-mapping section keyed by their destination path, and it is not possible for multiple entries to have the same key. If "prepend" were a separate mode, then it would not be possible to both prepend and append to the same file.

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_11 "Permalink to this headline")gitignore

The `gitignore` configuration setting controls whether or not this plugin will add scaffolded assets to `.gitignore` files in the web root.

- true: `.gitignore` files will be updated when scaffold files are written.
- false: `.gitignore` files will never be modified.
- Not set: `.gitignore` files will be updated if the target directory is a local working copy of a git repository, and the `vendor` directory is ignored in that repository.

When Composer Scaffold installs a scaffolded asset, it will append a corresponding line to a `.gitignore` file in the web root (one of `docroot/.gitignore`, `docroot/sites/.gitignore`, or `docroot/sites/default/.gitignore`). Composer Scaffold generates and manages these gitignore files dynamically; these files are not related to the [example.gitignore](https://git.drupalcode.org/project/drupal/-/blob/9.2.x/example.gitignore) file provided by Drupal core.

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_12 "Permalink to this headline")locations

The `locations` configuration setting contains a list of named locations that may be used in placing scaffold files. The only required location is `web-root`. Other locations may also be defined if desired.

```php
"locations": {
  "web-root": "./docroot"
},
```

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_13 "Permalink to this headline")symlink

The `symlink` property causes `replace` operations to make a symlink to the source file rather than copying it. This is useful when doing core development, as the symlink files themselves should not be edited. Note that `append` operations override the `symlink` option, to prevent the original scaffold assets from being altered.

```php
"symlink": true,
```

## [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_14 "Permalink to this headline")Managing scaffold files

Scaffold files should be treated the same way that the `vendor` directory is handled. If you need to commit `vendor` (e.g. in order to deploy your site), then you should also commit your scaffold files. You should not commit your `vendor` directory or scaffold files unless it is necessary.

If a dependency provides a scaffold file with `overwrite` set to `false`, that file should be committed to your repository.

By default, `.gitignore` files will be automatically updated if needed when scaffold files are written. See the `gitignore` setting in the Specifications section above.

## [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#s-migrating-composer-scaffold "Permalink to this headline")Migrating Composer Scaffold

This information was sourced from [DrupalCon Amsterdam 2019: Composer and Drupal: Past, Present and Future](https://youtu.be/ddPL91oHQdU?t=1740). To migrate from a legacy configuration to the Drupal Core Composer Scaffold, complete the following:

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#s-remove-legacy-configuration "Permalink to this headline")Remove legacy configuration

Remove these dependencies:

```php
drupal-composer/drupal-scaffold
drupal/core
webflo/drupal-core-strict
webflo/drupal-core-require-dev
```

Under `extra`  
    Under `drupal-scaffold`  
        Remove the `initial` section

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#s-add-new-configuration "Permalink to this headline")Add new configuration

#### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#s-drupal-8 "Permalink to this headline")Drupal 8

Add these dependencies:

```php
drupal/core-composer-scaffold:^8.9
drupal/core-recommended:^8.9
```

Under `extra`  
    Under `drupal-scaffold`  
        Add:

```php
            "locations": {
                "web-root": "web/"
            }
```

#### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#s-drupal-9 "Permalink to this headline")Drupal 9

Add these dependencies:

```php
drupal/core-composer-scaffold
drupal/core-recommended
```

Under `extra`  
    Under `drupal-scaffold`  
        Add:

```php
            "locations": {
                "web-root": "web/"
            }
```

## [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#s-examples "Permalink to this headline")Examples

Some full-length examples appear below.

Sample composer.json for a project that relies on packages that use composer-scaffold:

```php
{
  "name": "my/project",
  "require": {
    "drupal/core-composer-scaffold": "*",
    "composer/installers": "^1.2",
    "cweagans/composer-patches": "^1.6.5",
    "drupal/core": "^8.8.x-dev",
    "service-provider/d8-scaffold-files": "^1"
  },
  "config": {
    "optimize-autoloader": true,
    "sort-packages": true
  },
  "extra": {
    "drupal-scaffold": {
      "locations": {
        "web-root": "./docroot"
      },
      "symlink": true,
      "overwrite": true,
      "file-mapping": {
        "[web-root]/.htaccess": false,
        "[web-root]/robots.txt": "assets/robots-default.txt"
      }
    }
  }
}
```

Sample composer.json for drupal/core, with assets placed in a different project:

```php
{
  "name": "drupal/core",
  "extra": {
    "drupal-scaffold": {
      "allowed-packages": [
        "drupal/assets"
      ]
    }
  }
}
```

Sample composer.json for composer-scaffold files in drupal/assets:

```php
{
  "name": "drupal/assets",
  "extra": {
    "drupal-scaffold": {
      "file-mapping": {
        "[web-root]/.csslintrc": "assets/.csslintrc",
        "[web-root]/.editorconfig": "assets/.editorconfig",
        "[web-root]/.eslintignore": "assets/.eslintignore",
        "[web-root]/.eslintrc.json": "assets/.eslintrc.json",
        "[web-root]/.gitattributes": "assets/.gitattributes",
        "[web-root]/.ht.router.php": "assets/.ht.router.php",
        "[web-root]/.htaccess": "assets/.htaccess",
        "[web-root]/sites/default/default.services.yml": "assets/default.services.yml",
        "[web-root]/sites/default/default.settings.php": "assets/default.settings.php",
        "[web-root]/sites/example.settings.local.php": "assets/example.settings.local.php",
        "[web-root]/sites/example.sites.php": "assets/example.sites.php",
        "[web-root]/index.php": "assets/index.php",
        "[web-root]/robots.txt": "assets/robots.txt",
        "[web-root]/update.php": "assets/update.php",
        "[web-root]/web.config": "assets/web.config"
      }
    }
  }
}
```

Sample composer.json for a library that implements composer-scaffold:

```php
{
  "name": "service-provider/d8-scaffold-files",
  "extra": {
    "drupal-scaffold": {
      "file-mapping": {
        "[web-root]/sites/default/settings.php": "assets/sites/default/settings.php"
      }
    }
  }
}
```

Append to robots.txt:

```php
{
  "name": "service-provider/d8-scaffold-files",
  "extra": {
    "drupal-scaffold": {
      "file-mapping": {
        "[web-root]/robots.txt": {
          "append": "assets/my-robots-additions.txt"
        }
      }
    }
  }
}
```

Patch a file after it's copied:

```php
"post-drupal-scaffold-cmd": [
  "cd docroot && patch -p1 <../patches/htaccess-ssl.patch"
]
```

Append a file from a remote file:

```php
"post-drupal-scaffold-cmd": [
  "cd web && curl https://www.drupal.org/files/issues/2020-02-09/gitattributes-to-append.txt >> .gitattributes"
]
```

## [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_16 "Permalink to this headline")Related plugins

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_17 "Permalink to this headline")drupal-composer/drupal-scaffold

Previous versions of drupal-scaffold (see community project, [drupal-composer/drupal-scaffold](https://github.com/drupal-composer/drupal-project)) downloaded each scaffold file directly from its distribution server (e.g. `https://cgit.drupalcode.org`) to the desired destination directory. This was necessary, because there was no subtree split of the scaffold files available. Copying the scaffold assets from projects already downloaded by Composer is more effective, as downloading and unpacking archive files is more efficient than downloading each scaffold file individually.

### [](https://www.drupal.org/docs/develop/using-composer/using-drupals-composer-scaffold#toc_18 "Permalink to this headline")composer/installers

The [composer/installers](https://github.com/composer/installers) plugin is similar to this plugin in that it allows dependencies to be installed in locations other than the `vendor` directory. However, Composer and the `composer/installers` plugin have a limitation that one project cannot be moved inside of another project. Therefore, if you use `composer/installers` to place Drupal modules inside the directory `web/modules/contrib`, then you cannot also use `composer/installers` to place files such as `index.php` and `robots.txt` into the `web` directory. The drupal-scaffold plugin was created to work around this limitation.