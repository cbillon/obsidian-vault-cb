---
link: https://wizardzines.com/comics/inside-git/
site: wizard zines
date: 2024-01-24T14:55
excerpt: HEAD is a tiny file that just contains the name of your current branch
slurped: 2024-08-08T17:10
title: inside .git
tags:
  - lindsay-wardell
---

### `HEAD`

`HEAD` is a tiny file that just contains the name of your current branch

`.git/HEAD`  
`ref: refs/heads/main`

`HEAD` can also be a commit ID, that’s called “detached `HEAD` state”

### branches

a branch is stored as a tiny file that just contains 1 commit ID. It’s stored in a folder called `refs/heads`.

`7622629` - (actually 40 characters)

tags are in `refs/tags`, the stash is in `refs/stash`

### commit

a commit is a small file containing its parent(s), message, tree, and author

`.git/objects/7622629`

```
tree c4e6559 
parent 037ab87 
author Julia <x@y.com> 1697682215 
committer Julia <x@y.com> 1697682215 
commit message goes here 
```

these are compressed, the best way to see objects is with `git cat-file -p HASH`

### trees

trees are small files with directory listings. The files in it are called “blobs”

`.git/objects/c4e6559`

```
100644 blob e351d93 404.html 
100644 blob cab4165 hello.py
040000 tree 9de29f7 lib
```

the permissions here LOOK like unix permissions, but they’re actually super restricted, only 644 and 755 are allowed

### blobs

blobs are the files that contain your actual code

`.git/objects/cab4165`  
`print("hello world!!!!")`

### reflog

the reflog stores the history of every branch, tag, and `HEAD`

`.git/logs/refs/heads/main`

```
2028ee0 c1f9a4c 
Julia Evans <x@y.com> 
1683751582 
commit: no ligatures in code
```

each line of the reflog has: - before/after commit IDs - user + - timestamp - log message

### remote-tracking branches

remote-tracking branches store the most recently seen commit ID for a remote branch

`.git/refs/remotes/origin/main`  
`a9bbcae`

when git status says “you’re up to date with `origin/main`”, it’s just looking at this

### .git/config

.git/config is a config file for the repository. it’s where you configure your remotes

`.git/config`

```
[remote "origin"] 
url = git@github.com: jvns/int-exposed 
fetch = +refs/heads/*: refs/remotes/origin/* 
[branch "main"] 
remote = origin 
merge refs/heads/main
```

git has and local global settings, the local settings are here and the global ones are in `~/.gitconfig`

### hooks

hooks are optional scripts that you can set up to run (eg before a commit) to do anything you want

`.git/hooks/pre-commit`

```
#!/bin/bash 
any-commands-you-want
```

### the staging area

the staging area stores files when you’re preparing to commit

`.git/index`  
`(binary file)`