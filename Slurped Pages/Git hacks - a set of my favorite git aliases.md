---
link: https://gagor.pro/2024/01/git-hacks-a-set-of-my-favorite-git-aliases/
byline: Tom
site: Tom's Blog
date: 2024-01-27T01:00
excerpt: |-
  I use Git a lot, even writing this article i will commit text few times. There&rsquo;s a set of aliases I rely on daily and they&rsquo;re first I add in new place.
  Some Git commands are unnecessarily verbose. You can make your life much easier with bash-completions, but if you write it tens of times per day, it&rsquo;s anyway a lot of typing&hellip; and I&rsquo;m a lazy man &#x1f604;
  Simple status/log checks git s s = status --short --branch --untracked-files Shows a short, branch-focused status with untracked files.
tags:
  - git
  - alias
slurped: 2026-01-26T19:56
title: Git hacks - a set of my favorite git aliases
---

I use Git a lot, even writing this article i will commit text few times. There’s a set of aliases I rely on daily and they’re first I add in new place.

Some Git commands are unnecessarily verbose. You can make your life much easier with `bash-completions`, but if you write it tens of times per day, it’s anyway a lot of typing… and I’m a lazy man 😄

## Simple status/log checks

### **git s**

```
  s = status --short --branch --untracked-files
```

Shows a short, branch-focused status with untracked files.

  

---

### **git tags**

```
  tags = !sh -c 'git tag -n1 | sort -V'
```

Lists tags along with their annotations, sorted by the version.

  

---

### **git graph**

```
  graph = log --graph --oneline --all
```

Displays a compact graph of the commit history.

  

---

### **git tree**

```
  tree = log --graph --decorate --pretty=oneline --abbrev-commit
```

Displays a tree-like view of the commit history with decorations.

  

## Saving progress, committing

I rarely use `git commit -a -m ...`. I have simpler ways to do it

### **git jira**

```
  jira = !"f() { git rev-parse --abbrev-ref HEAD | sed -n -E 's#^(feature|(bug|hot)-?(fix)?)/([A-Z]+-[0-9]+)[^a-zA-Z0-9].*#\\4#p' ; }; f"
```

Extracts the Jira ticket number from the current branch name. I use it to feed `git cm`, as some of the projects I work require Jira ticket ID on every commit.

```
git switch -c feature/ABC-123-awesome-stuff
git jira
ABC-123
```

  

---

### **git cm**

```
  # cm = !git add -A && git commit -m
  # cm = !"f() { if echo \"$1\" | egrep -q '^[A-Z]+-[0-9]+ '; then git add -A && git commit -m \"$1\"; else JIRA=$(git jira); if [ -z "$JIRA" ]; then echo >&2 '#### Start message with Jira ticket number! ####'; exit 1; else git add -A && git commit -m \"$JIRA $1\"; fi; fi; }; f"
  cm = !"f() { SCOPE=''; JIRA=''; if echo \"$1\" | egrep -q -i '^(BUILD|REVISION|PATCH|MINOR|MAJOR) '; then SCOPE=$(echo \"$1\" | sed -n -E 's#^(BUILD|REVISION|PATCH|MINOR|MAJOR).*#\\1#p'); fi; if echo \"$1\" | egrep -q '^((BUILD|REVISION|PATCH|MINOR|MAJOR)[ ]+)?[A-Z]+-[0-9]+ '; then JIRA=$(echo \"$1\" | sed -n -E 's#^((BUILD|REVISION|PATCH|MINOR|MAJOR)[ ]+)?([A-Z]+-[0-9]+).*#\\3#p'); else JIRA=$(git jira); if [ -z "$JIRA" ]; then  echo >&2 '#### Start message with Jira ticket number! ####';  exit 1;  fi;  fi; COMMIT_MESSAGE=$(echo \"$1\" | sed -n -E 's#^((BUILD|REVISION|PATCH|MINOR|MAJOR)[ ]+)?([A-Z]+-[0-9]+)?[ ]?##p'); if [ -z "$SCOPE" ]; then  git add -A && git commit -m \"$JIRA $COMMIT_MESSAGE\";  else git add -A && git commit -m \"$SCOPE $JIRA $COMMIT_MESSAGE\"; fi; }; f"
```

Commits changes with an optional Jira ticket number and scope in the message. I started with the first one, but as some corpo requirements forced me to prefix each message with a ticket ID, I hacked it 😄

```
git cm "Just the commint message"

# But the actual result is:

git tree | tail -n1
* 1c7dfed ABC-123 Just the commint message
```

  

---

### **git save**

```
  save = !git add -A && git commit -m "SAVEPOINT"
```

Adds all changes and commits with a “SAVEPOINT” message.

  

---

### **git wip**

Commits all changes with a “WIP” (Work In Progress) message.

  

## Branch management and dirty reverts

### **git co**

Shortcut for `git checkout`. I prefer `git switch` recently.

  

---

### **git cob**

Creates a new branch and checks it out. To be honest, I recently more often use `git switch -c`.

  

---

### **git undo**

```
  undo = reset HEAD~1 --mixed
```

Undoes the last commit, keeping changes in the working directory.

  

---

### **git amend**

```
  amend = commit -a --amend
```

Amends the last commit with any changes in the working directory.

  

---

### **git wipe**

```
  wipe = !git add -A && git commit -qm 'SAVEPOINT before WIPE' && git reset HEAD~1 --hard
```

