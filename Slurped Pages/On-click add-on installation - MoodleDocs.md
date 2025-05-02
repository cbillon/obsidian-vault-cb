---
link: https://docs.moodle.org/dev/On-click_add-on_installation
excerpt: Moodle 2.5
slurped: 2025-03-16T09:00
title: On-click add-on installation - MoodleDocs
tags:
  - moodle
  - plugin
  - install
---

|**On-click add-on installation**|   |
|---|---|
|Project state|Implemented|
|Tracker issue|[MDLSITE-2107](https://tracker.moodle.org/browse/MDLSITE-2107)|
|Discussion|N/A|
|Assignee|[David Mudrak](https://docs.moodle.org/dev/User:David_Mudrak), Aparup Banerjee|

Moodle 2.5

  
This is a follow-up project on [Automatic updates deployment](https://docs.moodle.org/dev/Automatic_updates_deployment "Automatic updates deployment") aiming to implement the Episode III of the deployment saga.

## Use cases

[![Sue the admin](https://docs.moodle.org/dev/images_dev/a/ac/sue.png)](https://docs.moodle.org/dev/File:sue.png "Sue the admin")

Meet Sue, a Moodle administrator. She runs her Moodle at a cheap web hosting company where she gets an FTP access only (and some basic web tool to administer one SQL database). No shell access, no way to use Git etc. She never modifies source codes manually. And she likes the way how new apps can be installed into her Android smart phone by browsing the Google Play site.

### Use case 1

- Sue logs in her Moodle site and goes to the Plugins overview page. She clicks the "Get more add-ons!" link at the top of the page.
- She is redirected to the [Plugins directory](https://moodle.org/plugins). The Moodle version there is automatically set to the version of her Moodle site she just came from.
- She browses the list of plugins in the directory and finally finds an add-on she want to install.
- There is a big nice icon "Install" at the page so she clicks it.
- A popup-like overlay window is displayed with the list of Sue's sites. She has one site only so she clicks the "Install to this site" link next to it.
- She is redirected back to her site, to a page requiring her confirmation for the plugin installation. She confirms that she really wants to install that add-on by click at "Install" button.
- The Plugins check page appears, showing that there is the plugin ready to be installed (as if she just unzipped to her site).

## Analysis

There are two basic scenarios: The user (admin) starts at their site and clicks the "Get more add-ons!" link, or the user visits moodle.org/plugins directly. In both scenarios, the user may or may not have an account at moodle.org and may or may not be logged in there. In the second scenario the user may and may not be logged in at their site. Their site may or may not be Moodle 2.5 (and higher).

[![addon-install-uml-1.png](https://docs.moodle.org/dev/images_dev/1/13/addon-install-uml-1.png)](https://docs.moodle.org/dev/File:addon-install-uml-1.png)

(1) Admin logs in their Moodle site and navigates to the "Plugins overview" page.

(2) Admin clicks the "Get more add-ons!" link at the top of the page.

(3) The link leads to a script like [https://moodle.org/plugins/get.php?site=](https://moodle.org/plugins/get.php?site=)... where the parameter holds the encoded information about the site name, URL and branch.

(4) Plugins directory decodes the site information. Then ...

(4.1) If the user is not logged-in at moodle.org, they are asked if they want to log in (with a help icon to explain why login is useful).

(4.1.1) If they decide to continue in anonymous mode, the site information is stored in the MUC session cache, the Moodle version is set to the branch provided by the link (3) and they are redirected to the main moodle.org/plugins page.

(4.1.2) If they decide to log in, they are redirected to the login form, having the link (3) set in their $SESSION->wantsurl - thence jumping to (4.2).

(4.2) If the user is logged-in at moodle.org, the passed site information is compared with the list of their "own" sites recorded in their user preferences. If there is a change in the name name or the branch, or such URL is not recorded yet, they are asked to update it/add it to the list of their Moodle sites.

(5) The user navigates to the plugin they want to install. The "Install" button is displayed at the page.

(5.1) If they are logged-in, clicking the button opens a popup-like overlay window with the list of all recorded user's sites with a possibility to add a new site or delete an existing site. The "Install to this site" link is displayed next to each recorded site if its branch >= 2.5.

(5.2) If they are not logged in, clicking the button opens a popup-like overlay window with an input field for the site URL and the submit button labelled "Install to this site". If the MUC session cache contains the information about the site they came from, that site's URL is pre-filled in the field.

(6) Clicking the "Install to this site" button takes them to that site's /index.php. Additional information about the plugin version to be installed is passed via encoded HTTP GET parameter 'installaddon'.

(7) If optional_param('installaddon') is passed to /index.php, the user must log in as the admin (if they are not yet). A screen requiring their confirmation to install the given add-on is displayed. The sesskey is checked during the confirmation. For sites running 2.4.x and lower, nothing will happen.

(8) The site uses cURL call via HTTPS to fetch meta-information about the plugin to be installed, most notably the URL and the MD5 hash of the ZIP package to be downloaded. This information is to be provided by [https://download.moodle.org/api/1.2/describe.php?plugin=frankenstyle_name&version=the_plugin_version](https://download.moodle.org/api/1.2/describe.php?plugin=frankenstyle_name&version=the_plugin_version) (which in turn pulls data from local_plugins WS every now and then and when needed).

(9) The information about the given plugin and it's version is loaded from the database and the JSON response is prepared.

(10) The JSON response is sent back to the site.

(11) The parameters for the mdeploy.php script are prepared.

(12) The mdeploy.php utility is executed.

(13) The cURL GET call to download the ZIP.

(14) ZIP package is downloaded.

(15) ZIP is extracted into the correct location.

(16) The browser is redirected into the /admin to perform the installation.

## Implementation plan

The existing standalone utility mdeploy.php will be responsible for actual fetching and deploying the ZIP packages from moodle.org/plugins.

### Required functionality for Moodle sites

- mdeploy.php supports the add-on installation mode
- UI to jump to moodle.org/plugins
- UI to confirm installation
- fetching the plugin meta-info

### Required functionality for the Plugins directory

- Owned sites management - how to store them?
- MUC session cache for anonymous landings
- API providing the meta-information about the plugin
- UI to handle "Install" button clicks

### Communication protocols

#### Moodle site -> moodle.org/plugins

For the step (3), when the admin clicks "Install add-ons from Moodle plugins directory" at their site, they will be redirected to [https://moodle.org/plugins/get.php?site=xxx](https://moodle.org/plugins/get.php?site=xxx) where the xxx value is encoded using a code like this:

$siteinfo = array(
    'fullname' => strip_tags($SITE->fullname),  // e.g. 'My School Online Courses'
    'url' => $CFG->wwwroot,                     // e.g. 'http://my.school.edu/moodle'
    'majorversion' => moodle_major_version(),   // e.g. '2.5'
);

$jsonized = json_encode($siteinfo);

$encoded = base64_encode($jsonized);

$goto = new moodle_url('https://moodle.org/plugins/get.php', array('site' => $encoded));

#### moodle.org/plugins -> Moodle site

For the step (6), when the admin clicks "Install" at a plugin page at moodle.org/plugins, they must be redirected to [http://url.of.their.site/admin/index.php?installaddonrequest=xxx](http://url.of.their.site/admin/index.php?installaddonrequest=xxx) where the xxx value is encoded using a code like this:

$installaddonrequest = array(
    'name' => $plugin->name,                    // human readable plugin name, e.g. 'Stamp collection'
    'component' => $plugin->component,          // frankenstyle component name, e.g. 'mod_stampcoll'
    'version' => $pluginversion->version,       // plugin version in form 2YYYMMDDXX, e.g. 2013032800
);

$jsonized = json_encode($installaddonrequest);

$encoded = base64_encode($jsonized);

$goto = new moodle_url('http://url.of.their.site/admin/index.php', array('installaddonrequest' => $encoded));

It is important to highlight that the API expects plugin versions in the described format. Plugins that do not have their version specified or that violate the documented rules of declaring their version in [version.php](https://docs.moodle.org/dev/version.php "version.php") file, cannot be installed via this method.

Also note that some sites may use non-standard $CFG->admin and the UI in moodle.org/plugins should let users to modify the URL in that case.

#### Moodle site <-> download.moodle.org/api/1.2/pluginfo.php

If the parameter installaddonrequest is detected at admin/index.php, its value is dispatched to the new admin tool_installaddon. The tool makes the admin confirm their intention, checks if the expected target location is writable and that the plugin is not installed yet.

If confirmed, the tool fetches additional data about the requested plugin (among which the most important are the ZIP URL and the ZIP content hash etc) from the API service at [https://download.moodle.org/api/1.2/plugininfo.php?plugin=component@version](https://download.moodle.org/api/1.2/plugininfo.php?plugin=component@version)

This service is sort of proxy/mirror between Moodle sites and moodle.org/plugins. It fetches data about all available plugins (via secret token based web service provided by local_plugins) every hour and distributes it publicly. This way, local_plugins (and thence the moodle.org itself) does not need to handle all the requests (similar design is used for obtaining information about available updates, too).

On successful response, the ZIP is downloaded from the given URL and its content hash is compared with the declared value. The content of the ZIP is validated (various checks are performed, such as matching $plugin->component and the root directory name etc) and the admin has to confirm again they really want to install the ZIP.

Once confirmed, the ZIP is extracted into the plugin type root location and the browser is redirected to the main admin page to perform the actual database upgrade.

## See also

- [Available update notifications](https://docs.moodle.org/dev/Available_update_notifications "Available update notifications") project (since Moodle 2.3).
- [Automatic updates deployment](https://docs.moodle.org/dev/Automatic_updates_deployment "Automatic updates deployment") project (since Moodle 2.4).