---
link: https://amitmerchant.com/composer-scripts/
byline: Amit Merchant
site: Amit Merchant - A blog on PHP, JavaScript, and more
date: 2024-10-22T02:00
excerpt: I was watching this video by Nuno Maduro the other day where he was explaining how they are using Composer Scripts to simplify the process of running multiple commands in their flagship product, Laravel Cloud.
slurped: 2025-10-13T11:37
title: Combine and simplify multiple commands using Composer Scripts
tags:
  - composer
---

I was watching [this video](https://www.youtube.com/watch?v=iVcLGN5Kcyc) by Nuno Maduro the other day where he was explaining how they are using Composer Scripts to simplify the process of running multiple commands in their flagship product, [Laravel Cloud](https://cloud.laravel.com/).

Essentially, when your project is big, it relies upon a lot of smaller packages to work together. For instance, your project might rely upon different packages to test both the front-end and back-end of your application. Let’s say PHPUnit/Pest and Jest respectively.

Running these commands separately will obviously work but it’s time-consuming as you can imagine and this is where Composer Scripts can simplify this to some extent.

[Composer Scripts](https://getcomposer.org/doc/articles/scripts.md) lets you use a command or a combination of commands under a named event.

We can define Composer Scripts in the root JSON object in `composer.json` should have a property called **“scripts”**.

For instance, if we were to check the linting for both the frontend and backend, we can do it by combining the commands in a single named event like so.

```
{
    "scripts": {
        "lint": [
            "pint",
            "npm run lint"
        ]
    }
}
```

This will run the commands in the order they are defined. So, in this case, it will first run the `pint` command and then run `npm run lint`.

**So, when you run `composer lint`, it will run both commands in the order they are defined.**

You can get even further by referencing the existing Composer Script events using `@` to create a new event.

**For instance, if we want to combine all the test suite commands under a single command called `composer test`, we can do it like so.**

```
{
    "scripts": {
        "test:lint": [
            "pint",
            "npm run lint"
        ],
        "test:types": [
            "phpstan analysis",
            "npm run test:types"
        ],
        "test:unit": "pest --parallel --ci --coverage",
        "test:browser": [
            "php artisan dusk --bail",
        ],
        "test": [
            "@test:lint",
            "@test:types",
            "@test:unit",
            "@test:browser"
        ]
    }
}
```

As you can tell, the `test` script will run all the tests. The `test:lint` script will run the linters, the `test:types` script will run the type checker, the `test:unit` script will run the unit tests, and the `test:browser` script will run the browser tests.

This is pretty cool since you don’t have to remember the commands yourself and you can still run the individual commands if you want to. Apart from this, it also makes it easy to configure the CI/CD pipeline to run all the tests.