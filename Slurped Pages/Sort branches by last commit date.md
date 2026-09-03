---
link: https://ryangreenberg.com/til/git-branches-by-commit-date/
site: Ryan Greenberg
excerpt: August 24, 20269:55 AM
slurped: 2026-08-31T09:45
title: Sort branches by last commit date
tags:
  - git
---

### August 24, 2026  
9:55 AM

I have too many git branches and like many people I developed a convoluted script to show me the most recent ones. I ran this command many times a day for years:

```
function branches {
  for k in `git branch | sed "s/^..//"`; do
    echo -e `git log --color -1 --pretty=format:"%Cgreen%ci %Cblue%cr%Creset" "$k"`\\t"$k";
  done | sort
}
```

This outputs the date as the first column in the output so that `sort` works. Turns out with modern git it’s totally unnecessary. Just configure git branch to sort by the last commit date:

```
git config branch.sort committerdate
```

You can also supply this on a one-off basis as `git branch --sort committerdate`.

What else can you sort by? The manual says “The keys supported are the same as those in `git-for-each-ref(1)`.” The manpage for [`git-for-each-ref`](https://man.freebsd.org/cgi/man.cgi?query=git-for-each-ref&sektion=1&format=html) lists a ton of fields, including some that don’t make sense to me as sort keys. Here are a few that stood out as interesting:

- `authordate`, `committerdate`, `creatordate`, `taggerdate`
- `contents:size`: The size in bytes of the commit or tag message.
- `push`: The name of a local ref which represents the @{push} location for the displayed ref.
- `refname`: The name of the ref (the part after $GIT_DIR/) (this is the default).
- `upstream`: The name of a local ref which can be considered “upstream” from the displayed ref.

Git