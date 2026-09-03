---
link: https://journal.hexmos.com/git-uses-dag/
byline: Ganesh Kumar
site: Hexmos Journal
date: 2025-12-28T17:06
excerpt: >-
  Most developers use Git daily, often memorizing commands without fully
  understanding what happens under the hood. We know how to use git add and git
  commit, but do we really know how it works?




  At its core, Git is a simple key-value data store. It is a database, and
  understanding its fundamental unit—the commit—is crucial to mastering it.




  In this blog, we will demystify Git by exploring its internal structure,
  starting with the most important building block.





  What Is a Commit?




  A c
twitter: https://twitter.com/@AmazingStardom
slurped: 2026-08-30T12:20
title: "From Commits to DAGs: How Git Uses a DAG to Track Your Code History"
---

Most developers use [Git](https://git-scm.com/?ref=journal.hexmos.com) daily, often memorizing commands without fully understanding what happens under the hood. We know _how_ to use `git add` and `git commit`, but do we really know _how it works_?

At its core, Git is a simple key-value data store. It is a database, and understanding its fundamental unit—the **commit**—is crucial to mastering it.

In this blog, we will demystify Git by exploring its internal structure, starting with the most important building block.

## What Is a Commit?

A commit is not just a diff; it represents the state of your project at a specific moment in time.  
Internally, a commit points to a **tree object**, which in turn points to **blob objects** (the actual file contents). Git uses content-addressable storage, so unchanged files are not duplicated—they are simply referenced again.

Each commit has 3 things.

1. **Snapshot:** Pointer to the project's directory tree.
2. **Metadata:** Author, timestamp, and message.
3. **Parent:** Pointer to the previous commit.

This creates a chain where each commit points to its parent, tracing back to the initial commit.

The first commit has no parent and is called the root commit.

A merge commit typically has 2 parents.

We will explore more about Git below.

## Why Git Uses a DAG

[![Git DAG Diagram](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/0381e09dfe586b486eec05b3bd91e04b2045aba10a9a78b1d547be40ebfe13f8.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/0381e09dfe586b486eec05b3bd91e04b2045aba10a9a78b1d547be40ebfe13f8.png?ref=journal.hexmos.com)

Each commit contains a Snapshot, Metadata, and the parent commit ID.  
Each commit points to its parent.

Hence, there is no way for the commit history to form a cycle or loop.

This structure is known as a Directed Acyclic Graph (DAG).

The resulting graph contains the history of every commit, capturing every decision made by the developer.

This makes Git powerful, as we can return to the state of any past commit.

## How Branching Works

A common misconception is that a branch contains a full copy of the project files. This is not how Git works.

A branch is simply a lightweight movable pointer to a commit. It doesn't contain files; the commit it points to references a tree object, which in turn points to the files (blobs).

Let's understand with an example.  
Master Branch.

[![Master Branch Diagram](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/cb19de69ff06cfcff22b0311818fa6311ed9f1f26844d0ea538615af665efada.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/cb19de69ff06cfcff22b0311818fa6311ed9f1f26844d0ea538615af665efada.png?ref=journal.hexmos.com)

Create New Branch

```
git checkout -b iss53
# Switched to a new branch "iss53"
```

[![New Branch Diagram](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/6b6c335e0e80e3b3bd5b07d9b2625b6625aa7d5d369ab24d01c09f9b305f4d8f.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/6b6c335e0e80e3b3bd5b07d9b2625b6625aa7d5d369ab24d01c09f9b305f4d8f.png?ref=journal.hexmos.com)  
Commit changes

[![Commit on New Branch](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/75081fa09f3827234b13c4f3e866c27164aaa207acdca6e82e3a0c7c10965320.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/75081fa09f3827234b13c4f3e866c27164aaa207acdca6e82e3a0c7c10965320.png?ref=journal.hexmos.com)

If you want to change anything in the master branch

Checkout to the branch

```
git checkout master
Switched to branch 'master'
```

And commit changes

[![Commit on Master Branch](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/cccec82259a4004c790bc7c51313ab0b23672ab99eb0b33146fc7da1c83aeaf5.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/cccec82259a4004c790bc7c51313ab0b23672ab99eb0b33146fc7da1c83aeaf5.png?ref=journal.hexmos.com)

[![Git Simulation Animation](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/677c623eb6c3f76db9d6effa2c4c461cf7b219f2344ce81e64f670496682167d.gif)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/677c623eb6c3f76db9d6effa2c4c461cf7b219f2344ce81e64f670496682167d.gif?ref=journal.hexmos.com)

We will understand how Git knows in which commit we are working and the structure of git.

## HEAD: The Pointer for Git Commits

Git uses a special pointer called HEAD to track the current location. HEAD usually points to the current branch name, which in turn points to the latest commit.

Let's assume we are in the main branch.

HEAD points to `main`, and `main` points to the latest commit.

Commit will be your current location.

If we want to create a branch "feature"

```
git checkout feature
```

Suppose we want to check a specific commit. We will use git to point to the specific commit. This will be known as `detached HEAD`. There will be no branch pointing between head and commit.

Actually, we can commit a few more changes. But when we switch to a different commit, it will be orphaned, and no branch will be pointing to it; this will be known as an orphaned commit.

Git's garbage collector will eventually delete these unreachable commits (usually after 30 days by default).

> Actual Reality: We usually try to see in old commits in detached mode. We fix and commit changes. When we move back to the main branch, all commits will be lost.  
> That is why git warns that the head is in detached mode.

## The Structure of Git

Git has 3 stages:

1. **Working Directory:** This consists of the actual files on your disk that you are currently modifying.
2. **Staging Area (Index):** A file that stores information about what will go into your next commit. (Uses `git add .`)
3. **Repository:** Database of commits, also called permanent history. (Uses `git commit -m "update"`)

## Git Undo Commands

In this section, we will look at `git checkout`, `git reset`, and `git revert`.

These three commands are all used to undo changes, but they function differently, which can often be confusing.

### Git Checkout

This command will move only the head.

Git head is now pointing to `c3`.

[![Git Checkout Diagram](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/1a84b0423d680213941a72e26286785d07893e5f9540ebfafad14ab324717804.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/1a84b0423d680213941a72e26286785d07893e5f9540ebfafad14ab324717804.png?ref=journal.hexmos.com)  
Now, when we check out to c1

```
git checkout c1
```

[![Detached HEAD Diagram](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/40a00f6bd9815df49ccbde701ca69e072ca26958a9c0e1105e8ad7a24d0b76b5.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/40a00f6bd9815df49ccbde701ca69e072ca26958a9c0e1105e8ad7a24d0b76b5.png?ref=journal.hexmos.com)

Now, the head is detached, and it is pointing to c1.

> Note: No commits are changed, no history is changed, and no branch is moved.

This means we are just seeing changes made in a particular commit.

### Git Reset

`Git reset` is more powerful than `checkout` because it moves the branch pointer itself, not just the HEAD.

Here we have 4 commits

[![Initial Commit State](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/1a84b0423d680213941a72e26286785d07893e5f9540ebfafad14ab324717804.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/1a84b0423d680213941a72e26286785d07893e5f9540ebfafad14ab324717804.png?ref=journal.hexmos.com)

When we use `git reset c1`

[![Git Reset Diagram](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/a90acef01611f79ec39933391a52d2e3ed81b6a07ade3335b94ec7bc2bd37d23.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/a90acef01611f79ec39933391a52d2e3ed81b6a07ade3335b94ec7bc2bd37d23.png?ref=journal.hexmos.com)

Head moves, and remaining commits will be orphaned.

1. `--soft`  
    Moves HEAD to the target commit. Staged changes and working directory are preserved (changes appear as staged).
    
2. `--mixed` (Default)  
    Moves HEAD to the target commit. Staging area is reset to match the commit, but working directory is preserved (changes appear as modified but unstaged).
    
3. `--hard`  
    Moves HEAD to the target commit. Staging area and working directory are reset to match the commit (all uncommitted changes are lost).
    

### Git Revert

This command creates a new commit that undoes the changes made in a previous commit. It does not remove the old commit from history.

For example:

I want to remove `c1` from the main branch.

[![Pre-Revert State](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/676a50c3be77bc08204af8d24c68af24102ed222c1c8572738e07a92e9ac5848.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/676a50c3be77bc08204af8d24c68af24102ed222c1c8572738e07a92e9ac5848.png?ref=journal.hexmos.com)

Git creates a new commit that is the exact opposite of `c1`, effectively canceling out the changes while keeping the history intact.

[![Git Revert Result](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/2c096d63b47cd68708bf45426e702a99005b770234cd26da17dab5b97d059789.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/2c096d63b47cd68708bf45426e702a99005b770234cd26da17dab5b97d059789.png?ref=journal.hexmos.com)

### Summary of Undo Commands

Here, a simple overview.

|Command|What moves?|Risk Level|Primary Use Case|
|---|---|---|---|
|`git checkout`|HEAD only|Safe|Exploring old history (read-only view)|
|`git reset`|HEAD & Branch Ref|High|Rewriting local history (undoing recent commits)|
|`git revert`|HEAD (New Commit)|Safe|Undoing changes in shared/public history|

## Git Rebase

When you create a new branch and start committing changes, you create a divergent history (DAG) for that branch.

```
# 1. Create and switch to a new branch
git checkout -b new-feature

# 2. Make some commits
git add .
git commit -m "D"
```

Meanwhile, other contributors may have pushed new commits to the main branch, meaning your branch is now behind.

```
# Switch to main and pull latest changes
git checkout main
git pull origin main
```

When you perform a rebase, Git takes the commits from your new branch and "replays" them on top of the latest commit of the main branch.

[![Git Rebase Concept](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/98eb81ab9e7c262913334a9e70fabf21d212a9a0e5fe33dc6661e73d3eea63d7.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/98eb81ab9e7c262913334a9e70fabf21d212a9a0e5fe33dc6661e73d3eea63d7.png?ref=journal.hexmos.com)

Once the rebase is done, the original commits from your new branch are orphaned (no longer used), and your branch now has a clean, linear history extending from the latest main.

[![Git Rebase Result](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/e2a779469c6a1e0c05e173d6be56efc53eea433a021a43b5cdbb649b5820e339.png)](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/e2a779469c6a1e0c05e173d6be56efc53eea433a021a43b5cdbb649b5820e339.png?ref=journal.hexmos.com)

> Note: If you run into conflicts during the rebase, fix the files, add them, and run: `git rebase --continue`

Here are commands to work with it.

|Command|Description|
|---|---|
|`git checkout -b <branch>`|Creates and switches to a new branch.|
|`git rebase main`|Moves your current branch's commits to the tip of `main`.|
|`git rebase --continue`|Continues the rebase process after resolving conflicts.|
|`git rebase --abort`|Stops the rebase and returns the branch to its original state.|

## Conclusion

This blog post provides a detailed introduction to Git, version control, and related concepts.

However, it is obvious that I have missed out on some important topics like merge conflicts and pull requests. I tried to explain the importance of version control in software development and then explain what Git is.

[![FreeDevTools](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/231112a15fa586809fe09486893d8d91ab8873580bca6f5c734120a907d53150.png)](https://hexmos.com/freedevtools/?ref=journal.hexmos.com)

I’ve been building for **FreeDevTools**.

A collection of UI/UX-focused tools crafted to simplify workflows, save time, and reduce friction when searching for tools and materials.

Any feedback or contributions are welcome!

It’s online, open-source, and ready for anyone to use.

👉 Check it out: [FreeDevTools](https://hexmos.com/freedevtools?ref=journal.hexmos.com)  
⭐ Star it on GitHub: [freedevtools](https://github.com/HexmosTech/FreeDevTools?ref=journal.hexmos.com)