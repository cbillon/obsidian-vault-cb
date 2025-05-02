---
link: https://blog.sffc.xyz/post/185195398930/why-you-should-use-git-pull-ff-only
site: sffc's Tech Blog
date: 2019-05-28T09:13
excerpt: |-
  Git is a simple, elegant version control system.  Like many great things, if you are new to Git, it takes trial and error before you can come to appreciate its power and simplicity.



   Having introduced Git to many dozens of people, first as a teaching assistant and now as a professional software engineer, I have seen many of the mistakes that Git novices commonly make.  Often, there is one root cause: Git trying to be helpful and performing operations on your repository that you did not intend.



   In this blog post, I will advocate for using the --ff-only option for git pull, and furthermore setting it as the default option for git pull.  If you get nothing else out of this blog post, it should be that running the following simple command will save you many headaches as a novice Git user.



  $ git config --global pull.ff only




   



   Why the default Git Pull is a Problem



   The description of git pull in git pull --help looks rather harmless.




    
   Incorporates changes from a remote repository into the current branch.





   However, what the command actually does is explained in the next sentence.




    
   In its default mode, git pull is shorthand for git fetch followed by git merge FETCH_HEAD.





   There’s the problem: git pull performs a merge, which often creates a merge commit.  Therefore, by default, pulling from the remote is not a harmless operation: it can creates a new commit sha that didn’t exist before!  I find that this behavior confuses Git novices, because what feels like it should be a harmless download operation actually changes the commit history in unpredictable ways.



   Consider the following situation.  You have committed work on your local branch, and someone else made a new commit that is only on remote.



   



   If you run git pull or git pull origin master, you end up with this:



   



   Another problem is when you are on a different branch.  If you have my-branch checked out, and you run git pull origin master, this does not simply update your local master.  Instead, Git will happily merge origin/master into your branch!



   How Git Pull –ff-only Works



   Fortunately, Git gives us an option that prevents the accidental creation of new shas in your repository.  With git pull --ff-only, Git will update your branch only if it can be “fast-forwarded” without creating new commits.  If this can’t be done (if local and remote have diverged), git pull --ff-only simply aborts with an error message:



  $ git pull --ff-only upstream master
  # ...
  fatal: Not possible to fast-forward, aborting.




   When this happens, inspect why the branches diverged.  Sometimes, you may discover that you made a mistake, like trying to pull master into a local branch.  In cases where you really did intend to create a merge commit, you can now follow with git merge.  By doing this, you perform the “downloading” and the “committing” in two different steps, which helps mentally separate these two operations.



   Why use --ff-only over --rebase



   Another popular setup is to use the --rebase option for git pull instead of --ff-only.  Visually, if local and remote had diverged, git pull --rebase gives you something like this:



   



   Many people consider this a clean outcome.  Although the outcome is cleaner than the default, I still recommend --ff-only.  --rebase can permanently mutate the history of your current branch, and often it does it without asking.



   Conclusion



   By using --ff-only, you can be confident that no matter what the state is of your local repository, you will not change the version history.  Separating the download from the commit creation makes Git easier to swallow.



   If you are convinced, configure your Git client to always use --ff-only by default, so you get this behavior even if you forget the command-line flag!



  $ git config --global pull.ff only






   If you learned something from this post, please follow me on Twitter and/or post a comment below.  Thanks!
tags:
  - git
  - git-pull
slurped: 2024-08-07T20:11
title: Why You Should Use git pull --ff-only
---

[Git](https://git-scm.com/) is a simple, elegant version control system. Like many great things, if you are new to Git, it takes trial and error before you can come to appreciate its power and simplicity.

Having introduced Git to many dozens of people, first as a teaching assistant and now as a professional software engineer, I have seen many of the mistakes that Git novices commonly make. Often, there is one root cause: Git trying to be helpful and performing operations on your repository that you did not intend.

In this blog post, I will advocate for using the `--ff-only` option for `git pull`, and furthermore setting it as the default option for `git pull`. If you get nothing else out of this blog post, it should be that running the following simple command will save you many headaches as a novice Git user.

```
$ git config --global pull.ff only
```

## Why the default Git Pull is a Problem

The description of `git pull` in `git pull --help` looks rather harmless.

> Incorporates changes from a remote repository into the current branch.

However, what the command actually does is explained in the next sentence.

> In its default mode, git pull is shorthand for git fetch followed by git merge FETCH_HEAD.

There’s the problem: `git pull` performs a _merge_, which often creates a merge commit. Therefore, by default, pulling from the remote is _not_ a harmless operation: it can creates a new commit sha that didn’t exist before! I find that this behavior confuses Git novices, because what feels like it should be a harmless download operation actually changes the commit history in unpredictable ways.

Consider the following situation. You have committed work on your local branch, and someone else made a new commit that is only on remote.

![](https://64.media.tumblr.com/9fb0685ca9c513197d3be0f4fd186e12/tumblr_inline_ps7dd2ta811wthf4f_540.png)

If you run `git pull` or `git pull origin master`, you end up with this:

![](https://64.media.tumblr.com/eea76d484de575ea94c8203cbba4b80b/tumblr_inline_ps7ddfddO01wthf4f_540.png)

Another problem is when you are on a different branch. If you have `my-branch` checked out, and you run `git pull origin master`, this does not simply update your local master. Instead, Git will happily merge origin/master into your branch!

## How Git Pull –ff-only Works

Fortunately, Git gives us an option that prevents the accidental creation of new shas in your repository. With `git pull --ff-only`, Git will update your branch only if it can be “fast-forwarded” without creating new commits. If this can’t be done (if local and remote have diverged), `git pull --ff-only` simply aborts with an error message:

```
$ git pull --ff-only upstream master
# ...
fatal: Not possible to fast-forward, aborting.
```

When this happens, inspect why the branches diverged. Sometimes, you may discover that you made a mistake, like trying to pull master into a local branch. In cases where you really did intend to create a merge commit, you can now follow with `git merge`. By doing this, you perform the “downloading” and the “committing” in two different steps, which helps mentally separate these two operations.

## Why use `--ff-only` over `--rebase`

Another popular setup is to use the `--rebase` option for git pull instead of `--ff-only`. Visually, if local and remote had diverged, `git pull --rebase` gives you something like this:

![](https://64.media.tumblr.com/d7d28b11ee8169beabc0f862236e8c38/tumblr_inline_ps7ddqN5jG1wthf4f_540.png)

Many people consider this a clean outcome. Although the outcome is cleaner than the default, I still recommend `--ff-only`. `--rebase` can permanently mutate the history of your current branch, and often it does it without asking.

## Conclusion

By using `--ff-only`, you can be confident that no matter what the state is of your local repository, you will not change the version history. Separating the download from the commit creation makes Git easier to swallow.

If you are convinced, configure your Git client to always use `--ff-only` by default, so you get this behavior even if you forget the command-line flag!

```
$ git config --global pull.ff only
```

---