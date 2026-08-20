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
The registry’s governance had always been there, embedded in terms of service and incident responses. It just hadn’t been tested publicly at scale.
In my many years experience, I've never regretted learning a 'plan B' in case 'plan A' gets disrupted or worse, never materializes. Ken Moodle forum
not a one-off check.
As the Germans say, “_Vertrauen ist gut, kontrolle ist besser._”  
"Simplicity is not the absence of power. It is power without pretense." ~ Atiksh Sharma

There were long discussions about the question bank design in the quiz forum - and elsewhere. But, as Marcus has said, that ship sailed years ago.
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

### Proposed resolution

Figure out why, and perhaps try to get all Drupal projects listed on [https://packagist.org/](https://packagist.org/)?

Would this do the job? If yes, we could add it as a recommended step of maintaining Drupal contrib modules:

> **GitLab Service**  
> To enable the GitLab service integration, go to your GitLab repository, open the Settings > Integrations page from the menu. Search for Packagist in the list of Project Services. Check the "Active" box, enter your packagist.org username and API token. Save your changes and you're done.

From [https://packagist.org/about](https://packagist.org/about)

Or, instead of requiring the individual contrib module maintainers to do this, perhaps a drupal.org staff member has the permissions to set this for all contrib modules and themes on GitLab?

Git tag immutable
steady state, push a git tag

How are tags immutable? They can be deleted and added again to point at a different commit id.
Setting aside the distributed nature of git, if I delete a tag and add it again to point at a different commit id, how can the tag be considered immutable? If I step away from my desk and come back and look at a commit id, I know it is exactly the same thing as before. I can’t say the same thing about a tag.

The v0.0.1 is just a label for the tag object itself.

E.g. if you clone git.git you can see a tag for v2.16.2, the latest release. That you can re-create point to something entirely different.

But if you do:

```
    $ git rev-parse v2.16.2
    86aabcca24951ccfb9392014c8a379992434a7df
    $ git rev-parse v2.16.2^{commit}
    ffa952497288d29d94b16675c6789ef83850def3
```

You can see that the annotated tag object (GPG signature and all) is really called 86aabcca24951ccfb9392014c8a379992434a7df, and that it points to a commit ffa952497288d29d94b16675c6789ef83850def3.

That 86aabcca... is the immutable part of tags, just as commit objects are immutable.

Note that this only applies to annotated tags, lightweight tags just point to the commit object themselves, so they're really just a sort of branch name that git treats differently in not ever advancing it to another commit.

Annotated tags, however, are stored as full objects in the Git database. They're checksummed; contain the tagger name, email, and date; have a tagging message; and **can be signed and verified with GNU Privacy Guard (GPG)**.

the commit hash doesn't verify who made the commit. Someone else may add a commit to your repo with your name. But if you signed with your key that only you have then anyone else can verify it with your public key that you sent them before.

An annotated tag is made by `git tag -a` and then entering the message when prompted or `git tag -m 'message'`.  A signed tag is made with `git tag -s` and optionally the `-a` or `-m` to include a message
  

In both cases the structure is the same, but the signed one has the gpg signature appended on the message field.

  

See the following sequence of commands for example:

[source](https://git-scm.com/docs/git-rev-parse)
If you want to make sure that the output actually names an object in your object database and/or can be used as a specific type of object you require, you can add the `^{type}` peeling operator to the parameter. For example, `git` `rev-parse` `"$VAR^{commit}"` will make sure `$VAR` names an existing object that is a commit-ish (i.e. a commit, or an annotated tag that points at a commit). To make sure that `$VAR` names an existing object of any type, `git` `rev-parse` `"$VAR^{object}"` can be used.


This experience highlighted an important point — not all Git tags are created equal. Depending on the type of tag used, you may or may not get access to helpful metadata such as:

- Who created the tag
- When it was created
- What the purpose or message behind it was
- Whether the tag is verifiable (i.e., cryptographically signed)

This metadata can be crucial when you’re working on a team, managing multiple environments, or trying to trace the history of a release. But **lightweight tags**? They give you none of that. They’re just simple pointers to a commit — quick and easy to create, but lacking context.

Types of Git Tags Git supports three main types of tags:

- Lightweight Tags — the simplest kind, essentially just a label for a commit.
- Annotated Tags — enriched with metadata like author, date, and a custom message.
- Signed Tags — like annotated tags, but with an added cryptographic signature for authenticity.

Let’s take a closer look at each of them and when to use which.

**Lightweight Tags:** A **lightweight tag** is the most basic type of Git tag. It’s essentially just a **named pointer** to a specific commit nothing more then that just a plain bookmark.

git tag  v1.1

Above command create a git l**ightweight** tag which just point to the commit without any metadata.

Press enter or click to view image in full size

LightWeight Tag

As we can see above, the tag simply points to a commit that was made way back on **June 20th**, even though the tag itself was created today for this demo.

**Annotated Tag:**This is a better way of tagging then lightweight tag.An annotated tag is a full Git object that stores rich metadata along with the tag. This includes:

- The tagger’s name and email
- The date the tag was created
- A message describing the tag.

git tag -a v1.0 -m "Release Final Version"

Above command which has a message as well creates an annotated tag.

Press enter or click to view image in full size

Annotated Tag

As shown in the example above, this tag contains **much more information** — including **when** the tag was created, **who** created it, and the **message** describing the purpose of the tag.

**Signed Tag:**This is a better version of annotated tag**.I**t includes a cryptographic signature. This signature verifies that the tag was created by you, and that it hasn’t been tampered with.

git tag -s v1.2 -m "Release Final Version with signed "

Above command creates a signed tag with message and signed by the developer who is relasing the version .

Press enter or click to view image in full size

Signed Tag

At first glance, a **signed tag** may look almost identical to an **annotated tag** — same message, same tagger info, same creation date. So you might wonder: **_what’s actually different here?_**

git tag -v v1.2  
object 1406f4f7b28223305f42f6531fcd49a159f27e92  
type commit  
tag v1.2  
tagger Rajesh Kumar  Dash <dashrajesh49@gmail.com> 1753469617 +0530  
  
Release Final Version with signed tag  
gpg: Signature made Sat Jul 26 00:23:37 2025 IST  
gpg:                using RSA key somekey  
gpg: Good signature from "rajesh (signed rsa) <dashrajesh49@gmail.com>" [ultimate]

So, what sets it apart is the **additional layer of information**: we can see **who signed** the tag, **when the signature was made**, and even details like the **key ID** and **email address** of the signer.

This makes it easy to **verify the authenticity** of the tag and trace it back to a trusted source.

After going through all this, you might be wondering: “**If signed tags offer the most features and security, why not just use them all the time?**”

- Use light weight tag when we need a quick reference to a commit for personal or temporary use.Avoid using lightweight tags for production or release workflows — they don’t carry any context or traceability
- Use Annoated tag when you need a tag that’s **traceable** normally preffered in production release; I use annotated tags  for all official releases and major milestones.
- Use signed tags when you’re publishing to an open-source project or distributing binaries. The **cryptographic signature ensures that others can verify the authenticity of your release**, **preventing anyone from mirroring your repository and pointing the same release tag to a malicious or broken commit.


**`git push --follow-tags`**
- [git push --follow-tags](https://stackoverflow.com/a/26438076/895245) will only push annotated tags
This is a sane option introduced in Git 1.8.3:

```
git push --follow-tags
```

It pushes both commits and only tags that are both:

- annotated
- reachable (an ancestor) from the pushed commits

This is sane because:

- you should only push annotated tags to the remote, and keep lightweight tags for local development to avoid tag clashes. See also: [What is the difference between an annotated and unannotated tag?](https://stackoverflow.com/questions/11514075)
- it won't push annotated tags on unrelated branches

It is for those reasons that `--tags` should be avoided.

Git 2.4 [has added](https://github.com/git/git/commit/61ca378275e83c48343c74a849ff0dcdef9abc91) the `push.followTags` option to turn that flag on by default which you can set with:

```
git config --global push.followTags true
```

or by adding `followTags = true` to the `[push]` section of your `~/.gitconfig` file.