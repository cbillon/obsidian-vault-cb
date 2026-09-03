---
tags:
  - drupal
  - composer
---
[Add a composer.json file to module](https://www.drupal.org/docs/develop/using-composer/add-a-composerjson-file)

When developing custom modules there are several scenarios that require the developer to add a composer.json file to their module. Some of these scenarios depend upon whether the custom module is intended to be contributed back to the community as a project on drupal.org. Here are some reasons for creating a composer.json file:

- A module that uses a PHP library that is hosted on packagist.org, or that depends on other contributed modules from drupal.org, should have a composer.json file so that people downloading the module using Composer will automatically also install the dependencies ([see section Drupal core compatibility](https://www.drupal.org/docs/develop/using-composer/add-a-composerjson-file#)).
- A contributed module that depends on other modules/packages and has [automated tests](https://www.drupal.org/docs/develop/managing-a-drupalorg-theme-module-or-distribution-project/maintainership/automated) that run on the DrupalCI environment must have a composer.json that expresses the dependencies. Tests of merge requests on the module will fail without a composer.json file, as will tests of patches that change the dependencies.
- If a module developer wishes to use the more expressive constraints provided by composer.json, such as the caret or tilde operators, those are only possible in composer.json. See [Managing dependencies for a custom project](https://www.drupal.org/docs/develop/using-composer/managing-dependencies-for-a-custom-project) for details. Note, however, that Drupal itself will not be bound by those constraints -- they will only be enforced when Composer is used to download and update code.
- If the module would like to [use live previews with Tugboat](https://www.drupal.org/docs/develop/git/using-git-to-contribute-to-drupal/using-live-previews-on-drupal-core-and-contrib-merge-requests), the project must have a composer.json file. Tugboat needs it in order to install the project and its dependencies.
- If a module does not have any dependencies, or the dependencies are solely other Drupal modules, then a composer.json is **not required**. However, having a composer.json does not have a negative impact either.

Regardless of whether or not a developer has a composer.json file, their Drupal module dependencies must still be expressed in their `.info.yml`files so that Drupal can ensure that the correct modules are **enabled**.

## [](https://www.drupal.org/docs/develop/using-composer/add-a-composerjson-file#s-define-your-project-as-a-php-package "Permalink to this headline")Define your project as a PHP package

The wider PHP community uses Composer to manage packages; this is also done in Drupal. For example, the Drupal project has a dependency on the "drupal/core" package. The "drupal/core" package has a type defined as "drupal-core" so Composer knows what to do with it, e.g. how to install it. The [composer/installers](https://github.com/composer/installers) library defines a number of Drupal types (see that page for a full list of types). For project types that are publicly available, one of the following Drupal types should work:

- drupal-module
- drupal-theme
- drupal-library
- drupal-profile
- drupal-drush
- drupal-database-driver

For custom code which is typically not available publicly, choose one of the following Drupal types:

- drupal-custom-module
- drupal-custom-theme
- drupal-custom-profile

You can use the composer `init` command to generate a composer.json file:

1. Run: `composer init`
2. When prompted for `Package name (<vendor>/<name>):` enter `drupal/your-project-name`. If your code is hosted on drupal.org, the project name should match the URL used for the project, e.g. `drupal/components`.
3. When prompted for `Description []:` enter a concise description of your project.
    
4. When prompted for `Author [YourGitAuthorName <me@example.com>, n to skip]:`, hit enter as the default value should already be the one you prefer in Git.
    
5. When prompted for `Minimum Stability []`:, hit enter to use the the default of "stable" stability. This will have no effect on websites where your code is installed, but may affect which versions of dependencies are used when running your project's tests. Stable dependencies during testing is preferred to unstable ones.
    
6. When prompted for `Package Type (e.g. library, project, metapackage, composer-plugin) []:`, use one of the Drupal types shown above, e.g. `drupal-module`.
    
7. When prompted for `License []:`, you must enter `GPL-2.0-or-later` for projects hosted on drupal.org.
    
8. When prompted for `Would you like to define your dependencies (require) interactively [yes]?`, hit enter and then enter the name of each dependency that is _required_ for you project's code to run correctly on a Drupal website.
    
9. When prompted for `Would you like to define your dev dependencies (require-dev) interactively [yes]?`, hit enter and then enter the name of each dependency that is required to develop or test your project's code and _not_ required for you project's code to run correctly on a Drupal website.
    
10. When prompted for `Do you confirm generation [yes]?`, look at the proposed JSON and hit enter to generate the `composer.json` file or type `no` to abort and start over.
    

Here is a full example of how a project uses composer.json to depend on the external project `exampledetect/exampledetectlib`:

```php
{
    "name": "drupal/example_project",
    "description": "Example Project is a lightweight PHP class for detecting examples",
    "type": "drupal-module",
    "license": "GPL-2.0-or-later",
    "authors": [
        {
            "name": "Matthew Donadio (mpdonadio)",
            "homepage": "https://www.drupal.org/u/mpdonadio",
            "role": "Maintainer"
        },
        {
            "name": "Darryl Norris (darol100)",
            "email": "admin@darrylnorris.com",
            "homepage": "https://www.drupal.org/u/darol100",
            "role": "Co-maintainer"
        }
    ],
    "homepage": "https://drupal.org/project/example_project",
    "support": {
        "issues": "https://drupal.org/project/issues/example_project",
        "source": "https://git.drupalcode.org/project/example_project"
    },
    "require": {
        "exampledetect/exampledetectlib": "^2.8"
    }
}

```

Note that the example composer.json shows how to list multiple maintainers and link to the project's homepage, support queue and source code. You should edit your composer.json, if needed, to add those items.

For naming your package, you must follow Drupal's [Composer package naming conventions](https://www.drupal.org/node/2471927).

Always run the [`composer validate`](https://getcomposer.org/doc/03-cli.md#validate "Command-line interface / Commands - Composer") command before committing a new or changed _composer.json_ file. It will check if your _composer.json_ is valid.

## [](https://www.drupal.org/docs/develop/using-composer/add-a-composerjson-file#s-defining-dependencies-in-composerjson "Permalink to this headline")Defining dependencies in composer.json

You may optionally define external dependencies for your module in composer.json. **Drupal core will not automatically discover or manage these dependencies**. To utilize dependencies defined in a project's composer.json file, you must use one of the following maintenance strategies:

- [Install Drupal core and the module using composer.](https://www.drupal.org/node/2718229)
- [Manually modify the composer.json file](https://www.drupal.org/node/2404989) in your Drupal installation's root directory.

For more information on composer as a dependency manager for Drupal, review a [comparison of Composer and Drush Make](https://www.drupal.org/node/2471553) as dependency managers.

### [](https://www.drupal.org/docs/develop/using-composer/add-a-composerjson-file#s-adding-dependencies-on-other-drupal-modules "Permalink to this headline")Adding dependencies on other Drupal modules

By default Composer only will look at packages that are published on [Packagist](https://packagist.org/) when it is resolving its dependencies. Most Drupal modules are not published there, since Drupal has its own repository. Because of this you might get error messages such as the following:

> The requested package drupal/module could not be found in any version, there may be a typo in the package name.

You can instruct Composer to look for Drupal modules in the `packages.drupal.org` repository by executing the following command:

```php
$ composer config repositories.drupal composer https://packages.drupal.org/8
```

This command will add the following section to your `composer.json` file:

```php
  "repositories": {
    "drupal": {
      "type": "composer",
      "url": "https://packages.drupal.org/8"
    }
  }
```

## [](https://www.drupal.org/docs/develop/using-composer/add-a-composerjson-file#core-compatibility "Permalink to this headline")Drupal core compatibility

Having a `composer.json` file is not required for Drupal compatibility.  A [Drupal compatible info.yml file](https://www.drupal.org/docs/develop/creating-modules/let-drupal-know-about-your-module-with-an-infoyml-file#s-specifying-core-version-requirement) with a `core_version_requirement` is required and is sufficient to meet Core minimal requirements.

Composer generally requires a composer.json to be present in a project to correctly function in all scenarios.

If your project does have a `composer.json` file without a  `drupal/core` version requirement the [Drupal's composer facade](https://www.drupal.org/project/project_composer) will generate the appropriate metadata based on the info.yml file.server will add a `drupal/core` version requirement if it is not already present.

The composer facade can not generate this metadata for GIT based checkouts that do not use the facade. As a maintainer to ensure compatibility with composer an entry for `drupal/core` may be required in your `composer.json` file.

If you place a `drupal/core` version requirement in your `composer.json` file's `require` section, then it should be compatible with the same version set in the info file's `core_version_requirement`. A mismatch in compatibility may create a situation that Composer will download a module that can not be installed by Drupal, or that a user may install a module obtained manually when composer would not permit download.

## Related Content

## [Using Composer with Drupal](https://www.drupal.org/docs/develop/using-composer/using-composer-with-drupal)

Composer can be used to add dependencies to a Drupal project. It can be used in the following ways.

## Composer/installers

The [composer/installers](https://github.com/composer/installers) plugin is similar to this plugin in that it allows dependencies to be installed in locations other than the `vendor` directory. However, Composer and the `composer/installers` plugin have a limitation that one project cannot be moved inside of another project. Therefore, if you use `composer/installers` to place Drupal modules inside the directory `web/modules/contrib`, then you cannot also use `composer/installers` to place files such as `index.php` and `robots.txt` into the `web` directory. The drupal-scaffold plugin was created to work around this limitation.