---
tags:
  - git
  - options
---
[source](https://git-scm.com/docs/git-pull/2.25.1)

--ff

--no-ff

--ff-only

Specifies how a merge is handled when the merged-in history is already a descendant of the current history. `--ff` is the default unless merging an annotated (and possibly signed) tag that is not stored in its natural place in the `refs/tags/` hierarchy, in which case `--no-ff` is assumed.

With `--ff`, when possible resolve the merge as a fast-forward (only update the branch pointer to match the merged branch; do not create a merge commit). When not possible (when the merged-in history is not a descendant of the current history), create a merge commit.

With `--no-ff`, create a merge commit in all cases, even when the merge could instead be resolved as a fast-forward.

With `--ff-only`, resolve the merge as a fast-forward when possible. When not possible, refuse to merge and exit with a non-zero status.

--rebase[=false|true|merges|preserve|interactive]

When true, rebase the current branch on top of the upstream branch after fetching. If there is a remote-tracking branch corresponding to the upstream branch and the upstream branch was rebased since last fetched, the rebase uses that information to avoid rebasing non-local changes.

When set to `merges`, rebase using `git rebase --rebase-merges` so that the local merge commits are included in the rebase (see [git-rebase[1]](https://git-scm.com/docs/git-rebase) for details).

When set to `preserve` (deprecated in favor of `merges`), rebase with the `--preserve-merges` option passed to `git rebase` so that locally created merge commits will not be flattened.

When false, merge the current branch into the upstream branch.

When `interactive`, enable the interactive mode of rebase.

See `pull.rebase`, `branch.<name>.rebase` and `branch.autoSetupRebase` in [git-config[1]](https://git-scm.com/docs/git-config) if you want to make `git pull` always use `--rebase` instead of merging.

|   |   |
|---|---|
|Note|This is a potentially _dangerous_ mode of operation. It rewrites history, which does not bode well when you published that history already. Do **not** use this option unless you have read [git-rebase[1]](https://git-scm.com/docs/git-rebase) carefully.|