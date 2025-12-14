---
link: https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/
site: EasyEngine
excerpt: Using composer to define themes and plugin needed for a WordPress site. Covers wordpress.org hosted, git/svn-hosted, public and private/premium zip files.
twitter: https://twitter.com/@easyengine
slurped: 2025-12-01T11:18
title: Managing WordPress themes and plugins with Composer
tags:
  - composer
---

As our basic [composer project is ready with desired version of WordPress](https://easyengine.io/tutorials/composer-wordpress/git-file-system-layout/), we can proceed to add themes and plugin to our project.

As WordPress ecosystem has tons of free and paid themes/plugins from different sources, we need to follow few different methods to handle them. We will see them one-by-one.

Composer makes it easy to use anything as dependency, even if target library/repo is not a composer package!

This involves fair amount of work, as we will see later in this chapter alone. But thanks to [wpackagist.org](http://wpackagist.org/) – a composer package repository created  by fine folks at [Outlandish](http://outlandish.com/), working with WordPress.org hosted themes and plugins becomes cakewalk!

[wpackagist.org](http://wpackagist.org/) homepage explains usage in nice details but we will try to over-simplify things for composer newbies here by breaking process in two steps.

### Adding wpackagist.org repository to composer.json[](https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/#addingwpackagistorgrepository-to-composerjson)

Composer by default only looks for packages hosted on [packagist.org](https://packagist.org/). So first thing we need to tell composer is to look for WordPress themes and plugins at [wpackagist.org](http://wpackagist.org/). This can be done by adding following lines to `composer.json` file:

```
"repositories":[
        {
            "type":"composer",
            "url":"https://wpackagist.org"
        }
    ],
```

You can add above lines automatically by running command `composer config repositories.0 composer https://wpackagist.org/` So `composer.json` file will look like:

```
{
    "name": "rtcamp/wordpress-composer-template",
    "description": "Using Composer to manage complete WordPress Project",
    "license": "MIT",
    "repositories": [
        {
            "type": "composer",
            "url": "https://wpackagist.org"
        }
    ],
    "require": {
        "johnpbloch/wordpress": "4.2.2",
    }
}
```

### Adding actual wordpress.org themes and plugins[](https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/#adding-actual-wordpressorg-themes-and-plugins)

Each wordpress theme and plugin has a unique slug. You can use that slug to search theme/plugin on [wpackagist.org](http://wpackagist.org/).  wpackagist.org will show you a result page with versions available trough composer. You can click on a version and composer will give one-line statements like:

- `"wpackagist-plugin/nginx-helper": "1.9.3"` – for [nginx-helper](https://wordpress.org/plugins/nginx-helper/) plugin
- `"wpackagist-theme/twentyfifteen": "1.2"` – for [twentyfifteen](https://wordpress.org/themes/twentyfifteen/) theme

You can copy code blocks to require section in composer.json like below:

```
{
    "name": "rtcamp/wordpress-composer-template",
    "description": "Using Composer to manage complete WordPress Project",
    "license": "MIT",
    "repositories": [
        {
            "type": "composer",
            "url": "https://wpackagist.org"
        }
    ],
    "require": {
        "johnpbloch/wordpress": "4.2.2",
        "wpackagist-theme/twentyfifteen": "1.2",
        "wpackagist-plugin/nginx-helper": "1.9.3"
    }
}
```

Once `composer.json` file is updated, you can run composer install/update to download dependencies. **Command Line Way** Alternatively, you can run following composer command to add dependency without opening composer.json file

`composer require wpackagist-theme/twentyfifteen:1.2 wpackagist-plugin/nginx-helper:1.9.3`

Please note that composer require will update `composer.json` and also download packages right away.  If you want to prevent automatic downloading of dependencies, you can pass flag `--no-update` to above command.

## Managing non-WordPress.org themes and plugins[](https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/#managing-non-wordpressorg-themes-and-plugins)

Unlike above, this is very broad. Mainly we need to deal with:

1. Theme/Plugin published as composer package via [packagist.org](https://packagist.org/)
2. Theme/Plugin under version control – **with** composer support
3. Theme/Plugin under version control – **without** composer support
4. Theme/Plugin **without** version control e.g. zip file

Let’s deal with each of above with an example.

### Theme/Plugin published as composer package via [packagist.org](https://packagist.org/)[](https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/#themeplugin-published-as-composer-package-viapackagistorg)

As an example, let’s use [github-updater](https://github.com/afragen/github-updater) WordPress plugin which is [not allowed](https://github.com/afragen/git-updater/issues/34) on wordpress.org repo but available as composer package at – [https://packagist.org/packages/afragen/github-updater](https://packagist.org/packages/afragen/github-updater) You can add a line like below to composer.json file’s require section and it will work just fine

`"afragen/github-updater": "^4.6"`

Or can run composer command `composer require afragen/github-updater`

Above works so easily because github-updater is composer package. Things will not be easy for remaining cases!

### Theme/Plugin under version control – with composer support[](https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/#themeplugin-under-version-control-with-composer-support)

There could be a case that a project is neither hosted on wordpress.org or in some composer repo _but_ still has composer.json in it.

As an example consider this plugin – [https://github.com/rtCamp/rtAntiSpam](https://github.com/rtCamp/rtAntiSpam) we use on our sites but did not release it on wordpress.org! This plugin has a [composer.json file](https://github.com/rtCamp/rtAntiSpam/blob/master/composer.json) you may like to have a look at.

To install such theme/plugin, we need to perform two steps:

**Add version control project as a new composer repository (like wpackagist.org)**

We can do that by adding following to composer.json file:

```
"repositories": [
        {
            "type": "composer",
            "url": "https://wpackagist.org/"
        },
        {
            "type": "vcs",
            "url": "[email protected]:rtCamp/rtAntiSpam.git"
        }
    ],
```

Please note that repo type is`vcs` (or we can use git). For wpackagist.org it was`composer` since wpackagist.org is a composer repository.

Above can be done via command line as well:

`composer config repositories.rtantispam vcs [[email protected]](https://easyengine.io/cdn-cgi/l/email-protection):rtCamp/rtAntiSpam.git`

Adding repos via command updates composer.json file in slight different way:

```
"repositories": {
        "0": {
            "type": "composer",
            "url": "https://wpackagist.org/"
        },
        "rtantispam": {
            "type": "vcs",
            "url": "[email protected]:rtCamp/rtAntiSpam.git"
        }
    },
```

The difference really don’t matter so ignore it if it’s not making sense.

**Add actual composer package**

Once we add our git repo, we can add it as a package like below.

```
"require": {
        "rtcamp/rtantispam": "dev-master"
    }
```

`dev-master` will point to git’s master branch. This is really bad practise but we are explaining it this way because many developers (including us) do not use git tags for internal projects. 😐

For command-line interface, we can achieve above by running:

`composer require rtcamp/rtantispam:dev-master`

If you have taken look at [composer.json for rtAntiSpam plugin](https://github.com/rtCamp/rtAntiSpam/blob/master/composer.json), you may notice it has package name equal to `rtcamp/rtantispam`.

We need to match package name with `composer.json`.

Please note that `composer.json` in your project is different than composer.json in other project.

### Theme/Plugin under version control – without composer support[](https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/#themeplugin-under-version-control-without-composer-support)

Composer allow you to dynamically include any version controlled repo as a package, even if destination doesn’t contain `composer.json` file!

As an example, we will use a plugin which is neither hosted on wordpress.org, nor has composer.json.

Again this will need two steps.

**Add version control project as a new composer repository**

```
{
    "type": "package",
    "package": {
        "name": "digitalmedia/rollbar",
        "type": "wordpress-plugin",
        "version": "dev-master",
        "source": {
            "type": "git",
            "url": "[email protected]:digitalmedia/rollbar.git",
            "reference": "master"
        }
    }
}
```

If you have taken look at [composer.json for rtAntiSpam](https://github.com/rtCamp/rtAntiSpam/blob/master/composer.json) then some fields like `name` and `type` are similar.

You may notice repo type is now **package**, earlier it was **vcs** and before that it was **composer**. So when a target repo is not a composer package, we can declare it as a package on our own end by adding it to repositories list.

To avoid confusion and clear context, below is complete repositories block from composer.json

```
"repositories": {
        "0": {
            "type": "composer",
            "url": "https://wpackagist.org/"
        },
        "rtantispam": {
            "type": "vcs",
            "url": "[email protected]:rtCamp/rtAntiSpam.git"
        },
        "rollbar":{
            "type": "package",
            "package": {
                "name": "digitalmedia/rollbar",
                "type": "wordpress-plugin",
                "version": "dev-master",
                "source": {
                    "type": "git",
                    "url": "[email protected]:digitalmedia/rollbar.git",
                    "reference": "master"
                }
            }
        }
    },
```

**Notes:**

1. In reality, package type `wordpress-plugin` has dependency on `composer/installers` but `wordpress-core` composer package already satisfies it. So we are avoid duplicate lines in our composer.json file to keep it lean.
2. We really don’t know how to define entire package via composer command. But as you can see it has plenty of lines, we will prefer to copy-paste and edit.
3. Composer doesn’t directly support custom path if package doesn’t have `composer.json` in it’s source code with `composer/installers` as dependency but because we are using `wpackagist` if we specify “`type"` as “`wordpress-plugin"` or “`wordpress-theme"` it will take care of it.

**Add actual composer package**

Once we add our git repo, we can add it as a package like below.

```
"require": {
        "digitalmedia/rollbar": "dev-master"
    }
```

`dev-master` will point to git’s master branch. Rollbar git repo also doesn’t have any tags. For command-line interface, we can achieve above by running:

`composer require digitalmedia/rollbar:dev-master`

**Note:** when defining package like this, you can use any package name. It no need to be **githuborg/githubrepo**. Only care you need to take is to keep **githubrepo** unique so it won’t conflict with any other WordPress package.

### Theme/Plugin without version control _e.g. zip file_[](https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/#themeplugin-without-version-control-eg-zip-file)

This applies to packages available without any version control. This is common for premium themes and plugins which are often delivered as zip file.

For sake of example, let’s take example of our premium product [rtBiz Helpdesk](https://rtbiz.io/helpdesk/). We sell this plugin using [easydigitaldownloads.com](https://easydigitaldownloads.com/) which generates a zip file link like below for each download:

https://easyengine.io/index.php?eddfile=[FILE]&ttl=[TTL]&file=0&token=[TOKEN]

We can add such zip file links directly as a composer package by following two steps like above.

**Add zip file as a new composer repository**

```
{
    "type": "package",
    "package": {
        "name": "rtcamp/rtbiz-helpdesk",
        "type": "wordpress-plugin",
        "version": "1.0.0",
        "dist": {
            "type": "zip",
            "url": "https://easyengine.io/index.php?eddfile=[FILE]&ttl=[TTL]&file=0&token=[TOKEN]"
        }
    }
}
```

Above example will look familiar.

**Notes:**

1. If you do not have a unique download link for zip file, you can upload zip file on a private server and use that link. As long as it’s a valid link, composer won’t complain.
2. You may notice version field is set to 1.0.0. This won’t matter as zip packages won’t have version control data. You may keep this version updated to remind yourself which package version you are using.

**Add actual composer package**

Once we add our zip repo, we can add it as a package like below.

```
"require": {
        "rtcamp/rtbiz-helpdesk": "dev-master"
    }
```

Again version won’t matter for zip files. **For command-line interface, we can achieve above by running:**

`composer require rtcamp/rtbiz-helpdesk:dev-master`

## Note about private git/svn repos[](https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/#note-about-private-gitsvn-repos)

Composer can works transparently with private git/svn repo, provided machine on which composer running is pre-authenticated.

As an example, if you are cloning git repos from Github using git-based URL and SSH public key for that machine is already added to your Github profile, composer will work without any issue.

Please avoid HTTP based URLs when defining vcs repositories in composer.

## Summary[](https://easyengine.io/tutorials/composer-wordpress/manage-themes-plugins/#summary)

As you can see, we tried to cover all cases so you can define all themes and plugins as composer dependency and manage entire WordPress projects from a single `composer.json` file.

Composer by default will only download dependencies. In next chapter, we will deal with theme/plugin activation, again with the help of composer!