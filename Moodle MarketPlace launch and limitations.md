---
tags:
  - moodle
---
# Upload plugin versions through the API

Uploading new plugin versions to Moodle Marketplace through the API is not available at launch. Restoring this functionality is the team’s highest development priority after launch, and we aim to release it within the first month.

In the Plugins Directory, some providers used the publishing API and associated GitHub workflows to automate plugin version releases. These workflows cannot currently be used to publish versions to Moodle Marketplace.

Moodle Marketplace runs automated pre-checks when a new plugin version is submitted. The current API upload workflow cannot provide reliable feedback if one of these checks fails. For this reason, API uploads are not supported until the upload and validation process can be handled properly.

For now, upload new plugin versions manually through your Plugin dashboard in Moodle Marketplace.

We are reviewing how best to support automated publishing workflows, including workflows that use GitHub and existing developer tooling. We will update this documentation as work progresses.

# Install plugins directly from Moodle LMS

Installing plugins directly from Moodle LMS through the Plugins Directory is no longer supported.

Existing Moodle LMS installations cannot retrieve plugins directly from Moodle Marketplace because Marketplace downloads require authentication.

For now, download the plugin ZIP file from Moodle Marketplace and install it manually from Site administration > Plugins > Install plugins in Moodle LMS.

Future Moodle releases are expected to introduce changes to plugin installation workflows, including support for Composer-based installations. The Moodle Marketplace and Moodle LMS teams will work together on future integrations and will consider how a direct installation experience could be supported in later releases.

For more info, see [Why direct plugin installation has changed](https://moodle.atlassian.net/wiki/external/MTJiOTkyNzBmNTNlNDhjNDliM2Y1N2E3YzFmNGRkZjY "https://moodle.atlassian.net/wiki/external/MTJiOTkyNzBmNTNlNDhjNDliM2Y1N2E3YzFmNGRkZjY")
# Why direct plugin installation has changed

The Plugins Directory has moved to Moodle Marketplace. As part of this change, direct installation from your Moodle site is not currently available.

You can still install plugins by downloading them from Moodle Marketplace and uploading the ZIP file to your Moodle site.

# What has changed?

Previously, some Moodle sites could install plugins directly from the Plugins Directory using the Install plugin flow in site administration.

This flow used the Plugins Directory’s Sites functionality and redirected the installation back to your Moodle site. It was only available on sites where installation through the web interface was enabled and the Moodle code directory was writable.

With the move to Moodle Marketplace, this direct installation flow is not currently supported.

# Why has this changed?

Moodle Marketplace uses authenticated download endpoints. This means existing Moodle LMS installations cannot retrieve plugin files from Moodle Marketplace in the same way they previously retrieved them from the Plugins Directory.

Direct installation through the web interface also requires the Moodle code directory on the server to be writable. This setup is not recommended from a security perspective.

# How can I install plugins now?

To install a plugin:

1. Go to Moodle Marketplace.
    
2. Find the plugin you want to install.
    
3. Download the plugin ZIP file.
    
4. In your Moodle site, go to Site administration > Plugins > Install plugins.
    
5. Upload and install the ZIP file.
    

# Will direct installation be available again?

Plugin installation is expected to change in future Moodle releases, including upcoming work around Composer-based installation.

The Moodle Marketplace team will work with the Moodle platform team on future integration options. This may include reintroducing a click-to-install experience for supported Moodle versions.
# Change a plugin from free to paid

If your plugin is currently listed as free, you cannot change its pricing model after it has been published. This includes plugins that were migrated from the Plugins Directory. If you would like to offer a paid plugin or integration, you can submit it as a paid listing in Moodle Marketplace.

Support for changing an existing plugin from free to paid is planned for a future release. We will update this documentation when more information becomes available.