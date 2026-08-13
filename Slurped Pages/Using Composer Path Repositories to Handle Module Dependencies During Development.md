---
link: https://drupalize.me/blog/using-composer-path-repositories-handle-module-dependencies-during-development
site: Drupalize.Me
excerpt: Sometimes working on a Drupal contributed module requires making changes to the module’s composer.json so that you can update a dependency. This blog post looks at how to accomplish that in a local development environment.
twitter: https://twitter.com/@drupalizeme
slurped: 2026-06-27T15:14
title: Using Composer Path Repositories to Handle Module Dependencies During Development
tags:
  - composer
---

Sometimes working on a Drupal contributed module requires making changes to the module’s _composer.json_ so that you can update a dependency.

Have you run into this before? You’re working on a Drupal contrib module, and that module has a dependency on either another module or an external Composer package. That dependency is expressed in the module’s _composer.json_ file and likely has a version constraint. So what happens when you want to update the module to use a newer version of the dependency, but the new version is outside the existing version constraint?

This is super common when the contributed module relies on a third-party library, like the Recurly module using the Recurly API SDK, or the OAuth module using the PHP library common to many different projects.

Here’s a recent example. I was working on the [drupal/mcp_server module](https://www.drupal.org/project/mcp_server), which currently has a dependency on the [modelcontextprotocol/php-sdk](https://github.com/modelcontextprotocol/php-sdk) (mcp/sdk). I started by installing the module with `composer require drupal/mcp_server`. This gets the module and any dependencies.

The `require` section of the drupal/mcp_server module’s _composer.json_ file looks like this:

```
  "require": {
    "php": "^8.3",
    "drupal/tool": "1.0.0-alpha9",
    "drupal/simple_oauth": "^6",
    "e0ipso/simple_oauth_21": "^1",
    "mcp/sdk": "^0.1.0"
  },
```

I want to update the module to use the newer `0.3.x` version of the mcp/sdk. I can’t just `composer update mcp/sdk` because the version constraint doesn’t allow the newest release, and `composer require mcp/sdk^0.3.0` won’t work because it’ll result in a dependency conflict. I can’t just modify the local _composer.json_ file in the module because, when resolving the dependency tree, Composer will ping Drupal.org (Packagist) and use the version constraint from the hosted package, **not from the locally installed module**.

So how do I get my project to install `mcp/sdk^0.3.0` so that I can work on updating the module’s code to use the new library?

## Composer path repositories

One way to accomplish this is using a _path_ repository. Composer installs packages from a _repository_: a list of packages and versions. By default, only the Packagist.org repository is registered. You can change this by adding repository configuration to your project’s _composer.json_ file and telling Composer to use additional repositories when resolving dependencies.

Drupal actually does this already. If you look at your root _composer.json_ file, you’ll see something like this:

```
    "repositories": [
        {
            "type": "composer",
            "url": "https://packages.drupal.org/8"
        }
    ],
```

That configuration tells Composer that, in addition to Packagist.org, it should use packages.drupal.org/8 as a place to search for packages. And that’s why packages like `drupal/mcp_server` work.

There are a bunch of different types of repositories, including _path_, which allows you to depend on a local directory. Read more about it here: [https://getcomposer.org/doc/05-repositories.md#path](https://getcomposer.org/doc/05-repositories.md#path)

This approach works really well if you’re working on a contributed module within the context of a complete Drupal project. For example, working on adding an MCP server to the Drupalize.Me site and wanting to make (and test) the updates for the mcp/sdk in the context of that site.

### Set up a path repository to work on a Drupal module

In the project’s root _composer.json_ (the one that currently requires drupal/mcp_server, or whatever module you want to work on), add the following configuration:

```
  "repositories": [
    {
      "type": "path",
      "url": "web/modules/contrib/mcp_server",
      "options": {
        "symlink": false
      }
    },
    {
      "type": "composer",
      "url": "https://packages.drupal.org/8"
    }
  ],
```

This tells Composer to use _web/modules/contrib/mcp_server_ as a repository. So when it goes to resolve `drupal/mcp_server`, it’ll find _web/modules/contrib/mcp_server/composer.json_, which has `"name": "drupal/mcp_server",` and say “Got it!” No more pinging Drupal.org for that specific package. Dependencies are resolved by searching through the repositories top to bottom until a match is found.

After making those changes, we can remove the Composer-managed version, clone the Git repo, and then tell Composer to use the locally cloned package:

```
# Remove the composer-managed version.
composer remove drupal/mcp_server

# Git clone your module.
cd web/modules/contrib
git clone https://github.com/yourusername/mcp_server.git mcp_server
cd ../../..

# Tell Composer to use the local path.
composer require drupal/mcp_server:@dev
```

### Update the MCP SDK

Finally, we can update the mcp/sdk dependency.

Edit _web/modules/contrib/mcp_server/composer.json_:

```
{
  "require": {
    "mcp/sdk": "^0.3.0"
  }
}
```

Then run:

```
composer update mcp/sdk --with-all-dependencies
```

This should result in the `0.3.0` version of the `mcp/sdk` being downloaded into the _vendor/_ directory. And you can now begin the work of updating the contributed module’s code to use the latest version of the third-party dependency.