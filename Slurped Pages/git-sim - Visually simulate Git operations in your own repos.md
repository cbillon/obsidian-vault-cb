---
link: https://initialcommit.com/blog/git-sim
site: Initial Commit
date: 2023-01-22T00:00
excerpt: "git-sim: Visually simulate Git operations in your own repos with a single terminal command"
slurped: 2025-02-21T13:20
title: "git-sim: Visually simulate Git operations in your own repos"
tags:
  - git
  - git_sim
  - sim
---

![Image of git-sim: Visually simulate Git operations in your own repos](https://initialcommit.com/img/initialcommit/git-sim.jpg "git-sim: Visually simulate Git operations in your own repos")

## Table of Contents

- [Introduction](https://initialcommit.com/blog/git-sim#introduction)
- [When --dry-runs aren't enough](https://initialcommit.com/blog/git-sim#when---dry-runs-arent-enough)
- [What is Git-Sim?](https://initialcommit.com/blog/git-sim#what-is-git-sim)
- [Git-Sim Goals](https://initialcommit.com/blog/git-sim#git-sim-goals)
- [What Does Git Sim Do?](https://initialcommit.com/blog/git-sim#what-does-git-sim-do)
- [Full list of Git-Sim supported commands](https://initialcommit.com/blog/git-sim#full-list-of-git-sim-supported-commands)
- [Git-Sim features](https://initialcommit.com/blog/git-sim#git-sim-features)
- [How to Install and Run Git-Sim](https://initialcommit.com/blog/git-sim#how-to-install-and-run-git-sim)
- [How Does Git-Sim Work?](https://initialcommit.com/blog/git-sim#how-does-git-sim-work)
- [Contributing to Git-Sim](https://initialcommit.com/blog/git-sim#contributing-to-git-sim)
- [Summary](https://initialcommit.com/blog/git-sim#summary)
- [Next Steps](https://initialcommit.com/blog/git-sim#next-steps)

## Introduction

Despite its simple design under the hood, Git is a notoriously confusing tool for new devs to learn to use and understand. In my opinion this is due to the following 3 factors:

1. The wide breadth of concepts that underlie Git's model (working directory, staging area, object database, Git objects, SHA-1, local repo, remote repo, DAG, branching, merging, etc, etc, etc...)
2. The situational awareness required to be versatile with Git
3. The large number of commands that Git provides

And despite repeatedly being told by experienced users that Git is a very "safe" tool in that it is actually quite hard to lose your work, new Git users often operate with a low-level of constant underlying anxiety that their data will be lost or corrupted around every corner. It is also common practice for newbies to copy entire repos before running commands in case they end up in a state they don't know how to recover from.

Even intermediate and advanced Git users often find themselves in situations where they aren't 100% sure what the outcome of a particular command might be. This usually leads to Googling and Stackoverflow to hopefully find someone in a parallel situation and extrapolate their solution to your local repo.

This got me thinking about ways to help Git users of all experience levels understand exactly how running a specific Git command will impact their own local repo before running it. Specifically:

What if you could easily get a visual picture of how any Git command would impact your local repo, without interrupting your dev workflow?

## When --dry-runs aren't enough

Some Git commands implement a `-n` or `--dry-run` flag that enables users to get some idea of how that command will affect the state of their repository. Although these can be useful, not all commands have them, and the purely text-based output can be quite sparse as is typical of Git's command-line interface.

Moreover, many people out there are visual learners and could benefit greatly from a visual approach to simulating the impact of a Git command before running it.

In order to address this, I created [Git-Sim](https://initialcommit.com/tools/git-sim) - a free and open-source command-line tool written in Python.

## What is Git-Sim?

Git-Sim is a command-line tool written in Python that allows Git users to quickly and easily generate images (or even video animations) illustrating the impact of a Git command will have on the local repo.

For example, you can simulate a [git reset](https://initialcommit.com/blog/git-reset) of your branch to the previous commit using the following command:

```
$ git-sim reset HEAD^
```

Output:

![Git-Sim reset command](https://user-images.githubusercontent.com/49353917/210941835-80f032d2-4f06-4032-8dd0-98c8a2569049.jpg "Git-Sim reset command")

Or, you can simulate a [Git merge](https://initialcommit.com/blog/git-merge) of the branch `dev` into the active branch `main` using the following command:

```
$ git-sim merge dev
```

Output:

![Git-Sim merge command](https://user-images.githubusercontent.com/49353917/210942030-c7229488-571a-4943-a1f4-c6e4a0c8ccf3.jpg "Git-Sim merge command")

The point in these examples is to enable Git users review these images to ensure that they understand how the reset/merge will impact their local repo before executing the actual command.

Furthermore, these visualizations can be animated using the `--animate` flag as follows (keep in mind the performance is significantly slower when animating as the Git-Sim program needs to generate an mp4 video as output instead of a static jpg image):

```
$ git-sim --animate merge dev
```

Output:

These visualizations can be especially useful for notoriously confusing commands such as `git reset`, `git restore`, `git merge`, `git rebase`, and `git cherry-pick`, although it never hurts to simulate even simpler commands (such as `git add` or `git commit`), especially if you're a Git newbie or are creating documentation for your team :D.

## Git-Sim Goals

The primary goal of Git-Sim is to quickly and easily create and visualize the effects of Git commands, while _**minimizing interruption to the developer workflow**_. More specifically, Git-Sim should:

1. Provide a command-line interface so devs can run Git-Sim directly in the terminal inside their local Git repos
2. Maintain a parallel syntax to Git commands (subcommands and options/flags), so that using Git-Sim is as familiar as possible
3. Be efficient and fast, generating simulated output images in a matter of seconds

Future updates and enhancements to Git-Sim will primarily be focused around these goals.

## What Does Git Sim Do?

More generally, Git-Sim allows users to run terminal commands in the form:

```
$ git-sim [global options] <subcommand> [subcommand options]
```

The `[global options]` apply to the overarching git-sim simulation itself, including:

`--light-mode`: Use a light mode color scheme instead of default dark mode.

`--animate`: Instead of outputting a static image, animate the Git command behavior in a .mp4 video.

`--reverse`: Display commit history in the reverse direction.

The `[subcommand options]` mimic regular Git options specific to the specified subcommand.

See the [Git-Sim readme Commands section](https://initialcommit.com/tools/git-sim#markdown-header-commands) for a full list of supported subcommands and options along with example simulated output for each subcommand.

## Full list of Git-Sim supported commands

Here is a list of Git subcommands that can currently be simulated with Git-Sim:

- git log
- git status
- git add
- git restore
- git commit
- git stash
- git branch
- git tag
- git reset
- git revert
- git merge
- git rebase
- git cherry-pick

## Git-Sim features

- Run a one-liner git-sim command in the terminal to generate a custom Git command visualization (.jpg) from your repo
- Supported commands: `log`, `status`, `add`, `restore`, `commit`, `stash`, `branch`, `tag`, `reset`, `revert`, `merge`, `rebase`, `cherry-pick`
- Generate an animated video (.mp4) instead of a static image using the `--animate` flag (note: significant performance slowdown, it is recommended to use `--low-quality` to speed up testing and remove when ready to generate presentation-quality video)
- Choose between dark mode (default) and light mode
- Animation only: Add custom branded intro/outro sequences if desired
- Animation only: Speed up or slow down animation speed as desired

## How to Install and Run Git-Sim

To install Git-Sim:

1. Install [Manim and Manim dependencies](https://www.manim.community/) for your OS
2. Install git-sim: `$ pip3 install git-sim`

To run Git-Sim:

1. Open a command-line terminal
2. `cd` to the root directory of your project (at the same level as the .git folder)
3. Run your Git-Sim command in the form: `git-sim [global options] <subcommand> [subcommand options]`

This executes Git-Sim with default settings, which will create and automatically open a `.jpg` image simulating the Git operations in question.

Note that the output image file will be created in a new subdirectory within your project, at the path "./git-sim_media/images/"

## How Does Git-Sim Work?

Git-Sim is a simple Python package that makes use of the [Manim (Math-animation)](https://www.manim.community/) Python library. Manim was originally written by Grant Sanderson, a popular content creator of the [3blue1brown](https://www.3blue1brown.com/) math channel. Grant wrote Manim to easily create explanatory math videos using Python code. These videos often require visualizing graphs, geometries, and equations. Eventually, Grant's original codebase was forked and is now maintained as the Manim "community" edition.

Git-Sim uses Manim to draw the circles, arrows, text, branch names, and refs that represent the Git objects and operations taking place in your repository.

But, before drawing anything, Manim needs to know what to draw. In this context, Git-Sim queries your Git repo using the [GitPython](https://gitpython.readthedocs.io/en/stable/intro.html) library. This enables the Python code to interact with your Git repo to identify a list of commits and associated information to draw in the animation.

Currently Git-Sim doesn't perform any "write" operations on your Git repo, despite this being possible with GitPython. Git-Sim will only ever read information from your local repo to generate simulated Git command output.

Note: I am considering adding a feature in the future that will allow the user to decide whether to execute the actual Git command at the same time as running the Git-Sim simulation. This would likely be accomplished with an optional command-line flag called `--execute`, or similar.

## Contributing to Git-Sim

So far I have just been working on Git-Sim myself, but I'd love to have some help!

If you're interested in contributing to Git-Sim, feel free to [reach out to me by email](mailto:jacob@initialcommit.io) with any ideas, suggestions, enhancements, feature requests, feedback, installation issues, or bugs!

If you prefer to let your code speak for itself, pull requests are always welcome on [our GitHub page](https://github.com/initialcommit-com/git-sim).

## Summary

**Git-Sim** is a command-line tool written in Python that allows Git users to quickly and easily generate images (or even video animations) visualizing the impact of a Git command will have on the local repo.

It is written in Python and primarily leverages the Manim and GitPython dependencies.

The main goal of the tool is to allow Git users of all experience levels to fully understand how executing a Git command will impact the state of their repo, before actually running the command.

## Next Steps

**[Learn more and install Git-Sim now](https://initialcommit.com/tools/git-sim) to start visualizing your Git commands!**

[Follow @initcommit](https://twitter.com/initcommit?ref_src=twsrc%5Etfw) to stay updated with Git-Sim

If you have any questions or suggestions regarding Git-Sim, feel free to email me at [jacob@initialcommit.io](mailto:jacob@initialcommit.io).

If you'd like to contribute to Git-Sim, pull requests are always welcome on our [GitHub page](https://github.com/initialcommit-com/git-sim).

Thanks and happy coding!

## Final Notes