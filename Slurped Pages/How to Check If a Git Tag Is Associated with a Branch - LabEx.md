---
link: https://labex.io/tutorials/git-how-to-check-if-a-git-tag-is-associated-with-a-branch-560110
site: LabEx
excerpt: Learn how to check if a Git tag is associated with a branch using `git branch --contains` and `git log`. Verify tagged commits and test unrelated branches in this hands-on Git lab.
twitter: https://twitter.com/@WeAreLabEx
tags:
  - git
  - tag
  - branch
slurped: 2025-12-14T18:09
title: How to Check If a Git Tag Is Associated with a Branch | LabEx
---

Contents

- [Introduction](https://labex.io/tutorials/git-how-to-check-if-a-git-tag-is-associated-with-a-branch-560110#introduction)
- [Run git branch --contains Tag](https://labex.io/tutorials/git-how-to-check-if-a-git-tag-is-associated-with-a-branch-560110#run-git-branch---contains-tag)
- [Use git log to Verify Branch](https://labex.io/tutorials/git-how-to-check-if-a-git-tag-is-associated-with-a-branch-560110#use-git-log-to-verify-branch)
- [Test Unrelated Branches](https://labex.io/tutorials/git-how-to-check-if-a-git-tag-is-associated-with-a-branch-560110#test-unrelated-branches)
- [Summary](https://labex.io/tutorials/git-how-to-check-if-a-git-tag-is-associated-with-a-branch-560110#summary)

## Introduction

In this lab, you will learn how to check if a Git tag is associated with a specific branch. We will explore the `git branch --contains` command to identify which branches include the commit pointed to by a tag.

You will also use `git log` to visually verify the presence of the tagged commit within a branch's history and test the behavior of `git branch --contains` with branches that do not contain the tagged commit.

## Run git branch --contains Tag

In this step, we will explore how to use the `git branch --contains` command with a tag. This command is useful for finding out which branches contain a specific commit, and since tags point to specific commits, we can use it to see which branches include the commit that a tag points to.

First, let's make sure we are in our project directory. Open your terminal and navigate to the `my-time-machine` directory if you are not already there:

```
cd ~/project/my-time-machine
```

Now, let's create a new file and make a commit. This will give us a new commit to tag.

```
echo "Another message for the future" > message2.txt
git add message2.txt
git commit -m "Add another message"
```

You should see output similar to this after the commit:

```
[master a1b2c3d] Add another message
 1 file changed, 1 insertion(+)
 create mode 100644 message2.txt
```

Now, let's create a tag for this commit. We'll call it `v1.0`.

```
git tag v1.0
```

This command creates a lightweight tag named `v1.0` pointing to the current commit.

To see which branches contain the commit pointed to by the `v1.0` tag, we use `git branch --contains`:

```
git branch --contains v1.0
```

Since we only have the `master` branch and we created the tag on the latest commit of `master`, the output should show `master`:

```
* master
```

The asterisk (`*`) indicates the currently checked out branch.

This command is powerful because it allows you to quickly identify which branches have incorporated a specific milestone or release, represented by a tag.

## Use git log to Verify Branch

In this step, we will use the `git log` command to visually verify that the commit pointed to by our tag `v1.0` is indeed present in the `master` branch's history. While `git branch --contains` tells us which branches contain a commit, `git log` allows us to see the commit history in detail.

Make sure you are still in the `~/project/my-time-machine` directory.

Now, let's view the commit history of the `master` branch using `git log`. We can use the `--decorate` option to show tags and branch pointers.

```
git log --decorate --oneline
```

The `--oneline` option provides a concise view, showing each commit on a single line. The `--decorate` option shows the references (like branch names and tags) that point to each commit.

You should see output similar to this:

```
a1b2c3d (HEAD -> master, tag: v1.0) Add another message
e4f5g6h Send a message to the future
```

In this output, you can see the commit hash (a1b2c3d in this example), followed by the commit message. Crucially, next to the latest commit, you should see `(HEAD -> master, tag: v1.0)`. This confirms that the `master` branch's latest commit is the same commit that the `v1.0` tag points to.

This visual confirmation using `git log` complements the information provided by `git branch --contains`. It helps you understand the relationship between branches, tags, and commits in your project's history.

Remember, `git log` is a powerful tool for exploring your project's timeline. You can use various options to customize the output and find the information you need. Press `q` to exit the log view if it's in a pager.

## Test Unrelated Branches

In this step, we will create a new branch and make a commit on it that is _not_ based on the commit pointed to by our `v1.0` tag. This will help us understand how `git branch --contains` behaves with branches that do not contain the specified commit.

First, let's create a new branch called `feature-branch`.

```
git branch feature-branch
```

Now, let's switch to this new branch.

```
git checkout feature-branch
```

You should see output indicating that you have switched branches:

```
Switched to branch 'feature-branch'
```

Next, let's make a new commit on this `feature-branch`. We'll create a new file.

```
echo "This is a new feature" > feature.txt
git add feature.txt
git commit -m "Add new feature file"
```

You should see output similar to this after the commit:

```
[feature-branch a1b2c3d] Add new feature file
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

Now, let's use `git branch --contains v1.0` again. Remember, `v1.0` points to a commit on the `master` branch, and our new commit on `feature-branch` is different.

```
git branch --contains v1.0
```

This time, the output should only show the `master` branch:

```
master
```

The `feature-branch` is not listed because it does not contain the specific commit that the `v1.0` tag points to. This demonstrates how `git branch --contains` accurately identifies which branches have a particular commit in their history.

This is useful when you want to know if a bug fix (tagged with a version) has been included in a release branch, or if a specific feature (also potentially tagged) has been merged into different development lines.

## Summary

In this lab, we learned how to check if a Git tag is associated with a branch. We started by using the `git branch --contains <tag>` command, which effectively identifies branches that include the commit pointed to by the specified tag. We demonstrated this by creating a new commit, tagging it, and then verifying that the `master` branch, where the tag was created, was listed by the command.

We then used the `git log` command to visually confirm that the commit associated with the tag was present in the branch's history, providing a detailed view of the commit lineage. Finally, we explored how the `git branch --contains` command behaves with branches that do not contain the tagged commit, reinforcing its utility in quickly determining branch inclusion of specific tagged milestones.
