---
link: https://amitmerchant.com/lesser-known-composer-tricks-tips/
byline: Amit Merchant
site: Amit Merchant - A blog on PHP, JavaScript, and more
date: 2020-07-09T02:00
excerpt: The Composer dependency manager is currently the de-facto way of managing dependencies in your PHP projects. All the modern PHP development can’t be imagined without Composer. But do you know there are a lot of things that you can do with Composer other than just installing and updating dependencies?
slurped: 2025-10-13T11:26
title: Lesser known Composer tricks and tips you should know
tags:
  - composer
---

The [Composer](https://getcomposer.org/) dependency manager is currently the de-facto way of managing dependencies in your PHP projects. All the modern PHP development can’t be imagined without Composer. But do you know there are a lot of things that you can do with Composer other than just installing and updating dependencies?

That’s right! I’m going to enlist such features in this article that you might be unaware of Composer and can help you improve your Composer workflow.

- [List all the installed dependencies of the project](https://amitmerchant.com/lesser-known-composer-tricks-tips/#list-all-the-installed-dependencies-of-the-project)
- [Get information about a certain package](https://amitmerchant.com/lesser-known-composer-tricks-tips/#get-information-about-a-certain-package)
- [Navigate to package’s GitHub repository/Homepage](https://amitmerchant.com/lesser-known-composer-tricks-tips/#navigate-to-packages-github-repositoryhomepage)
- [Validate `composer.json`](https://amitmerchant.com/lesser-known-composer-tricks-tips/#validate-composerjson)
- [Find outdated packages](https://amitmerchant.com/lesser-known-composer-tricks-tips/#find-outdated-packages)
- [Get funding information of all the packages](https://amitmerchant.com/lesser-known-composer-tricks-tips/#get-funding-information-of-all-the-packages)
- [Self update the Composer from Composer](https://amitmerchant.com/lesser-known-composer-tricks-tips/#self-update-the-composer-from-composer)
- [In closing](https://amitmerchant.com/lesser-known-composer-tricks-tips/#in-closing)

### List all the installed dependencies of the project

Using the `show` command of the Composer, you can get the list of all the packages installed in the project or all your repositories in the following format.

```
$ composer show 
# List the packages that are installed (this is enabled by default, and deprecated).

$ composer show --all 
# List all packages available in all your repositories.
```

This can come in handy when you want to take an overview of which all kinds of dependencies your project is using.

### Get information about a certain package

Using the same `show` command you can get the information about a certain package. For instance, if you want to get information about `spatie/laravel-web-tinker` package, you can do it like so.

```
$ composer show spatie/laravel-web-tinker
```

The command will fetch the details from the package’s `composer.json` file and presents in a nice and readable way like so.


You can even get the information about the certain package version, which will only tell you the details of that specific version like so.

```
$ composer show spatie/laravel-web-tinker 1.0.0
```

### Navigate to package’s GitHub repository/Homepage

Using the `browse` command, you navigate to the package’s repository URL or homepage in your browser.

```
$ composer browse spatie/laravel-web-tinker 
# Navigate to package's repository URL

$ composer browse spatie/laravel-web-tinker --homepage 
# Navigate to package's homepage

$ composer browse spatie/laravel-web-tinker --show
# Only show the homepage or repository URL
```

### Validate `composer.json`

You can validate your project’s `composer.json` which helps find issues with your `composer.json` (if there are any).

For instance, when I ran the command in my project, I got the following output with few issues such as using `dev-master` as a version for certain packages which should be avoided.


### Find outdated packages

If you’re keeping all your dependencies latest, you can keep a check on outdated dependencies using an `outdated` command like so.

It will list all the outdated dependencies like `composer show` with the following color coding:

- **green (=)**: Dependency is in the latest version and is up to date.
- **yellow (~)**: Dependency has a new version available that includes backward compatibility breaks according to semver, so upgrade when you can but it may involve work.
- **red (!)**: Dependency has a new version that is semver-compatible and you should upgrade it.

### Get funding information of all the packages

If you’re ever interested in funding/donating to your favorite packages, you can get all the funding links from the installed dependencies of your project using the `fund` command.

It will show all the funding details like so.


## Self update the Composer from Composer

Lastly, you can even update the composer to the latest from the composer itself. To update Composer itself to the latest version, run the `self-update` command. It will replace your `composer.phar` with the latest version.

Or if you want to update to the stable version of Composer, you can use `--stable` like so. This is the recommended way to update the Composer.

```
$ composer self-update --stable
```

## In closing

That’s about it. I never knew all these cool features until I stumbled upon them. I hope you learned something new about our beloved Composer. Let me know in the comments which one of all these lesser-known features you liked the most.

Until next time!