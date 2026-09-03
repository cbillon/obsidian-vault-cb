---
tags:
  - cbm
  - moodle
  - composer
---

Please see:

[https://docs.moodle.org/503/en/Installing_plugins](https://docs.moodle.org/503/en/Installing_plugins)

[https://moodledev.io/general/development/tools/composer](https://moodledev.io/general/development/tools/composer)

As well as:

[https://docs.moodle.org/502/en/Composer_vendor_directory_not_found](https://docs.moodle.org/502/en/Composer_vendor_directory_not_found)


[IDEA 330][https://moodle.atlassian.net/browse/IDEA-330]

- We currently include all libraries we use in Moodle in the core code which leads to:
    
    - A larger install file for users
        
    - Us having to manually maintain versions of the code rather than just pointing to the next version that works with our software
        

- Install Third Party library dependencies using Composer
    
- Implement Plugin Installation using Composer

[Add tooling](https://moodle.atlassian.net/browse/MDL-86889

## Add tooling to help install Composer dependencies

In 5.1 we started to require Composer be run to install various dependencies.

There have been some support requests in the forum asking for help with this. It _may_ be possible to help people by downloading the `composer.phar` tool, and possibly running it too.

If these things are possible, then the chances are that the Moodle hosting directory is already web writable, but we should still consider any security concerns here.


[Video wisecat]([ttps://www.youtube.com/watch?foo=bar&v=izVUPboucNk](https://www.youtube.com/watch?foo=bar&v=izVUPboucNk))

if you can not configure router 
config.php
$CFG->routerconfigured = false;

# Gitlab as registry


To upload a new package to a GitLab Composer registry, you ==tag a Git commit in a project belonging to a GitLab group, and GitLab automatically indexes it via its built-in Composer repository==. Unlike other package managers, GitLab's Composer registry works directly with your Git repository and does not require a manual file upload or an explicit `composer publish` command. [[1](https://forum.gitlab.com/t/better-understanding-composer-registry/54243), [2](https://www.youtube.com/watch?v=e_HqOOWuRoI&t=46), [3](https://drupalviking.com/blog/2026-05/publishing-custom-drupal-modules-and-recipes-gitlabs-composer-registry), [4](https://stackoverflow.com/questions/76530637/gitlab-how-to-include-build-artifacts-when-adding-a-package-via-ci-pipeline-to)]

1. Create and Configure `composer.json`

Create your PHP project inside a **GitLab group** (GitLab exposes Composer registries at the group level, not personal namespaces). [[1](https://www.philipp-doblhofer.at/en/blog/custom-php-composer-package-on-gitlab-with-phpunit-tests/), [2](https://drupalviking.com/blog/2026-05/publishing-custom-drupal-modules-and-recipes-gitlabs-composer-registry)]

- Run `composer init` in your project.

- Ensure the `"name"` property in your `composer.json` uses your group or subgroup name as the vendor prefix (e.g., `<group-name>/<package-name>`). [[1](https://docs.gitlab.com/user/packages/workflows/build_packages/), [2](https://www.philipp-doblhofer.at/en/blog/custom-php-composer-package-on-gitlab-with-phpunit-tests/), [3](https://www.youtube.com/watch?v=e_HqOOWuRoI&t=46)]

Example `composer.json`:

json

```
{
    "name": "my-group/my-composer-package",
    "description": "A custom PHP package",
    "type": "library",
    "license": "MIT",
    "require": {
        "php": ">=8.1"
    },
    "autoload": {
        "psr-4": {
            "MyGroup\\MyPackage\\": "src/"
        }
    }
}
```

Utilisez le code avec précaution.

2. Commit and Tag the Release

GitLab's Composer registry links packages directly to **Git tags** representing semantic versions. [[1](https://forum.gitlab.com/t/better-understanding-composer-registry/54243), [2](https://www.youtube.com/watch?v=e_HqOOWuRoI&t=46)]

- Add, commit, and push your code to your repository.

- Create an annotated Git tag matching a valid version (e.g., `v1.0.0`) and push it to GitLab: [[1](https://turndevopseasier.com/publish-a-composer-package-by-using-ci-cd/), [2](https://www.youtube.com/watch?v=e_HqOOWuRoI&t=46)]

bash

```
git add .
git commit -m "Initial release of composer package"
git push origin main

git tag v1.0.0
git push origin v1.0.0
```

Utilisez le code avec précaution.

3. Verify the Package in GitLab

- Once pushed, GitLab automatically detects the tag and registers the package metadata.

- Navigate to your **Group** or **Project** in GitLab, then go to **Deploy** > **Package Registry** to see your newly indexed Composer package. [[1](https://www.youtube.com/watch?v=ZUi4HCPHQno&t=3), [2](https://forum.gitlab.com/t/better-understanding-composer-registry/54243), [3](https://turndevopseasier.com/publish-a-composer-package-by-using-ci-cd/)]

If you want, tell me:

- Do you need to **authenticate Composer** to download this private package in another project?
- Are you trying to **automate this process** using GitLab CI/CD pipelines?

I can provide the exact configuration commands you need.
