---
link: https://medium.com/@mazraara/create-a-composer-package-and-publish-3683596dec45
byline: Azraar Azward
site: Medium
date: 2019-08-11T08:19
excerpt: Create a composer package and publish I created and published a
  Composer package this time, so i thought to share the steps involved. (Also
  some tips specific to Laravel.) Following is the package …
twitter: https://twitter.com/@mazraara
slurped: 2026-08-16T09:07
title: Create a composer package and publish
---

I created and published a Composer package this time, so i thought to share the steps involved. (Also some tips specific to Laravel.)

Following is the package that was published and i will be using it as an example to explain.  
[https://github.com/mazraara/random-password](https://github.com/mazraara/random-password)

Let’s start now.

1. Create a public repository on GitHub.

2. Create composer.json at the top level of the repository.

> _With `composer init` you can make a quick template_

{  
  "name": "mazraara/random-password",  
  "description": "Generate random password of a given length",  
  "type": "library",  
  "require": {  
    "php": ">=5.3.0"  
  },  
  "license": "proprietary",  
  "authors": [  
    {  
      "name": "Azraar Azward",  
      "email": "[mazraara@gmail.com](mailto:mazraara@gmail.com)"  
    }  
  ],  
  "minimum-stability": "dev"  
}

3. Write a readme.

Be sure to include a README for users at least with following,

The name of the package  
How to install  
How to use  
License (GitHub function can create LICENSE file)

4. Write the package code.

Let’s make an actual package.

> Please note the following points.

src directory is not mandatory, but it seems that you often put code in a directory  
Write as many tests as possible  
Once ready, add an autoload section to composer.json to allow to use the namespace as RandomPassword\Password

"autoload": {  
    "psr-4": {  
      "RandomPassword\\": "src/RandomPassword/"  
    }

> Tips for Laravel package

In Laravel 5.5 and later, there is a function called auto-discovery that automatically registers ServiceProvider when it is created by itself.

Write the following in the extra section of composer.json:

"extra": {  
    "laravel": {  
      "providers": [  
        "RandomPassword\\PasswordServiceProvider"  
      ]  
    }  
  }

When developing Laravel packages you can’t use Laravel-specific features like service containers, migrations, etc for writing tests so need to use the following package for that purpose. [orchestra/testbench](https://github.com/orchestral/testbench)

"require-dev": {  
    "orchestra/testbench": "^3.8"  
  }

5. Finally, Register at Packagist ([https://packagist.org/](https://packagist.org/))

Signup using GitHub is easy  
Register the repository at [https://packagist.org/packages/submit](https://packagist.org/packages/submit)  
Thats it.

Now your package should be available at: [https://packagist.org/packages/mazraara/random-password](https://packagist.org/packages/mazraara/random-password)

You can version your package using with git, then users can install with tag specification and it is easy to maintain.

$ git tag 1.0.0  
$ git push origin 1.0.0

That’s it, you have now contributed to OSS ;-)