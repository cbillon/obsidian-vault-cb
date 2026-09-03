---
link: https://medium.com/packagist/custom-package-definitions-3f433629861e
byline: Nils Adermann
site: Private Packagist
date: 2017-11-28T21:42
excerpt: Custom Package Definitions Using code distributed in zip files with
  Composer Would you like to use a piece of code in your project which is only
  available for download as a zip file but you’re …
twitter: https://twitter.com/@naderman
slurped: 2026-08-27T09:06
title: Custom Package Definitions
---

## Using code distributed in zip files with Composer

Would you like to use a piece of code in your project which is only available for download as a zip file but you’re managing dependencies with Composer? There are a few options to consider: the **package repository** type, creating your own **Git repository to track the zip file**’s state or the **artifact repository** type.

They all have their own pros and cons and it’s not widely understood how managing multiple versions and updating works with these, so this article aims to give you an overview of the options and which ones to pick in which situation. **Scroll to the bottom for a quick summary of pros & cons.**

## The Package Repository

Composer recognizes a special type of repository: [package](https://getcomposer.org/doc/05-repositories.md#package-2). A package repository lets you define all the information usually retrieved from packagist.org, a VCS repository and the package’s composer.json. This is the only approach which is designed to work even if the zip file does not contain a composer.json file.

"repositories": [  
    {  
        "type": "package",  
        "package": {  
            "name": "old-school/magic",  
            "version": "1.2.3",  
            "dist": {  
                "url": "https://old-school-php.com/magic.zip",  
                "type": "zip"  
            },  
            "autoload": {  
                "classmap": ["libs/"]  
            }  
        }  
    }  
],  
"require": {  
    "old-school/magic": "1.2.*"  
}

You’ll recognize the options from composer.json like _name_ or _autoload_ but you’ll also have to define _dist_ and optionally _source_ to tell Composer where to download the code from. This is usually added automatically by Packagist or in the case of a VCS repository the repository URL itself is the _source_.

If you are using [**Private Packagist**](https://packagist.com/?utm_source=blog&utm_medium=blog&utm_content=custom-package) you can use the same JSON syntax to define a package using the “Add Package -> **Custom Package**” option. This way you don’t have to add anything to your composer.json, the package will be available in your Composer repository at https://repo.packagist.com/your-org/ like all other packages!

Press enter or click to view image in full size

Editing a custom Package in Private Packagist

### Multiple package versions in a package repository

The package-type repository allows you to specify multiple packages, or multiple versions of the same package. Simply turn the package definition into an array.

This is particularly useful if you use the same repository definition in multiple projects which may need different versions of your zip file. In Private Packagist this is the way to make sure multiple versions of your custom package show up in the Composer repository you use for all your projects.

"repositories": [  
    {  
        "type": "package",  
        "package": [{  
                "name": "old-school/magic",  
                "version": "1.2.3",  
                "dist": {  
                    "url": "https://old-school-php.com/magic.zip",  
                    "type": "zip"  
                }  
            },{  
                "name": "old-school/magic",  
                "version": "1.2.4",  
                "dist": {  
                    "url": "https://old-school-php.com/magic4.zip",  
                    "type": "zip"  
                }  
            }  
        ]  
    }  
],  
"require": {  
    "old-school/magic": "1.2.*"  
}

## Tracking the Zip Contents in a Git Repository

Instead of adding the metadata for the package to your project’s composer.json you can create a Git repository and commit the contents of the unzipped archive you are trying to use. If the zip file already contains a composer.json just tag the contents with the version number (make sure it matches the number in composer.json, or better yet, delete the version from composer.json). If your zip file came without a composer.json create one yourself.

unzip magic.zip  
cd magic  
git init .  
git remote add origin [https://github.com/your-org/old-school-magic](https://github.com/your-org/old-school-magic)  
# create/edit composer.json here if needed  
git add --all  
git commit  
git tag 1.2.3  
git push --tags origin

Using this approach it’s easier to reuse the file in multiple places because you only have to add the [VCS repository](https://getcomposer.org/doc/05-repositories.md#loading-a-package-from-a-vcs-repository) URL to each project where you’re trying to use the zip file contents. With Private Packagist this Git repository will automatically show up as a package in all your projects using synchronization.

"repositories": [  
    {  
        "type": "vcs",  
        "url": "https://github.com/your-org/old-school-magic"  
    }  
],  
"require": {  
    "old-school/magic": "1.2.*"  
}

### Adding a new version of the zip file to the tracking Git repository

If a new version of the zip file becomes available you can update the contents of the repository and tag the new release. If it doesn’t come with a composer.json make sure to update it as necessary. You need to take special care in this step to ensure deleted files will actually be deleted from your Git repository.

cd magic  
rm -r *  
unzip ../magic4.zip  
# restore/edit composer.json here if needed  
git add --all  
git commit  
git tag 1.2.4  
git push origin

Now you can run composer update on any project using the code and it’ll update its dependency on old-school/magic to the latest version 1.2.4.

## The Artifact Repository

If your zip file contains a composer.json you have another alternative available to you: the [artifact repository type](https://getcomposer.org/doc/05-repositories.md#artifact) (Of course you could also repackage your zip file to include a composer.json). The artifact repository let’s you specify a local path to a directory containing any number of zip files. Composer will load metadata from composer.json files in all of the zip files. So you can easily store many different packages and versions of these packages in a directory.

"repositories": [  
    {  
        "type": "artifact",  
        "url": "/srv/artifacts/composer/"  
    }  
],  
"require": {  
    "old-school/magic": "1.2.*"  
}

If you wish to share this project or work on it together however you need to make sure everyone can access the zip files. So you’ll have to either mount a shared filesystem for these artifacts, come up with a distribution mechanism yourself, or actually commit all the zip files into a directory in your project so the artifact repository points to a relative path in your project.

## Summary

### **Repository type: artifact**

**Pro  
-** Only one repository entry in composer.json for many packages  
**Con  
-** Requires composer.json in zip files  
- Zip files need to be on local path on every machine or you have to commit all zip files into project repository which is not really an option if used in multiple projects

### **Repository type: package**

**Pro  
-** Zip file can be used without modification  
**Con  
-** Metadata is copied into every project’s composer.json  
- New version requires change in every composer.json referencing the package  
- Does not use data in composer.json if zip file already contains one

### Custom Package on Private Packagist

**Pro**  
- Configuration like package repository type- Use zip file without modification  
- No copying of metadata, no composer.json changes needed  
- Single location to update for a new version — even with many projects  
**Con  
**- Does not use data in composer.json if zip file already contains one

### **Git repository to track zip contents**

**Pro  
-** Git tools (e.g. diff, log) can be used to view history of the package  
- Only short VCS repository entry in every composer.json using the package (Not needed if you use Private Packagist)  
**Con  
-** Complex process to update the Git repo every time a new version is released  
- Potentially lots of Git repositories to manage if you use many zip files

### Head over to [https://packagist.com](https://packagist.com/?utm_source=blog&utm_medium=trial&utm_content=custom-package) to try Private Packagist for free!