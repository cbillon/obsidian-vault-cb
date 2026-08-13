---
link: https://medium.com/@xenmayer/composer-create-project-from-local-package-hack-29f9b5c3e9ed
byline: Vasilchenko Nikita
site: Medium
date: 2018-05-15T21:56
excerpt: "Composer create-project from local package (hack) Problem Suppose such situation: you need to create a service that will be the basis for a number of subsequent ones. The children of this service …"
twitter: https://twitter.com/@xenmayer
slurped: 2026-06-26T14:14
title: Composer create-project from local package (hack)
tags:
  - composer
  - git
---

Suppose such situation: you need to create a service that will be the basis for a number of subsequent ones. The children of this service, for example, can be created by copying the parent folder and setting dependencies within itself. And they all must live in a mono-repository.

What solutions immediately come to mind? The simplest thing is probably to create a similar alias (if you are in * nix):

Of course, there is a desire to make everything more elegant and therefore the next in a row, but the first for a reason comes to mind to use the capabilities of Composer.

### Official solutions

Solutions [offered by Composer](https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md#handling-private-packages):

1. Paid [Private Packagist](https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md#private-packagist);
2. Free [Satis](https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md#satis);
3. Own [packages.json](https://getcomposer.org/doc/05-repositories.md#packages).

The first option is good, but it will not work if you do not want to pay, and you want to do it yourself on your own.

The second option is quite rude, especially if you need to come up with a solution for a small number of your libraries. You shall agree that deploying the service for hosting one or two libraries is not an attractive decision.

The third option is very poorly described in the official documentation. And as I was told in the official composer GitHub, it is a trick.

### More about packages.json

The [documentation](https://getcomposer.org/doc/05-repositories.md#types) says that the composer uses the packages.json file to get the basic meta-information of the package:

This is a ready-made file that can be passed to the create-project command as a repository argument:

Unfortunately, without dancing with a tambourine, the above configuration and command do not create a new folder with the project. After executing the command, we get symlink to the folder of the original project with the name of the new project. We are not satisfied with this, so we are looking for answers to the documentation.

### Everything is described in the documentation

Fortunately, it is. A lot of things are described in the documentation, but, unfortunately, Composer has a lot of white spots there.

In the packages.json scheme there should be properties:

But neither the first nor the second does not work as expected, wherever they are attached. However, there are no errors for small violations of the scheme.

To establish the truth, I had to create an issue in the github repository composer.

### Decision

The documentation has a description of the environment variable [COMPOSER_MIRROR_PATH_REPOS](https://getcomposer.org/doc/03-cli.md#composer-mirror-path-repos) . It controls the behavior we need. Using the command below, we’ll add it to the system:

This line must be executed on the command line and, preferably, added to .bashrc so that it is automatically executed when opening a new terminal session. You can do this, for example, like this:

Now when you run the command to create a project, composer will create a copy of the local package, not symlink to the source folder.