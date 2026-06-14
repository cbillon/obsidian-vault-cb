---
link: https://tylercipriani.com/blog/2025/05/21/git-commits/
byline: Tyler Cipriani
excerpt: |-
  People should try to compare the quality of the kernel git logs with
  some other projects, and cry themselves to sleep.
slurped: 2026-06-01T08:14
title: Per-project git commit templates - Tyler Cipriani
tags:
  - git
---

> People should try to compare the quality of the kernel git logs with some other projects, and cry themselves to sleep.
> 
> – Linus Torvalds

I’ll never remember your project’s commit guidelines.

Every project insists on something different:

- [Conventional commits](https://www.conventionalcommits.org/en/v1.0.0/)
- [Problem/Solution format](https://zeromq.org/how-to-contribute/#write-good-commit-messages)
- [Gitmoji](https://gitmoji.dev/)
- The twisty maze of [trailers in the Linux Kernel](https://docs.kernel.org/process/submitting-patches.html#when-to-use-acked-by-cc-and-co-developed-by)

But [git commit templates](https://git-scm.com/docs/git-config#Documentation/git-config.txt-codecommittemplatecode) help. Commit templates provide a scaffold for commit messages, offering documentation where you need it: inside the editor where you’re writing your commit message.

## What is a git commit template?

When you type `git commit`, git pops open your text editor[1](https://tylercipriani.com/blog/2025/05/21/git-commits/#fn1). Git can pre-fill your editor with a commit template—it’s like a form you fill out.

Creating a commit template is simple.

- Create a plaintext file – mine lives at `~/.config/git/message.txt`
- Tell git to use it:

```
git config --global \
    commit.template '~/.config/git/message.txt'
```

[My default template](https://gist.github.com/thcipriani/5e79a11b69b66a493249bac1439bfe02) packs everything I know about writing a commit.

## Project-specific templates with `IncludeIf`

The real magic of commit templates is you can have different templates for each project.

Different projects can use different templates with git’s `includeIf` configuration setting.[2](https://tylercipriani.com/blog/2025/05/21/git-commits/#fn2)

Large projects, such as [the Linux kernel](https://web.git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/SubmittingPatches?id=4e8a2372f9255a1464ef488ed925455f53fbdaa1), [git](https://git-scm.com/docs/SubmittingPatches#describe-changes), and [MediaWiki](https://www.mediawiki.org/wiki/Gerrit/Commit_message_guidelines), have their own commit guidelines.

For Wikimedia work, I stow git repos in `~/Projects/Wikimedia` and at the bottom of my global git config (`~/.config/git/config`) I have:

```
[includeIf "gitdir:~/Projects/Wikimedia/**"]
    path = ~/.config/git/config.wikimedia
```

In `config.wikimedia`, I point to my Wikimedia-specific commit template. I also override other git config settings like my `user.email` or `core.hooksPath`.

## An example: my global template

My default commit template contains three sections:

1. Subject – 50 characters or less, capitalized, no end punctuation.
2. Body – Wrap at 72 characters with a blank line separating it from the subject.
3. Trailers – Standard formats with a blank line separating them from the body.

In each section, I added pointers for both format[3](https://tylercipriani.com/blog/2025/05/21/git-commits/#fn3) and content.

For the header, the guidance is quick:

```

# 50ch. wide ----------------------------- SUBJECT
#                                                |
#     "If applied, this commit will..."          |
#                                                |
#     Change / Add / Fix                         |
#     Remove / Update / Document                 |
#                                                |
# ------- ↓ LEAVE BLANK LINE ↓ ---------- /SUBJECT
```

For the body, I remind myself to answer basic questions:

```
# 72ch. wide ------------------------------------------------------ BODY
#                                                                      |
#     - Why should this change be made?                                |
#       - What problem are you solving?                                |
#       - Why this solution?                                           |
#     - What's wrong with the current code?                            |
#     - Are there other ways to do it?                                 |
#     - How can the reviewer confirm it works?                         |
#                                                                      |
```

And that’s it, except for git trailers.

## The twisty maze of git trailers

My template has a section for trailers used by the projects I work on.

```
#     TRAILERS                                                         |
#     --------                                                         |
#     (optional) Uncomment as needed.                                  |
#     Leave a blank line before the trailers.                          |
#                                                                      |
# Bug: #xxxx
# Acked-by: Example User <user@example.com>
# Cc: Example User <user@example.com>
# Co-Authored-by: Example User <user@example.com>
# Requested-by: Example User <user@example.com>
# Reported-by: Example User <user@example.com>
# Reviewed-by: Example User <user@example.com>
# Suggested-by: Example User <user@example.com>
# Tested-by: Example User <user@example.com>
# Thanks: Example User <user@example.com>
```

These trailers serve as useful breadcrumbs of documentation. Git can parse them using standard commands.

For example, if I wanted a tab-separated list of commits and their related tasks, I could find `Bug` trailers using `git log`:

```
$ TAB=%x09
$ BUG_TRAILER='%(trailers:key=Bug,valueonly=true,separator=%x2C )'
$ SHORT_HASH=%h
$ SUBJ=%s
$ FORMAT="${SHORT_HASH}${TAB}${BUG_TRAILER}${TAB}${SUBJ}"
$ git log --topo-order --no-merges \
      --format="$FORMAT"
d2b09deb12f     T359762 Rewrite Kurdish (ku) Latin to Arabic converter
28123a6a262     T332865 tests: Remove non-static fallback in HookRunnerTestBase
4e919a307a4     T328919 tests: Remove unused argument from data provider in PageUpdaterTest
bedd0f685f9             objectcache: Improve `RESTBagOStuff::handleError()`
2182a0c4490     T393219 tests: Remove two data provider in RestStructureTest
```

## Stop remembering commit message guidelines

Git commit templates free your brain from remembering what to write, allowing you to focus on the story you need to tell.

Save your brain for what it’s good at.

---

1. Starting with `core.editor` in your git config, `$VISUAL` or `$EDITOR` in your shell, finally falling back to `vi`.[↩︎](https://tylercipriani.com/blog/2025/05/21/git-commits/#fnref1)
    
2. You could also set it inside a repo’s `.git/config`, `includeIf` is useful if you have multiple repos with the same standards under one directory.[↩︎](https://tylercipriani.com/blog/2025/05/21/git-commits/#fnref2)
    
3. All cribbed from [Tim Pope](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)[↩︎](https://tylercipriani.com/blog/2025/05/21/git-commits/#fnref3)