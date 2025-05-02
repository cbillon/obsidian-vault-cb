---
tags:
  - moodle
  - behat
---
#### [General developer forum](https://moodle.org/mod/forum/view.php?id=55) -> [Coding best practices for creating open source Moodle plugins](https://moodle.org/mod/forum/discuss.php?d=462107) -> [Re: Coding best practices for creating open source Moodle plugins](https://moodle.org/mod/forum/discuss.php?d=462107#p1855297)

par [Marcus Green](https://moodle.org/user/view.php?id=2246&course=5), mardi 17 septembre 2024, 04:35

I am sure others will come in with advice but one of my top suggestions is that you run the moodle-ci (continuous integration) plugin, which typically runs against Github actions.  
[https://moodledev.io/general/development/tools/gha](https://moodledev.io/general/development/tools/gha)  
  
This checks a whole bunch of stuff. It will run behat and phpunit tests, will lint php, [javascript](https://moodle.org/mod/glossary/showentry.php?eid=25&displayformat=dictionary "Glossary of common terms : Javascript"), mustache etc etc and some moodle specific stuff such as upgrade scripts. This is an example I was using yesterday.  
[https://github.com/marcusgreen/moodle-qbank_bulktags/blob/main/.github/workflows/moodle-ci.yml](https://github.com/marcusgreen/moodle-qbank_bulktags/blob/main/.github/workflows/moodle-ci.yml)

https://moodle.org/mod/forum/discuss.php?d=462107#p1855297

[Fulldecent Plugin](https://github.com/fulldecent/moodle-enrol_course_tokens)

[Test Behat](https://tracker.moodle.org/browse/MDL-75081)

[Valery Fremaux ](https://tracker.moodle.org/browse/MDLSITE-4781)

while we are having more and more power tools to test and validate, thus raising trustability of our plugins, it would be soooo great we could complete a fully integrated code providing process by scripting the update of plugins archives, push new versions and submit new plugin material using a script API.

Ideally it might directly retrieve code in Github but some processing should be done on package name, such as reading version file and extracting the folder name. I'm not sure i remember that is has been proposed for a short time, but there was a mismatch in the package construction, that is, guthub should contain the immediate code of the plugin at root level (for travis-ci integration chain to worl properly), but the github puller was asking having the code in a named dir. Now this seems to be quitte old stuff...

I would of course be one of the lead consumer of this ability, having around 130 contribs/plugins now on the way of being validated in 5 branches each. This now asks me about 650 submission and package updates to do, sometime several times a month, obvioulsy impossible to do unless i can script and trigger version update uploads.

What would you think about it ?

In mypoint of view it would be a huge time saved and thus much more time affected in Unittest or Behats and in documentation.

[![](https://tracker.moodle.org/secure/useravatar?size=xsmall&ownerId=mudrd8mz&avatarId=22398)David Mudrák (@mudrd8mz)](https://tracker.moodle.org/secure/ViewProfile.jspa?name=mudrd8mz) added a comment - [20/Jan/21 6:20 PM](https://tracker.moodle.org/browse/MDLSITE-4781?focusedId=835615&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-835615)

As I envision it, there are several ways to approach this feature. Ideally we end up having implemented most (if not all) in some clever way. Please share your thoughts if this would cover your needs.

- **1. Releasing a plugin in the Plugins directory automatically once it is released in Github.** Github has support for releases described at [https://docs.github.com/en/github/administering-a-repository/about-releases](https://docs.github.com/en/github/administering-a-repository/about-releases)
- **2. Releasing a plugin once it is tagged** - This needs some more thinking.
    - We already do support tagged versions partially and allow them to be easily picked while manually releasing the plugin. So adding a bit more automation to this could help.
    - My only worry is that I don't like releasing _every_ tagged version as a new plugin version. Tags should not imply a new version automatically. Maybe unless the maintainer says so. So we could have a new per-plugin checkbox like "Automatically release every tag as a new version" for people who use tags exclusively for releasing.
    - This could be both based on triggers (external repository calls Plugins directory that a new tag/release is available) or regular checking of the repository for new not-yet-released tags.
- **3. Having an API allowing for custom scripts and tools to release new version.** That will also allow us to re-implement the current UI in plain ES6 client side.

Other related:

- Nice to have - API and/or generally a way to trigger prechecking before releasing, as mentioned above (but I agree it should not be a blocker).
- Should have - Others than Github: Releases are Github specific feature, tagging is not. Maybe we should focus on release-by-tagging as it would allow to support wider range of repository hosting platforms.

[![](https://tracker.moodle.org/secure/useravatar?size=xsmall&ownerId=marina&avatarId=25763)Marina Glancy](https://tracker.moodle.org/secure/ViewProfile.jspa?name=marina) added a comment - [22/Jan/21 11:00 PM](https://tracker.moodle.org/browse/MDLSITE-4781?focusedId=836044&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-836044) - edited

Is it possible to specify in version.php the list of Moodle versions the plugin is compatible with? Normally when I upload a new version of a plugin I have to select them from the dropdown manually.

The reason I'm asking is that I would like to avoid adding any metadata (suggested by Valery) and have all necessary information inside the plugin code

### Improved validation, API for maintainers and ability to release by tagging at GitHub
[David Mudrak](https://moodle.org/mod/forum/discuss.php?d=423251)
### Releasing Moodle plugins in the Plugins directory automatically from GitHub Actions
[README](https://github.com/moodlehq/moodle-plugin-release)

### Moodle-tool_vault  Backup
[plugin Vault ](https://github.com/lmscloud-io/moodle-tool_vault/blob/main/README.md)