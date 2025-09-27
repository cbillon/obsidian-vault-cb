---
link: https://alchemists.io/articles/git_configuration
byline: Alchemists
excerpt: A collective devoted to the craft of software engineering where expertise is transmuted into joy.
slurped: 2025-09-13T15:38
title: Git Configuration | Alchemists
tags:
  - git
  - configuration
---

it Configuration

I’ve written about [Git](https://git-scm.com/) several times in the past but haven’t spent much time talking about how to properly configure your global configuration. Maintaining a global configuration allows your local configurations to inherit common defaults while improving consistency across all managed repositories. In this article, we’ll focus on a [Git Rebase](https://alchemists.io/articles/git_rebase) workflow while leveraging [XDG](https://alchemists.io/articles/xdg_base_directory_specification) to manage our global Git configuration. OK, let’s jump in!

Table of Contents

- [Overview](https://alchemists.io/articles/git_configuration#_overview)
- [Details](https://alchemists.io/articles/git_configuration#_details)
- [Include If](https://alchemists.io/articles/git_configuration#_include_if)
- [Conclusion](https://alchemists.io/articles/git_configuration#_conclusion)


```

[advice]
  detachedHead = false
[branch]
  autoSetupRebase = always
[commit]
  gpgSign = true
  template = ~/.config/git/commit_message.txt
  verbose = true
[core]
  abbrev = 12
  editor = "$EDITOR --wait"
  fsmonitor = false
  hooksPath = ~/.config/git/hooks
  pager = delta
  quotePath = false
  untrackedCache = true
  whitespace = fix,-indent-with-non-tab,trailing-space,cr-at-eol
[color]
  pager = true
  ui = true
[color "branch"]
  current = yellow reverse
  local = yellow
  remote = green
[color "diff"]
  commit = 227 bold
  frag = magenta bold
  meta = 227
  new = green bold
  old = red bold
  whitespace = red reverse
[color "diff-highlight"]
  newHighlight = green bold 22
  newNormal = green bold
  oldHighlight = red bold 52
  oldNormal = red bold
[color "status"]
  added = yellow
  changed = green
  untracked = cyan
[credential]
  helper = cache --timeout=3600
[delta]
  commit-decoration-style = bold yellow box ul
  file-style = bold yellow ul
  hunk-header-decoration-style = yellow box
  line-numbers = true
  minus-color = "#340001"
  plus-color = "#012800"
  side-by-side = true
  whitespace-error-style = 22 reverse
[diff]
  algorithm = histogram
  colorMoved = default
  dstPrefix = after
  indentHeuristic = true
  mnemonicPrefix = true
  renames = copies
  srcPrefix = before
  tool = difftastic
[diff "exif"]
  textconv = exiftool
[difftool]
  prompt = false
[difftool "difftastic"]
  cmd = difft "$LOCAL" "$REMOTE"
[feature]
  experimental = true
[fetch]
  prune = true
  pruneTags = true
  writeCommitGraph = true
[gpg "ssh"]
  allowedSignersFile = ~/.ssh/allowed_signers
[grep]
  column = true
  fullname = true
  lineNumber = true
[init]
  defaultBranch = main
  templateDir = ~/.config/git/template
[interactive]
  diffFilter = delta --color-only
  singleKey = true
[maintenance]
  auto = false
  strategy = incremental
[merge]
  conflictStyle = zdiff3
  ff = only
  tool = difftastic
[mergetool]
  prompt = false
[mergetool "difftastic"]
  cmd = difft "$LOCAL" "$REMOTE"
  trustExitCode = true
[notes]
  rewriteRef = refs/notes/commits
[pack]
  useBitmapBoundaryTraversal = true
[pager]
  difftool = true
[pull]
  rebase = merges
[push]
  autoSetupRemote = true
  default = simple
  followTags = true
  useForceIfIncludes = true
[rebase]
  abbreviateCommands = true
  autoSquash = true
  autoStash = true
  updateRefs = true
[remote "origin"]
  fetch = refs/pull/*/head:refs/remotes/pull_requests/*
  fetch = refs/stashes/*:refs/remote/origin/stashes/*
[rerere]
  autoUpdate = true
  enabled = true
[revert]
  reference = true
[stash]
  showIncludeUntracked = true
  showPatch = true
  showStat = false
[status]
  showUntrackedFiles = all
[tag]
  gpgSign = true
  sort = version:refname
[transfer]
  fsckObjects = true
[url "https://github.com/bkuhlmann/"]
  insteadOf = bk:
[url "https://github.com/dry-rb/dry-"]
  insteadOf = dry:
[url "https://github.com/"]
  insteadOf = gh:
[url "https://github.com/hanami/"]
  insteadOf = hanami:
[url "https://git.heroku.com"]
  insteadOf = heroku:
[url "https://github.com/rom-rb/rom-"]
  insteadOf = rom:
[url "https://github.com/ruby/"]
  insteadOf = rb:
[user]
  email = <redacted>
  name = <redacted>
  signingKey = <redacted>
  useConfigOnly = true
[includeIf "gitdir:~/Engineering/Companies/Example/"]
  path = ~/.config/git/profiles/example
```

## Overview

Below is my current global configuration with a few edits made for security reasons:

```
[advice]
```The first thing you might notice with the above configuration is that each section — denoted by the use of brackets (i.e. `[]`) — is alphabetically sorted. Only the `includeIf` section is listed last due to order of precedence but we’ll get to that later. For simplicity, I’m going to document each key in the configuration using Git’s namespace dot notation which means:

```
[advice]
  detachedHead = false
```

…​becomes `advice.detachedHead = false`. This is roughly the same syntax you’d use if wanting to immediately adjust your global configuration via the command line:

```
git config --global advice.detachedHead false
```

With the above in mind, let’s walk through each setting so you can learn how they work and determine if you’d like to apply these changes to your own configuration. Also, keep in mind that the official documentation for all of these settings can be found via Git’s own [documentation](https://git-scm.com/docs/git-config). Anyway, here’s the breakdown:

- `advice.detachedHead = false`: When set to `true` (default), this will print help text each time you use `git switch` or `git checkout` to move into a detached HEAD state. This can quickly become annoying when doing this often so I’ve permanently disabled it to reduce verbosity.
    
- `branch.autoSetupRebase = always`: Ensures new branches are automatically configured to use a [Git Rebase](https://alchemists.io/articles/git_rebase) workflow.
    
- `commit.gpgSign = true`: Ensures all commits are signed for improved security and author verification. See the [Milestoner](https://alchemists.io/projects/milestoner/#_security) documentation for further details on how to setup and configure GPG if you haven’t already.
    
- `commit.template = ~/.config/git/commit_message.txt`: References my [Dotfiles](https://alchemists.io/projects/dotfiles) template in a [XDG](https://alchemists.io/articles/xdg_base_directory_specification) structure which provides helpful text and a reminder checklist when creating a new commit message.
    
- `commit.verbose = true`: Shows diff of changes, by file, at the bottom of the commit message which helps when composing the commit message. The diff is only for documentation purposes and isn’t included once the commit message is saved.
    
- `core.abbrev = 12`: Uses only the first 12 characters of a commit SHA instead of the full 40 characters which saves screen real-estate while still being uniquely identifiable.
    
- `core.editor = "$EDITOR --wait"`: Instructs my default editor to wait for me to edit and save my commit message. Without this being set, I’d never be able to finish a commit message, rebase, etc. I use `EDITOR` as a global environment variable which provides the path to my [Sublime Text](https://alchemists.io/projects/sublime_text_setup) IDE.
    
- `core.fsmonitor = false`: Enables Git’s built-in file system monitor daemon which will watch all projects for changes and speed up commands that use the index. Useful for large repositories but there is a significant performance hit for small projects especially when checking the status of all projects because if the daemon is not running it will have to be started first.
    
- `core.hooksPath = ~/.config/git/hooks`: Tells Git where to look for my [Dotfiles](https://alchemists.io/projects/dotfiles) managed Git Hooks [XDG](https://alchemists.io/articles/xdg_base_directory_specification) configuration. This is one of the most powerful aspects of Git because it allows you to wire in additional behavior to automate your workflow and improve your performance.
    
- `core.pager = delta`: Tells Git to use the [Delta](https://github.com/dandavison/delta) program for pagination purposes as installed via my [macOS Configuration](https://alchemists.io/projects/mac_os-config).
    
- `core.quotePath = false`: Ensures path output is not quoted and escaped by default.
    
- `core.untrackedCache = true`: Ensures any untracked cache is added to your index so you don’t lose information.
    
- `core.whitespace = fix,-indent-with-non-tab,trailing-space,cr-at-eol`: Defines how to deal with whitespace problems in your diff. You generally want this comma delimited list but I’ll refer you to the Git documentation since it details each value.
    
- `color.pager = true`: Ensures output is colorized for improved readability.
    
- `color.ui = true`: Enables color for all commands who support colorized output.
    
- `color.branch.current = yellow reverse`: Shows current branch using a yellow background with foreground text in a reverse color for improved readability contrast.
    
- `color.branch.local = yellow`: Shows local branches in yellow foreground text.
    
- `color.branch.remote = green`: Shows remote branches in green foreground text.
    
- `color.diff.*`: I won’t individually list these but know this is how you customize the color for diffs. In my case, all colors are defined with the assumption of reading in a black console.
    
- `color.diff-highlight.*`: Similar to the above, these are the colors used for diff highlights.
    
- `color.status.added = yellow`: Shows added status in yellow color.
    
- `color.status.changed = green`: Shows changed status in green color.
    
- `color.status.untracked = cyan`: Shows untracked status in cyan color.
    
- `credential.helper = cache --timeout = 3600`: Instructs your credential helper for username/password information to cache for 1 hour to reduce strain when needing to resupply credentials.
    
- `delta.*`: I won’t walk through each [Delta](https://github.com/dandavison/delta) setting used in my configuration but refer you to Delta’s documentation instead since that is where I sourced these settings to begin with and you might want to take as is or customize further.
    
- `diff.algorithm = histogram`: Defines the `histogram` diffing algorithm which further enhances the `patience` algorithm to support low occurrence common elements for better diffing analysis that helps speed up your reading time.
    
- `diff.colorMoved = default`: Determines the color of moved lines in a diff. In this case, the defaults are used.
    
- `dstPrefix = after`: Provide more information when looking at diffs instead of seeing only `b/` for the comparison.
    
- `diff.indentHeuristic = true`: Ensures the heuristics for determining diff hunk boundaries are easier to read.
    
- `diff.mnemonicPrefix = true`: Ensures the the default `a/` and `b/` diff prefixes are dropped in favor of using `i/` (index) and `w/` (working copy) which are more informative than alphabetic letters.
    
- `diff.renames = copies`: Ensures Git not only detects file renames but also file copies where the code is the same but the file has been moved to a different location for improved diff reporting.
    
- `diff.srcPrefix = before`: Provide more information when looking at diffs instead of seeing only `a/` for the comparison.
    
- `diff.tool = difftastic`: Defines your favorite external diff tool. In my case, I use [Difftastic](https://difftastic.wilfred.me.uk/) as found via my [macOS Configuration](https://alchemists.io/projects/mac_os-config) project.
    
- `diff.exif.textConv = exiftool`: Defines which tool to use when converting binary files, like images, so you can see metadata information. I use the [ExifTool](https://exiftool.org/) as found via my [macOS Configuration](https://alchemists.io/projects/mac_os-config) project.
    
- `difftool.prompt = false`: Disables the annoying confirmation prompt when launching your custom diff tool.
    
- `difftool.difftastic.cmd = cmd = difft "$LOCAL" "$REMOTE"`: Defines how to execute [Difftastic](https://difftastic.wilfred.me.uk/). See the Difftastic CLI documentation for further information.
    it Configuration
I’ve written about Git several times in the past but haven’t spent much time talking about how to properly configure your global configuration. Maintaining a global configuration allows your local configurations to inherit common defaults while improving consistency across all managed repositories. In this article, we’ll focus on a Git Rebase workflow while leveraging XDG to manage our global Git configuration. OK, let’s jump in!

Table of Contents
Overview
Details
Include If
Conclusion
Overview
Below is my current global configuration with a few edits made for security reasons:

[advice]
- `feature.experimental = true`: Enables config options that are experimental/new but not fully supported or might change in the future. I like to live on the edge by using the latest and greatest Git has to offer. 😅
    
- `fetch.prune = true`: Ensures each fetch removes local objects and remote branches that no longer exist. Ensures you automatically have a clean local repository without manual maintenance.
    
- `fetch.pruneTags = true`: Ensures each fetch prunes remote tags that no longer exist for a clean tag history you don’t have to manually maintain.
    
- `fetch.writeCommitGraph = true`: Informs `git gc` to update the commit graph after each fetch for improved performance of several Git commands (for example: `git log`). This is less demanding than a full GC operation for non-trivial garbage collection.
    
- `gpg.ssh.allowedSignersFile = ~/.ssh/allowed_signers`: Defines the file path used to store SSH public keys that you trust.
    
- `grep.column = true`: Ensures column information is printed when searching for improved readability of output.
    
- `grep.fullName = true`: Ensures paths, relative to the top of your repository, are used instead of being relative to the current directory you are in for more actionable output.
    
- `grep.lineNumber = true`: Ensures line numbers are printed when searching.
    
- `init.defaultBranch = main`: Ensures `main` is always defined as the default branch.
    
- `init.templateDir = ~/.config/git/template`: Defines where my [XDG](https://alchemists.io/articles/xdg_base_directory_specification) configuration template is located for initializing new Git repositories with custom settings as provided by the [Dotfiles](https://alchemists.io/projects/dotfiles) project.
    
- `interactive.diffFilter = delta --color-only`: Ensures [Delta](https://github.com/dandavison/delta) is used to handle the diff filter for interactive commands.
    
- `interactive.singleKey = true`: Ensures only a single key is used without having to hit enter for interactive commands.
    
- `maintenance.auto = false`: Disables some Git commands from executing maintenance after completing their work. This allows me to control when maintenance mode is executed via my [Dotfiles](https://alchemists.io/projects/dotfiles) aliases/functions.
    
- `maintenance.strategy = incremental`: Ensures the maintenance strategy is incremental which optimizes for performing small activities only.
    
- `merge.conflictStyle = zdiff3`: Specifies the diff conflict style used. In this case, I’m able to see what is on HEAD, what is common, and what is local for improved diffing.
    
- `merge.ff = only`: Ensures no extra commit is made when merging. This helps ensure a clean Git history while also ensuring the merge can only be performed when there are no errors.
    
- `merge.tool = difftastic`: Defines [Difftastic](https://difftastic.wilfred.me.uk/) as my default merge tool.
    
- `mergetool.prompt = false`: Ensures there is no prompt when using a custom merge tool.
    
- `mergetool.difftastic.cmd = difft "$LOCAL" "$REMOTE"`: Informs Git how to use [Difftastic](https://difftastic.wilfred.me.uk/) for displaying merges.
    
- `mergetool.difftastic.trustexitcode = true`: Ensures the merge tool properly exits when given a non-zero exit code.
    
- `notes.rewriteRef = refs/notes/commits`: Ensures notes are not lost when rebasing.
    
- `pack.useBitmapBoundaryTraversal = true`: Ensures bitmaps are used for improved index traversal performance.
    
- `pager.difftool = true`: Enabled [Difftastic](https://difftastic.wilfred.me.uk/) pagination.
    
- `pull.rebase = merges`: Enforces a [Git Rebase](https://alchemists.io/articles/git_rebase) workflow instead of the default merge workflow when pulling new content from the remote repository. By using `merges` instead of `true`, you also gain the ability to rebase merge bubbles when working in a team environment that is unfortunately a mix of merging and rebasing.
    
- `push.autoSetupRemote = true`: When enabled, assumes that the current branch is set upstream (i.e. same name as branch) where remote tracking is enabled by default so you don’t have to type the current branch name each time you are pushing.
    
- `push.default = simple`: Works in conjunction with the above by using the same branch name locally and remotely.
    
- `push.followTags = true`: Ensures, when pushing, any tags not on the remote are pushed up.
    
- `push.useForceIfIncludes = true`: Ensures you can’t force push changes to remote branch if commit history doesn’t have same parent. This is extremely handy for preventing obliteration of remote changes if the histories don’t align especially in rare cases your local branch managed to get misaligned with the remote branch. See my [Git Rebase](https://alchemists.io/articles/git_rebase) article for details.
    
- `rebase.abbreviateCommands = true`: Ensures rebase editor commands are appreviated to a single letter for less typing. See my [Git Rebase](https://alchemists.io/articles/git_rebase) article for details.
    
- `rebase.autoSquash = true`: Ensures each rebase automatically moves `fixup!`, `squash!`, and `amend!` commits to theirit Configuration
I’ve written about Git several times in the past but haven’t spent much time talking about how to properly configure your global configuration. Maintaining a global configuration allows your local configurations to inherit common defaults while improving consistency across all managed repositories. In this article, we’ll focus on a Git Rebase workflow while leveraging XDG to manage our global Git configuration. OK, let’s jump in!

Table of Contents
Overview
Details
Include If
Conclusion
Overview
Below is my current global configuration with a few edits made for security reasons:

[advice] corresponding commit for reduced manual labor. See my [Git Rebase](https://alchemists.io/articles/git_rebase) article for details.
    
- `rebase.autoStash = true`: Ensures your untracked, modified, or staged files are automatically pushed onto the stash before a rebase and auto-popped after the rebase completes so you don’t have to manage this yourself. See my [Git Rebase](https://alchemists.io/articles/git_rebase) article for details.
    
- `rebase.updateRefs = true`: Ensures any branches, which references the same commits, are automatically rebased along with the current branch being rebased. Reduces the maintenance burden when working on multiple branches in parallel or chained together.
    
- `remote.origin.fetch = refs/pull/**/head:refs/remotes/pull_requests/**`: Enables remote tracking by fetching pull request references which can be used via a [Dotfiles](https://alchemists.io/projects/dotfiles) function for interacting with pull request information locally via an interactive prompt.
    
- `fetch = refs/stashes/**:refs/remote/origin/stashes/**`: Enables remote tracking by fetching remote stash references to aid in the [exporting](https://git-scm.com/docs/git-stash#Documentation/git-stash.txt-export%E2%80%94%E2%80%8Bprint%E2%80%94%E2%80%8Bto-refrefstash) and [importing](https://git-scm.com/docs/git-stash#Documentation/git-stash.txt-importcommit) of your stashes. _Ensure you use `remote` instead of `remotes` to avoid clashing with the default refspec for branches_.
    
- `rerere.autoUpdate = true`: Ensures the rerere cache is automatically updated when resolving a rebase conflict for improved rebase speed. See my [Git Rebase](https://alchemists.io/articles/git_rebase) article for details.
    
- `rerere.enabled = true`: Ensures Reuse Recorded Resolution (rerere) is enabled for an improved rebase workflow so you can reduce the number of times you have to resolve the same conflict when rebasing multiple times. See my [Git Rebase](https://alchemists.io/articles/git_rebase) article for details.
    
- `revert.reference = true`: Disables the verbose commit revert message and uses a formatted SHA for reference instead which you can describe with more useful information.
    
- `stash.showIncludeUntracked = true`: Ensures untracked changes are included when showing a stash.
    
- `stash.showPatch = true`: Ensures the stash is shown in patch form.
    
- `stash.showStat = false`: Disables showing stash stats so you can customize the format instead.
    
- `status.showUntrackedFiles = all`: Ensures all untracked files in all directories are shown so you have a better view.
    
- `tag.gpgSign = true`: Ensures tags are always cryptographically signed as done for commits (shown earlier).
    
- `tag.sort = version:refname`: Ensures repository tags are sorted by version. This is especially useful when using [Semantic Versioning](https://semver.org/).
    
- `transfer.fsckObjects = true`: Ensures fetch will abort when receiving a malformed object. This is great to have for repository integrity and when cloning new repositories that have been poorly maintained.
    
- `url.https://github.com/bkuhlmann/.insteadOf = bk:`: Allows you to use the `bk` prefix when cloning my GitHub repositories for less typing. Example: `git clone bk:infusible`.
    
- `url.https://github.com/dry-rb/dry-.insteadOf = dry:`: Allows you to use the `dry` prefix when cloning Dry RB repositories for less typing. Example: `git clone dry:monads`.
    
- `url.https://github.com/.insteadOf = gh:`: Allows you to use the `gh` prefix when cloning Github repositories for less typing. Example: `git clone gh:jill/demo`.
    
- `url.https://github.com/hanami/.insteadOf = hanami:`: Allows you to use the `hanami` prefix when cloning Hanami repositories for less typing. Example: `git clone hanami:views`.
    
- `url.https://git.heroku.com.insteadOf = heroku:`: Allows you to use the `heroku` prefix when cloning Heroku repositories for less typing. Example: `git clone heroku:demo`.
    
- `url.https://github.com/rom-rb/rom-.insteadOf = rom:`: Allows you to use the `rom` prefix when cloning ROM repositories for less typing. Example: `git clone rom:sql`.
    
- `url.https://github.com/ruby/.insteadOf = rb:`: Allows you to use the `rb` prefix when cloning Ruby repositories for less typing. Example: `git clone rb:logger`.
    
- `user.email = <redacted>`: Defines your global/default email address.
    
- `user.name = <redacted>`: Defines your global/default name.
    
- `user.signingKey = <redacted>`: Defines your GPG signing key (never share this publicly).
    
- `user.useConfigOnly = true`: Ensures Git never attempts to guess user email and name from the local system but uses your configuration instead.
    
- `includeif.gitdir:/Engineering/Companies/Example/.path=/.config/git/profiles/example`: This is an example of a powerful feature which allows you to specify a global configuration override based on the root directory you are currently in. You always want to define this at the bottom of your configuration because it is meant to override settings defined above it.