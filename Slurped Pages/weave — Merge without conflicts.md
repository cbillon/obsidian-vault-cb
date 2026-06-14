---
link: https://ataraxy-labs.github.io/weave/
excerpt: Entity-level semantic merge driver for Git. Two agents edit different functions in the same file? Clean merge. Every time.
slurped: 2026-06-14T15:32
title: weave — Merge without conflicts
tags:
  - git
  - merge
---

[weave](https://ataraxy-labs.github.io/weave/index.html)

Entity-level semantic merge driver for Git. Two agents edit different functions in the same file? Clean merge. Every time.

# Two agents edited different functions

$ git merge feature-b
CONFLICT (content): Merge conflict in src/lib.ts
Automatic merge failed.

# Git sees overlapping lines.
# The functions don't actually overlap.

# Same merge, with weave configured

$ git merge feature-b
weave [src/lib.ts]:
  2 entities matched, 2 modified, 0 conflicts
Merge made by the 'ort' strategy.

# Different functions = no conflict.

copied $ brew install weave

## 31 out of 31.

31 merge scenarios across 7 languages. [Full breakdown →](https://ataraxy-labs.github.io/weave/benchmarks.html)

83 real-world wins

0 regressions on C, Python, Go

4,917 file merges tested

1,500+ downloads

## Three layers

Use just the merge driver. Or add coordination for multi-agent workflows. [Full docs →](https://ataraxy-labs.github.io/weave/docs.html)

MERGE

### Merge Driver

Replaces git's line-level merge. Parses code with tree-sitter, merges by function and class.

COORDINATE

### CRDT State

Agents claim entities before editing. Detect conflicts before they happen.

CONNECT

### MCP Server

15 tools via Model Context Protocol. Claude and other AI agents call them directly.

## 28 languages

Entity extraction powered by [sem-core](https://github.com/Ataraxy-Labs/sem) and tree-sitter. Plus 5 data formats. [Full list →](https://ataraxy-labs.github.io/weave/docs.html#languages)

TypeScript JavaScript Python Go Rust Java C C++ C# Ruby PHP Swift Kotlin Elixir Bash HCL Fortran Dart Perl OCaml Scala Zig Vue Svelte XML ERB JSON YAML TOML CSV Markdown

## Try it. 5 seconds.

$ brew install weave

$ cd my-project && weave setup
✓ Merge driver configured

$ git merge feature-branch
Merge made by the 'ort' strategy.