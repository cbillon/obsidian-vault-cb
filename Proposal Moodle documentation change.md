
- After any change in source code , you must increase version number

https://moodledev.io/docs/5.0/guides/upgrade

- [`version.php`](https://moodledev.io/docs/5.0/apis/commonfiles/version.php): This records the version of the plugin code. You must increase version in `version.php` after any change in the `db/` folder, any change in JavaScript code, any new auto-loaded class, any new setting and also after any change in language pack, because a new version triggers the upgrade procedure and resets all caches.

Formulation is ambigous : after any change in source  , you must increase version.

 - [`version.php`](https://moodledev.io/docs/5.0/apis/commonfiles/version.php): This records the version of the plugin code. You must increase version in `version.php` after any change in code. A new version triggers the upgrade procedure and resets all caches.
 
Moodle version


Moodle core does not follow Semantic Versioning== (SemVer), despite using a three-part dot-separated number. [[1](https://moodle.org/mod/forum/discuss.php?d=318866), [2](https://docs.moodle.org/dev/Moodle_versions)]

How Moodle Versioning Works, 

- **First two numbers**: Combine to represent the major version (e.g., `4.1` or `4.5`), which releases every six months.

- **Third number**: Acts as a minor/incremental bug-fix release within that major version (e.g., the `2` in `4.5.2`), released about every two months.

- **Incompatible changes**: Can happen in major releases without adhering to SemVer's strict breaking-change rules for major indices.

- **Exceptions**: Some individual Moodle sub-projects or developer tools (like `moodle-plugin-ci`) may choose to use SemVer for their own distinct components, but the core LMS platform does not. [[1](https://moodlehq.github.io/moodle-plugin-ci/), [2](https://moodlehq.github.io/moodle-plugin-ci/CHANGELOG.html), [3](https://gist.github.com/jashkenas/cbd2b088e20279ae2c8e), [4](https://moodle.org/mod/forum/discuss.php?d=318866), [5](https://docs.moodle.org/dev/Moodle_versions)]

If you're working on a custom extension, would you like information on how to format your plugin's **version.php** file for Moodle compatibility?

Plan du document
Semver
- Moodle version  major/minor before 5.0
-  Semver
- Introduction  series
	- breaking change appears with new serie
	-  major version  with new serie 5.X.Y
	- minor 5.1,5.2,5.3 : new feature but no breaking changes
	- patch : 5.1.1, ... corr 5.1
- last version of series is LTS



David Mudrak
https://moodle.org/mod/forum/discuss.php?d=396152
I personally tried various versioning and release naming schemes in my plugins. I have not found a nice & working solution for having the list of supported Moodle versions reflected in the plugin's version number. What finally seems to work best for me is to have my own version sequence without a numerical dependency on the Moodle version number, and mention the relation to the Moodle version in the release notes, version.php and the plugin description.

Additionally I try to adapt the ideas and spirit of the [semantic versioning](https://semver.org/). When the list of supported Moodle versions change (typically I drop support for older Moodle versions), I release a new plugin version with the higher major version number etc. That way, one major plugin version supports a range of Moodle core versions. And I do breaks only for good enough reasons (e.g. Moodle 3.8 is a good candidate due to its new options of handling the JS modules). 

in version.php
version release   X.Y.Z preferably follows semver
moodle->version [4.5 5.2]
