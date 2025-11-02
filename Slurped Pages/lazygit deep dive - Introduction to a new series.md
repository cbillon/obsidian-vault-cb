---
link: https://oliverguenther.de/2021/04/lazygit-an-introduction-series/
excerpt: You might consider yourself a person that has a working knowledge of
  git. I certainly do. I use git daily, swinging between branches and worktrees,
  doing interactive rebases and using features that many might label "advanced"
  git features.
slurped: 2025-11-02T19:38
title: lazygit deep dive - Introduction to a new series
---

You might consider yourself a person that has a working knowledge of git. I certainly do. I use git daily, swinging between branches and worktrees, doing interactive rebases and using features that many might label "advanced" git features.

But still, I keep struggling with some of the features that are not used everyday. Not necessarily because I don't get the concept behind them. I either keep forgetting the correct command and its order to use or find typing the command tedious, even with aliasing it to make it shorter.

I recently integrated [lazygit](https://github.com/jesseduffield/lazygit) into my daily working with git. It helps with making some operations faster, and provides an interface to perform some of the magic whose commands I would need to look up often. In this tutorial series, I will demonstrate this tool from its basic usage to advanced features, starting with an introduction in this post.

## Series index

- _You're here:_ [Introduction to lazygit](https://oliverguenther.de/2021/04/lazygit-an-introduction-series/)
- [Deep dive: status panel](https://oliverguenther.de/2021/04/lazygit-status-panel-deep-dive)
- [Deep dive: files panel](https://oliverguenther.de/2021/04/lazygit-the-files-panel)
- [Deep dive: branches panel](https://oliverguenther.de/2021/07/lazygit-the-branches-panel)
- [Deep dive: commits panel](https://oliverguenther.de/2021/12/lazygit-the-commits-panel)
- [Deep dive: stash panel](https://oliverguenther.de/2022/10/lazygit-the-stash-panel)
- [Use-case: backporting with rebase onto](https://oliverguenther.de/2023/09/lazygit-use-case-backporting)

## A first glimpse

I'm not covering installation here, as you will find excellent [installation instructions on the GitHub repository of lazygit](https://github.com/jesseduffield/lazygit#installation). All my examples will be working on [OpenProject](https://openproject.org/) as my daily work rather than another made up tiny example repository with git.

Once you installed lazygit (I aliased it to `lg` right away) and run it, you will be greeted with the following screen:

![A first view of lazygit on dev branch](https://oliverguenther.de/2021/04/lazygit-an-introduction-series/lazygit-intro.png)

This will give you an overview over the current state of your git repo with the help of different panels. Each panel shows a different aspect of your repository. They can be navigated using the left and right arrow keys → and ←, or with the panels . Within each panel, you can move between items using the up and down arrow key ↓ and ↑.

On every panel, you can hit ? to get a quick menu with the available options in this menu. Every panel activates their own actions, and is highlighted by a colored border (depending on your terminal scheme) once you activate them.

Let's go through the panels one by one:

#### 1. Status

The status panel shows your current branch and tracking branch, and any local or remote changes that are not integrated. In my example, I'm on our `dev` branch with one commit on my local and one on my remote `origin/dev` branch.

#### 2. Files

The files panel shows your locally modified and/or staged files. Navigate files with up and down arrow keys, stage or unstage them with space ␣.

#### 3. Branches

Shows your recently used branches. The active branch is marked with a *, other branches are shown with their last activity. Each branch shows you outstanding local and remote changes, just like the status panel does for the current branch. Once activated, use the up and down keys to navigate between branches. Use space ␣ to checkout a selected branch.

This panel also has tabs for remotes and tags, which we will look at in another post.

#### 4. Commits

The commits panel shows the recent commits for your active branch. Again, you can move between the commits with the up and down arrow keys.

#### 5. Stash

The stash panel shows your stash stack items, if any. In my case, this is currently empty, so it does not show anything.

#### Right hand side

The right hand side shows information depending on the active panel. It will show the most recent log commits from your current branch (status, branch panels), or the diff from the selected files, commits, or stash item in the respective panels.

## A simple example: Creating a new pull request

My daily work includes creating a lot of branches and pull requests for individual bug fixes and features.

Here's a quick rundown on how I would create a bug fix pull request:

#### Step 1: Fast-forward dev branch

First I'll want to make sure I fast-forwarded the `dev` branch. For that I switch to the branches tab (2), search for the dev branch (checkout by name c dev ⏎) and fast-forward to get any remote changes (f).

#### Step 2: Create new branch based on dev

From there, I would check out a new branch (n) and type the branch name (e.g., `fix/36346/moment-locale`) and press enter to confirm.

#### Step 2: Create your changes

Then I will continue working and switch back to the files panel (2) to create commits. Select modified or new files with space and hit c to commit the files with a quick inline message. If you want to type using your $EDITOR, use an uppercase C.

#### Step 3: Push and open pull request

When ready to create a pull request, I switch back to the branches panel (3) to push the branch to origin (P) followed by open pull request (o). This will spawn a browser window with the compare window.

If you're now ready to try your own, feel free to [help contributing to OpenProject](https://github.com/opf/openproject/blob/dev/CONTRIBUTING.md)!

**Next post in series**

[Click here for the next post: Deep dive: status panel](https://oliverguenther.de/2021/04/lazygit-status-panel-deep-dive)