Creates a ‘SAVEPOINT’ commit before resetting the branch to the previous commit.

  

---

### **git dropmerged**

```
  dropmerged = "!git branch --merged | grep -v '* ' | xargs git branch -d; git checkout -q master && git for-each-ref refs/heads/ \"--format=%(refname:short)\" | while read branch; do mergeBase=$(git merge-base master $branch) && [[ $(git cherry master $(git commit-tree $(git rev-parse \"$branch^{tree}\") -p $mergeBase -m _)) == \"-\"* ]] && git branch -D $branch; done"
```

Deletes branches that have been merged into master and have no unmerged changes. It can even recognize squash merges.

I love this one. I often have 5+ branches on different projects and when I get back to them, I don’t remember which stuff is still useful. I call this command and only uncommitted stuff is left.

  

## Bitbucket helpers

I work with Bitbucket a lot, so it’s helpful to jump from the code into repo or pull requests page quickly.

### **git bb**

```
  bb = ! open $(git config --get remote.origin.url | sed 's/^ssh:\\/\\/git@\\([^\\/]*\\)\\/\\([^\\/]*\\)\\/\\([^\\/]*\\).git/https:\\/\\/\\1\\/projects\\/\\2\\/repos\\/\\3\\/browse/')
```

Opens Bitbucket repository in the default web browser.

  

---

### **git prs**

```
  prs = ! open $(git config --get remote.origin.url | sed 's/^ssh:\\/\\/git@\\([^\\/]*\\)\\/\\([^\\/]*\\)\\/\\([^\\/]*\\).git/https:\\/\\/\\1\\/projects\\/\\2\\/repos\\/\\3\\/pull-requests/')
```

Opens Bitbucket Pull Requests page in the default web browser.

  

## Voodoo/Black magic

Warning

Those are helpful in some situations, but might be dangerous or made your palls angry. You’ve been warned 😉

```
  authorupdate = !git filter-branch -f --env-filter "GIT_AUTHOR_NAME='My Name'; GIT_AUTHOR_EMAIL='my-mail@github.com'; GIT_COMMITTER_NAME='My Name'; GIT_COMMITTER_EMAIL='my-mail@github.com';" HEAD && git push origin master --force
```

Updates the author information for all commits and force-pushes to master.

One time I wanted to unify commits on one repo, that I’ve done from multiple computers with different names and emails. This hack allowed me to achieve it.

Yes, it allows to overwrite with any name. That’s why it’s worth to sign your commits 😄

  

## My whole `.gitconfig` [alias]’es section

For those as lazy as I’m, my config file. Just edit your `~/.gitconfig` file and add aliases you like.

```
[alias]
  retag = "!f() { git tag -d $1 && git push origin :refs/tags/$1 && git tag -a $1 -m \"$1\" && git push && git push --tags ; }; f"
  s = status --short --branch --untracked-files
  tags = !sh -c 'git tag -n1 | sort -V'
  co = checkout
  cob = checkout -b
  jira = !"f() { git rev-parse --abbrev-ref HEAD | sed -n -E 's#^(feature|(bug|hot)-?(fix)?)/([A-Z]+-[0-9]+)[^a-zA-Z0-9].*#\\4#p' ; }; f"
  cm = !git add -A && git commit -m
  cm = !"f() { if echo \"$1\" | egrep -q '^[A-Z]+-[0-9]+ '; then git add -A && git commit -m \"$1\"; else JIRA=$(git jira); if [ -z "$JIRA" ]; then echo >&2 '#### Start message with Jira ticket number! ####'; exit 1; else git add -A && git commit -m \"$JIRA $1\"; fi; fi; }; f"
  save = !git add -A && git commit -m 'SAVEPOINT'
  wip = commit -am "WIP"
  undo = reset HEAD~1 --mixed
  amend = commit -a --amend
  wipe = !git add -A && git commit -qm 'SAVEPOINT before WIPE' && git reset HEAD~1 --hard
  graph = log --graph --oneline --all
  tree = log --graph --decorate --pretty=oneline --abbrev-commit
  authorupdate = !git filter-branch -f --env-filter "GIT_AUTHOR_NAME='My Name'; GIT_AUTHOR_EMAIL='my-mail@github.com'; GIT_COMMITTER_NAME='My Name'; GIT_COMMITTER_EMAIL='my-mail@github.com';" HEAD && git push origin master --force
  dropmerged = "!git branch --merged | grep -v '* ' | xargs git branch -d; git checkout -q master && git for-each-ref refs/heads/ \"--format=%(refname:short)\" | while read branch; do mergeBase=$(git merge-base master $branch) && [[ $(git cherry master $(git commit-tree $(git rev-parse \"$branch^{tree}\") -p $mergeBase -m _)) == \"-\"* ]] && git branch -D $branch; done"
  bb = ! open $(git config --get remote.origin.url | sed 's/^ssh:\\/\\/git@\\([^\\/]*\\)\\/\\([^\\/]*\\)\\/\\([^\\/]*\\).git/https:\\/\\/\\1\\/projects\\/\\2\\/repos\\/\\3\\/browse/')
  prs = ! open $(git config --get remote.origin.url | sed 's/^ssh:\\/\\/git@\\([^\\/]*\\)\\/\\([^\\/]*\\)\\/\\([^\\/]*\\).git/https:\\/\\/\\1\\/projects\\/\\2\\/repos\\/\\3\\/pull-requests/')
```

Feel free to share in comments if you have done it better, or which might be a nice addition to my collection.