---
link: https://moodledev.io/general/development/tools/composer
excerpt: "Use composer to download Moodle code {/ #use-composer-to-download-moodle-code /}"
slurped: 2026-06-28T19:01
title: Composer | Moodle Developer Resources
tags:
  - composer
  - moodle
---

## Use composer to download Moodle code[​](#use-composer-to-download-moodle-code "Direct link to Use composer to download Moodle code")

`Composer.json` now includes meta information and hence composer can be used to download the Moodle code base. You can do it by creating `composer.json` file with following information

```json
{    
  "repositories": [
    {
      "type": "vcs",
      "url": "https://github.com/moodle/moodle.git"
    }
  ],
  "require": {
      "moodle/moodle": "v4.2.0"
  }
}

```

And then execute:

```
php composer.phar install
```

## How to prepare and submit composer changes[​](#how-to-prepare-and-submit-composer-changes "Direct link to How to prepare and submit composer changes")

There are a number of situations where we need to update the bundled [composer.json](https://github.com/moodle/moodle/blob/main/composer.json)] in core. When we upgrade, for a given branch, [the phpunit](https://moodle.atlassian.net/browse/MDL-71036) or [the behat-extension](https://moodle.atlassian.net/browse/MDL-70637) versions... we also have to update the [composer.lock](https://github.com/moodle/moodle/blob/main/composer.lock) file, in order to guarantee that all the tests will run in a stable, verified environment.

As far as there are a number of variables affecting how that lock file will be generated, here there are some standard steps to follow, in order to guarantee that any change to composer will be always applied in the same, standard and verified way.

1. Pre-requisite: Always use the **lower PHP version supported** in the branch the changes are happening. So, if a given branch (`MOODLE_37_STABLE`) works with PHP 7.1, 7.2 and 7.3, all the following steps will be executed using PHP 7.1 (the lower version supported).
2. Perform the required changes to the `composer.json` file. Normally.
3. Remove the composer.lock file that is going to be regenerated.  
    `rm composer.lock`
4. Remove completely the vendor directory.  
    `rm -fr vendor`
5. Clean all composer caches.  
    `php composer.phar clearcache`
6. Run `php composer.phar update`, a new `composer.lock` will be regenerated.
7. Check that the `composer.lock` file, together with other changes, does include the changes you've performed in the `composer.json` file.
8. Ideally, run both phpunit and behat tests and verify that there isn't any problem, using all the supported PHP versions.
9. Done, you can send the changes for review, integration and, if everything goes ok, will be applied upstream without problem.

## Runtime status checks[​](#runtime-status-checks "Direct link to Runtime status checks")

Moodle provides a Composer runtime status API for inspecting the state of Composer-managed dependencies.

The API can be used to:

- Check whether Composer dependencies are installed.
- Verify that installed package versions match those recorded in `composer.lock`.
- Identify missing packages.
- Identify outdated packages.

### Obtaining the service[​](#obtaining-the-service "Direct link to Obtaining the service")

The Composer service must be obtained through Moodle's dependency injection container.

```
$composer = \core\di::get(\core\composer::class);
```

### Checking Composer installation status[​](#checking-composer-installation-status "Direct link to Checking Composer installation status")

To determine whether Composer dependencies are installed:

```
if ($composer->is_installed()) {    echo 'Composer dependencies are installed.';}
```

### Retrieving overall status[​](#retrieving-overall-status "Direct link to Retrieving overall status")

The `get_status()` method returns a `\core\composer\status` object containing the overall Composer runtime status and the status of all packages defined in `composer.lock`.

```
$status = $composer->get_status();if (!$status->installed) {    echo 'Composer dependencies are not installed.';} else if (!$status->current) {    echo 'One or more Composer packages require attention.';}
```

The status object also provides convenience methods for identifying packages in a particular state:

```
$current = $status->current_packages();$missing = $status->missing_packages();$outdated = $status->outdated_packages();
```

### Checking a specific package[​](#checking-a-specific-package "Direct link to Checking a specific package")

The `get_package_status()` method returns a `\core\composer\package_status` object containing information about the package's installation state and version information.

```
$package = $composer->get_package_status('composer/installers');if (!$package->installed) {    echo 'Package is not installed.';}if (!$package->current) {    echo 'Package is not up to date.';}echo "Required version: {$package->requiredversion}";echo "Installed version: {$package->installedversion}";
```