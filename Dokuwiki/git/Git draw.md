---
link: https://github.com/sensorflo/git-draw/wiki
byline: sensorflo
site: GitHub
excerpt: draws nearly the full content of a tiny git repository as a graph - sensorflo/git-draw
slurped: 2024-06-07T09:55:14.492Z
title: Home
tags:
  - git
  - graph
---

`git-draw` draws nearly the full content of a tiny git repository as a graph. For example after the following few git commands

```
git init
echo 'hello world' > greeting.txt
git add greeting.txt
git commit -m 'initial commit'
git tag R1 -m R1
echo 'bye bye' > parting.txt
git add parting.txt
git commit -m 'added parting'
echo 'welcome' > greeting.txt
git add greeting.txt
```

`git-draw` will display the following image

![](https://github.com/sensorflo/git-draw/wiki/example.png)

## Documentation

[](https://github.com/sensorflo/git-draw/wiki#documentation)

The text provided here is a copy of the documentation the `git-draw` script contains.

```
NAME
  git-draw - draws nearly the full content of a tiny git repository as a graph

SYNOPSIS
  git-draw [OPTION]...

PREREQUISITES
  You don't need all of these if you use git-draw with certain options.

  - graphviz (http://www.graphviz.org/)
  - imagemagick (http://www.imagemagick.org)

  If you have apt you can install these with:

    sudo apt-get install graphviz imagemagick

DESCRIPTION
  git-draw is composed of three main steps, where the 2nd and 3rd are just for
  convenience and are not part of git-draw's core responsibility.

  1) A .dot file describing the repository's content as a graph is created.
  2) dot (see graphviz) is called to produce an image out of that .dot file.
  3) display (see imagemagick) is called to display that image.

  The current working directory must be at the root of the working tree of
  your project, i.e. the directory which contains the .git directory.

  The intention is to help learning Git's basic concepts (references, Git
  objects, SHA-1 checksum over content as id). Virtually all information
  concerning Git's basic concepts is contained in the drawing. Thus git-draw
  is aimed at tiny toy Git repositories and at users with an engineer
  background, i.e. users which are not scared off by terms like checksum,
  references aka pointers and graphs.

OPTIONS
  -p, --print-only
    Only prints the .dot file to STDOUT. Mutually exclusive with --image-only.

  -i, --image-only
    Only generates an image of the graph, and a .dot file beforehand. Mutually
    exclusive with --print-only.

AUTHOR
  Written by Florian Kaufmann <sensorflo@gmail.com>

COPYRIGHT
  Florian Kaufmann 2014. License GPLv3+: GNU GPL version 3 or later
  <http://gnu.org/licenses/gpl.html>. This is free software: you are free to
  change and redistribute it. There is NO WARRANTY, to the extent permitted by
  law.
```