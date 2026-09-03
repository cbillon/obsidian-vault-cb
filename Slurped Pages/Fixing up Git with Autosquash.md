---
link: https://blog.gitbutler.com/git-autosquash
byline: Scott Chacon
site: GitButler
date: 2024-03-11T15:16
updated: 2024-03-11T15:16
excerpt: How can you amend a series of commits automatically to keep a clean
  history? Let's take a look at git autosquash.
twitter: https://twitter.com/@gitbutler
tags:
  - slurp/git
  - slurp/Git
  - slurp/GitButler
  - slurp/Version-Control
  - slurp/Software-Development
  - slurp/Programming
  - slurp/Developer-Tools
slurped: 2026-08-24T07:42
title: Fixing up Git with Autosquash
---

This year I've been giving talks all over the place, pointing out some cool features of Git that I've found aren't super well known and writing up some of these features here. Previously I've written about [worktrees](https://blog.gitbutler.com/git-worktrees/), [large repository](https://blog.gitbutler.com/git-tips-3-really-large-repositories/) features and [a few other things](https://blog.gitbutler.com/git-tips-and-tricks/).

Today I want to show you a really handy way to incorporate fixups into a series of commits using `git commit --fixup` and `git rebase --autosquash`.

But first, let's take a look at the problem that this is designed to solve.

As you may or may not know, Git actually doesn't store patches or changesets, it stores snapshots. However, lots of Git commands can convert a list of them into the differences between each one, which makes it seem like each commit is a patch.

What makes things a little more confusing is that the commit message generally describes this difference, rather than the actual data that it points to.

This is more or less what Git was originally written to do, keep a series of snapshots of the Linux kernel and easily convert them into a series of patches that can easily be e-mailed to a mailing list and re-applied locally.

So this is the generally intended original workflow of a Git user. You make a change and document it. You make another change and document it. Then you use [`git send-email`](https://git-scm.com/docs/git-send-email) to mail those changes to the mailing list for feedback.

Today this is often done as a branch with a Pull Request on GitHub, but the idea is the same: you have commits that describe your changes that people can inspect in isolation.

If you're using GitHub and PRs to collaborate on changes, it's possible that you think of the atomic unit of work as the branch rather than the commit for this reason. That the PR description is actually what's most important, not individual commit messages, largely because GitHub makes the PR reviewable and not individual commits. Whereas in mailing lists, the individual commits are quite important for review purposes.

In a mailing list, one normally sends a patch series, gets feedback on each patch and then you need to re-roll the series and re-send it. This article is about trying to do that in an easier way. Maintain a nice series of commits as "patches" and incorporate feedback into that series, rather than just pushing new commits on top of the branch.

Ok, so let's imagine a scenario. Let's say that you have two commits that implement some feature and document it.

```
❯ git log --oneline sc-feature-A ^origin/master
18e9c6b997a (sc-feature-A) add documentation for new feature
e57c40da089 add new feature
```

We can also see that each change has modified some files (with `--stat`):

```
❯ git log --oneline --stat sc-feature-A ^origin/master
18e9c6b997a (sc-feature-A) add documentation for new feature
 README.md | 1 +
 1 file changed, 1 insertion(+)
e57c40da089 add new feature
 gitbutler-app/src/virtual_branches/errors.rs              | 1 +
 gitbutler-app/src/virtual_branches/files.rs               | 1 +
 gitbutler-ui/src/lib/components/Differ/CodeHighlighter.ts | 2 ++
 3 files changed, 4 insertions(+)
```

You send your patch series to a mailing list (or push them to a Pull Request).

Now you have two commits that are viewed as patches and commented on. The feedback is valuable and you want to incorporate changes to address the feedback into your commits.

Here is where things become a little difficult. You want to still have _two_ commits with the same good descriptive commit messages, but you need to incorporate changes from your feedback into different files in each commit.

One way to do this is to interactively rebase, stopping at each commit, incorporating your changes destined for that commit, and then continuing. This is a little difficult because you can't really make the changes to make sure they work and _then_ do this very easily.

Or, you could make the changes, commit it into temporary commits, then rebase interactively, rearrange the commits and squash them together.

We make our temporary commits:

```
❯ # fix the documenation
❯ git commit -am 'feedback on documentation'

❯ # fix the feature
❯ git commit -am 'feedback on feature'

❯ git log --oneline sc-feature-A ^origin/master
d7c50392f51 (sc-feature-A) feedback on feature
c679989f4a7 feedback on documentation
18e9c6b997a add documentation for new feature
e57c40da089 add new feature
```

Then we interactively rebase and rearrange the pick lines:

```
❯ git rebase -i origin/master

# REBASE FILE:

---
pick e57c40da08 add new feature
pick 18e9c6b997 add documentation for new feature
pick c679989f4a feedback on documentation
pick d7c50392f5 feedback on feature
---

# BECOMES:

---
pick e57c40da08 add new feature
squash d7c50392f5 feedback on feature
pick 18e9c6b997 add documentation for new feature
squash c679989f4a feedback on documentation
---
```

This rebase should work without issues (here I also changed the commit message to add `w / feedback`, just to be clear that it's incorporated the changes)

```
❯ git rebase -i origin/master
[detached HEAD 3224c58f81] add new feature, w/feedback
Date: Mon Mar 11 11:17:21 2024 +0100
 3 files changed, 5 insertions(+)
[detached HEAD d0a90ff020] add documentation, w/ feedback
 Date: Mon Mar 11 11:17:30 2024 +0100
 1 file changed, 2 insertions(+)
Successfully rebased and updated refs/heads/sc-feature-A.
```

And viola, we have two commits again, this time with the feedback incorporated.

```
❯ git log --oneline sc-feature-A ^origin/master
eb4262e9e46 (sc-feature-A) add documentation, w/ feedback
3224c58f81c add new feature, w/feedback
```

Now we can re-send the series to the mailing list, or force-push to our branch.

However, this is common enough of an issue, that Git has a clever way to do this a little more automatically. Instead of committing something that lets _us_ know that we fixed up a previous commit, we can let _Git_ know that a commit is a fixup.

With, you may have guessed it, the `--fixup` flag. So let's say we get _more_ feedback on our series and need to fixup both commits again for a third round of the series. This time, we'll use the `--fixup` flag:

```
❯ # do more work on documentation
❯ git commit -a --fixup=eb4262e9e46
[sc-feature-A f77a3f5909] fixup! add documentation, w/ feedback
 1 file changed, 1 insertion(+)

❯ # do more work on the feature

❯ git commit -a --fixup=3224c58f81c
[sc-feature-A 6149ef8966] fixup! add new feature, w/feedback
 1 file changed, 1 insertion(+)

❯ git log --oneline sc-feature-A ^origin/master
6149ef89666 (sc-feature-A) fixup! add new feature, w/feedback
f77a3f5909a fixup! add documentation, w/ feedback
eb4262e9e46 add documentation, w/ feedback
3224c58f81c add new feature, w/feedback
```

It's not super complicated, but in the case of committing with a `--fixup=[sha]` flag, Git will write a specially formatted commit message that it can recognize to mean "this commit fixes up this other commit".

Now, instead of manually interactively rebasing and rearranging the pick/squash commands, you can simply run `git rebase --autosquash`:

```
❯ git rebase --autosquash origin/master
Successfully rebased and updated refs/heads/sc-feature-A.

❯ git log --oneline sc-feature-A ^origin/master
0273a58c32e (sc-feature-A) add documentation, w/ feedback
7c102015437 add new feature, w/feedback
```

It basically does exactly what we just did, but automatically - no editor is opened, no instructions manually moved around.

💡

The `--fixup` option also takes `amend` and `reword` options, to handle rewording the commit message as well. For example, `git commit --fixup:amend=[sha]`. Check out the Git [commit docs](https://git-scm.com/docs/git-commit#Documentation/git-commit.txt---fixupamendrewordltcommitgt) for more options.

If you're trying to maintain a clean patch series as a continuously updated set of commits, `--fixup` and `--autosquash` may make things a little simpler. Enjoy your squashing!