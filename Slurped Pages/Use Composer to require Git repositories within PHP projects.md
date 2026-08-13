---
link: https://opensource.com/article/22/5/composer-git-repositories
byline: Jonathan Daggerhart
site: Opensource.com
excerpt: "The dependency management tool Composer provides multiple ways to include Git repositories within a PHP: Hypertext Pr"
twitter: https://twitter.com/@daggerhart
slurped: 2026-06-26T15:17
title: Use Composer to require Git repositories within PHP projects
tags:
  - git
---

The dependency management tool [Composer](https://getcomposer.org/) provides multiple ways to include Git repositories within a PHP: Hypertext Preprocessor (PHP) project.

In many cases, repositories have been created on Packagist, so requiring them with Composer is very straightforward. But what do you do when a repository has not been created as a package on Packagist? You use Composer to require the package directly from the repository. This article explains how.

**Note:** Some of the terminology in this post is confusing because multiple words are used to describe different things. Here is a quick vocabulary list that will help:

- **Project:** The custom software you are building. This can be a website, a command-line utility, an application, or anything else you dream up.  
     
- **Package:** Any third-party software you want to download and use within your project. It can be a library, Drupal theme, WordPress plugin, or any other number of things.  
     
- **Git repository:** Also called the Git repo, this is the version-control host for a package. Common hosts include GitHub, GitLab, or Bitbucket, but any URL-accessible Git repository will work for this tutorial.  
     
- **Composer repositories:** In a `composer.json` file, there is an optional property named "repositories." This property is where you can define new places for Composer to look when downloading packages.

When adding a Git repo to your project with Composer, you can find yourself in two situations: Either the repo contains a `composer.json` file, which defines how the repo should be handled when required, or it does not. You can add the Git repository to your project in both cases, with different methods.

## Git repo with composer.json

When a repository includes a `composer.json` file, it defines aspects of itself that are important to how Composer manages the package. Here is an example of a simple `composer.json` file a package may include:

```
{
    "name": "mynamespace/my-custom-library",
    "type": "library"
}
```

This example shows two important properties that a `composer.json` file can define:

- **Name:** The package's namespaced name. In this case, "mynamespace" is the namespace for the package "my-custom-library."
- **Type:** The type of package the repo represents. Package types are used for installation logic. Out of the box, Composer allows for the following package types: library, project, metapackage, and composer-plugin.

You can verify this by looking at the `composer.json` file of any popular GitHub project. They each define their package name and the package type near the top of the file. When a repository has this information defined in its `composer.json` file, requiring the repository within your project is quite simple.

## Require a Git repository that has a composer.json file

After you've identified a Git repository with a `composer.json` file, you can require that repository as a package within your project.

Within your project's `composer.json` file, you need to define a new property (assuming it doesn't exist already) named "repositories." The value of the repositories property is an array of objects, each containing information about the repository you want to include in your project. Consider this `composer.json` file for your custom project.

```
{
  "name":  "mynamespace/my-project-that-uses-composer",
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/mynamespace/my-custom-library.git"
    }
  ],
  "require": {
    "mynamespace/my-custom-library": "dev-master"
  }
}
```

You are doing two important things here. First, you're defining a new repository that Composer can reference when requiring packages. And second, you're requiring the package from the newly defined repository.

**Note:** The package version is a part of the _require_ statement and not a part of the _repository_ property. You can require a specific branch of a repo by choosing a version named "dev-<branch name>".

If you were to run `composer install` in the context of this file, Composer would look for a project at the defined URL. If that URL represents a Git repo that contains a `composer.json` file that defines its name and type, Composer will download that package to your project and place it in the appropriate location.

## Custom package types

The example package shown above is of the type "library," but with WordPress and Drupal you're dealing with plugins, modules, and themes. When requiring special package types in your project, it's important to install them in specific locations within the project file structure. Wouldn't it be nice if you could convince Composer to treat these types of packages as special cases? Well, you're in luck. There is an official Composer plugin that will do that for you.

