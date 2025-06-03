---
tags:
  - linux
  - tools
---
## Bat

[Bat](https://github.com/sharkdp/bat?ref=blog.yaakov.online)

`cat` is really light and efficient and just takes either a file or stdin, and writes that to the screen.

Bat is a bit smarter. For one, if I try print a binary file, it doesn't do that by default, which always risks stuffing up my terminal.

It also has syntax highlighting for some file formats, and can integrate with `git` to show you changes.
## Diffstatic

[diffstatic](https://difftastic.wilfred.me.uk/?ref=blog.yaakov.online)
Difftastic is a CLI diff tool that compares files based on their syntax, not line-by-line. Difftastic produces accurate diffs that are easier for humans to read.