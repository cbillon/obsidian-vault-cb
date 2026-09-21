---
tags:
  - moodle
  - plugins
---

# moodle-test-sandbox_ynh

[moodle-test-sandbox_ynh](https://github.com/verzog/moodle-test-sandbox_ynh/blob/main/README.md#moodle-test-sandbox_ynh)

A sandbox for testing **unsupported / development versions of Moodle** in Docker, isolated from any real server. The first setup here runs **Moodle 5.3** with the PostgreSQL 17 version it needs — something YunoHost 12 (Debian 12) can't run natively — without touching your host's packages, database, or other apps.

> **This is a throwaway test environment, not a YunoHost package.** It does not use YunoHost's SSO/LDAP and is not meant for production data. It exists so you can explore new Moodle versions safely before they're officially supported.
# Automate (tool_automate)

[tool_auromate](https://github.com/verzog/moodle-tool_automate#automate-tool_automate)

A no-code, rules-based automation tool for Moodle site administration. Each rule is a **trigger** (manual, scheduled, or a Moodle event) plus optional **conditions** (who or what it applies to) and one or more **bounded, named actions** (what to do). No graph editor, no scripting — every rule is a single guided form, and there is no raw-SQL or arbitrary-code action anywhere in the plugin.
*# Server Monitor Block for Moodle (`block_servermon`)

[block_servermon](https://github.com/verzog/moodle-block_servermon#server-monitor-block-for-moodle-block_servermon)

A lightweight Moodle block that displays live server health metrics on the admin Dashboard. Covers CPU, RAM, disk, top processes, Moodle page-performance metrics, cache store health, session info, a shared-server isolation audit (OS users, PHP-FPM pools, `/proc` `hidepid`), and historical metric logging with CSV export. The whole block can be printed or saved as a PDF.

# Plugin Stash (tool_pluginstash)

[tool_pluginstash](https://github.com/verzog/moodle-tool_pluginstash#plugin-stash-tool_pluginstash)

A test/dev-Moodle convenience tool. It lists the non-core (add-on) plugins installed on a site as a checklist and copies the ticked ones to a **stash directory outside the code tree**. A CLI script copies them back after an upgrade or rebuild, before you run Notifications.

This is intended for throwaway test and development sites where you rebuild the Moodle code tree often and want to keep your add-on plugins handy.

# Remote LibreOffice document converter (fileconverter_remotelibre)

[fileconverter_remotelibre](https://github.com/verzog/fileconverter_remotelibre#remote-libreoffice-document-converter-fileconverter_remotelibre)

A Moodle **document converter** (`\core_files\converter`) that turns office documents into PDF by posting them to a remote render service ([pptx_render_ynh](https://github.com/verzog/pptx_render_ynh)'s `/convert` endpoint) instead of running LibreOffice/unoconv on the Moodle server.

Once configured as the site's document converter, features that rely on doc→PDF conversion — notably the Assignment **"Annotate PDF"** feedback (`assignfeedback_editpdf`) — offload the conversion to the remote box. (Moodle still uses Ghostscript locally to rasterise the PDF for the annotation canvas; this plugin only replaces the conversion step.)