The [Installers](https://github.com/composer/installers) plugin for Composer contains the custom logic required for handling many different package types for a large variety of projects. It is extremely helpful when working with projects that have well-known and supported package installation steps.

This project allows you to define package types like drupal-theme, drupal-module, wordpress-plugin, wordpress-theme, and many more, for a variety of projects. In the case of a drupal-theme package, the Installers plugin will place the required repo within the `/themes/contrib` folder of your Drupal installation.

Here is an example of a `composer.json` file that might live within a Drupal theme project as its own Git repository:

```
{
    "name": "mynamespace/whatever-i-call-my-theme",
    "type": "drupal-theme",
    "description": "Drupal 8 theme",
    "license": "GPL-2.0+"
}
```

Note that the only meaningful difference here is that the type is now drupal-theme. With the drupal-theme type defined, any project that uses the Installers plugin can easily require your repo in its Drupal project, and it will be treated as a contributed theme.

## Require any Git repository with Composer

What happens when the repo you want to include in your project does not define anything about itself with a `composer.json` file? When a repo does not define its name or type, you have to define that information for the repo within your project's `composer.json` file. Take a look at this example:

```
{
  "name": "mynamespace/my-project-that-uses-composer",
  "repositories": [
    {
      "type": "package",
      "package": {
        "name": "mynamespace/my-custom-theme",
        "version": "1.2.3",
        "type": "drupal-theme",
        "source": {
          "url": "https://github.com/mynamespace/my-custom-theme.git",
          "type": "git",
          "reference": "master"
        }
      }
    }
  ],
  "require": {
    "mynamespace/my-custom-theme": "^1",
    "composer/installers": "^1"
  }
}
```

Notice that your repository type is now "package." That is where you will define everything about the package you want to require.

Create a new object named "package" where you define all the essential information that Composer needs to know to be able to include this arbitrary git repo within your project, including:

- **Name:** The namespaced package name. It should probably match the repository you're requiring but doesn't have to.
- **Type:** The type of package. This reflects how you want Composer to treat this repository.
- **Version:** A version number for the repo. You will need to make this up.
- **Source:** An object that contains the following repository information:
    - **URL:** The Git or other version-control system (VCS) URL where the package repo can be found
    - **Type:** The VCS type for the package, such as git, svn, cvs, and so on
    - **Reference:** The branch or tag you want to download

I recommend reviewing the [official documentation](https://getcomposer.org/doc/05-repositories.md#package-2) on Composer package repositories. Note that it is possible to include zip files as Composer packages as well. Essentially, you are now responsible for all parts of how Composer treats this repository. Since the repository itself is not providing Composer with any information, you are responsible for determining almost everything, including the current version number for the package.

This approach allows you to include almost anything as a Composer package in your project, but it has some notable drawbacks:

- Composer will not update the package unless you change the `version` field.  
     
- Composer will not update the commit references. If you use `master` as a reference, you will have to delete the package to force an update, and you will have to deal with an unstable lock file.

## Custom package versions

Maintaining the package version in your `composer.json` file isn't always necessary. Composer is smart enough to look for GitHub releases and use them as the package versions. But eventually you will likely want to include a simple project that has only a few branches and no official releases.

When a repository does not have releases, you will be responsible for deciding what version the repository branch represents to your project. In other words, if you want composer to update the package, you will need to increment the "version” defined in your project's `composer.json` file before running `composer update`.

## Overriding a Git repository's composer.json

When defining a new Composer repository of the type package, you can override a package's own `composer.json` definitions. Consider a Git repository that defines itself as a library in its `composer.json`, but you know that the code is actually a drupal-theme. You can use the above approach to include the Git repository within your project as a drupal-theme, allowing Composer to treat the code appropriately when required.

Example: Require Guzzle as a drupal-theme just to prove that you can.

```
{
  "name": "mynamespace/my-project-that-uses-composer",
  "repositories": [
    {
      "type": "package",
      "package": {
        "name": "mynamespace/guzzle-theme",
        "version": "1.2.3",
        "type": "drupal-theme",
        "source": {
          "url": "https://github.com/guzzle/guzzle.git",
          "type": "git",
          "reference": "master"
        }
      }
    }
  ],
  "require": {
    "mynamespace/guzzle-theme": "^1",
    "composer/installers": "^1"
  }
}
```

This works! You've downloaded the Guzzle library and placed it within the `/themes` folder of your Drupal project. This is not a very practical example, but it highlights how much control the package type approach provides.

## Summary

Composer offers plenty of options for including arbitrary packages within a project. Determining how those packages are included in the project primarily comes down to who defines the package information. If the Git repository includes a `composer.json` file that defines its name and type, you can have Composer rely on the repository itself for the definition.

But if you want to include a repository that does not define its name and type, then it is up to your project to define and maintain that information for your own internal use. Alternatively, if a repository doesn't define a `composer.json` file, consider submitting a pull request that adds it.

---

_This article originally appeared on the [Daggerhart Lab blog](https://www.daggerhartlab.com/composer-how-to-use-git-repositories) and is republished with permission._

[![Creative Commons License](https://opensource.com/themes/osdc/assets/img/cc-by-sa-4.png "This work is licensed under a Creative Commons Attribution-Share Alike 4.0 International License.")](https://creativecommons.org/licenses/by-sa/4.0/)This work is licensed under a Creative Commons Attribution-Share Alike 4.0 International License.