---
link: https://tylercipriani.com/blog/2022/09/30/githubs-missing-merge-option/
byline: Tyler Cipriani
excerpt: |-
  github is a perfectly fine hosting site, and it does a number of
  other things well too, but merges is not one of those things.
slurped: 2026-06-01T08:31
title: GitHub's missing merge option - Tyler Cipriani
tags:
  - git
---

> github is a perfectly fine hosting site, and it does a number of other things well too, but merges is not one of those things.
> 
> – [Linus Torvalds](https://lore.kernel.org/lkml/CAHk-=wjbtip559HcMG9VQLGPmkurh5Kc50y5BceL8Q8=aL0H3Q@mail.gmail.com/)

Git possesses parts of a decent [software forge](https://en.wikipedia.org/wiki/Forge_\(software\)).

When git was developed by the Linux kernel community, they already had bug tracking, documentation, and a mailing list, so (unlike [fossil](https://fossil-scm.org/home/doc/trunk/www/fossil-v-git.wiki)) git has none of those things.

Enter GitHub. It uses “issues” for bug tracking and discussion, and its code browser is unrivaled.

But for all of its features, GitHub implements only a subset of git. For instance, GitHub lacks the default merge strategy of git—the fast-forward merge.

And after some pondering I realized there’s a good reason for that: it’s a cop-out.

## `git log` can be clean or accurate, not both

> I want clean history, but that really means (a) clean and (b) history.
> 
> – [Linus Torvalds](https://www.mail-archive.com/dri-devel@lists.sourceforge.net/msg39091.html)

Git log will always suck for someone.

An eternal war rages between team “git log should be clean” vs. team “git log should have an accurate history.”

- 📚 **Team History**
    
    - Method: `git merge --no-ff`
    - 🟢 Pros: A complete history of how everything was developed
    - 🔴 Cons: You’ve opened a pandoras box of strange git situations. And your git log looks like this now[1](https://tylercipriani.com/blog/2022/09/30/githubs-missing-merge-option/#fn1):
    
    ```
    * (refs/heads/B)
    * * Merge 'C' into 'B'
    * |\
    * | | * (refs/heads/C -- git revert B8)
    * | | * Merge 'B' into 'C'
    * | |/|
    * | |/
    * |/|
    * * | B8
    * | * C3
    * |/
    * * A (refs/heads/A)
    ```
    

![Team history uses git merge --no-ff to ensure a merge commit is always created](https://photos.tylercipriani.com/2022-09-30_git-merge.png)

- ✨ **Team Clean**
    - Methods: `git merge --ff-only` or `git rebase && git merge` (extreme clean freaks add the `--squash` option)
    - 🟢 Pros: Linear history, `git log` is easy to read, `git revert` requires no thought.
    - 🔴 Cons: You’re erasing history—you can no longer tell if two commits were written together on a single feature branch.

![Team clean uses git rebase && git merge --ff-only to make it appear there was never a feature branch at all](https://photos.tylercipriani.com/2022-09-30_git-rebase.png)

## Why is `git merge` bad?

`git merge` opted out.

If a branch can be fast-forwarded, `git merge` sticks the commits on the end of the branch and never tells you there was a merge—team clean.

But if a branch has conflicts, you’ll need to fix them and create a merge commit to say what you did—team history.

Sometimes there’s a merge commit; sometimes not: Madness.

## What does GitHub do?

When you mash “merge” in GitHub it never executes plain `git merge`.

![The GitHub merge options (the command line instructions are a lie)](https://photos.tylercipriani.com/2022-09-30_git-merge-button.png)

And sussing out what git command it will run is kafkaesque. I spent some time mapping all the checkboxes and merge strategies into something you could type into bash.

|command|GitHub|Alignment|
|---|---|---|
|`git merge`|**not implemented**|`¯\_(ツ)_/¯`|
|`git merge --ff-only`|**not implemented**|✨ Team Clean|
|`git rebase && git merge --ff-only`|Rebase and Merge|✨ Team Clean|
|`git merge --no-ff`|Create a merge commit|📚 Team History|
|`git merge --squash --ff-only BRANCH`|Squash and merge|✨ Team Clean|
|`git merge --is-ancestor && git merge --no-ff`|Create a merge commit + [Require linear history](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches#require-linear-history)|✨ Team Clean[2](https://tylercipriani.com/blog/2022/09/30/githubs-missing-merge-option/#fn2)|

There is a distinction between `git rebase && git merge --ff-only` and `git merge --ff-only`. Rebasing modifies the commit—you end up with a different SHA1.

By not using the “merge if necessary” strategy of `git merge`, GitHub forces you to choose a side in the eternal war. And that’s a good thing.

---

1. This is a contrived example of a “[criss-cross merge](https://gist.github.com/thcipriani/0749be2a13815f398550a2865197dc49)”[↩︎](https://tylercipriani.com/blog/2022/09/30/githubs-missing-merge-option/#fnref1)
    
2. In GitLab this is “Merge Commit with Semi-linear history” which seems like a nicer UI vs the buried option to “Require linear history”. This option mitigates some of the pain of an ugly git log.[↩︎](https://tylercipriani.com/blog/2022/09/30/githubs-missing-merge-option/#fnref2)