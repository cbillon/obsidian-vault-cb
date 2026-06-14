---
link: https://tylercipriani.com/blog/2023/07/31/git-files-hidden-in-plain-sight/
byline: Tyler Cipriani
excerpt: |-
  I doubt that it is a good practice to ship the public key used to
  sign things in the repository in the repository itself
slurped: 2026-06-01T08:25
title: Git files hidden in plain sight 🫥
tags:
  - git
---

> I doubt that it is a good practice to ship the public key used to sign things in the repository in the repository itself
> 
> – Junio C Hamano, [git@vger.kernel.org: expired key in junio-gpg-pub](https://lore.kernel.org/git/20210907204100.ptapvn4wrqn3qrzq@meerkat.local/T/)

Git ships with the maintainer’s public key.

But you won’t find it in your worktree—it’s hidden in plain sight.

Junio Hamano’s public key is a blob in the `git` object database. It’s tagged with `junio-gpg-pub`, so you can only see it with `git cat-file`:

```
(/^ヮ^)/*:・ﾟ✧ git cat-file blob junio-gpg-pub
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1
...
```

In 2021, Junio pretty much said that this was a [bad idea](https://lore.kernel.org/git/20210907204100.ptapvn4wrqn3qrzq@meerkat.local/T/).

But it led me to think about some other wonderful bad ideas.

## Fake empty GitHub repos 📦

I made an empty GitHub repo called [hidden-zangief](https://github.com/thcipriani/hidden-zangief).

![hidden-zangief](https://photos.tylercipriani.com/2023-07-31_empty-github.png)

Except it’s not empty.

Instead, it’s chockfull of sweet ANSI art—Zangief from Street Fighter II.

![Zangief + Figlet = magic](https://photos.tylercipriani.com/2023-07-31_zangief-ansii.png)

And if you clone it, after an initial warning, you can see Zangief is still in there:

```
(/^ヮ^)/*:・ﾟ✧ git clone https://github.com/thcipriani/hidden-zangief && cd hidden-zangief
Cloning into 'hidden-zangief'...
warning: You appear to have cloned an empty repository.
(/^ヮ^)/*:・ﾟ✧ git fetch origin refs/atomic/piledriver
remote: Enumerating objects: 1, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 1 (delta 0), reused 1 (delta 0), pack-reused 0
Unpacking objects: 100% (1/1), 1.71 KiB | 1.71 MiB/s, done.
From https://github.com/thcipriani/hidden-zangief
 * branch            refs/atomic/piledriver -> FETCH_HEAD
(/^ヮ^)/*:・ﾟ✧ git show FETCH_HEAD

                        [...sweet zangief ansi art...]
                        _____                 _       __ 
                       |__  /__ _ _ __   __ _(_) ___ / _|
                         / // _` | '_ \ / _` | |/ _ \ |_ 
                        / /| (_| | | | | (_| | |  __/  _|
                       /____\__,_|_| |_|\__, |_|\___|_|  
                                        |___/        
```

## Dubious git plumbing hacks 🪓

Inspired by Junio, I misused and finagled a couple of [git plumbing commands](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain) to make this fake empty repo:

1. `git hash-object`
2. `git update-ref`

First, I used `hash-object` to create a dangling git object with the `~/zangief.txt` contents.

```
(/^ヮ^)/*:・ﾟ✧ mkdir /tmp/hidden-zangief && cd /tmp/hidden-zangief
(/^ヮ^)/*:・ﾟ✧ git init
(/^ヮ^)/*:・ﾟ✧ git hash-object -w ~/zangief.txt
7dd9e2d2d2d8b5107d225b4708e1177abb08e7c8
```

Now Zangief is lurking in your git plumbing, atomic-suplexing your other git objects.

I imagine this is how Junio added his public key to the git object database. Then he tagged it with `junio-gpg-pub` and pushed it to the `git` repo.

But a tag would appear in the GitHub UI, and I wondered whether I could hide it.

So I opted to abuse the wide-open git ref namespace, imagining a ref beyond tags and branches: `refs/atomic/piledriver`.

Then I schlepped that ref to GitHub.

```
(/^ヮ^)/*:・ﾟ✧ git update-ref refs/atomic/piledriver 7dd9e2d2d2d8b5107d225b4708e1177abb08e7c8
(/^ヮ^)/*:・ﾟ✧ git remote add origin https://github.com/thcipriani/hidden-zangief
(/^ヮ^)/*:・ﾟ✧ git push origin refs/atomic/piledriver:refs/atomic/piledriver
```

And, of course, Microsoft GitHub foolishly neglects the `refs/atomic/*` namespace in their UI, rendering our 400 lb wrestler friend invisible.

Infinite magic awaits the intrepid developer willing to abuse `git` plumbing. After all, git is just a database with a terrible interface.