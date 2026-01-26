---
link: https://joshtronic.com/2020/08/09/how-to-get-the-default-git-branch/
byline: Josh Sherman
excerpt: |-
  Posted 09-Aug-2020
              
              in Command-line Interface
              
                tagged
                
                  Version Control
slurped: 2026-01-22T10:00
title: How to get the default git branch
tags:
  - git
---

As the world has been shifting away from using terms like `master` and `slave` in technology, I was left wondering what the heck I was going to do about my `git` aliases that relied explicitly on the word `master`.

`git` itself makes it really easy to swap the default branch from `master` to `main`, `production`, `trunk`, or whatever, but out of the box doesn't have a way to easily reference the default branch.

Because I want to keep my work flows as they are, and maintain my high level of productivity, I set out of swap out my aliases that have `master` hard coded for a method that would return the default branch for the repository.

My aliases that used `master` were as follows:

```
alias gcm='git checkout master'
alias gfm='git fetch origin master:master'
alias gmm='git merge master'
alias grbm='git rebase master'
```

I'm not stranger to creating more complicated aliases, so I already had a good idea of what I wanted to do. For some of my other aliases, I reference a method called `git_current_branch` that pulls the current branch:

```
alias gl='git pull origin $(git_current_branch)'
```

So I set out to create a new method called `git_default_branch`. The method needed to be able to figure out what the default branch is and spit it back out.

`git` has a lot of great commands, but often times, even with the flags to suppress information, they return too much information and need to be massaged.

In this scenario, I was able to use `git symbolic-ref` to get the remote origin branch (which was used to initially clone the repository):

```
git symbolic-ref refs/remotes/origin/HEAD
```

Which spits back:

```
refs/remotes/origin/master
```

Great, but the only part of that we want is `master`. Nothing a little `sed` action couldn't handle:

```
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
```

Perfect, now we have the last part of the string, which corresponds with the default branch of the repository.

To make it reusable across my aliases, I simply wrapped it up in a method:

```
git_default_branch() {
  (git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@') 2>/dev/null
}
```

And updated my aliases to reference it:

```
alias gcm='git checkout $(git_default_branch)'
alias gfm='git fetch origin $(git_default_branch):$(git_default_branch)'
alias gmm='git merge $(git_default_branch)'
alias grbm='git rebase $(git_default_branch)'
```

Sure as heck beats maintaining multiple aliases or even worse, abandoning my aliases in favor of just typing things out manually, which still would require remembering what the default branch for the particular repository is.

Now all you need to do is follow one of the thousand other posts out there to rename your default branch from `master` to `whatever` and you're nearly off to the races.

As mentioned earlier, the default branch returned seems to be the one that was used to originally clone the repository. I found on a repository where I switched from `master` to `main`, the method still returned `master`.

There's probably a more sophisticated solution, but for me, I just went ahead and re-cloned the repository and called it a day.

If you happen to be interested in my other shell aliases and other command-line shenanigans, feel free to check out my [dotfiles](https://git.sherver.org/joshtronic/dotfiles).