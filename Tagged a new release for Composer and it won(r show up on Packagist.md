---
tags:
  - composer
  - packagist
link: https://medium.com/packagist/tagged-a-new-release-for-composer-and-it-wont-show-up-on-packagist-org-or-on-private-packagist-efaf21c212ff
---
# Tagged a new release for Composer and it won’t show up on Packagist?

This is probably the most common support question we see both on [Packagist.org](https://packagist.org/) and on [Private Packagist](https://packagist.com/?utm_source=blog&utm_medium=blog&utm_content=versions): A Composer user tags a new library version but they cannot install it because it won’t show up on Packagist.

**Packagist will not list versions and branches containing invalid composer.json files** because Composer users will not be able to install these versions. If this happened to a package you maintain, **tag a new release** with a fixed composer.json. Don’t edit or replace the existing git tag because many tools and services have caches that expect tags to never change.
### Run “`composer validate”` in your project

The [validate command](https://getcomposer.org/doc/03-cli.md#validate) will analyze your composer.json file and list errors as well as warnings and recommendations for publishing your package. You’ll be able to spot JSON syntax errors, but also issues with the contents of your package definition. The output of this command could look like the following example:

**~/projects/private-test2$** composer.phar validate                                                                                                                              
./composer.json is valid for simple usage with composer but has                                                                                                                                                       
strict errors that make it unable to be published as a package:                                                                                                                                                       
See https://getcomposer.org/doc/04-schema.md for details on the schema                                                                                                                                                
description : The property description is required                                                                                                                                                                    
Name "seld/PRivate-test2" does not match the best practice (e.g. lower-cased/with-dashes). We suggest using "seld/p-rivate-test2" instead. As such you will not be able to submit it to Packagist.                    
The version field is present, it is recommended to leave it out if the package is published on Packagist.

### Check the Update Log

Private Packagist has a link to a package’s update log on the view package page.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:1138/1*Vz2jTTLApNbGUHgis5j6Ww.png)

Follow the View Log link to see why a version may be missing

The update log will highlight any errors or warnings encountered while updating the package information.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:1138/1*Vy1Tqtkjs2cicLrayxI1xw.png)

## Delete the version attribute from composer.json

Both of the previous examples show you the **most common reason for missing releases: A package containing a hardcoded version number in the JSON file** which does not match the git tag.

Usually the version attribute gets added to a composer.json early on in a project and everything works fine. But sooner or later someone tags a new release and forgets to increment the version number in composer.json. When Composer now looks at the Git repository it cannot tell whether the tag (1.3.0 in my example) or the version attribute in composer.json (1.2.0 in my example) is the right number. Either option may lead to problems down the road so Composer entirely ignores the broken tag.

The fix is quite simple, **delete the version attribute from your composer.json**. Composer has great integration with version control systems like Git, Mercurial and Subversion and there is **no need to manually track version numbers** in a text file for Composer at all. The field really only exists for special situations where a version control system is not in use.