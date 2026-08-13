---
tags:
  - git
---
**As of Git 1.8.0:**

```
git branch -u upstream/foo
```

Or, if local branch `foo` is not the current branch:

```
git branch -u upstream/foo foo
```

Or, if you like to type longer commands, these are equivalent to the above two:

```
git branch --set-upstream-to=upstream/foo

git branch --set-upstream-to=upstream/foo foo
```