---
link: https://dudi.dev/create-composer-php-package
excerpt: Learn how to create a composer php package, host it on github and
  import it into a project using composer.
slurped: 2026-08-27T19:52
title: Learn How to create and publish a composer php package | dudi.dev
---

Composer is a great tool to import php packages hosted on [packagist.org](https://packagist.org/). All the packages hosted on packagist are public packages and anyone can use them with in their package.

In this article, I am going to show you

- how to create a composer package
- Host it on github
- Submit to packagist.org
- Use the package in an existing project using composer.

First, We are going to create a simple hello world composer php package.

### 1. Create composer package

create a directory called `hello-world-package`.

```
1mkdir hello-world-package && cd hello-world-package
```

Every composer package must have a `composer.json` file in the root directory. `composer.json` file holds information about a package like its name, dependencies, autoload classes, etc.

Create a new file called `composer.json` in the project root directory.

```
1sudo nano composer.json
```

Copy and paste the following code.

```
 1{ 2    "name": "your-github-username/hello-world-package", 3    "description": "A hello world composer private package.", 4    "autoload": { 5        "psr-4": { 6            "YourGithubUserName\\HelloWorldPackage\\": "src/" 7        } 8    }, 9    "minimum-stability": "dev",10    "prefer-stable": true11}
```

Replace `your-github-username` with your github profile name and save the file.

> Full `composer.json` file schema can be found [here](https://getcomposer.org/doc/04-schema.md)

Now that we have our `composer.json` file all setup, We can start writing actual code.

Create a new directory called `src` in the project root.

```
1mkdir src
```

Create a file called `HelloWorld.php` inside the `src` directory.

```
1sudo nano HelloWorld.php
```

Copy and paste the below php code into the file and save the file.

```
 1<?php 2  3namespace HelloWorldPackage; 4  5class HelloWorld 6{ 7    public static function sayHello() 8    { 9        echo "Hello World";10    }11}
```

Now our package is complete and ready to be distributed to the world. We first need to host this package on github before we start using it.

### 2. Setup Github Repository

- create an empty repository called `hello-world-package` in your github account.
- Mark the repository as `public`.

> In order to host packages on packagist.org and use them with composer, All the packages in github must be marked as public. I wrote a separate article on [how to use composer with private repositories](https://dudi.dev/composer-github-private-repositories).

### 3. Sync package with github

Now that we have our package and github repository ready, We can push our package code to github. - Navigate to your project root directory in a terminal - Run `git init` to initialize your package as a github repository. - If you are using https for git, Run the following command

```
1git remote add origin https://github.com/your-github-username/hello-world-package.git
```

- If you are using ssh for git, Run the following command

```
1git remote add origin [email protected]:your-github-username/hello-world-package.git
```

- Replace `your-github-username` with your github username before running the above command.
- Commit and push the package code to your github repository by running the following commands

```
1git add .2git commit -m "my first composer package"3git push --set-upstream origin master
```

### 4. Release package

Our package is finally on github. But before we can make use of our package, We need to release our version on github and provide a version label.

- go to `https://github.com/your-github-username/hello-world-package`
- click on releases tab.
- click on `create a new release` button.
- Give a version number, Title, and short info about the release(For version number, It is recommended to follow [semantic versioning](https://semver.org/)).
- Click on `publish release` button to release the package.

### 5. Host package on packagist.org

- Go to packagist.org and create an account if you do not have yet.
- Once logged in, Click on [submit](https://packagist.org/packages/submit) in the navigation menu
- Enter your github repository url
- If your composer.json file is valid, It will display the package name. Confirm and submit your package.

After successfully submitting your package to packagist, You can access it from the url `https://packagist.org/packages/<your-username>/<package-name>`

### 6. Import the package in a project

Now that our package is all setup on packagist.org, We can start importing and using it in our existing projects.

Import your package by running

```
1composer require <your-username>/<your-package-name>
```

Add the composer autoload file at the beginning of script(If you are not already doing it)

```
1<?php2 3require 'path-to-vendor-directory/autoload.php';
```

You can start calling the method in the package as below

```
1<?php2 3require 'path-to-vendor-directory/autoload.php';4 5 6\YourGithubUserName\HelloWorldPackage\HelloWorld::sayHello();
```

### Tips for creating composer packages

- Composer package names can only contain letters, numbers and hypehns.
- Composer package names cannot contain uppercase characters.
- It is recommended to keep composer package names short and memorable.
- Add proper keywords, so your package can be found when the user is searching for it on packagist.org
- It is recommended to follow [semantic versioning](https://semver.org/) while versioning your package.