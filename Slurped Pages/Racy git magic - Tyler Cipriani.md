---
link: https://tylercipriani.com/blog/2023/11/30/racy-git/
byline: Tyler Cipriani
excerpt: Exploting a long-standing git bug for my own amusement.
slurped: 2026-06-01T08:24
title: Racy git magic - Tyler Cipriani
tags:
  - git
---

Exploting a long-standing git bug for my own amusement.

> And I think there is one known race: the index mtime itself is not race-free.
> 
> – Linus Torvalds, [Re:git bugs](https://lore.kernel.org/git/alpine.LFD.1.10.0806101249580.3101@woody.linux-foundation.org/), 2008

A well-known race condition skulks through git’s plumbing.

And I can demo it via a **git magic trick 🪄**[1](https://tylercipriani.com/blog/2023/11/30/racy-git/#fn1)

```
$ tree -L 1 -a .
.
├── file
└── .git

$ cat file
okbye
$ git status
On branch main
Changes to be committed:
        new file:   file

$ git ls-files --modified  # No output. A clean working directory.
$ git commit --message="The file sez $(cat file)"
$ git log --oneline HEAD -1
600fcac (HEAD -> main) The file sez okbye
$ git status
On branch main
nothing to commit, working tree clean
```

Nothing up my sleeves:

- The git repo had one staged file, `file`, containing `okbye`—nothing else
- I committed it
- And the commit message is `The file sez okbye`

**Now, the big reveal**:

```
$ cat file
okbye
$ git show HEAD:file
hello
$ git status
nothing to commit, working tree clean
```

Boom. ✨Magic✨

Git is clueless that the wrong file is in your work tree.

Even `git restore` has no effect:

```
$ git restore file
$ cat file
okbye
```

Git maintainers know this sleight-of-hand well—dubbing it the “[Racy Git Problem](https://git-scm.com/docs/racy-git/en)” circa 2006.

But it can still generate [heisenbugs](https://en.wikipedia.org/wiki/Heisenbug) in unexpected places.

## What is the racy git problem?

The two biggest problems in computer science are:

1. cache invalidation,
2. naming things, and
3. off-by-one

Racy git is a cache invalidation problem.

Git speeds operations by stowing two bits of data about each file in your work tree:

1. The file size
2. The last time the file was modified—its `mtime`

So, if you tweak a file, without changing the mtime or the size—how would git know?

Before 2006, git was oblivious.

Now, it’s wised up—if the file looks unchanged (via size and mtime), then git performs another check. If file mtime >= the mtime of the index file (`.git/index`), then git rebuilds the index.

Thus, the core of my lame magic trick:

```
#!/usr/bin/env bash
echo hello > file

# Stage the file for commit
git update-index --add file

# Send .git/index's mtime INTO THE FUTURE!!!1!
touch --date='1 second' .git/index

# Now modify "file" behind git's back. So. Sneaky.
echo okbye > file

# CAVEAT OF DOOoooM: this has to happen within a single second to work---yeah. git's good. :D
```

## How this happens in the real world

This problem can still happen in the real world.

I stumbled onto this while working with `git fat` (a forerunner of GitHub’s “Git Large File Storage”—git-lfs) in 2017.

At that time, we had a tool that pulled git code onto hundreds of servers. And every so often, a server would fail to fetch large files.

The basic steps were:

- ssh into 100s of servers in parallel
- `git clone <repo>`
- `git fat pull`

`git fat pull` rsync’d large binary files, mostly jars, into the working directory.

Frustratingly, when this failed, I could jump on the server and rerun `git fat pull`, which worked every time.

This was the racy git problem in disguise.

See, `git fat` uses git [filters](https://git-scm.com/docs/gitattributes#_filter) (just like git-lfs). Filters are scripts that git runs automagically at checkout or commit time.

But git is smart—it only triggers filters when it has to—when your working directory differs from its index.

So, when we ran `git clone` within the same second as `git fat pull`, the files on disk were the same as in the index—so, git never triggered the filter.

Why did it work when I ran it manually? Because `git fat` was smart, too—it `touch`’d files on disk, which invalidated the index which caused git to run the filter command.

And so the racy git problem persists. It plagues every system that relies on the git filters[2](https://tylercipriani.com/blog/2023/11/30/racy-git/#fn2) to. this. day.

---

1. This will only seem like magic if you can distiguish normal git operations from magic. For nerds only.[↩︎](https://tylercipriani.com/blog/2023/11/30/racy-git/#fnref1)
    
2. This means almost all large file storage mechanisms in git. [git annex](https://git-annex.branchable.com/todo/git_smudge_clean_interface_suboptiomal/)’s “indirect mode” eschewed git’s smudge and clean filters. But I see both “direct” and “indirect” mode are now deprecated. I’m unclear how this works today `¯\_(ツ)_/¯`.[↩︎](https://tylercipriani.com/blog/2023/11/30/racy-git/#fnref2)