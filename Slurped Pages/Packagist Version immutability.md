---
link: https://packagist.org/about/version-immutability
byline: Jordi Boggiano
excerpt: The PHP Package Repository
slurped: 2026-09-03T10:03
title: Packagist The PHP Package Repository
---

## Version immutability

Once a stable (non-dev) version of a package is published on Packagist, its `source` and `dist` reference are locked. The Packagist crawler will not update them, even if the upstream tag is moved or rewritten in the git repository. Dev branches (versions matching `dev-*` or `*-dev`) continue to track their branches as they always have — this rule applies only to stable releases.

### Why?

A stable release is a contract: every downstream user who installed `vendor/pkg:1.2.3` expects to receive the exact same code as everyone else, today and a year from now. Allowing the source reference of a published version to change opens the door to supply-chain attacks (e.g. taking over a tag after release) and to maintainer footguns (e.g. a silent re-tag introducing a regression that nobody can audit). Making stable versions immutable closes both.

### What you'll see if a re-tag is detected

- A warning is shown in the update log on the package page.
- An email is sent to the package maintainers explaining the previous and attempted reference.
- An audit record is written.
- The version page displays a badge noting that the upstream re-tag was blocked and that Packagist's stored snapshot may no longer match what is currently in git.

The audit/email/badge fires at most once per attempted reference — subsequent crawls that observe the same diverged reference do not re-send.

### How to publish the change you want to make

Tag a new version with the fix. For example, if `1.2.3` had a regression, publish `1.2.4` (or even `1.2.3.1`). The new version will be crawled and made available as normal.

### Takedowns and recovery

Maintainers can soft-delete a version they own from the package page. Such versions are hidden from Composer metadata but still listed on the page (grayed out), and can be recovered by the maintainer at any time. Administrator takedowns (for malware, abuse, or legal reasons) cannot be recovered by maintainers — if you believe a takedown was applied in error, please contact Packagist support.