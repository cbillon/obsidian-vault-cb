---
link: https://alchemists.io/articles/git_revert
byline: Alchemists
excerpt: A collective devoted to the craft of software engineering where
  expertise is transmuted into joy.
slurped: 2025-02-08T08:59
title: Git Revert | Alchemists
---

Git Revert

When committing changes to your [Git](https://git-scm.com/) repository, you’ll sometimes make mistakes. If on a feature branch, you can use [Git Rebase](https://alchemists.io/articles/git_rebase) to erase mistakes for a clean Git history and improved [Code Reviews](https://alchemists.io/articles/code_reviews). Otherwise, if the mistake is made on your default branch or, worse, deployed into production you’ll need to use `git revert` because you need to document the change being made and explain why you had to unwind a previous change.

The problem everyone makes is using `git revert <sha>` instead of `git revert --no-commit <sha>`. The former results in an ugly and hard to read commit with no further modification to the commit message while the latter allows you to explain your reasoning.

To illustrate, we can use a `demo` project and learn why using `git revert --no-commit` is important:

```
mkdir demo
cd demo
git init
touch bad.txt
git commit --all --message "Added bad file" --message "For demonstration purposes."
```

Assuming the above bad commit passed code review, was deployed, and now needs to be reverted, you’d need to revert this commit by doing the following:

```
git log --oneline
# 4cad7350269d (HEAD -> main) Added bad file

git revert 4cad7350269d
```

You’ll then see the following message pop up in your editor:

```
Revert "Added bad file"

This reverts commit 59985773c3bc439dd520d7a8255b17d12ce95e8d.
```

…​which most people save, exit, and move on. That’s not what we want because we need to explain ourselves not only for our own recollection but also for our team and eventual [Milestones](https://alchemists.io/articles/milestones). To fix, let’s first delete the previously reverted commit:

```
git reset --hard HEAD~1
git log --oneline
# 59985773c3bc (HEAD -> main) Added bad file
```

Now, with a clean slate, we can make a couple of corrections. The first is to modify your Git global configuration so you have this configuration:

[revert]
  reference = true

💡 You can also use `git config set --global revert.reference true` if you don’t want to edit your configuration directly.

The above ensures Git provides a full reference to the commit you are reverting (i.e. SHA, subject, and date) by saving you time by not having to hunt down the original commit details.

Next, you need to use `git revert --no-commit` which allows you to provide your own commit message. If we put all this together, we can do the following:

```
git log --oneline
# 59985773c3bc (HEAD -> main) Added bad file

git revert --no-commit 59985773c3bc
git status --short
# D  bad.txt
```

The above reverts the bad commit without making a commit message. The best part is [Git](https://git-scm.com/) has saved the relevant details you need to make a proper commit message. Here’s what you’ll see in your editor when using `git commit`:

This reverts commit 59985773c3bc (Added bad file, 2024-10-31).

Notice Git provided the commit SHA, subject, and date thanks to our `revert.reference = true` configuration. 🎉 Now we can use this information to finesse and save the following commit message:

Removed bad text file

Necessary to revert commit 59985773c3bc (Added bad file, 2024-10-31) due to accidentally committing this change while demo'ing `git revert` functionality.

Milestoner: patch

The above takes what Git initially provided us so we could craft a detailed commit message as explained when adhering to good [Git Commit Anatomy](https://alchemists.io/articles/git_commit_anatomy). Now, if we look at our Git commits, we have history that reads well:

```
git log --oneline --reverse

59985773c3bc Added bad file
33af8a04ab21 (HEAD -> main) Removed bad text file
```

Now you know how to use `git revert` professionally. Enjoy!