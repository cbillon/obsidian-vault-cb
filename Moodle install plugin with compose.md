---
tags:
  - moodle
  - install
  - plugin
  - composer
---
# Installing plugins with Composer (Packagist)

If your Moodle site is Composer-managed, you can install compatible plugins directly from Packagist.

## Before you start

- Confirm your site is Composer-managed.
- Confirm your project requires `moodle/composer-installer` in production dependencies.
- Confirm the target plugin is available on Packagist and declares correct Moodle Composer metadata.
## Install a plugin


From your project root:

```
  composer require abgreeve/moodle-block_stash

```

Then complete the Moodle upgrade process if prompted.

## Update a plugin

  composer update abgreeve/moodle-block_stash


## Remove a plugin

composer remove abgreeve/moodle-block_stash


After removal, complete the Moodle uninstall or upgrade flow as needed.

## Good practice for Composer-managed sites

- Commit changes to `composer.json` and `composer.lock` together.
- Test plugin changes in a non-production environment first.
- Use one installation method per plugin to avoid drift.

Notes
This method does not apply to non-Composer Moodle sites.
If a plugin is not in Packagist or is not Composer-compatible, use the standard installation methods described elsewhere on this page.