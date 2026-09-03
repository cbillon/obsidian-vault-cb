---
link: https://lalitm.com/post/git-history/
byline: Lalit Maganti
site: Lalit Maganti
date: 2026-07-13T20:40
excerpt: Working with lots of changes in parallel on git can be painful. You end
  up juggling branches and commits, and running scary rebase -i commands that
  can leave your tree in a half-broken state if you so …
slurped: 2026-08-24T08:20
title: The git history command deserves more attention
---

Working with lots of changes in parallel on git can be painful. You end up juggling branches and commits, and running scary `rebase -i` commands that can leave your tree in a half-broken state if you so much as sneeze.

[`jj`](https://github.com/jj-vcs/jj), an alternative to `git`, gets discussed a lot these days ([1](https://news.ycombinator.com/item?id=48692083), [2](https://lobste.rs/s/beqyuc/evan_s_jujutsu_tutorial), [3](https://lobste.rs/s/fg3sgh/jj_tui_terminal_user_interface_jujutsu), [4](https://lobste.rs/s/27dxjg/petty_reason_we_didn_t_end_up_using_jj)) and is often pitched as a solution. While I’m very sold on the _problems_ `jj` is trying to solve, the way it solves them hasn’t quite hit home with me. Every 3 months, for the last 1.5 years, I try it out for a few days, really trying to make it part of my workflow but eventually I give up and go back to git.[1](#fn:1)

That’s where `git history` comes in. It’s an **experimental** [command](https://git-scm.com/docs/git-history) that arrived across two releases, [2.54](https://github.com/git/git/blob/master/Documentation/RelNotes/2.54.0.adoc) (April, `reword` and `split` subcommands) and [2.55](https://github.com/git/git/blob/master/Documentation/RelNotes/2.55.0.adoc) (June, `fixup` subcommand). It got a flurry of attention on each release day, and then, as far as I can tell, not much community discussion since. Which is a shame, because IMO it already delivers several of the benefits people tout for `jj` without needing to switch your whole workflow. And the cool thing is that it’s part of the core git distribution, so you can try it without installing anything.

There are three subcommands: `fixup`, `reword` and `split`.

## fixup[#](#fixup)

[`git history fixup`](https://git-scm.com/docs/git-history#Documentation/git-history.txt-fixupcommit) fixes an old commit that has something wrong in it, then autorebases _all_ your branches to match.

You stage the fix as usual with `git add`, then run `git history fixup <commit>` to fold those staged changes into the target commit. It’s like a `git commit --fixup` plus an autosquash rebase but with the extra magic that it _also_ updates any other branch which contained that commit.

That last part goes further than `git rebase --update-refs`, which only moves refs sitting inside the range you’re actively rebasing. `git history` instead finds and rewrites _every_ local branch descended from the commit (while also having an option to limit it to only the current branch). On the other hand it does _not_ work in the presence of merge commits which, for some usages of git, is going to be a dealbreaker.

Here’s how it works in practice:

Before, with a fix staged for `B`:

 ![Commits A, B (buggy), C on feat-1 and D on feat-2 in a line, with a staged fix sitting on top of the tip D.](app://obsidian.md/diagrams/fixup-before-light.svg) ![Commits A, B (buggy), C on feat-1 and D on feat-2 in a line, with a staged fix sitting on top of the tip D.](app://obsidian.md/diagrams/fixup-before-dark.svg)

After `git history fixup B`:

 ![After fixup: A unchanged; B (now fixed), C and D rewritten with new hashes, branch tips feat-1 and feat-2 following.](app://obsidian.md/diagrams/fixup-after-light.svg) ![After fixup: A unchanged; B (now fixed), C and D rewritten with new hashes, branch tips feat-1 and feat-2 following.](app://obsidian.md/diagrams/fixup-after-dark.svg)

_`B*` is `B` with the fix folded in. Rewriting a commit gives it a new hash, so `C` and `D` are automatically re-created on top as `C*` and `D*`, and the `feat-1` and `feat-2` branch tips move with them._

The most important property, common to all three commands, is that it’s atomic: it never leaves your tree in a half-broken state. It manages this by refusing any operation that could produce a conflict.

To be clear, this is strictly _less_ powerful than `jj`. `jj` [treats conflicts as first class](https://docs.jj-vcs.dev/latest/conflicts/) so it can carry a conflicted state through a rebase and let you sort it out later. `git history` doesn’t do this yet _but_ the [docs](https://git-scm.com/docs/git-history#_limitations) leave the door open:

> “This limitation is by design as history rewrites are not intended to be stateful operations. The limitation can be lifted once (if) Git learns about first-class conflicts.”

So basically, this limitation may change in the future; excited to see if it does!

## reword[#](#reword)

[`git history reword`](https://git-scm.com/docs/git-history#Documentation/git-history.txt-rewordcommit) updates the commit message on an old commit and automatically rebases everything on top. This is very useful for going back and fixing commit messages when the design shifts as you iterate.

`git history reword <commit>` opens your editor with that commit’s existing message. You edit it, save, and the rest of the stack is rebuilt on top with the branches following along. It’s exactly like fixup but for commit messages instead of the tree contents.

Because it only changes a message, `reword` (like `split` later) never touches your index or working tree at all; it works purely on the commit graph. So both let you rewrite a commit on a branch you don’t have checked out without disturbing whatever you’re in the middle of.

Before:

 ![Commits A, B with message 'fix bug', and C on feat-1.](app://obsidian.md/diagrams/reword-before-light.svg) ![Commits A, B with message 'fix bug', and C on feat-1.](app://obsidian.md/diagrams/reword-before-dark.svg)

After `git history reword B`:

 ![After reword: A unchanged; B rewritten with message 'fix null deref in parser' and C rewritten on top.](app://obsidian.md/diagrams/reword-after-light.svg) ![After reword: A unchanged; B rewritten with message 'fix null deref in parser' and C rewritten on top.](app://obsidian.md/diagrams/reword-after-dark.svg)

_Only `B`’s message changes, but that still gives it a new hash, so `C` is rebuilt on top as `C*` and `feat-1` follows along._

## split[#](#split)

[`git history split`](https://git-scm.com/docs/git-history#Documentation/git-history.txt-splitcommit--pathspec) takes one commit and splits it into two, interactively picking what you care about from each. It’s the equivalent of `git add -p`, but without needing gymnastics with `git rebase`. I’ve found this to be the most specialized of the three, but invaluable when I need it.

Specifically, `git history split <commit>` drops you into a hunk-by-hunk prompt over that commit’s diff. The hunks you keep make up the first commit, the rest fall into the second.

Before, with `B` bundling two unrelated changes:

 ![Commit B bundling two unrelated changes, between A and C on feat-1.](app://obsidian.md/diagrams/split-before-light.svg) ![Commit B bundling two unrelated changes, between A and C on feat-1.](app://obsidian.md/diagrams/split-before-dark.svg)

After `git history split B`:

 ![After split: A unchanged; B split into B1 and B2, and C rewritten on top as C*.](app://obsidian.md/diagrams/split-after-light.svg) ![After split: A unchanged; B split into B1 and B2, and C rewritten on top as C*.](app://obsidian.md/diagrams/split-after-dark.svg)

_`B` becomes `B1` and `B2`, and `C` is rebuilt on top of the pair as `C*`._

## Conclusion[#](#conclusion)

Judging by how many people are using `jj`, I do think there’s still some key mental shift which I’m not yet making. And to be clear, `git history` doesn’t close the full gap: `jj` still gives you an [operation log with easy undo](https://docs.jj-vcs.dev/latest/operation-log/), [models your working copy as a commit](https://docs.jj-vcs.dev/latest/working-copy/), and can [carry conflicts through a rebase](https://docs.jj-vcs.dev/latest/conflicts/), none of which this is trying to do.

But for now, `git history` is a _big_ step forward in adopting many of the pieces that attract people to `jj`, and it’s already in the tool I use every day. And the way the documentation is written makes me hopeful that more improvements will be coming in upcoming releases!