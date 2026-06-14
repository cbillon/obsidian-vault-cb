---
tags:
  - ai
---
[Mimo ](https://mimo.xiaomi.com/mimocode/start)
To use MiMo Code in your terminal, set up your environment, then install it with any of the methods below.

---

#### Prerequisites

To use MiMo Code in your terminal, you'll need:

1. A modern terminal emulator like:
    
    - [WezTerm](https://wezterm.org/), cross-platform
    - [Alacritty](https://alacritty.org/), cross-platform
    - [Ghostty](https://ghostty.org/), Linux and macOS
    - [Kitty](https://sw.kovidgoyal.net/kitty/), Linux and macOS
2. API keys for the LLM providers you want to use.
    

---

## Install

The easiest way to install MiMo Code is through the install script.
MiMo Code provides an interactive terminal interface or TUI for working on your projects with an LLM. This page covers how to enter messages, reference files, run commands, and steer the model while you work in the TUI.

Running MiMo Code starts the TUI for the current directory.

```bash
mimo
```

Or you can start it for a specific working directory.

```bash
mimo /path/to/project
```

Once you're in the TUI, you can prompt it with a message.

```text
Give me a quick summary of the codebase.
```

To run MiMo Code programmatically from the command line instead, see [Command Line](https://mimo.xiaomi.com/mimocode/cli-options).