---
tags:
  - git
---
[stack overflow](https://stackoverflow.com/questions/21609781/why-call-git-branch-unset-upstream-to-fixup)
Today, while working on a new post, `git status` showed the following message:

```
On branch source
Your branch is based on 'origin/master', but the upstream is gone.
  (use "git branch --unset-upstream" to fixup)
```

TL;DR version: remote-tracking branch `origin/master` used to exist, but does not now, so local branch `source` is tracking something that does not exist, which is suspicious at best—it means a different Git feature is unable to do anything for you—and Git is warning you about it. You have been getting along just fine without having the "upstream tracking" feature work as intended, so it's up to you whether to change anything.

This warning is a new thing in Git, appearing first in Git 1.8.5. The release notes contain just one short bullet-item about it:

> - "git branch -v -v" (and "git status") did not distinguish among a branch that is not based on any other branch, a branch that is in sync with its upstream branch, and a branch that is configured with an upstream branch that no longer exists.

To describe what it means, you first need to know about "remotes", "remote-tracking branches", and how Git handles "tracking an upstream". (_Remote-tracking branches_ is a terribly flawed term—I've started using _remote-tracking names_ instead, which I think is a slight improvement. Below, though, I'll use "remote-tracking branch" for consistency with Git documentation.)

Each "remote" is simply a name, like `origin` or `octopress` in this case. Their purpose is to record things like the full URL of the places from which you `git fetch` or `git pull` updates. When you use `git fetch _remote_,[¹] Git goes to that remote (using the saved URL) and brings over the appropriate set of updates. It also _records_ the updates, using "remote-tracking branches".

A "remote-tracking branch" (or remote-tracking name) is simply a recording of a branch name as-last-seen on some "remote". Each remote is itself a Git repository, so it has branches. The branches on remote "origin" are recorded in your local repository under `remotes/origin/`. The text you showed says that there's a branch named `source` on `origin`, and branches named `2.1`, `linklog`, and so on on `octopress`.

(A "normal" or "local" branch, of course, is just a branch-name that you have created in your own repository.)

Last, you can set up a (local) branch to "track" a "remote-tracking branch". Once local branch _`L`_ is set to track remote-tracking branch _`R`_, Git will call _`R`_ its "upstream" and tell you whether you're "ahead" and/or "behind" the upstream (in terms of commits). It's normal (even recommend-able) for the local branch and remote-tracking branches to use the same name (except for the remote prefix part), like `source` and `origin/source`, but that's not actually necessary.

And in this case, that's not happening. You have a local branch `source` tracking a remote-tracking branch `origin/master`.

You're not supposed to need to know the exact mechanics of _how_ Git sets up a local branch to track a remote one, but they are relevant below, so I'll show how this works. We start with your local branch name, `source`. There are two configuration entries using this name, spelled `branch.source.remote` and `branch.source.merge`. From the output you showed, it's clear that these are both set, so that you'd see the following if you ran the given commands:

```
$ cat .git/config

$ git config --get branch.source.remote
origin
$ git config --get branch.source.merge
refs/heads/master
```

Putting these together,"[²]" this tells Git that your branch `source` tracks your "remote-tracking branch", `origin/master`.

But now look at the output of `git branch -a`, which shows all the local and remote-tracking branch names in your repository. The remote-tracking names are listed under `remotes/` ... and _there is no `remotes/origin/master`_. Presumably there was, at one time, but it's gone now.

Git is telling you that you can _remove_ the tracking information with `--unset-upstream`. This will clear out both `branch.source.origin` and `branch.source.merge`, and stop the warning.
```
git switch <branch-name>
git branch --unset-upstream
```

It seems fairly likely that what you want, though, is to _switch_ from tracking `origin/master`, to tracking something else: probably `origin/source`, but maybe one of the `octopress/` names.

You can do this with `git branch --set-upstream-to`,[³] e.g.:

```
$ git branch --set-upstream-to=origin/source
```


"[¹]:`git pull` uses `git fetch`, and as of Git 1.8.4, this (finally!) also updates the "remote-tracking branch" information. In older versions of Git, the updates did not get recorded in remote-tracking branches with `git pull`, only with `git fetch`. Since your Git must be at least version 1.8.5 this is not an issue for you."

"[²]:Well, this plus a configuration line I'm deliberately ignoring that is found under `remote.origin.fetch`. Git has to map the "merge" name to figure out that the full local name for the remote-branch is `refs/remotes/origin/master`. The mapping almost always works just like this, though, so it's predictable that `master` goes to `origin/master`."

"[³]:Or, with `git config`. If you just want to set the upstream to `origin/source` the only part that has to change is `branch.source.merge`, and `git config branch.source.merge refs/heads/source` would do it. But `--set-upstream-to` says _what_ you want done, rather than making you go do it yourself manually, so that's a "better way"."