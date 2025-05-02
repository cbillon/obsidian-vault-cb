---
tags:
  - linux
  - command
  - direvent
---
**Direvent** is a program that monitors a set of directories on the file system and reacts when their content changes. When a change is detected, the daemon reacts by invoking an external command configured for that kind of change.
direvent` has two other flag that may be useful for you.

`--debug`(`-d`) to give extra information.

There's also `--lint`(`t`) that check the configuration file for errors, but I suspect this isn't your issue if `direvent` is running.

Source: [https://www.gnu.org.ua/software/direvent/manual/direvent.html](https://www.gnu.org.ua/software/direvent/manual/direvent.html)