---
link: https://bhoot.dev/2024/memorising-git-rebase/
byline: |-
  Jayesh Bhoot's Ghost
    Town
excerpt: In a git rebase x command, is x being rebased on the current branch, or vice versa?
slurped: 2025-09-01T12:14
title: Memorising branch order in git rebase
tags:
  - git
  - rebase
---

I mentally always read _rebase_ as _rebase on top of_.

So, `git rebase master` reads as **_git, rebase on top of master_**, i.e., rebase the current branch on top of master.

But what about the alternate version of the command: `git rebase master feat/x` ?

Well, git always rebases the **current** branch.

So `git rebase master feat/x` is actually `git checkout feat/x && git rebase master` . So the reading _git rebase on top of master_ still applies.

A bit clunky, but good enough for me.