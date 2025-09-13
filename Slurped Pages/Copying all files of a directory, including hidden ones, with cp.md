---
link: https://bhoot.dev/2025/cp-dot-copies-everything/
byline: |-
  Jayesh Bhoot's Ghost
    Town
excerpt: I outline my exploration of why cp -R src/. dest copies contents of src – that too all its files, including hidden ones – and not src itself, as src/. had initially led me to believe. This is because of how the algorithm of cp is defined.
slurped: 2025-09-01T12:25
title: Copying all files of a directory, including hidden ones, with cp
tags:
  - bash
  - cp
---

I outline my exploration of why `cp -R src/. dest` copies contents of `src` – that too _all_ its files, including hidden ones – and not `src` itself, as `src/.` had initially led me to believe. This is because of how the algorithm of `cp` is defined.

There are two directories - a non-empty `src` and an empty `dest`. My **aim** is to copy all the files in `src` inside `dest`, including hidden files.

```
$ ls -a src
```

`src` dir contains a normal file named `unhidden` and a hidden file called `.hidden`

## Solution - `cp -R src/. dest`

I want to cut to the chase, so here is the command: `**cp -R **src/.** dest**`.

```
$ cp -R src/. dest
```

The `src/.` construct performs the magic.

I was more interested in figuring out why `src/.` makes this happen. It felt counter-intuitive to me – `src/.` simply refers to the `.` entry in the `src` directory, i.e., it refers to `src` itself. A command like `cp -R src dest` should have copied the entire `src` directory inside `dest`, ending up with a structure like `dest/src/{.hidden,unhidden}` instead of `dest/{.hidden,unhidden}`.

## Why does it work?

After some faffing about, I found the algorithm [used by `cp` as documented in POSIX](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/cp.html). Below is the relevant portion of the algorithm.

> `cp -R source_file target`
> 
> If `target` exists and names an existing directory, the name of the corresponding destination path for each file in the file hierarchy rooted in each _source_file_ shall be the **concatenation** of `target`, a single >slash< character if `target` did not end in a >slash<, and the **pathname of the file relative to the directory** containing `source_file`. [Emphasis added]

I tried to understand the algorithm by building path for `src/.hidden` file. According to the algorithm, the path of a target file is a concatenation of:

1. _target_ (`dest` in our case) +
2. _/_ if needed (which we do) +
3. relative pathname of the target file, starting from the directory that _contains_ the `source_file`. I am interpreting _heavily_ now – in our case of `src/.`, the directory containing `.` is `src`. So the pathname for the file `.hidden` , relative from `src`, would become: `./.hidden`.

Thus, the built path for the hidden file `.hidden` becomes: `dest/./.hidden`, which is equal to `dest/.hidden`.

Similarly, for the normal file `unhidden`, the built path becomes `dest/./unhidden`, which is equal to `dest/unhidden`.