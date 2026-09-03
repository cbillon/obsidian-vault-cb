---
link: https://developer.upsun.com/posts/insights/the-only-correct-way-to-git-pull-and-merge
site: Upsun Developer
excerpt: A strongly opinionated guide to git pull and merge strategies that will
  make your commit history actually useful. Learn when to rebase, when to merge,
  and why the defaults are lying to you.
slurped: 2026-08-25T09:25
title: The only correct way to git pull and merge - Upsun Developer
---

If your git history is full of merge commits that don’t seem to mean anything, you’re in good company. Git’s default behavior is actively working against you. The good news? A couple of configuration changes will fix it.

## The problem with `git pull`

Here’s what happens when you run `git pull` with the default settings. You’re working on a branch with your teammates. You’ve got a few commits. Everything’s fine. Then you think, “I should grab the latest changes.” Reasonable! So you pull. And git creates a merge commit. You do this three times during the day because you want to stay up to date. Now you have three merge commits that say absolutely nothing except “I, a developer, updated my local copy of the repository.” Thanks for that vital historical information, git.

These commits don’t tell you anything about how the code evolved. They don’t show meaningful changes. They’re noise. Your git history now reads like a diary of your sync habits rather than a story of your codebase.

## What GitLab figured out

Here’s the thing: GitLab doesn’t even offer you the option to create a merge commit when updating your branch. When your merge request is behind the target branch, you get a “Rebase” button. That’s it. No merge commit option. ![GitLab's Rebase button - no merge commit option in sight](https://mintcdn.com/upsun-c9761871/eXV27YLeu5lESGxN/images/posts/insights/the-only-correct-way-to-git-pull-and-merge/gitlab-rebase-button.webp?fit=max&auto=format&n=eXV27YLeu5lESGxN&q=85&s=2c96da34a6d43665d758ba01deed997f) GitLab understood that there’s no value in a merge commit that only says “I got the latest code.” The history should show what actually happened to the code, not your personal workflow habits.

No merge commits. Your commits sit on top of the latest main. The history tells a clean story.

## When merge commits actually matter

Merge commits aren’t evil. They’re valuable when you’re merging a branch into main. That’s the moment when you want to show: “All of these commits were developed together, and they’re landing in main as a unit.” The merge commit groups related work. When you’re debugging six months from now and you find a problematic line, you can trace it back to its merge commit and see the full context. That refactoring commit? It was part of a larger feature. That feature? Here’s every commit that went into it, all grouped together.

This is useful information. “I pulled at 2:47 PM” is not.

## The fast-forward trap

When you merge without the `--no-ff` flag, git will fast-forward if it can. Your branch commits get added directly on top of main, and it looks like they were developed there all along. This erases context. You lose the information about which commits belonged together. The history looks linear, but it’s a lie. Those commits were developed on a branch, tested together, and merged as a unit. That information matters when you’re trying to understand why code exists.

Use `--no-ff` when merging to main. Always.

## The squash merge problem

Squash merging takes all the commits from your feature branch and combines them into a single commit on main. It sounds tidy. It’s actually throwing away information.

When you squash, you lose the story of how the feature was built. That refactoring commit that seemed unrelated? It was preparing for the next step. That bug fix at the end? It reveals an edge case you might hit again. The individual commits show the developer’s reasoning, the problems they encountered, the decisions they made. Six months from now, when you’re debugging and you find that the auth service is doing something unexpected, you won’t be able to trace back through the thought process. You’ll just see one big commit that says “Add user authentication” and wonder why things are the way they are.

## The configuration you actually want

Here’s how to fix your git defaults:

Now `git pull` rebases by default instead of creating pointless merge commits. Your history stays clean. Your teammates will thank you. Or at least they won’t curse your name when reviewing the commit log. For merges, get in the habit:

Or configure your merge strategy in your repository settings if your git hosting provider supports it.

## The summary

1. **Configure git to rebase on pull**: `git config --global pull.rebase true`
2. **Use `--no-ff` when merging branches**: Preserve the context of grouped work
3. **Avoid squash merges**: Keep the development history intact

## A note on opinions

This is an opinionated piece, and reasonable people disagree on these things. Some teams love squash merges because they keep the main branch looking clean. Some developers prefer merge commits on pull because they mark a clear point in time. There are arguments for all of these approaches, and your team might have good reasons for doing things differently. What matters most is that your team agrees on a workflow and sticks to it consistently. A consistent approach beats the “right” approach applied inconsistently. That said, if you’re starting fresh or reconsidering your workflow, the approach outlined here optimizes for one thing: making your git history useful when you need to understand why code exists. Your future self, debugging at 11 PM, will appreciate a history that tells a story rather than a logbook of sync operations or a series of opaque squashed commits.