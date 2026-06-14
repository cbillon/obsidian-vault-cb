---
link: https://tylercipriani.com/blog/2020/10/05/git-squash-is-not-nice-history/
byline: Tyler Cipriani
excerpt: |-
  Today I read a brilliant article about effective use of git
  bisect, but I disagreed with a small nuance of one of its
  conclusions (and, by internet law, was honor bound to write a blog post
  about it):
slurped: 2026-06-01T08:53
title: Git squash is not nice history
tags:
  - git
---

Today I read a brilliant article about effective use of [git bisect](https://blog.ploeh.dk/2020/10/05/fortunately-i-dont-squash-my-commits/), but I disagreed with a small nuance of one of its conclusions (and, by internet law, was honor bound to write a blog post about it):

> Had this happened in a code base with a ‘nice history’ (as the squash proponents like to present it), that small commit would have been bundled with various other commits. The problem wouldn’t have jumped at me if buried in dozens of other changes.

It’s true that `git merge --squash` obscures history; whether or not this makes a nice history is entirely dependent on the situation.

First we need to agree on background and introduce terms.

## Lossless merges

Let’s say you have a git history that looks like this:

    `C - D         feature/magic      / A - B               main`
    

A standard `git merge feature/magic` issued on the `main` branch results in this history:

    `C - D         feature/magic      / A - B - C - D       main`
    

This is a fast-forward merge. Since the `main` ref is at `B` and `B` is the parent of `C` when we merge `feature/magic` into `main`, `main`’s ref is updated to point at the commit at `D`.

There is no loss of fidelity from the point of development. Every development commit is kept and the relationships between commits maintained.

## Lossy merges

Using `--squash` instead of the default merge strategy is lossy: the fidelity of git history is lost. Squash, in our example, results in a new commit being added to `main`’s history that is an amalgam of the commits on the `feature/magic` branch:

    `C - D        feature/magic      / A - B - - - CD'    main`
    

You can no longer see that `C` and `D` were two separate commits.

## Helpful Loss

There are reasons to choose a lossy merge over a lossless merge.

There are blogs that advocate heavily for a [squash workflow](https://blog.dnsimple.com/2019/01/two-years-of-squash-merge/). Which strategy to choose is dependent on the content of the commits you are merging. The strategy chosen should maintain the principal that a commit in a mainline branch’s history should make sense on its own.

In the above example, the content of the feature branch isn’t shown. A new example might be a `feature/lossless` branch that contains a refactor and a new feature that depends on that refactor:

    `| * (feature/lossless) feature: method is dynamic | * refactor: method instead of global |/ * (main) Initial Commit`
    

This is an example where the desired outcome is lossless: both of these commits are meaningful on their own and can be vectors for bugs. After the merge, in an ideal case, the main branch should look like:

    `* (main, feature/lossless) feature: method is dynamic * refactor: method instead of global * Initial Commit`
    

Now, imagine a different branch – a bugfix, with only a single commit that is up for review.

    `| * (bugfix/lossy) bugfix: validate user input |/ * (main) Initial Commit`
    

During review, there’s a typo in a comment that needs fixing. Now the branch graph looks like:

    `| * (bugfix/lossy) fix comment typo | * bugfix: validate user input |/ * (main) Initial Commit`
    

I would argue that the typo commit doesn’t make sense on its own. There’s no need to persist that commit into the main branch: it’s noise. It’s not a vector for meaningful error, it’s a development detail that shouldn’t leak back into the main branch. In short, a more functional history for a merge would be to use a lossy strategy:

    `* (main, bugfix/lossy) bugfix: validate user input * Initial Commit`
    

## This is all an oversimplification

Of course, the example above completely ignores merge commits, repository merge strategies, and any shared agreements about the state of feature or mainline branches, code review, testing strategies, deployment pipelines, and so so (so!) much more!

Many of the blog posts on git I read make broad generalizations about the Right™ way to use some particularly controversial features of git (pull, merge, rebase, branching, commit messages …wait. 🤔 Is every feature controversial?), but the reality is that there is a lot of nuance in the world and the only right answer depends on your situation.