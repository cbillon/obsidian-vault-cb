---
link: https://blog.tjll.net/shell-kung-fu/
byline: Tyler Langlois
site: Tyblog
excerpt: Level up your shell game by going beyond the basics.
slurped: 2026-06-03T09:51
title: Shell Kung Fu
tags:
  - bash
---

### [«](https://blog.tjll.net/distributed-homelab-cluster/) Shell Kung Fu [»](https://blog.tjll.net/the-best-of-2019/)

- 5 January, 2019
- 1,508 words
- 6 minute read time

[My blog post about ssh](https://blog.tjll.net/ssh-kung-fu/) is still the most frequently read content on my blog four years later. I've collected enough shell tricks that it's about time for one of these type of posts about my favorite software tool of all time: the shell.

My experience is mostly limited to zsh, so while the examples here will be zsh-based, I think other shells like fish and xonsh are great as well.

#### Upgrade Your Shell

Bash is fine, but at a certain point my opinion is that you should upgrade to either [fish](https://fishshell.com/) or [zsh](http://www.zsh.org/). Why bother?

- Personally, my number one reason is proper multi-session history handling. If you issue the command `echo yo` in one terminal and hit the up arrow another terminal window, it's not ever going to show up in that other shell's history until a logout or flush event occurs in the first terminal. A modern shell like zsh or fish will actively append commands to the history file so they're available across all sessions. This becomes more important when you shift to leveraging history heavily.
- Syntax highlighting makes writing extended pipelines and control structures far easier. fish ships with this out-of-the box, and it's easy to add to a zsh setup.
- There's lots of small quality-of-life niceties - for example, when reviewing history (up key) zsh won't repeat duplicate commands, and tab-completing filenames can occur with any substring of a file name, not just the prefix.

Here's an example of syntax highlighting in zsh. Note how each of the following is colorized: loop keywords (`for`, `done`, etc.), strings, variables, and found versus unfound commands (they'll transition from red to green to indicate it's in my `$PATH`):

And just for completeness, here's what tab-completing a substring that appears anywhere in a filename looks like:

These are some great features, and particularly in the case of zsh, POSIX compatibility means that there's lots of crossover for various tools that were originally built for bash. However, the number one argument I hear from people when there's a suggestion to use something not-`bash` is:

> If I stop using bash, I'll get rusty with the shell when I'm working over ssh or in a different environment (or, I'll feel crippled in anything other than zsh/fish).

Let me get ahead of this argument with my direct experience: I've used zsh exclusively for _many_ years and have noticed zero knowledge degradation (and zero frustration) when I drop into a bash shell for some reason. Do yourself a favor and don't artificially hamstring yourself with the "but I'll suddenly forget how to use bash" argument.

#### Loops, Loops Everywhere

One of the most important pieces of the shell that's important to learn is that at they're essentially a language [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop). Empowering your use of the shell with variables, loops, and functions is hugely beneficial and not that hard to get started with.

Here's the simplest scenario: `ssh` into four hosts (successively named bastion1, bastion2, etc.) and check system load.

shell

```
for host in bastion{1..4} ; do ssh $host w ; done
```

In case you haven't seen it before, shell expansion (`{1..4}`) permits you to expand number ranges or sets of strings without typing it all out - here's a similar example doing the same for bastions 1-4 in addition to four machines with the name `nat` as well:

shell

```
for host in {nat,bastion}{1..4} ; do ssh $host w ; done
```

For what it's worth, you can tinker with expansion by testing it out with `echo` as well (i.e., `echo {1..4}`). Moving on.

`for` is what I reach for most often, but have you tried `until`? It's particularly well-suited for the shell where a failing command can indicate that you want to retry something.

shell

```
until ping -c 1 192.168.1.100 ; do sleep 5 ; done ; echo Host is up
```

That'll keep attempting to ping an IP address until it responds to an ICMP ping and sleep between attempts. You can get fancy with this sort of thing with an attempt counter to backoff, etc.

Here's where things get more interesting. Did you know that you can pipe standard input _into_ loops? Here's one I used today:

shell

```
gluster volume list | while read -r volume ; do echo ----- $volume ----- ; gluster volume heal $volume info summary ; done | less
```

This also means that you can pipe _from_ loops _into_ other loops for maximum pipeline overdrive. Technically you can get a similar effect by nesting `$(command)` substitutions into for loops, but this ends up being a lot easier sometimes when you're building up pipelines incrementally.

#### Process Substitution

This is another niche use case, but it comes in really handy when you need it.

Say that you have a command that expects a file as an argument, but the input you want to provide is the output of a command. Being able to provide that file input from a _command_ can be useful.

Here's an example: I sometimes need to quickly diff what may be running on remote systems. Using process substitution, you can diff output of `ssh` commands:

shell

```
diff <(ssh host1 systemctl list-unit-files) <(ssh host2 systemctl list-unit-files)
```

`diff` will operate on the output of those `ssh` commands as if they were files. This applies to any command that accepts files as arguments, so you can get really creative for a variety of use cases. For example, you can retrieve a json response from `curl` and forward it along to another server with `curl -XPOST` something like `-d @<(curl ...)`. In my experience, not all CLI utilities will transparently accept temporary file handles (which is what `<()` constructs), but the majority do.

#### Fuzzy Searching

This is the most impactful change I've implemented over the past couple of years. If you're looking for the most immediate win in shell workflows, I'd suggest starting here.

In short: instead of hitting up repeatedly to re-use commands or use basic Ctrl-R reverse searching, [fzf](http://www.zsh.org/) re-imagines what it means to fuzzy search for commands or files at the shell. Coupled with a sufficiently broad and generalized shell history, I find that maybe 50% of the commands I enter day-to-day in my terminal are just reissued commands that already exist in my history. Fuzzy searching means that finding them is very natural and easy. You can see some examples on the [fzf examples](https://github.com/junegunn/fzf/wiki/examples) page. I'll cover fuzzy history search first, but you can repurpose `fzf` in other ways too.

##### Shell History

This is the biggest one: armed with `fzf`, you can just slam Ctrl-R and spew out some related strings for the command you want to easily find an invocation from your old history. Here's an example of me using it to quickly hit an old command to check my Elasticsearch cluster's health.

`fzf` works equally well in bash or zsh if you're still not sold on zsh, incidentally. If the functionality seems trivial to you, just try `fzf` for a week or so if you're a heavy terminal user. Fuzzy history searching a dramatic quality of life improvement.

##### Git Branches

Because my brain is full of holes like swiss cheese, I sometimes can't remember my git branch names. Fortunately, `fzf` is flexible enough that you can pipe _anything_ into it and kick the filtered input back out. This includes things like filtering a list of git branches to `git checkout`. I've created a git alias called `z` (for "fuzzy") that I can tab-complete quickly in order to invoke my `checkout-fuzzy` command. Here I find a branch with the string "container" _somewhere_ in the branch name:

I ruminated on this one for a while before writing the git alias, but it's actually pretty simple (with some functionality broken apart):

Conf[Unix]

- Font used to highlight strings.
- Font used to highlight variable names.
- Font used to highlight type and class names.

```
[alias]
  recent = for-each-ref --sort=-committerdate --format='%(refname:short)' refs/heads/
  checkout-fuzzy = !branch=$(git recent | fzf) && git checkout ${branch}
```

I've started to use this more often, and although it's not as load-bearing as my reverse history search, it's most definitely useful.

##### Deep File Recursion

Spelunking for specifically-named files used to be an exercise in `find` for me. On remote systems, this is often still the case, but if `fzf` is integrated into your shell, you can perform some fuzzy-finding tomfoolery to very easily reach into deeply-nested directories for files with specific names.

Here's an example of opening a file stored deep within a maze of nested directories - by default, you can trigger `fzf`'s file-finding fuzzy search with Ctrl-T:

As a convenience, `fzf` also invokes the file fuzzy-finder on-demand if you invoke tab-completion of the form `vim path/**` TAB for quick, ad-hoc file navigation.

#### Grab Bag

These points are the most impactful shell customizations I've used, but there's many more out there. If the topic interests you:

- Learn to love `jq`. I've coupled `jq` with httpie, and together you can get a surprising amount of REST-related work done on the command-line without needing to resort to an HTTP console.
- I didn't cover it here, but `tmux` is standard in my toolkit - I never use a shell without it. Being able to detach my shell is the most basic feature of `tmux` - it shines when you use it for things like reverse scrollback search, splits, and window linking.
- Codifying common operations as aliases - or shell functions - is exponentially powerful once you start to compose them together. I have some specialized functions that, coupled with the ability of a Yubikey to generate TOTP codes, makes 2fa really easy.

---

---