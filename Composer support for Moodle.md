---
link: https://moodle.org/mod/page/view.php?id=8917
tags:
  - moodle
  - composer
---
# Composer support for Moodle

Composer is the standard PHP dependency and package manager. Moodle predates Composer and historically bundled third-party libraries manually. This initiative delivers two related capabilities: Composer-based installation of Moodle’s PHP dependencies (already mandatory, with full enforcement arriving soon) and an optional path to install Moodle itself using Composer alongside the existing installer.

## History

For many years Moodle has relied on manually installed libraries without dedicated dependency management. Modern PHP packages now come with larger dependency graphs and sometimes call Composer APIs directly, making manual inclusion fragile and costly. Early experiments showed we can preserve Moodle’s layout while letting Composer resolve and place dependencies with only minor changes to core.

Community members have repeatedly asked for a way to install [Moodle plugins](https://moodle.org/plugins "Auto-link") using Composer. This becomes possible with these changes, allowing plugins to be pulled, versioned, and kept in sync automatically instead of downloading and managing them by hand.

## Rationale

OpenTelemetry integration depends on packages that require the Composer API, and removing that requirement would be high-effort and risky. Composer support was already planned and has been brought forward to unblock observability work, future dependency needs, and the planned REST API OAuth2 improvements. The goal is to enable Composer-based installation of dependencies and Moodle while maintaining a smooth transition path.

Adding Composer support for plugins also answers long-standing community requests. It lets administrators and developers use a single workflow to install and upgrade plugins, with Composer handling version management and dependency resolution automatically. That reduces manual effort and lowers the risk of version conflicts between plugins and core.

## What’s Changing

- Core (minimal):
    - Light `composer.json` adjustment.
    - Test infrastructure that runs against either Composer-installed or classic layouts.
    - Autoloader tweaks to prevent conflicts between Composer-managed dependencies and Moodle-bundled libraries.
- New Composer plugins and packages:
    - [moodle/composer-installer](https://github.com/moodlehq/composer-installer): places Moodle plugins into their correct locations.
    - [moodle/moodle-composer-scaffold](https://github.com/moodlehq/moodle-composer-scaffold): scaffolds Moodle for Composer installs, generating shims for configuration and autoloading plus helper tooling.
    - [moodle/moodle-testing](https://github.com/moodlehq/moodle-testing): shared development and testing dependencies for Composer-installed sites.
    - `moodle/seed` (not yet released): template to create a new Moodle site with `composer create-project`, wiring scaffold and installer together.
- Scope: most changes live in the new Composer plugins and packages rather than Moodle core.

## Timeline and Adoption

- Now through Moodle 5.2 and 5.3: Composer is already mandatory for installing PHP dependencies; enforcement is intentionally gradual during this period. Installing Moodle itself via Composer remains optional.
- Moodle 6.0: Full enforcement of Composer for dependency installation. We have not yet taken a stance on if or when Composer will be required to install Moodle itself.

## Next Steps

1. Finish remaining polish for 5.2 delivery.
2. Validate Composer-based and classic installation and testing flows to ensure parity.
3. Prepare guidance for administrators and plugin developers on the new Composer tooling.
4. Define and communicate future enforcement milestones once they are decided.

Modifié le: mardi 17 février 2026, 03:13