---
link: https://www.daggerhartlab.com/composer-how-to-use-git-repositories/
byline: Jonathan Daggerhart
site: Daggerhart Lab
date: 2019-09-13T15:21
excerpt: How to use composer to require Git repos within a PHP project.
twitter: https://twitter.com/@daggerhart
slurped: 2026-06-26T19:01
title: "Composer: How to use Git repositories - Daggerhart Lab"
tags:
  - composer
  - git
---

It’s not uncommon to find ourselves wanting to use composer to include Git repositories within our PHP project. In many cases repositories have been created on [packagist.org](https://packagist.org/) and requiring them with composer is very straight forward. But what do we do when a repository has not been created as a package on packagist? The answer is that we use composer to require the package directly from the repository.

**Note:** Some of the terminology in the post is confusing because multiple words are used to describe different things. Here is a quick vocabulary list that will help:

- **Project** – The custom software we are building. This can be a website, a command line utility, application, or anything else we dream up.
- **Package** – Any 3rd party software we want to download and use within our project. It can be a library, Drupal theme, WordPress plugin, or any other number of things.
- **Git repository** – AKA, _Git repo_. The version control host for a package. Common hosts are: GitHub, GitLab, or Bitbucket; but any URL accessible Git repository will work for this tutorial.
- **Composer repositories** – In a _composer.json_ file there is an optional property named “_repositories_“. This property is where we can define new places for Composer to look when downloading packages.

When adding a Git repo to our project with composer there are two situations we can find ourselves in: the repo contains a _composer.json_ file and it defines how the repo should be handled when required, or it does not. Regardless, in both cases we are able to add the Git repository to our project.

Let’s start with a Git repo that has defined a _composer.json_ file.

## Git repo with _composer.json_

When a repository includes a _composer.json_ file, it defines aspects of itself that are important to how composer manages the package. Here is an example of a simple _composer.json_ file a package may include.

```
{    "name": "daggerhart/my-custom-library",    "type": "library"}
```

composer.json

Shown are two important properties that a _composer.json_ file can define:

- _**name**_: The package’s namespaced name. In this case _daggerhart_ is the namespace for the package _my-custom-library_.
- _**type**_: The type of package the repo represents. [Package types](https://getcomposer.org/doc/04-schema.md#type) are used for installation logic. Out of the box, composer allows for the following package types: _library_, _project_, _metapackage_, _composer-plugin_.

You can verify this by looking at the [composer.json](https://github.com/guzzle/guzzle/blob/master/composer.json) file of any [popular](https://github.com/slimphp/Slim/blob/4.x/composer.json) [project](https://github.com/Seldaek/monolog/blob/master/composer.json). They each define their package name and the package type near the top of the file.

When a repository has this information defined in its _composer.json_ file, requiring the repository within our project is relatively straight-foward.

### Require a Git repository that has a _composer.json_ file

Now that we’ve identified a GIt repository with a _composer.json_ file, let’s require that repository as a package within our project.

Within our project’s _composer.json_ file we need to define a new property (assuming it doesn’t exist already) named “_repositories_“. The value of the repositories property is an array of objects. Each object containing information about the repository we want to include in our project. Consider this _composer.json_ file for our custom project.

```
{  "name":  "daggerhart/my-project-that-uses-composer",  "repositories": [    {      "type": "vcs",      "url": "https://github.com/daggerhart/my-custom-library.git"    }  ],  "require": {    "daggerhart/my-custom-library": "dev-master"  }}
```

composer.json

We are doing two important things here. First, we’re defining a new repository that composer can reference when requiring packages. And second, we’re requiring the package from the newly defined repository.

> **Note:** The package _version_ is a part of the _require_ statement, and not a part of the _repository_ property. We can require a specific branch of a repo by choosing a version named “_dev-<branch name>_“.

If we were to run `composer install` in the context of this file, composer would look for a project at the defined URL, and if that URL represents a Git repo that contains a composer.json file that defines its _name_ and _type_, composer will download that package to our project and place it in the appropriate location.

### Custom Package Types

The example package shown above is of the type “_library_“, but often in our work with WordPress and Drupal we’re dealing with plugins, modules, and themes. When requiring special package types in our project it’s important for them to be installed in specific locations within the project file structure. Wouldn’t it be nice if we could convince composer to treat these types of packages as special cases? Well we’re in luck. There is an official composer-plugin that will do that for us.

[composer/installers](https://github.com/composer/installers) – The _installers_ composer plugin contains the custom logic required for handling many different package types for a large variety of projects. It is extremely helpful when working with projects that have well known and supported package installation steps.

This project allows us to define package types like: _drupal-theme_, _drupal-module_, _wordpress-plugin_, _wordpress-theme_, and many more, for a variety of projects. In the case of a _drupal-theme_ package, the _composer/installers_ plugin will place the required repo within the /_themes/contrib_ folder of our Drupal installation.

Here is an example of a _composer.json_ file that might live within a Drupal theme project as its own Git repository:

```
{    "name": "daggerhart/whatever-i-call-my-theme",    "type": "drupal-theme",    "description": "Drupal 8 theme",    "license": "GPL-2.0+"}
```

composer.json

Note that the only important difference here is that the “_type_” is now “_drupal-theme_“. With the _drupal-theme_ type defined, any project that uses the _composer/installers_ plugin can easily require our repo in their Drupal project and it will be treated as a contributed theme.

## Require Any Git Repository with Composer

Now that we know how to require a Git repo that contains a _composer.json_ file, let’s consider the case where the repo we want to include in our project does not define anything about itself with a _composer.json_ file.

The short of it is, when a repo does not define its _name_ or _type_, we have to define that information for the repo within our project’s _composer.json_ file. Take a look at this example:

```
{  "name": "daggerhart/my-project-that-uses-composer",  "repositories": [    {      "type": "package",      "package": {        "name": "daggerhart/my-custom-theme",        "version": "1.2.3",        "type": "drupal-theme",        "source": {          "url": "https://github.com/daggerhart/my-custom-theme.git",          "type": "git",          "reference": "master"        }      }    }  ],  "require": {    "daggerhart/my-custom-theme": "^1",    "composer/installers": "^1"  }}
```

composer.json

Notice that our repository type is now “_package_“. That is where we are going to define everything about the package we want to require. Next, we create a new object named “_package_” where we define all the important information that composer needs to know to be able to include this arbitrary git repo within our project. Including:

- _**name**_ – The namespaced package name. It should probably match the repository we’re requiring, but doesn’t have to.
- _**type**_ – The type of package. How we want composer to treat this repository.
- _**version**_ – A made up version number for the repo.
- _**source**_ – An object that contains the following repository information:
    - _**url**_ –  The Git (or other VCS) URL where the package repo can be found.
    - _**type**_ – The VCS type for the package. _git_, _svn_, _cvs_ (?)… etc.
    - _**reference**_ – The branch or tag we want to download.

I recommend reviewing the [official documentation on composer package repositories](https://getcomposer.org/doc/05-repositories.md#package-2). Note that it is possible to include zip files as composer packages as well.

Essentially, we are now responsible for all parts of how composer treats this repository. Since the repository itself is not providing composer with any information, we are responsible for determining almost everything, including the current version number for the package.

This approach allows us to include almost anything as a composer package in our project, but it has some drawbacks. Notable drawbacks:

- Composer will not update the package unless you change the `version` field.
- Composer will not update the commit references, so if you use `master` as reference you will have to delete the package to force an update, and will have to deal with an unstable lock file.

### Custom Package Versions

Maintaining the package version in our _composer.json_ file isn’t always necessary. Composer is smart enough to look for GitHub releases and use them as the package versions. But eventually we will find ourselves wanting to include a simple project that only has a few branches and no official releases.

When a repository does not have releases, we will be responsible for deciding what version the repository branch represents to our project. In other words, if we want composer to update the package, we will need to increment the “_version_” defined in our project’s _composer.json_ file before running `composer update`.

### Overriding a Git Repository’s _composer.json_

An interesting thing we can do when defining a new composer repository of the type _package_, is override a package’s own _composer.json_ definitions. Consider a Git repository defines itself as a _library_ in its _composer.json_, but we know that the code is actually a _drupal-theme_. We can use this above approach to include the Git repository within our project as a _drupal-theme_, allowing composer to treat the code appropriately when required.

Example: require Guzzle as a drupal-theme just to prove that we can.

```
{  "name": "daggerhart/my-project-that-uses-composer",  "repositories": [    {      "type": "package",      "package": {        "name": "daggerhart/guzzle-theme",        "version": "1.2.3",        "type": "drupal-theme",        "source": {          "url": "https://github.com/guzzle/guzzle.git",          "type": "git",          "reference": "master"        }      }    }  ],  "require": {    "daggerhart/guzzle-theme": "^1",    "composer/installers": "^1"  }}
```

composer.json

This works! This will download the Guzzle library and place it within the _/themes_ folder of my Drupal project. Obviously this is not a very practical example, but hopefully it highlights how much control the _packge_ type approach provides.

## Summary

Composer offers us plenty of options for including arbitrary packages within our project. The question of how those packages are included in the project primarily comes down to “_who is definining the package information?_“. If the Git repository includes a _composer.json_ file that defines its _name_ and _type_, then we can have composer rely on the repository itself for the definition. But if we want to include a repository that does not define its _name_ and _type_, then it is up to our project to define and maintain that information for our own internal use.

Alternatively, if a repository doesn’t define a composer.json file, consider submitting a pull request that adds it. ?

**References:**

- [Composer: custom package definition](https://getcomposer.org/doc/05-repositories.md#package-2)
- [Composer: using VCS repos](https://getcomposer.org/doc/05-repositories.md#vcs)
- [Drupal StackExchange: How to add theme from GitHub with Composer](https://drupal.stackexchange.com/a/277473/25221)