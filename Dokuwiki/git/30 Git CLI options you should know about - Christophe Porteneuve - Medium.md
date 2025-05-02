---
link: https://medium.com/@porteneuve/30-git-cli-options-you-should-know-about-15423e8771df
byline: Christophe Porteneuve
site: Medium
date: 2014-09-15T09:23
excerpt: You think you know Git? Maybe you do… And yet, I’d bet my shirt that many cool little command-line options remain unknown to you. Indeed, as Git versions march on, a lot of such options surface, be…
slurped: 2024-06-07T09:47:32.620Z
title: 30 Git CLI options you should know about - Christophe Porteneuve - Medium
tags:
  - git
  - tips
  - porteneuve
---

[

![Christophe Porteneuve](https://miro.medium.com/v2/resize:fill:88:88/0*E5E12sAhnnsMPeAF.png)



](https://medium.com/@porteneuve?source=post_page-----15423e8771df--------------------------------)

You think you know Git? Maybe you do… And yet, I’d bet my shirt that many cool little command-line options remain unknown to you.

Indeed, as Git versions march on, a lot of such options surface, be it about more comfort, more raw power, or additional safeguards. As they are not a new command _per se_ though, they are usually not touted as much and go under your radar.

I selected here about thirty options, spread across roughly fifteen commands, that will make your Git life more enjoyable. This makes for an excellent ROI over your next few minutes of reading!

I will generally put the option right in the section title, intentionally. Still, do not skip a section because you think you know that option: I may use it on another command than the one you think, or for another reason, that may be news to you. Also, I often slap on extra info on associated options and configuration variables.

## Partial (un)staging with -p

So you opened a file for a specific reason, perhaps make that damned tracker asynchronous… And you notice in passing that ARIAL roles are missing from a few UX items, and that the footer is still hard-coded instead of coming from the layout, and what not…

When you’re about to commit, you realize that file contains a solid half-dozen (if not more) edits that **span multiple unrelated topics**. You then have three possible routes:

- You spew a big fat **ugly kitchen-sink commit**, complete with a lousy message full of “+” signs or, if you’re even lazier, the time-honored useless “Changes,” “Fixes,” “Lots of stuff,” etc.
- You **copy-paste the file** somewhere then start undo-ing, if that’s even possible, to only keep the first top, commit, re-apply changes for the second, commit again, then the third… All of this by hand, naturally. **Screw-up probability: 99%**.
- You read this, or attend our training classes, and now _-p_ !

The _git add -p_ command is actually a refinement of _git add -i_: it pre-selects the **interactive add patch mode**. In practice, you tell it what file you want to operate on, to go even faster. For instance:

git add -p index.html

Let me seize that opportunity to remind you that _git add_ is **not** about putting a file under version control, but to **stage** an edit, that is, to confirm that edit as a part of the next commit.

When you perform such an add, Git will auto-split the content in _hunks_, which are groups of edits, using proximity inside the file, and unchanged lines for splitting. If your edits are too close together, Git will probably not auto-split, and you’ll have to do it yourself using the _s_ key (Git will provide a plethora of possible commands, by their initials, in a prompt. If in à), use _?_ to display help), which here stands for _split_.

Note that even if you have _adjacent_ edits (edits without unchanged lines between them), you can edit the snapshot on the fly to make it look like what you intend to stage, using the _e_ (_edit_) command. It’s sort of express Photoshopping for your snapshot. Actually, if you know from the get-go that your file has such adjacent hunks, you can pre-select that mode using the _-e_ option instead of _-p_. In that case however, Git will not pre-split other hunks for you.

When you’re done, your file will normally appear **as both staged and modified**. That’s to be expected, as indeed:

- the latest committed version isn’t the same as the staged one: your file thus appears staged.
- the staged version isn’t the same as the file in the working directory: your file thus appears modified.

You can check out the diff for the staged version using _git diff --staged index.html_. If you want to the see the whole staged snapshot, instead of diffs, you can go with _git show :0:index.html_ (that’s a zero, not an O letter).

After that, be extra careful not to do a _git commit -a_ (for instance, _git commit -am “Asynchronous tracker”_), as that _-a_ will auto-stage every known edit, thereby overwriting the “sculpted” stage you had put together.

Finally, few people know that _git reset_ also features a _-p_ option, which has the exact same UX as in _add_, but obviously does the opposite: it _unstages selected hunks_. It’s often used to split the latest commit, by doing something like this:

git reset -p HEAD^

Edits are then presented as cancellations of those in the latest commit. You tell which cancellations you want, amend the commit (see below), then complete the extra commit(s) you want with the remaining modifications.

It’s a very “quick and pro” way of splitting a commit inside an [interactive _rebase_](https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa#541c), using its _edit_ command.

## Properly account for renames using -A

You may know that, by default (at least before 2.0), _git add_ behaved as _git add --no-all_ or, if you prefer, _git add --ignore-removal_. It only used the working directory as a basis to compute its list of files to take into account, which therefore included:

- Modifications to known files
- New files

On the other hand, files known to Git’s index but not found in the disk anymore, which appeared as removed, were left aside.

**This was a problem for renames and moves**, which result in both a “deletion” of the old path, and the apparition of the new one.

To deal with this, we use _git add -A_, or its longer form _git add --all_. **This takes everything into account**. When the index is then aware of both changes, it can “realize” it’s a rename (even if some of the content has changed in the file, too), which later allows _git log_ to follow the file across renames, for instance.

**Starting with Git 2.0**, this is the default behavior of _git add_ if you provide a path to it (e.g. _git add ._). By the way, another important change in 2.0: before it, when you did _git add -A_ with no path, it would only work on the current directory and its subfolders, but from 2.0 on, it will work on _the entire repository_, wherever you are in it.

## Get inside untracked directories for status

I’m sure you noticed: when you add a folder to a repo, _git status_ only lists the folder itself as untracked, not its contents. For instance, let’s say I just added a super plugin to my project, with a JS and CSS file inside:

Status does not enter untracked directories by default.

I find that annoying. We can ask _status_ to get inside using _-u_:

Status can enter untracked directories when asked to.

I find this so useful that I set the appropriate configuration variable in my global configuration, so it’s always on:

git config --global status.showUntrackedFiles all

## Produce more useful diffs

The diffs produced by _git diff_, _git log_ and _git show_, to name only these, are nice but definitely have room for improvement. Here are three tweaks that are near and dear to my heart:

git diff -w

Or its longer form, _git diff --ignore-all-space_ (which is indeed more explicit). This option lets diffs ignore any whitespace change inside or on the edge of lines, including going from no whitespace to some, and vice-versa. The one exception is no blank lines to some blank lines, and vice-versa.

This can come back and bite you when you work on files with significant indenting, but for most cases, it’s a great way to “unspam” the display and focus on useful bits.

Without _-w_:

A diff without -w can contain a lot of cruft

With _-w_:

Note how using -w lets us focus on more useful bits

Another option I love is about the core display of diffs. By default, it’s a line-by-line thing, which sometimes doesn’t quite cut it:

Line-by-line diffs can make it difficult to spot changes

You can start by asking _diff_ to only display the line once, using word delimiters, thanks to the _--word-diff_ option. The definition of “word” here is based on _whitespace_. For editorial content, that’s just fine:

Whitespace-delimited “words” are good enough for editorial content

And by the way, if you dislike these +/- brackets, you can use _--word-diff=color_ to spruce this up. Actually, there’s a shorter form called _--color-words_ (ain’t it cute…).

Using —color-words removes the +/- bracketing. Depending on the diff, this may improve, or impair, legibility of the result.

Anyway, this leaves us with a problem when diff’ing code, as _whitespace_ is seldom the only useful delimiter. Just look at this:

Whitespace-based word diffing kinda sucks on code

To fix this, we’ll go for a regex that says “this is a word.” Taking this to the extreme would be “any character, even just one,” using _._ (a single period), but if this goes overboard on your specific situation, you can lengthen it, for instance _.{3,}_ for at least 3 characters_._

This would result in quite verbose command lines: _--word-diff=color --word-diff-regex=._ or so. We’ll go for a shorter version:

git diff --color-words=.

Here goes:

Custom colored words diffing for the win!

If you want to make such an approach systematical (I often do), you can configure _diff.wordRegex_ to the proper value (e.g. _._), so any type of _word diff_ (e.g. _--color-words_) with no argument will use it (an explicit regex in the CLI will of course have priority).

## Fix the latest commit with --amend

When looking at the quick recap displayed after a _git commit_, one often realizes they just slipped up: forgot a file, committed one file too many, that kind of thing…

An easy way to fix this, as long as you haven’t pushed that commit to your friends, is to put yourself back in the proper situation (e.g. perform the necessary _git add_, _git reset_ or _git rm --cached_ on the problematic file, perhaps combined with a judicious addition to _.gitignore_…) then do this:

git commit --amend

This option is actually nothing more than a _git reset --soft HEAD^_ before the requested commit, but most people _do not master reset_, so this helps.

Also note that most of the time, the initial commit message was fine. I doubt that you had originally labeled it “Migrating to Bootstrap 3.1, and by the way mistakenly versioning the server’s root password.” In order to avoid having to re-type that message, or simply having the editor pop up, you can do this:

git commit --amend --no-edit

If you’re still running on a Git older than 1.7.9 (Gawd, [look at what you’re missing!](https://medium.com/@porteneuve/whats-new-with-git-since-1-7-5bbec76fa394)) do this instead:

git commit --amend -C HEAD

It’s such a common use case that I often see people alias this as _git oops_ :-)

In your log, only the latest version of the commit is visible: it’s as if you had never slipped up (how good are you!). The former version(s) of the commit of course remain in your [reflog](https://www.atlassian.com/en/git/tutorial/rewriting-git-history#!reflog), as the general Git principle holds: “if it’s been committed, it’s virtually impossible to lose.”

## Smart filtering of logs with -S and -G

The _git log_ command is packed with options (100+!), many of which it shares with its close cousin _git diff_.

A number of these options are there to **filter the log** even before displaying it (which is vastly faster and useful than grep’ing it afterwards): filtering based on dates, paths, branches, authors and committers, commit messages… but also **diff contents**. Specifically, active diff lines.

Diff filtering is extremely useful to **hunt down the origin of some code, especially of a bug**. Too many people think they should use _git blame_ for this, mostly because of a cargo-culted _svn blame_ reflex, but this command is just as dumb as its _svn_ counterpart:

- It only displays the commit _that touched the line last_, without telling you why; for all we know, it may just have trimmed trailing whitespace.
- It only displays _currently existing lines_, so if the issue is that a line was removed, it’s completely useless.

On the contrary, if you filter **diff contents**, you’ll indeed know **which commit introduced the change you’re interested in**.

If we’re only interested in the presence of a given text in the diff’s active lines (the +/- lines, not the context lines), regardless of why and how it got there, we’ll usually go with _-G_ (this is regex-based, so do remember to escape regex-special characters):

git log -G 'Secure_?Random' -2 -- path/to/problematic_file

(We’re usually only interested in the 1–2 latest commits when doing such a search.)

On the other hand, if we’re specifically looking for diffs that **removed or added the text**, we’ll go with _-S_, which only returns diffs that **changed the number of occurrences** of the text. By default, _-S_ takes a fixed string, but if you want it to be a regex, just add _--pickaxe-regex_:

git log -S 'Secure_?Random' --pickaxe-regex -2 -- path/to/file

If you need your texts, or regexes, to be case-insensitive, add _-i_. Regexes are always processed as extended-syntax (ERE). Finally, if you want to display diffs on the fly (which can make for heavy display, be warned), add the usual _-p_ (all the more reason to filter on the specific file you’re inspecting).

## Faster branch handling with -b, -v, -vv

Alright, so first, if you didn’t know yet that _git checkout -b_ creates your new branch on the fly, you now have no excuse.

I mean, why the heck bother with:

$ git branch ticket-12  
$ git checkout ticket-12

When you can just go with:

git checkout -b ticket-12

Of course, nothing stops you from using the 2nd argument to specify the base for the new branch (which defaults to _HEAD_, as is often the case with the Git CLI).

Extra tip: _checkout_ is smart about one case where _-b_ becomes superfluous: when you want to start working on a remote branch _super-feature_ and don’t have yet a local tracking branch. You can just go:

git checkout super-feature

Git will realize there’s no such local branch, but there _is_ such a branch on the default remote, and will **automatically** do the equivalent of what follows (assuming here your default remote is called _origin_, which is common):

git checkout -b -t super-feature origin/super-feature

So why bother with a long call when you can have it short and sweet?

Let’s know talk about _-v_ and its agressive brother, _-vv_.

You’re probably used to listing your local branches with a simple _git branch_:

A simple git branch call lists your local branches

Did you know that you can get much more info (SHA, spread with any upstream, first line of the commit message) with _-v_?

git branch -v displays a lot more useful info

You could even go so far as to look up the tracked upstreams with _-vv_:

git branch -vv even looks up your tracked upstreams

Ain’t it cool? By default upstreams appear as dark blue, but this sucks on my black background, so I setup _color.branch.upstream_ to _cyan_…

## Easier help with -w

Man pages are well and good, and work everywhere, including through an SSH session… Well, almost everywhere. Several environments don’t handle _man_ very well.

Not only that, but a vast majority of users don’t know how to interact with _man_ other than scroll through it. Leveraging hyperlinks, in particular, is extremely rare.

**Git publishes all its docs** not only in the _man_ format, but also **as HTML**, a type far easier to use and known to all, links included. To use this format, just add the _-w_ option to _git help_:

git help -w reset

In a few environments (such as the official Windows installer, I believe) this is actually the **default**. If you wish to make that happen for you too, you can configure this globally:

git config --global help.format html

If the default selected browser isn’t your preferred one (this is determined by _git-web--browse_, which is aware of a shit load of them), you can force-configure it with _help.browser_, setting the proper name or command.

These HTML files are stored locally (installed by Git), so you **don’t even need an Internet access**.

## Better stashing with save and -u

**Too few people are aware of _git stash_**, and those who do know it seldom look up the doc and learn how to **use it well**. I see most people just firing up a _git stash_ first, then a simple _git stash apply_ later on.

The default stashing behavior rather blows:

1. It leaves untracked files in the working directory, which is rarely what you want;
2. it uses a braindead default message, something along the lines of “_WIP on master: <whatever the latest commit message was>”_.

Such a message is completely useless, as it **doesn’t say anything about what the work in progress (WIP) actually is**, making it difficult later on to identify what the stash was about.

To fix these to issues, all we need to do is go with the **_save_ subcommand and its _-u_ option** (which includes untracked files), and provide our custom message. For instance:

git stash save -u 'Beginning of Bootstrap 3 refactoring'

You then find yourself on a _clean tree_: what’s in HEAD, plus ignored files.

**Properly getting your stash back is just as hard**, once you’re done with whatever emergency had you stash in the first place.

Most people just do a _git stash apply_, which is too bad because in case of success (i.e. no conflict with your new base state), your stash is kept in the stash list, which could entice you later to try re-applying it. Oops.

What we need is a way, when _apply_ succeeds, to automatically _drop_. And this is exactly what _git stash pop_ does.

Although its name may suggest otherwise, it doesn’t limit itself to the latest stash: you can specify any stash (e.g. _git stash pop stash@{2}_). That being said, I believe **stashes are meant to be short-lived**, as a workaround for a complex situation or obstacle, and you should seldom have more than one.

Another gotcha is that _apply_ and _pop_, by default, do not restore the stage. It is indeed saved individually by _save_, still by default, so why not auto-restore it, as it is an important piece of information?

This is because _stash_ is a bit of a coward: if you modified a file that was in the stash’s stage, it would have to merge both and re-stage the result. Git usually auto-stages merges (after a _merge_, _rebase_ or _cherry-pick_ for instance, with or without _rerere_ assistance), but in this instance, it’ll deny it.

So, in order not to have to yell _should a merge have to happen in the stage_, it will by default not restore the stage: its snapshots will become regular local edits again.

This annoys me to no end, so **I always explicitly ask it to restore the stage**. Worst-case scenario, it will bump on something and I’ll just have to re-do the command with no stage requirement. So I always use the _--index_ option:

git stash pop --index

Unfortunately, there’s no configuration variable to automate this…

Ah, and yes, **it’s super easy to forget you stashed**: so be sure to have a solid prompt (e.g. use the ones provided by the [builtin scripts](https://github.com/git/git/blob/master/contrib/completion/git-prompt.sh) for bash/zsh) and use them with a _GIT_PS1_SHOWSTASHSTATE=1_ environment setup.

## Previous active branch: -

You probably now that in most shells, _cd -_ takes you back to the directory you were in just before the current one (so using this multiple times toggles you between two directories).

As Git versions marched on, various commands have learned this trick: _checkout_, _merge_, _cherry-pick_ and lately _rebase_. Here’s a classic sequence:

(topic)  $ git checkout master  
(master) $ git merge -

And another one:

(2-3-stable +) $ git ci -m "fix: no more _ conflict. Fixes #217."  
(2-3-stable)   $ git checkout master  
(master)       $ git cherry-pick -

If you’re running a Git version that doesn’t support the dash notation for the command you want ([verify this](https://medium.com/@porteneuve/whats-new-with-git-since-1-7-5bbec76fa394)), you can fall back on the universal syntax that dash is syntactic sugar for: _@{-1}_.

## Cancel the current merge yet preserve previous local edits

**Git doesn’t really need a clean tree** to allow a merge to go ahead: it just needs its working directory to be [_in good order_](http://git-scm.com/docs/git-merge#_pre_merge_checks), which basically means that files to be changed by the merge should not have local edits, and that you shouldn’t have an ongoing stage (to avoid a multi-topic commit eventually).

So when you find yourself wading through merge conflicts, you may have in your working directory both merge conflicts and local edits that were there before you started the merge.

If you decide to cancel the merge for whatever reasons, it is tempting to go with good ol’ _git reset --hard_. This would actually be **dangerous**, as it would destroy all local edits **that were there before the merge**, too.

This is why we have _git reset --merge_ (or its more recent syntax: _git merge --abort_, which is more in line with its _rebase_ cousin): it resets **only changes brought on by the merge**.

(master *) $ git merge cool-feature  
Auto-merging index.html  
CONFLICT (content): Merge conflict in index.html  
Automatic merge failed; fix conflicts and then commit the result.  
(master *+) $ git merge --abort  
(master *) $

You can even do this **after a successful merge**!

(master *) $ git merge cool-feature  
Auto-merging index.html  
Merge made by the `recursive` strategy.  
[afbd564] Merged `cool-feature` branch(master *) $ git reset —merge ORIG_HEAD  
[ac3489b] Original master tip  
(master *) $

Super classy, isn’t it? This spares us some stashing…

## Avoid killing a merge when rebasing it

_Rebasing_ is definitely a wonderful [Swiss-army knife](https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa), with just one little potential risk: by default, **when rebasing a merge commit, it _inlines_ the merge**. In a nutshell, you run the following risk (here illustrated by a _pull_ that rebases instead of merging, usually a great idea):

By default, rebasing inlines merge commits, which is not such a great idea…

To avoid this painful scenario, we can ask rebase to **preserve merges**, using its _-p_ option, or the longer form --_preserve-merges_. The result will be similar to what follows, although it uses a different CLI context:

You can ask rebase to preserve merges, which is usually what you want

Try to avoid combining that with commit reordering in interactive rebase though, this would [yield unexpected results](http://git-scm.com/docs/git-rebase#_bugs).

## Be a rebase ninja with -i

Talking about interactive rebase, this is indeed where rebase **really shines**, a multi-daily use case being the best-practice reflex of [cleaning up your local history before pushing it](https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa#541c), which usually goes _git rebase -i @{u}_.

Stripping zero-sum commit pairs/groups (e.g. the original and its _revert_), reordering commits, merging multiple attempts at one fix, rewriting messages, splitting up kitchen sinks… [Everything’s possible!](https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa#541c)

## Safely cleaning up with -i and -n

The _git clean_ command is very useful, but **potentially destructive**: it does impact the working directory (WD), so it could **destroy local edits you never committed**, so if you slip up, Git won’t be able to recover your work!

This is probably why, by default, _git clean_ is a no-op, as _clean.requireForce_ defaults to _true_. You thus would have to _git clean -f_ to start pumping; even then, it would leave directories alone (unless _-d_) and ignored files too (unless _-x_).

The good news is, you can **see what your clean would do without any risk**, with the traditional _-n_ (or _--dry-run_) option that many Git commands feature: it will list files and folders to be removed, but will stop at listing.

And when you do go ahead, you can **gain some confidence** by using _-i_ (the traditional _--interactive_), that will launch a sort of shell listing candidates for removal, and letting you filter them, confirm each, etc. No more anguish!

## Set the upstream on the fly with -u

So you’re pushing a branch for the first time? You’ll always need to explictly state what the remote is (even if you only have one defined), and what branch you’re pushing (even if it’s the current one), for instance _git push origin topic_.

However, this simple _push_ **does not set up tracking**: your local configuration does not remember the matching between your local _topic_ branch and its _upstream_, here the _topic_ branch on the _origin_ remote.

To remedy that, you can at any time re-_push_ with an added _-u_ (or _--set-upstream_), which will persist that configuration for you, in addition to the push proper. This way you don’t have to specify anything for future pushes and pulls.

git push -u origin topic

Internally, this relies on _git branch --set-upstream-to=origin/topic topic_, so if you just want to **set this up without pushing just yet**, do that.

As a side reminder, you don’t have to track an homonymous upstream: if names need to differ, you’ll just need to use **the full push syntax**, for instance, to connect a remote _christophe-topic_ branch to your local _topic_ branch:

git push -u origin topic:christophe-topic

This is why the **remote branch deletion syntax** is as follows:

git push origin :old-remote-branch

You’re essentially saying “replace the remote branch _old-remote-branch_ with nothing at all” so… delete it.

## Ah well, 36 in the end

Yeah, that’s 36 options (not counting long form variations), and 4 configuration variables, and one environment setting. I can’t help it, I give, I give…

## Want to learn more?

I wrote [a number of Git articles](https://medium.com/@porteneuve), and you might be particularly interested in the following ones:

- [Our GitHub video series is out!](https://medium.com/@porteneuve/our-github-video-course-series-is-out-1fe829e04a59) (absolutely kick-ass, even for experts)
- [Getting solid at Git merge vs. rebase](https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa) (a must-read!)
- [Fix conflicts only once with git rerere](https://medium.com/@porteneuve/fix-conflicts-only-once-with-git-rerere-7d116b2cec67) (why do it twice?)
- [How to make Git preserve specific files while merging](https://medium.com/@porteneuve/how-to-make-git-preserve-specific-files-while-merging-18c92343826b) (sweet trick!)
- [Mastering Git subtrees](https://medium.com/p/mastering-git-subtrees-943d29a798ec) (because submodules, I mean, yuck!)

Also, if you enjoyed this post, say so: [upvote it on HN](https://news.ycombinator.com/item?id=9075360)! Thanks a bunch!

Although we don’t publicize it much for now, we do offer English-language Git training across Europe, based on our battle-tested, celebrated [Total Git](http://www.git-attitude.fr/total-git/) training course. If you fancy one, just [let us know](http://www.git-attitude.fr/request-intra-or-custom)!

_(We can absolutely come over to US/Canada or anywhere else in the world, but considering you’ll incur our travelling costs, despite us being super-reasonably priced, it’s likely you’ll find a more cost-effective deal using a closer provider, be it GitHub or someone else. Still, if you want_ **_us_**_, follow the link above and let’s talk!)_