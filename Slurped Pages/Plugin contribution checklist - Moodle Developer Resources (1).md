---
link: https://moodledev.io/general/community/plugincontribution/checklist?_gl=1*ywc0ds*_ga*NDk4Njg3OTA4LjE3NDM5NTY4NjI.*_ga_QWYJYEY9P5*MTc0NDE4MDk0OS4zLjEuMTc0NDE4MDk4MC4wLjAuMA..
excerpt: Before approaching the Moodle plugins directory and submitting your
  plugin (or a new plugin version), you are encouraged to go through the
  checklists below and fix eventual issues with your plugin. Doing so will make
  the reviewer of your plugin happy :-) and may have impact on how long your
  plugin has to spend in the approval queue before it lands smoothly.
slurped: 2025-04-09T06:47
title: Plugin contribution checklist | Moodle Developer Resources
---

[![Moodle.org](https://moodle.org/theme/moodleorg/pix/moodle_logo_TM.svg)](https://moodle.org/)

- [Forums](https://moodle.org/course)
- [Documentation](https://docs.moodle.org/)
- [Downloads](https://download.moodle.org/)
- [Demo](https://moodle.org/demo)
- [Tracker](https://tracker.moodle.org/)
- [Development](https://moodledev.io/)
- [Translation](https://lang.moodle.org/)
- [](https://moodle.org/public/search)

What are you looking for?

---

Learn about Moodle's products, like Moodle LMS or Moodle Workplace, or find a Moodle Certified Service Provider.[Moodle.com](https://moodle.com/)

---

Our social network to share and curate open educational resources.[MoodleNet](https://moodle.net/)

---

Courses and programs to develop your skills as a Moodle educator, administrator, designer or developer.[Moodle Academy](https://moodle.academy/)

[Moodle.com](https://moodle.com/)  
Learn about Moodle's products, like Moodle LMS or Moodle Workplace, or find a Moodle Certified Service Provider.  

---

[MoodleNet](https://moodle.net/)  
Our social network to share and curate open educational resources.  

---

[Moodle Academy](https://moodle.academy/)  
Courses and programs to develop your skills as a Moodle educator, administrator, designer or developer.  

- [](app://obsidian.md/)
- [Plugin contribution](app://obsidian.md/general/community/plugincontribution)
- Checklist

## Plugin contribution checklist

Before approaching the [Moodle plugins directory](https://moodle.org/plugins) and submitting your plugin (or a new plugin version), you are encouraged to go through the checklists below and fix eventual issues with your plugin. Doing so will make the reviewer of your plugin happy :-) and may have impact on how long your plugin has to spend in the approval queue before it lands smoothly.

### Plugin descriptions[​](#plugin-descriptions "Direct link to Plugin descriptions")

- Have a meaningful description of your plugin prepared in English.
- You will need a short concise description (just a sentence or two) for the short description field, and another elaborated one for the full description field.
- It is encouraged to have the same info at the plugin record page and in its `README` file.

### Supported Moodle versions[​](#supported-moodle-versions "Direct link to Supported Moodle versions")

- New plugins submitted into the plugins directory must support at least one of the currently maintained Moodle version.

See [Releases](app://obsidian.md/general/releases) for the list of currently maintained Moodle versions (policy issue [MDL-47579](https://tracker.moodle.org/browse/MDL-47579)).

### Code repository name[​](#code-repository-name "Direct link to Code repository name")

- Provide a consistent experience for other Moodle developers and site administrators - follow the repository naming convention for Moodle plugins: `moodle-{plugintype}_{pluginname}`.

### Source control URL[​](#source-control-url "Direct link to Source control URL")

- Facilitate sharing and further development of your open-source plugin - provide publicly accessible URL of your code repository.
- Github is a choice of most Moodle plugin developers.

### Bug tracker URL[​](#bug-tracker-url "Direct link to Bug tracker URL")

- Encourage participation and have a place to report issues, bugs, make feature requests, or suggest other types of improvements.
- Both [Moodle tracker](app://obsidian.md/general/development/tracker/guide) and [Github issues](https://guides.github.com/features/issues/) are common.

### Documentation URL[​](#documentation-url "Direct link to Documentation URL")

- Have a place where further documentation of your plugin will be located.
- [Moodle docs](app://obsidian.md/general/community/plugincontribution/documentation) is preferred location, [Github wikis](https://guides.github.com/features/wikis/) or your own website will work, too.

### Illustrative screenshots[​](#illustrative-screenshots "Direct link to Illustrative screenshots")

- Capture some screenshots of your plugin to help folks get an idea of what it looks like when installed.
- We will use these screenshots at more places in the plugins directory in the future.

### Licensing[​](#licensing "Direct link to Licensing")

- All files that implement the interface between the Moodle core and the plugin must be licensed under GNU GPL v3 (or later).
- Additional files contained in the plugin ZIP package (such as third party libraries used by the plugin, or included media) may eventually use other license as long as it is [GPL compatible](http://www.gnu.org/licenses/license-list.html#GPLCompatibleLicenses). See [Plugin files#thirdpartylibs.xml](app://obsidian.md/docs/5.0/apis/commonfiles#thirdpartylibsxml) for how to do it.
- Note that binary files violate GPL unless the source code is also included or available (e.g. Java classes or Flash).

### Intellectual property rights[​](#intellectual-property-rights "Direct link to Intellectual property rights")

You represent and warrant that any and all intellectual property that you upload or submit via the [Moodle plugins directory](https://moodle.org/plugins) is either your sole and exclusive property or you have secured any and all authorisation, licences and rights to use such intellectual property as applicable under relevant laws. You hereby grant Moodle a licence to use, copy, transmit, store, process and back-up your data, account information, intellectual property and other related information ("Data") and consent to Moodle removing all Data should it determine to do so.

### Subscription needed[​](#subscription-needed "Direct link to Subscription needed")

- If the plugin requires a third-party subscription based service, make sure the description states it very clearly.
- If the plugin requires some credentials to integrate with an external system (such as API keys etc), the description should provide clear information of where and how the users can obtain them.
- To allow the testing of the plugin functionality, please provide us with demo credentials (such as API keys etc) so that the approval team can use them to see the plugin in action.

## Usability[​](#usability "Direct link to Usability")

### Installation[​](#installation "Direct link to Installation")

- Make sure the plugin installs smoothly from the ZIP package using the in-build plugin installation interface.
- If any non-standard post-installation steps are needed, make sure they are clearly listed in both plugin description and the `README` file.

### Dependencies[​](#dependencies "Direct link to Dependencies")

- If the plugin depends on another additional plugin, make sure it is clearly stated in the description and in the `README` file.
- Also declare the dependency explicitly in the plugin's [version.php](app://obsidian.md/docs/5.0/apis/commonfiles/version.php) file.
- Plugins must not require the admin to run **composer** or similar dependency manager tools. Many Moodle admins do not have access to their server shell and/or are not experienced with PHP building tools. Moodle plugins are supposed to be distributed in packages that work out of the box.

### Functionality[​](#functionality "Direct link to Functionality")

- Test the plugin functionality with full developer debugging enabled.
- Make sure the code does not throw unexpected PHP warnings, notices or even errors.

### Cross-DB compatibility[​](#cross-db-compatibility "Direct link to Cross-DB compatibility")

- Test the plugin with multiple database engines supported by Moodle.
- At very least, the plugin is supposed to work with MySQL and PostgreSQL unless reasons are clearly explained in the description and the `README` file (such as the plugin is a wrapper for third-party DB specific utility).

## Coding[​](#coding "Direct link to Coding")

### Coding style[​](#coding-style "Direct link to Coding style")

- It is encouraged to follow [Moodle coding style](app://obsidian.md/general/development/policies/codingstyle) and other [coding guidelines](app://obsidian.md/general/development/policies).
- It's not always possible to achieve "all greens" in automated syntax checks (especially when third party libraries are involved) but you should aim to it.
- Consistent style helps others to read and understand your code (not only during the approval review).

### English[​](#english "Direct link to English")

- Moodle is an international project. To facilitate sharing, reviews of and contributions to your code, all comments, variable names, function names etc. should be in English.

### Boilerplate[​](#boilerplate "Direct link to Boilerplate")

- All files should contain the common boilerplate at the beginning with explicit GPL license statement.

### Copyrights[​](#copyrights "Direct link to Copyrights")

- All files should contain the `@copyright` tag with your name.
- If you are re-using someone else's file, keep the original copyrights reference to the previous author and add your name as a copyright holder.
- Both things should be clear: (1) that it is you to be blamed for the file code and (2) that your work is based on someone else's work.

```
/** * @copyright 2023 John Smith <john.smith@example.com> * @copyright based on work by 2020 Mary Stuart <mary.stuart@example.com> */
```

### CSS styles[​](#css-styles "Direct link to CSS styles")

- All styles.css files from all plugins are concatenated, cached and served as a one big resource on all pages of the given Moodle installation.
- For that reason, plugins must use properly namespaced selectors so that their style sheets can be safely combined with others without affecting other pages and elements beyond the plugin's scope.
- Plugin specific CSS selectors are needed to make sure that your styling does not accidentally affect other parts of Moodle outside your plugin scope.

Instead of the selector `.contentarea` it is better to use something like `.path-mod-mymodule .contentarea` as the `.path-\*` classes are automatically added by the Moodle core renderers to the HTML `<body>` tag.

### Namespace collisions[​](#namespace-collisions "Direct link to Namespace collisions")

- Check that all your DB tables, settings, functions, classes, constants and variables are named correctly. In most cases their names must start with the plugin type and plugin name, as in `block_yourname_something` (so called [frankenstyle](app://obsidian.md/general/development/policies/codingstyle/frankenstyle) prefix). Modules are an exception to this rule as functions such as get_coursemodule_from_id() rely on there being no preface of 'mod'.
- Do not define own classes, functions, variables or constants in the top-level scope (global space) without the valid frankenstyle prefix.

### Settings storage[​](#settings-storage "Direct link to Settings storage")

- Check that your settings are stored in the table `config_plugins` and not in the main `config` table.
- This helps to avoid `$CFG` bloat and potential collisions.
- Use `get_config()` to pull the settings data out of the `config_plugins` table.
- In the file `settings.php`, the setting names are supposed to be `plugintype_pluginname/settingname` (note the slash) and not `plugintype_pluginname_settingname` or even just `settingname`.
- If you eventually need to change the settings yourself, use `set_config()`.

### Strings[​](#strings "Direct link to Strings")

- Avoid hard-code texts in the code, always use `get_string()`.
- Just the English strings should ship with the plugin. All other translations are supposed to be submitted as contributions at [lang.moodle.org](https://lang.moodle.org/) once your plugin is approved - see [Translating plugins](https://docs.moodle.org/en/Translating_plugins).
- Your code must not rely on trailing and leading whitespace in strings.
- The string file must be considered as pure data file with the syntax `$string[]('id') = 'value';`. No other PHP syntax such as [concatenation](http://php.net/manual/en/language.operators.string.php), [heredoc and nowdoc](http://php.net/manual/en/language.types.string.php) is supported by the tools that we use when processing your strings (even if it may work in Moodle itself).
- The English language pack (`lang/en/`) in Moodle does not use "Capitalised Titles".

### Privacy[​](#privacy "Direct link to Privacy")

- Avoid gathering, storing, processing and sharing personal data unless they are needed for the plugin functionality.
- For plugins that integrate with an external system, privacy API implementation is required prior approval in the plugins directory (most notably the meta-data provider part).

Inform about all the personal data that the plugin processes, via both description and the [Privacy API](app://obsidian.md/docs/5.0/apis/subsystems/privacy).

### Security[​](#security "Direct link to Security")

- Never trust the user input.
- Do not access superglobals like `$_REQUEST` directly, use wrappers like required_param() with correct type declared to sanitize input.
- Always use placeholders in custom SQL queries (`?` or `:named`).
- Always check for the `sesskey` before taking an action on submitted data.
- Check for `require_login()`.
- Always check that the user has appropriate capabilities before displaying the widgets _and_ before taking the actual action.
- Avoid using malicious functions like `call_user_func()`, `eval()`, `unserialize()` and so on, especially when they would be called with user-supplied data.

## Approval blockers[​](#approval-blockers "Direct link to Approval blockers")

Examples of issues that will prevent your plugin from being approved:

1. There is no public and transparent issue tracker where the community members can leave feedback, report bugs and suggest improvements.
2. Your SQL fails to work on PostgreSQL even when working on MySQL.
3. [Namespace collisions](app://obsidian.md/general/development/policies/codingstyle/frankenstyle).
4. Compliance with [security guidelines](app://obsidian.md/general/development/policies/security).
5. It integrates with an external system and does not have the privacy API correctly implemented.
6. It is an activity module and does not have the backup and restore API implemented.
7. You must be in compliance with the [Moodle.org Site policy](https://moodle.org/admin/tool/policy/view.php?policyid=4).

## See also[​](#see-also "Direct link to See also")

- [Some common issues in submitted plugins](https://moodle.org/mod/forum/discuss.php?d=263614) post at the Plugins traffic forum at moodle.org
- [Moodle Coding Style](https://moodle.org/mod/forum/discuss.php?d=371967) forum thread at the General Developer Forum at moodle.org
- [Plugin files](app://obsidian.md/docs/5.0/apis/commonfiles) has a list and descriptions of files that work the same in all plugin types

[](app://obsidian.md/general/community/plugincontribution)[](app://obsidian.md/general/community/plugincontribution/codeprechecks)

- [Meta-data](#meta-data)
    - [Plugin descriptions](#plugin-descriptions)
    - [Supported Moodle versions](#supported-moodle-versions)
    - [Code repository name](#code-repository-name)
    - [Source control URL](#source-control-url)
    - [Bug tracker URL](#bug-tracker-url)
    - [Documentation URL](#documentation-url)
    - [Illustrative screenshots](#illustrative-screenshots)
    - [Licensing](#licensing)
    - [Intellectual property rights](#intellectual-property-rights)
    - [Subscription needed](#subscription-needed)
- [Usability](#usability)
    - [Installation](#installation)
    - [Dependencies](#dependencies)
    - [Functionality](#functionality)
    - [Cross-DB compatibility](#cross-db-compatibility)
- [Coding](#coding)
    - [Coding style](#coding-style)
    - [English](#english)
    - [Boilerplate](#boilerplate)
    - [Copyrights](#copyrights)
    - [CSS styles](#css-styles)
    - [Namespace collisions](#namespace-collisions)
    - [Settings storage](#settings-storage)
    - [Strings](#strings)
    - [Privacy](#privacy)
    - [Security](#security)
- [Approval blockers](#approval-blockers)
- [See also](#see-also)