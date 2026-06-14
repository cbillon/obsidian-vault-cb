---
link: https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/
byline: Tyler Cipriani
excerpt: |-
  the short of it is: they’re cool for appending notes from automated
  systems (like ticket or build systems) but not really for having
  interactive conversations with other developers (at least not yet)
slurped: 2026-06-01T08:29
title: "Git Notes: git's coolest, most unloved­ feature"
tags:
  - git
---

> the short of it is: they’re cool for appending notes from automated systems (like ticket or build systems) but not really for having interactive conversations with other developers (at least not yet)
> 
> – Scott Chacon, [GitHub.blog](https://github.blog/2010-08-25-git-notes-display/), Aug. 2010

Git notes are almost a secret.

They’re buried by their own distressing usability.

But git notes are continually rediscovered by engineers trying to stash metadata inside git.

![Sun, 30 Oct 2022 11:05 @simonw](https://photos.tylercipriani.com/2022-10-30_simonw-git-notes.png)

**Git notes are powerful tools.** And they could solve so many problems—if only they were better known and easier to use.

## 🧐 What are git notes?

A common use of git notes is tacking metadata onto commits.

Once a commit cements itself in git’s history—that’s it. It’s impossible to amend a commit message buried deep in a repo’s log[1](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/#fn1).

But git notes enable you to amend new information about old commits in a special namespace. And they’re capable of so much more.

**Notes stow metadata about anything tracked by git**—any object: commits, blobs, and trees. All without futzing with the object itself.

You add notes to the latest commit in a repo like this:

```
git notes add -m 'Acked-by: <tyler@tylercipriani.com>'
```

And then it shows up in `git log`:

```
commit 1ef8b30ab7fc218ccc85c9a6411b1d2dd2925a16
Author: Tyler Cipriani <thcipriani@gmail.com>
Date:   Thu Nov 17 16:51:43 2022 -0700

    Initial commit

    Notes:
        Acked-by: <tyler@tylercipriani.com>
```

## 🥾 Git notes in the wild

The git project itself offers an example of git notes in the wild. They link each commit to its discussion on their mailing list.

For example:

```
commit 00f09d0e4b1826ee0519ea64e919515032966450
Author: <redacted>
Date:   Thu Jan 28 02:05:55 2010 +0100

    bash: support 'git notes' and its subcommands
    ...

Notes (amlog):
    Message-Id: <1264640755-22447-1-git-send-email-szeder@ira.uka.de>
```

This commit’s notes point intrepid users to the [thread where this patch was discussed](https://lore.kernel.org/git/1264640755-22447-1-git-send-email-szeder@ira.uka.de/).

Other folks are using notes for things like:

- Tracking time spent per commit or branch
- Adding review and testing information to git log
- And even fully distributed code review

## 📦 Storing code reviews and test results in git notes

Here is a plea for all forges: make code review metadata available offline, inside git.

The [reviewnotes](https://gerrit.googlesource.com/plugins/reviewnotes/+/refs/heads/master/src/main/resources/Documentation/refs-notes-review.md) plugin for Gerrit[2](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/#fn2) is an example of how to do this well. It makes it easy to see who reviewed code in git log:

```
git fetch origin refs/notes/review:refs/notes/review
git log --show-notes=review
```

The command above shows me all the standard git log info alongside information about what tests ran and who reviewed the code. All without forcing me into my browser.

```
commit d1d17908d2a97f057887a4afbd99f6c40be56849
Author: User <user@example.com>
Date:   Sun Mar 27 18:10:51 2022 +0200

    Change the thing

Notes (review):
    Verified+1: SonarQube Bot
    Verified+2: jenkins-bot
    Code-Review+2: Reviewer Human <reviewerhuman@wikimedia.org>
    Submitted-by: jenkins-bot
    Submitted-at: Tue, 14 Jun 2022 21:59:58 +0000
    Reviewed-on: https://gerrit.wikimedia.org/r/c/mediawiki/core/+/774005
    Project: mediawiki/core
    Branch: refs/heads/master
```

## 💠 Distributed code review inside git notes

Motivated hackers can knead and extend git notes. Using them as distributed storage for any madcap idea.

Someone at Google cobbled together a full-on code review system teetering atop git notes called [git-appraise](https://github.com/google/git-appraise).

Its authors have declared it a “fully distributed code review”—independent of GitHub, GitLab, or any other code forge.

This system lets you:

- Request review of a change
- Comment on a change
- Review and merge a change

And you can do all this from your local computer, even if GitHub is down.

Plus, it’s equipped with an affectedly unaesthetic web interface, if that’s your thing.

![The git-appraise web interface, in all its NaN-line-numbering glory.](https://photos.tylercipriani.com/2022-11-27_git-appraise-web.png)

## 😭 No one uses git notes

Git notes are a pain to use.

And GitHub [opted to stop displaying commit notes in 2014](https://github.blog/2010-08-25-git-notes-display/) without much explanation.

For commits, you can make viewing and adding notes easier using fancy options in your gitconfig[3](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/#fn3). But for storing notes about blobs or trees? Forget it. You’d need to be comfortable rooting around in git’s [plumbing](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain) first.

So, for now: **git notes are relegated to obscurity**. Forever hamstrung by an obscure and clunky interface and limited adoption—I often forget they’re there.

## 🗽 Forge independence

Git is a distributed code review system. But much of the value of git repos ends up locked into forges, like GitHub.

Git notes are a path toward an alternative.

Git distributes the history of a piece of code. **Git notes could make it possible to distribute the history of an entire project.**

---

1. Without having to endure the [perils of a force push](https://groups.google.com/g/jenkinsci-dev/c/-myjRIPcVwU/m/mrwn8VkyXagJ), anyway.[↩︎](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/#fnref1)
    
2. The code review system used for [a](https://gerrit.wikimedia.org/r/) [couple](https://go-review.googlesource.com/) of [bigish](https://android-review.googlesource.com/) [projects](https://chromium-review.googlesource.com/).[↩︎](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/#fnref2)
    
3. Noteably by automagically fetching notes and displaying them in `git log` via:
    
    ```
    $ git config --add \
    remote.origin.fetch \
    '+refs/notes/*:refs/notes/*'
    $ git config \
    notes.displayRef \
    'refs/notes/*'
    ```
    
    [↩︎](https://tylercipriani.com/blog/2022/11/19/git-notes-gits-coolest-most-unloved-feature/#fnref3)