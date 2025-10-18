---
link: https://amitmerchant.com/trigger-deployments-using-empty-git-commits/
byline: Amit Merchant
site: Amit Merchant - A blog on PHP, JavaScript, and more
date: 2022-01-20T01:00
excerpt: One of the most common scenarios with modern software development is the use of CI/CD pipelines. The way these CI/CD pipelines work is, you set up a webhook on your Git repository and allow these deployments to run every time something is pushed to the repository.
slurped: 2025-10-13T11:13
title: Trigger CI/CD deployments using empty Git Commits
tags:
  - git
---

One of the most common scenarios with modern software development is the use of [CI/CD pipelines](https://en.wikipedia.org/wiki/CI/CD). The way these CI/CD pipelines work is, you set up a webhook on your Git repository and allow these deployments to run every time something is pushed to the repository.

Now, if for some reason the deployment doesn’t get triggered or you just want to re-trigger the deployment, the obvious thing you would do is to **do some random change in any file of your Git repository**, **commit it**, and **push it onto the branch on the remote**. And it will trigger the deployment.

This is fine and works but you would end up modifying files unnecessarily which in my opinion is not cool.

That’s where empty Git commits can come to your rescue.

## Empty Git Commits

Usually, when you want to commit in Git, you need to have some modified files added to the current index. Trying to commit without doing so would give **“nothing to commit, working tree clean”** message like so.

But there’s an option called `--allow-empty` that you can use with `git commit` which lets you perform an _empty commit_. Essentially, this option will let you commit without adding any modified file to the working tree.

Here’s how you can use this option.

```
$ git commit --allow-empty -m "this is an empty commit"
```

Running this will create an empty commit like so.

Since a commit has been recorded, you would be now able to push the changes to the remote.


Here’s how an empty commit looks like on remote.

And that’s how you can push an empty commit and trigger the deployments which are tied to this action.