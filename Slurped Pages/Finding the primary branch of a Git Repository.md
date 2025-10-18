---
link: https://amitmerchant.com/primary-branch-git/
byline: Amit Merchant
site: Amit Merchant - A blog on PHP, JavaScript, and more
date: 2023-10-20T02:00
excerpt: If you’re using Git, you might have noticed that the primary/default branch of a Git repository is called master. But, since the word master has a negative connotation, many organizations have started to use different names for the default branch.
slurped: 2025-10-13T10:47
title: Finding the primary branch of a Git Repository
tags:
  - git
---

If you’re using Git, you might have noticed that the primary/default branch of a Git repository is called `master`. But, since the word `master` has a negative connotation, many organizations have started to use different names for the default branch.

For instance, GitHub has renamed the default branch from `master` to `main` and GitLab has renamed it to `default`.

And if you’re working with multiple Git repositories, it can be hard to remember the default branch name of each repository. So, how do you find out the default branch of a Git repository?

Well, Liam Hammett has [recently shared](https://twitter.com/LiamHammett/status/1715077724904071277) a neat Git command that will let you do just that.

```
git symbolic-ref refs/remotes/origin/HEAD | cut -d'/' -f4
```

As you can tell, the `git symbolic-ref` command will return the symbolic reference of the `HEAD` of the remote `origin` and the `cut` command will cut the output of the `git symbolic-ref` command and return the 4th field which is the name of the default branch.

Now, if you want to make this command more accessible, you can create a [Git alias](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases) for it like so.

```
git config --global alias.default-branch '!git symbolic-ref refs/remotes/origin/HEAD | cut -d'/' -f4'
```

And now, you can use this alias like so.