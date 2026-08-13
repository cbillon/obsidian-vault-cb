---
link: https://www.mikestreety.co.uk/blog/add-custom-repositories-for-composer-to-pull-from/
byline: Mike Street
site: Mike Street
date: 2022-05-13T02:00
excerpt: This post covers several main types of custom composer repositories and how to use them
slurped: 2026-06-27T11:26
title: Add custom repositories for composer to pull from
tags:
  - composer
---

2022-05-13T00:00:00.000Z Posted on **13th May 2022**. **4 mins** reading time

Packagist is a great resource for Composer packages; the NPM of the PHP world and composer is already set up to work with it. However, there may be times where you wish to include a local package or pull in a package from somewhere else - such as a self-hosted [Private Packagist](https://packagist.com/) or [Gitlab](https://about.gitlab.com/) package repository.

With Composer, you can specify in your `composer.json` where you wish to pull the packages from.

## Local folder - type: path

If you wish to use a local folder, such as `src` or `app` (such as with this example), you can do so with the following addition to your `composer.json` file within the `repositories` array:

```
{
    "name": "project/name",
    "description": "Description of your app",
    "type": "project",
    "repositories": [
        {
            "type": "path",
            "url": "./app/*"
        }
    ],
    "require": {
        "app/name": "@dev"
    }
}
```

Once you've included your custom location, you can include the package/extension/plugin inside your `composer.json` as though it was remote. You can use `dev-*` branches, version numbers, `*` or `@dev`. Using `@dev` indicates the _current_ branch.

```
{
    "name": "project/name",
    "description": "Description of your app",
    "type": "project",
    "repositories": [
        {
            "type": "path",
            "url": "./app/*"
        }
    ],
    "require": {
        "app/name": "@dev"
    }
}
```

When using this method, Composer creates a symlink to the package inside of the path repository.

## Custom remote repositories - type: composer (Gitlab)

We use a self-hosted version of Gitlab to publish and host our private packages. This saves us having to use git repositories or symlinking and keeps us as close to the Composer ecosystem as possible.

Once our packages are [built and released using Gitlab CI](https://www.mikestreety.co.uk/blog/build-and-release-composer-packages-using-a-self-hosted-gitlab/), we set a custom composer location in our "repositories" section of the `composer.json`.

There are a few more bits to update with this one:

- `URL.TO.GITLAB` - this is the base URL to the Gitlab install where your packages are hosted, such as `gitlab.com`
- `ID` - this is the ID of the group where your packages are hosted

```
{
    "name": "project/name",
    "description": "Description of your app",
    "type": "project",
    "config": {
        "gitlab-domains": [
            "URL.TO.GITLAB"
        ]
    },
    "repositories": [
        {
            "type": "composer",
            "url": "https://URL.TO.GITLAB/api/v4/group/ID/-/packages/composer/packages.json"
        }
    ],
    "require": {
        "app/name": "^1.0"
    }
}
```

The `gitlab-domains` section under `config` tells Composer how to handle the JSON from the Gitlab API. If your packages are private, you will need to look at setting a [Gitlab auth token](https://getcomposer.org/doc/articles/authentication-for-private-packages.md) to access.

Once you've included your custom repository you can include your packages as you would off Packagist. When using this method, Composer downloads the files from the location as specified in the packages Composer file - this includes omitting the git repository too.

## Git repository - type: git

The last type we're going to cover in this post is `type: git`. This is useful if you are developing a package which will be uploaded. It tells Composer to _clone_ the repository instead of downloading the package or creating a symlink.

First step, as before is to include the custom repository location. Make sure you include this **before** it's original source (e.g. the Gitlab API) as composer works on a **first come, first served** basis when searching for packages

```
{
    "name": "project/name",
    "description": "Description of your app",
    "type": "project",
    "repositories": [
        {
            "type": "git",
            "url": "[email protected]:packages/path/to/repo.git"
        },
    ],
    "require": {
        "app/name": "dev-main"
    }
}
```

This uses your SSH key, so if you have access to the repo it will clone it down. You can then choose which branch to use (by preceding it with `dev-`) or you can still use a tag. This is useful if you've not published your package yet.

## Other means

There are plenty of [other types](https://getcomposer.org/doc/05-repositories.md#types) of repository for Composer, but these are the main 3 I use every day.