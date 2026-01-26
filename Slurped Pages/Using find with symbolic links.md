---
link: https://joshtronic.com/2021/06/13/using-find-with-symlinks/
byline: Josh Sherman
excerpt: |-
  Posted 13-Jun-2021
              
              in Command-line Interface
slurped: 2026-01-22T10:05
title: Using find with symbolic links
tags:
  - find
  - symbolic_link
---

Symbolic links, or symlinks come in handy when you need to share directories with multiple directories and not have to worry about keeping each linked instance in sync.

The problem with symlinks is that not every command-line utility supports them, and often times, they do support them, but it's not default behavior.

One such utility is `find` which allows you to search a directory for files and directories that match a pattern. Out of the box, `find` doesn't follow symbolic links.

To will `find` into following symbolic links, all you need to do is pass in the `-L` argument (as a predicate to the path):

```
# Finds all files, but won't follow symlinks
find . -type f
```

If you happen to want this behavior all of the time, you can simply alias `find` to include the argument:

```
alias find='find -L'
```