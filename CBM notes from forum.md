---
tags:
  - moodle
  - cbm
---
## upgrade vs migrate

==An **app upgrade** updates software to a newer version on the same system, while an **app migration** moves an application to a completely new environment, platform, or underlying architecture==. 

App Upgrade

- **Definition:** Installing a newer version of software on your current infrastructure.

- **Scope:** Keeps the core architecture, platform, and setup mostly the same.

- **Goal:** Fix bugs, patch security holes, and add new features.

- **Process:** Usually fast, direct, and done "in-place". 

App Migration

- **Definition:** Moving an application to a new environment, host, or framework.

- **Scope:** Changes where or how the app runs (such as moving from local servers to the Cloud Provider Services or changing tech stacks).

## from David Pesce
1. Quick reminder to check your AMD builds. CAMP now rebuilds minified js from source at each release tag and compares it byte for byte. Everything we've found so far is benign (a build file whose source was never committed, builds older than their source), but minified code that doesn't match its source is effectively unreviewable, and your own CI can't catch that since it runs whatever the repo contains. Worth a minute: make sure every amd/build file has its amd/src counterpart committed, and rerun grunt amd after source changes. Releases that match get a "rebuilt, byte for byte" note on their listing; mismatches show as warn-only notes that clear on your next consistent release.
    
2. To be clear about why we built this: a compromised maintainer account or laptop can hide a payload in a few lines of a minified file while the readable source stays clean, and that survives code review, CI, and package verification alike, because reviewers read src while every browser executes build. Rebuilding from source and comparing is the only place that gap closes, which is why it runs registry-side rather than in anyone's own CI.


