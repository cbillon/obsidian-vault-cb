---
link: https://github.com/bensadeh/circumflex
site: GitHub
excerpt: 🌿 It's Hacker News in your terminal. Contribute to bensadeh/circumflex development by creating an account on GitHub.
twitter: https://twitter.com/@github
slurped: 2026-04-19T15:06
title: "GitHub - bensadeh/circumflex: 🌿 It's Hacker News in your terminal"
tags:
  - golang
  - cli
---


`circumflex` is a command line tool for browsing Hacker News in your terminal

[![Main view](https://github.com/bensadeh/circumflex/raw/main/screenshots/main_view.png)](https://github.com/bensadeh/circumflex/blob/main/screenshots/main_view.png)

### Main features

[](https://github.com/bensadeh/circumflex#main-features)

- 🛋 **Everything in one place** — read both the comment section and articles in Reader Mode
- 🌈 **Syntax highlighting** — syntax-aware formatting for comments and headlines
- ⚡️ **Vim-style navigation** — scroll through, jump between and collapse threads with familiar keybindings

**You might also like:**

- 🤹 **Native terminal colors** — you bring your own color scheme, `circumflex` does the rest
- 💎 **Nerd Fonts** — full support for Nerd Fonts as icons
- ❤️ **Add to favorites** — save interesting submissions for later

## Installing

[](https://github.com/bensadeh/circumflex#installing)

The binary name for `circumflex` is `clx`.

# Homebrew
brew install circumflex

# Nix
nix-shell -p circumflex

# AUR
yay -S circumflex

# Go
go install github.com/bensadeh/circumflex/cmd/clx@latest

# From source
go run ./cmd/clx

## Features

[](https://github.com/bensadeh/circumflex#features)

### Comment section

[](https://github.com/bensadeh/circumflex#comment-section)

Press Enter to view the comment section.

The comment section has two modes: `read mode` and `navigate mode`.

In `read mode`, you can scroll using the usual vim bindings. You can also jump between top-level comments ( n/N), and you can expand and collapse threads by quote level (h/l) or all at once (Enter).

In `navigate mode`, you can individually select comments and collapse specific threads. This is useful in longer threads with many replies.

[![comment section](https://github.com/bensadeh/circumflex/raw/main/screenshots/comment_view.png)](https://github.com/bensadeh/circumflex/blob/main/screenshots/comment_view.png)

`circumflex` is read-only and does not support for logging in, voting or commenting.

### Reader Mode

[](https://github.com/bensadeh/circumflex#reader-mode)

Press Space to read the linked article in Reader Mode. Just like in the comment section, you can jump between headers using n/N, and you can scroll using the usual vim bindings.

[![reader mode](https://github.com/bensadeh/circumflex/raw/main/screenshots/reader_mode.png)](https://github.com/bensadeh/circumflex/blob/main/screenshots/reader_mode.png)

## Usage

[](https://github.com/bensadeh/circumflex#usage)

Run `clx help` or `man clx` for a full list of available commands, flags and keymaps. Press i to bring up all the keybindings for the current view.