---
link: https://amitmerchant.com/copy-last-terminal-command-to-clipboard-in-linux-and-macos/
byline: Amit Merchant
site: Amit Merchant - A blog on PHP, JavaScript, and more
date: 2023-01-12T01:00
excerpt: Sometimes, it’s useful to quickly copy over the command that you executed recently in your terminal when you want to share it with someone.
slurped: 2025-10-13T11:50
title: Copy last terminal command to clipboard in Linux and macOS
tags:
  - bash
  - clipboard
---

Sometimes, it’s useful to quickly copy over the command that you executed recently in your terminal when you want to share it with someone.

The easiest way to do so is by copying it to the system clipboard so that you can paste it wherever you want. Recently, Paul Redmond [shared](https://twitter.com/paulredmond/status/1613377566785835009) how to do it on the macOS.

- [macOS](https://amitmerchant.com/copy-last-terminal-command-to-clipboard-in-linux-and-macos/#macos)
- [Linux](https://amitmerchant.com/copy-last-terminal-command-to-clipboard-in-linux-and-macos/#linux)
- [Make it seamless using Alias](https://amitmerchant.com/copy-last-terminal-command-to-clipboard-in-linux-and-macos/#make-it-seamless-using-alias)

## macOS

Here’s the command to copy the last command to the clipboard.

As you can tell, the [fc](https://www.unix.com/man-page/linux/1/fc/) command is used to access the command history list. We can couple it with option `-ln` which invokes the list with suppressed command numbers. The `-1` represents the last executed command in the terminal.

Lastly, we can pass the output of this command to [pbcopy](https://ss64.com/osx/pbcopy.html) which is used to copy data from STDIN to the clipboard. It’s an in-built utility in macOS.

And that’s how you can copy the last command in the clipboard.

## Linux

If you’re using a Bash terminal on Linux, you can use the following command to copy the last command in the clipboard.

```
$ fc -nl -1 | xclip -selection clipboard
```

As you can tell, the first part of the command here is the same as macOS but instead of using `pbcopy`, we need to use `xclip` to copy the command to the clipboard. It’s something you need to install first.

For instance, you can install `xclip` on systems based on Ubuntu like so.

## Make it seamless using Alias

You can create an alias for these commands to make the entire process of copying the command more seamless and easy to invoke.

Here’s how I would do it.

On macOS,

```
$ alias lc="fc -ln -1 | pbcopy"
```

On Linux,

```
$ alias lc="fc -nl -1 | xclip -selection clipboard"
```

Now, you can do it just by running `lc` in your terminal!