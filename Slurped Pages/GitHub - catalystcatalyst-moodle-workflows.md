---
link: https://github.com/catalyst/catalyst-moodle-workflows
byline: catalyst
site: GitHub
excerpt: Contribute to catalyst/catalyst-moodle-workflows development by creating an account on GitHub.
twitter: https://twitter.com/@github
slurped: 2026-02-08T17:39
title: GitHub - catalyst/catalyst-moodle-workflows
tags:
  - github
  - ci
  - catalyst
---

## Reusable Workflows

[](https://github.com/catalyst/catalyst-moodle-workflows#reusable-workflows)

Thanks to the new GitHub Actions feature called "Reusable Workflows" we can now reference an existing workflow with a single line of configuration rather than copying and pasting from one workflow to another.

This massively reduces the amount of boilerplate setup in each plugin to the bare minimum and also means that we can maintain and revise our definition of best practice in one place and have all the Moodle plugins inherit improvements in lock step. Mostly these shared actions in turn wrap the Moodle plugin CI scripts:

[https://moodlehq.github.io/moodle-plugin-ci/](https://moodlehq.github.io/moodle-plugin-ci/)

## 🚀 Quick start

[](https://github.com/catalyst/catalyst-moodle-workflows#rocket-quick-start)

For most plugins, you'll want to go through this checklist:

1. Ensure branch section in `README.md` describes correct versions supported
2. Supported range in `version.php` has been configured correctly
3. Workflow file has been added `.github/workflows/ci.yml` and configured
4. CI badge has been properly added in the `README.md`

Some examples of usage: [tool_mfa](https://github.com/catalyst/moodle-tool_mfa#branches), [tool_dataflows](https://github.com/catalyst/moodle-tool_dataflows/#branches)

### Configure support range

[](https://github.com/catalyst/catalyst-moodle-workflows#configure-support-range)

Each plugin should have a measurable range of versions supported. It's recommended and ensures a predictable test range.

1. Open `version.php` in your plugin repository
2. Set the `$plugin->supported`* as the range the plugin supports, which then determines the versions the workflow tests for.

# version.php
$plugin->supported = [35, 402];

This example will run a matrix of tests from `MOODLE_35_STABLE` to `MOODLE_402_STABLE` - [see full test matrix here](https://github.com/catalyst/catalyst-moodle-workflows/blob/main/.github/actions/matrix/matrix_includes.yml).

Any number _greater_ than the [latest available stable branch](https://github.com/moodle/moodle/branches/active) will automatically include the [main branch](https://github.com/moodle/moodle/tree/main) for testing

* For more info on the `$plugin->supported` field, please see [https://docs.moodle.org/dev/version.php](https://docs.moodle.org/dev/version.php)

### Add the workflow

[](https://github.com/catalyst/catalyst-moodle-workflows#add-the-workflow)

For most cases, this following demonstrates how your workflow file would typically look.

Create `.github/workflows/ci.yml` with the below template in your plugin repository.

# .github/workflows/ci.yml
name: ci

on: [push, pull_request]

jobs:
  ci:
    uses: catalyst/catalyst-moodle-workflows/.github/workflows/ci.yml@main
    # Required if you plan to publish (uncomment the below)
    # secrets:
      # moodle_org_token: ${{ secrets.MOODLE_ORG_TOKEN }}
    with:
      # Any further options in this section

For how to set up the secret, please see the [_How does this automate releases_](https://github.com/catalyst/catalyst-moodle-workflows#how-does-this-automate-releases) section below.

You can add extra options to disable checks that you might not want, or to add additional dependencies under the `with` field. For example:

    with:
      disable_behat: true
      disable_grunt: true
      extra_plugin_runners: 'moodle-plugin-ci add-plugin catalyst/moodle-local_aws'

#### `with` options

[](https://github.com/catalyst/catalyst-moodle-workflows#with-options)

Below lists the available inputs which are _all optional_:

|Inputs|Description|
|---|---|
|codechecker_max_warnings|To fail on warnings, set this to 0|
|extra_plugin_runners|Command to install more dependencies|
|disable_behat|Set `true` to disable behat tests.|
|disable_phpdoc|Set `true` to disable phpdoc tests.|
|disable_phpcs|Set `true` to disable phpcs (codechecker) tests.|
|disable_phplint|Set `true` to disable phplint tests.|
|disable_phpunit|Set `true` to disable phpunit tests.|
|disable_grunt|Set `true` to disable grunt.|
|disable_mustache|Set `true` to disable mustache.|
|disable_master|If `true`, this will skip testing against moodle/master branch|
|disable_release|If `true`, this will skip the release job|
|disable_phpcpd|If `true`, this will skip phpcpd checks|
|disable_ci_validate|If `true`, this will skip moodle-plugin-ci validate checks|
|enable_phpmd|If `true`, to enable phpmd|
|release_branches|Name of the non-standardly named branch which should run the release job|
|moodle_branches|Specify the MOODLE_XX_STABLE branch you specifically want to test against. This is _not_ recommended, and instead you should configuring a supported range.|
|min_php|The minimum php version to test. Set this to support the minimum php version supported by the plugin. Defaults to '7.4', however more recent Moodle branches only test higher versions.|
|ignore_paths|Specify custom paths for CI to ignore. Third party libraries are ignored by default.|

### Running with dependencies

[](https://github.com/catalyst/catalyst-moodle-workflows#running-with-dependencies)

If you'd require to run your workflow against specific versions of a plugin you depend on, then you can specify a branch you'd like to run each job against. Like following:

jobs:
  moodle41:
    uses: catalyst/catalyst-moodle-workflows/.github/workflows/ci.yml@main
    with:
      disable_phpunit: true
      moodle_branches: MOODLE_401_STABLE
      extra_plugin_runners:
        moodle-plugin-ci add-plugin danmarsden/moodle-mod_attendance --branch MOODLE_401_STABLE

  moodle42:
    uses: catalyst/catalyst-moodle-workflows/.github/workflows/ci.yml@main
    with:
      disable_phpunit: true
      moodle_branches: MOODLE_402_STABLE
      extra_plugin_runners:
        moodle-plugin-ci add-plugin danmarsden/moodle-mod_attendance --branch MOODLE_402_STABLE

An example can be found here - [https://github.com/danmarsden/moodle-block_attendance/blob/main/.github/workflows/ci.yml](https://github.com/danmarsden/moodle-block_attendance/blob/main/.github/workflows/ci.yml)

### Add CI badge

[](https://github.com/catalyst/catalyst-moodle-workflows#add-ci-badge)

With badges, we will be able to see at a glance from the plugin's `README.md` whether or not the plugin is in a good state for usage.

```
[![ci](https://github.com/[user]/[plugin]/actions/workflows/ci.yml/badge.svg?branch=[branch])](https://github.com/[user]/[plugin]/actions/workflows/ci.yml?branch=[branch])
```

Please update `[USER]`, `[PLUGIN]` and `[BRANCH]` in the example above. This goes under the plugin title. Here is an example:

```
[![ci](https://github.com/catalyst/moodle-tool_excimer/actions/workflows/ci.yml/badge.svg?branch=MOODLE_35_STABLE)](https://github.com/catalyst/moodle-tool_excimer/actions/workflows/ci.yml?branch=MOODLE_35_STABLE)
</a>
```

which renders as:

[![ci](https://github.com/catalyst/moodle-tool_excimer/actions/workflows/ci.yml/badge.svg?branch=MOODLE_35_STABLE)](https://github.com/catalyst/moodle-tool_excimer/actions/workflows/ci.yml?branch=MOODLE_35_STABLE)

## How does this automate tests?

[](https://github.com/catalyst/catalyst-moodle-workflows#how-does-this-automate-tests)

When you call the reusable ci, it will:

1. a check to see what versions of moodle should run, based on the `version.php` file included in the plugin repository.
2. This will then build out the combination of tests to run, performing the tests based on the **MOODLE_XX_STABLE** version affected and will handle any version specific caveats and run a more optimally configured test suite for you.

To view or modify the full matrix, please see it here: [.github/actions/matrix/matrix_includes.yml](https://github.com/catalyst/catalyst-moodle-workflows/blob/main/.github/actions/matrix/matrix_includes.yml)

## How does this automate releases?

[](https://github.com/catalyst/catalyst-moodle-workflows#how-does-this-automate-releases)

Whenever a change is made to version.php, the workflow is triggered on a release branch (e.g. **main** / **MOODLE_XX_STABLE**), and tests pass, will it attempt to run the plugin/release action `.github/plugin/release/action.yml`. Doing so will automatically publish a release to the Moodle plugin directory at [https://moodle.org/plugins](https://moodle.org/plugins).

In the prepare step of the CI, it will have resolved the component name for you so you don't need to enter one manually, and the `MOODLE_ORG_TOKEN` secret should be set otherwise the plugin won't be published.

To configure the secret:

- Please check `MOODLE_ORG_TOKEN` is available in your plugin's **Settings > Secrets** section. If not, please create using below steps:
    - Log in to the Moodle Plugins directory at [https://moodle.org/plugins/](https://moodle.org/plugins/)
    - Locate the **Navigation block > Plugins > API access**.
        - Use that page to generate your personal token for the `plugins_maintenance` service.
    - Go back to your plugin repository at Github. Locate your plugin's **Settings > Secrets** section. Use the 'New repository secret' button to define a new repository secret to hold your access token. Use name `MOODLE_ORG_TOKEN` and set the value to the one you generated in previous step.
- For the latest branch/stable releases in plugin directory we **MUST** bump plugin version (e.g. by date).
- For the **older stables** with closed groups, the version **MUST** be a **micro bump**.

## Common concerns / issues

[](https://github.com/catalyst/catalyst-moodle-workflows#common-concerns--issues)

### Core patches

[](https://github.com/catalyst/catalyst-moodle-workflows#core-patches)

If you need to apply core patches to the moodle code to allow the plugin to work on older versions missing API support, this can be achieved by outputting an appliable diff to a file, in the `patch` directory in the top level of the repo. The version for a particular branch should be named `MOODLE_XX_STABLE.diff` in line with the naming of the branch. To generate a diff, these patches can be manually applied to the target branch, and a diff from the remote generated by doing `git format-patch MOODLE_XX_STABLE --stdout > my/plugin/path/patch/MOODLE_XX_STABLE.diff`.

### amd / grunt bundling issues

[](https://github.com/catalyst/catalyst-moodle-workflows#amd--grunt-bundling-issues)

Depending on your supported range, you might encounter an issue which outputs something along the lines of `File is stale and needs to be rebuilt`. The brief reason for this, is that along the way, Moodle has updated the way it bundles JS/CSS files, which results in different outputs across Moodle versions.

To fix this, you'll need to rebundle the relevant files, on the highest supported version of Moodle for your plugin's support range. For example, if the plugin supports up to **Moodle 4.0**, you'll need to bundle the changes, while on the `MOODLE_400_STABLE` branch of Moodle, and then commit those changes.

**NOTE:** This may involve having a clean copy of Moodle and installing the plugin code to run the necessary commands to rebuild the _stale_ files.

Grunt docs: [https://docs.moodle.org/dev/Grunt#Running_grunt](https://docs.moodle.org/dev/Grunt#Running_grunt)

### Testing changes

[](https://github.com/catalyst/catalyst-moodle-workflows#testing-changes)

Changes can be tested by using another plugin repository that uses these workflows.

Simply change the repository/branch it pulls from:

uses: catalyst/catalyst-moodle-workflows/.github/workflows/ci.yml@my-custom-branch

and specify the same branch as a `with` parameter:

with:
  internal_workflow_branch: 'my-custom-branch'