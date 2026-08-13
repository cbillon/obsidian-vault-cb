---
tags:
  - cbm
  - git
---
 [Johannes Burk](https://moodle.org/user/view.php?id=1789360&course=5)

Since I like managing things with git I'd love to see every plugin using some consistent model which links stable releases to specific commits. But I don't think it is desired or even possible to force plugin developers to something. 

My idea is to recommend git as the source control system for plugin development and make some recommendations regarding versioning, branch naming and tag usage. I know that it is not safe to assume that the latest commit of something is stable. Therefore I would always recommend to use git tags for releases. The Moodle plugin directory already supports linking a plugin version to a VCS tag! Additionally a stable branch per series could help finding the correct version when working with git only. When a developer takes "stable" branches serious (and makes use of the feature branching model) then it should be safe to use the latest commit of such a stable branch. And since branches are cheap why not use multiple branches pointing to the same commit if nothing on the plugin side changed between two or more Moodle series. 

What I'm currently missing is a recommendation for a human readable plugin version ($plugin->release). ... And a wider adoption of $plugin->supported, $plugin->incompatible and the specification of a VCS tag in the plugin directory. 

Regarding the discussion about the LTS version: I think making the last version per series LTS is the right decision.