[Moodle forum](https://moodle.org/mod/forum/discuss.php?d=471355#p1892453)

Brett Dalton
Which is fine for users who have a full development setup.  That's not the case for most users of Moodle

Petr Skoda

There is an easy way to test compatibility - run full PHPUnit and Behat test suites with all additional plugins. I’d personally stay away from plugins that do not have decent test coverage.

Composer seems to be a good solution to most versioning and distribution needs, I do not think we need to be reinventing plugin release processes.

There is much bigger problem in tool_installaddon - see [MDL-87520](https://moodle.atlassian.net/browse/MDL-87520)

I think it should be possible to create a small plugin that uses the existing tool_installaddon APIs to check add-on plugin compatibility with future versions (it does already something similar by accident).

## Sentences
**"In God we trust, all others must bring data"** W. Edwards Deming is ==a popular business and scientific maxim emphasizing that personal opinions and gut feelings must be backed by empirical evidence==.
_We shape our dwellings, and afterwards our dwellings shape us_ Winston Churchill
The registry’s governance had always been there, embedded in terms of service and incident responses. It just hadn’t been tested publicly at scale.
In my many years experience, I've never regretted learning a 'plan B' in case 'plan A' gets disrupted or worse, never materializes. Ken Moodle forum
not a one-off check. 
As the Germans say, “_Vertrauen ist gut, kontrolle ist besser._” Actually, I think it comes from something Lenin originally said, so maybe this was a bad example for me to have pulled up. Ignore that. But, let’s _kontrolle_ nonetheless.
"Simplicity is not the absence of power. It is power without pretense." ~ Atiksh Sharma

There were long discussions about the question bank design in the quiz forum - and elsewhere. But, as Marcus has said, that ship sailed years ago.

Complacency is also contrary to progress. We develop new solutions out of disatisfaction. Complaining has gotten us far.

Users often ask for a particular solution instead of explaining their root issue. Rather than taking the request at face value, I keep digging until I understand what they are trying to accomplish and why existing products do not work for them.
### Can we improve this?

 Keep complaining.

But there's a caveat: It is strictly speaking better to eventually do something about it.

## Reflexion

Price too high
They are re-building these plugins now with AI to keep the functionality.

I really wonder if this is the way it was planned when turning free plugins into paid ones was advertised. This example is a lose-lose-lose situation from a business, from a ecosystem and from a security perspective.

In traditional niche dev circles, respect is earned slowly through years of activity in their respective fora, sharing elegant code, displays of genuine curiosity, and through sharing deep domain knowledge along the way. At the end of the day, these communities don’t care if your code works at all, but instead care that you know why and how it works. To me, an LLM functions best as a force multiplier, not a surrogate. In the hands of an expert who already understands a domain deeply, it could act like a lever.[1](https://blog.fogus.me/llm/born-against.html#fn1) But in these niche communities, the entire exercise is in the learning. Using an LLM to generate the finished piece doesn’t make us craftsmen; it just robs us of the craft.
1 That said, expertise offers no natural immunity against being fooled by LLMs
## Reflexion de https://zwischenzugs.com/

The failure modes are not “the library has a bug.” They are: the maintainer disappears, the maintainer is compromised, the maintainer turns hostile, a transitive dependency changes, a package is unpublished, or the ecosystem shifts under you.

_compliance basically comes down to "We trust the [third party] enough, and [third party component] is vital to our business, so we accept the risks"

### Plugin approval taking longer that expectation

par [David Mudrák](https://moodle.org/user/view.php?id=1601&course=5), mardi 3 décembre 2019, 14:25


As you may be aware of, the approval checks of the submitted plugins are handled by the community developers in their free time on a volunteer basis. There is no prescribed order in which plugins are picked up for approvals. In most cases, it is the first-in-first-out style, but it is not guaranteed and may be affected by other aspects (such as developers' expertise fields etc).

On your plugin's Developer zone pahe you can see how long your plugin has spent in the approval queue. You can compare that with the overall performance published at [https://moodle.org/plugins/queue.php](https://moodle.org/plugins/queue.php) where you will see the recent median is around 50 days. That should give you a rough estimate of how long you are likely to wait. Again, the only thing I can guarantee is that your plugin will not be ignored and that we will eventually get to it.

Thank you for your understanding and patience.(https://moodledev.io/general/community/plugincontribution), [2](https://moodle.org/mod/forum/discuss.php?d=394018)]

## Plugin subtype

Originally I introduced the `plugintypes` object as a one-to-one replacement for the legacy `subplugins.php` file.

When I performed the directory restructure of Moodle I realised that we actually need to calculate the path relative to the new _root_ directory, because not all plugins will be in the the `public` folder in the future. We know that the subplugins will always (for the time being) live as a subdirectory of the parent plugin, so we actually only need the part after this in the path.

To handle this we have added a new `subplugintypes` object which contains the same data,m but without the full path to the plugin. The legacy data is no longer used, except if the `subplugintypes` data is not provided, but is available for two reasons:

1. in the contrib plugin world it's kept to allow a plugin to support multiple Moodle versions; and
2. we wanted to give people a chance to update any CLI tooling which currently reads the `subplugins.json`

You can find the documentation for this file here: [https://moodledev.io/general/development/tools/metadata#subplugins](https://moodledev.io/general/development/tools/metadata#subplugins)

The `plugininfo` class is a class in `[parent/plugin/path]/classes/plugininfo/[subpluginname].php` which extends `\core\plugininfo\base`. You can see an example here: [https://github.com/moodle/moodle/blob/main/public/mod/assign/classes/plugininfo/assignfeedback.php](https://github.com/moodle/moodle/blob/main/public/mod/assign/classes/plugininfo/assignfeedback.php)

It provides the system information about things like:

- can the plugin be uninstalled
- how are settings loaded
- what plugins are available
- are plugins ordered in some fashion?
- and so on.

You can find the source of this file here: [https://github.com/moodle/moodle/blob/main/public/lib/classes/plugininfo/base.php](https://github.com/moodle/moodle/blob/main/public/lib/classes/plugininfo/base.php)

There is a little bit of information about it in the legacy developers docs here: [https://docs.moodle.org/dev/Subplugins#plugininfo_class](https://docs.moodle.org/dev/Subplugins#plugininfo_class)

## Composer
We are not satisfied with the security model of Composer. We believe a package manager has a substantial burden to protect and inform users, and that Composer currently fails to uphold that burden.

When you type composer require package/name, you implicitly trust both packagist.org and the package owner on packagist.org, who is unverifiable and not vetted. This default chain of trust is not made obvious to many users, and the package upstream may be essentially uninvolved. The circumstances in which packagist.org makes package changes are not documented, the changes are not signed, and these changes are not auditable. Package owners on packagist.org are not verifiable, changes they make are not signed, and their changes are not auditable. There is no chain of trust between the package upstream and packagist.org. None of this is very clear to the average user.

You can find more details on a specific case of this at: [https://github.com/phacility/xhprof/pull/40](https://github.com/phacility/xhprof/pull/40)

We may support Composer in the future, but this upstream's attitudes toward security are currently very different from Composer's attitudes toward security.

We understand that a lot of users don't care about this, and Composer works well and is easy to use, but this is important to us.

### Shared Hosting problem

par [Howard Miller](https://moodle.org/user/view.php?id=1473&course=5), lundi 20 octobre 2025, 11:43



[Moodle shared hosting](https://moodle.org/mod/forum/discuss.php?d=470526#p1888712)
Moodle is slowly but surely getting harder to deploy. Which is making it beyond the reach of "amateurs" and those with limited resources. I suppose in some ways this is inevitable.

### Proposed resolution

Figure out why, and perhaps try to get all Drupal projects listed on [https://packagist.org/](https://packagist.org/)?

Would this do the job? If yes, we could add it as a recommended step of maintaining Drupal contrib modules:

> **GitLab Service**  
> To enable the GitLab service integration, go to your GitLab repository, open the Settings > Integrations page from the menu. Search for Packagist in the list of Project Services. Check the "Active" box, enter your packagist.org username and API token. Save your changes and you're done.

From [https://packagist.org/about](https://packagist.org/about)

Or, instead of requiring the individual contrib module maintainers to do this, perhaps a drupal.org staff member has the permissions to set this for all contrib modules and themes on GitLab?

