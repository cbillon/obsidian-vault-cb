---
tags:
  - moodle
  - gitpod
---

## Quick start with Gitpod

[](https://github.com/moodlehq/moodle-docker/#quick-start-with-gitpod)

Gitpod is a free, cloud-based, development environment providing VS Code and a suitable development environment right in your browser.

When launching a workspace in Gitpod, it will automatically:

- Clone the Moodle repo into the `<workspace>/moodle` folder.
- Initialise the Moodle database.
- Start the Moodle webserver.

[![Open in Gitpod](https://camo.githubusercontent.com/b04f5659467d23b5109ba935a40c00decd264eea25c22d50a118021349eea94f/68747470733a2f2f676974706f642e696f2f627574746f6e2f6f70656e2d696e2d676974706f642e737667)](https://gitpod.io/#https://github.com/moodlehq/moodle-docker)

> **IMPORTANT**: Gitpod is an alternative to local development and completely optional. We recommend setting up a local development environment if you plan to contribute regularly.

The Moodle Gitpod template supports the following environment variables:

- `MOODLE_REPOSITORY`. The Moodle repository to be cloned. The value should be URL encoded. If left undefined, the default repository `https://github.com/moodle/moodle.git` is used.
- `MOODLE_BRANCH`. The Moodle branch to be cloned. If left undefined, the default branch `main` is employed.
- `DATAFILE`. When specified, this feature URL initializes the Moodle site with essential data. The value should be URL encoded. The content of this file adheres to the [Behat generators format](https://moodledev.io/general/development/tools/generator#create-a-testing-scenario-using-behat-generators) for creating testing scenarios.
- `INSTALLADMINER`. Add this variable, set to any value, to install [adminer](https://www.adminer.org/).
- `CLONEALL`. If not set, a shallow clone is created, truncating the history to reduce the clone size. Set to `1` for a full clone.

For a practical demonstration, launch a Gitpod workspace with the 'main' branch patch for [MDL-79912](https://tracker.moodle.org/browse/MDL-79912). Simply open the following URL in your web browser (note that MOODLE_REPOSITORY should be URL encoded). The password for the **admin** user is **test**:

```
https://gitpod.io/#MOODLE_REPOSITORY=https%3A%2F%2Fgithub.com%2Fsarjona%2Fmoodle.git,MOODLE_BRANCH=MDL-79912-main/https://github.com/moodlehq/moodle-docker
```

To optimize your browsing experience, consider integrating the [Tampermonkey extension](https://www.tampermonkey.net/) into your preferred web browser for added benefits. Afterward, install the Gitpod script, which can be accessed via the following URL: [Gitpod script](https://gist.githubusercontent.com/sarjona/9fc728eb2d2b41a783ea03afd6a6161e/raw/gitpod.js). This script efficiently incorporates a button adjacent to each branch within the Moodle tracker, facilitating the effortless initiation of a Gitpod workspace tailored to the corresponding patch for the issue you're currently viewing.

## Companion docker images

[](https://github.com/moodlehq/moodle-docker/#companion-docker-images)

The following Moodle customised docker images are close companions of this project:

- [moodle-php-apache](https://github.com/moodlehq/moodle-php-apache): Apache/PHP Environment preconfigured for all Moodle environments
- [moodle-db-mssql](https://github.com/moodlehq/moodle-db-mssql): Microsoft SQL Server for Linux configured for Moodle
- [moodle-db-oracle](https://github.com/moodlehq/moodle-db-oracle): Oracle XE configured for Moodle