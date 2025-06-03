---
link: https://arnaudr.io/2015/12/06/bash-logging-helpers/
excerpt: Some smart bash lines that provide powerful logging for your scripts, especially useful when run by Cron.
slurped: 2025-05-11T14:25
title: Bash Logging Helpers
tags:
  - bash
  - log
---

## Introduction

I wrote plenty of bash scripts lately. And when it comes to logging info and errors, it's always the same. I start simple, with just `echo` for info and `echo >&2` for errors. I always hope I can get away with it, but it's not that simple.

It starts to be tricky when the script is launched automatically by [Cron](https://en.wikipedia.org/wiki/Cron). How to get the output of the script in the system logs?

## Piping the Script Output

The first solution, and the easiest, is to pipe the script output to `logger`, directly in the cronjob definition. So that the job definition looks like that:

`@daily /usr/local/bin/myscript 2>&1 | /usr/bin/logger -t myscript`

This solution is damn simple, but it has a slight disadvantage: the error output is mixed with the standard output, therefore you loose the notion of error in the logs. You error messages will appear as standard messages, with nothing warning you that it's an error.

## Defining Log Functions in the Script

This is probably a bit overkill for small and simple scripts. But if you think you can afford a few extra lines to have a proper logging system in your script, here's how I do it.

```
  tty -s && function log() { echo "$@"; }
  tty -s && function log_err() { >&2 echo "$@"; }
  tty -s || function log() { logger -t $(basename $0) "$@"; }
  tty -s || function log_err() { logger -t $(basename $0) -p user.err "$@"; }

```
I put these lines at the beginning of my script, or in a kind of bash library that is included by my other scripts. Then, I use these `log*` functions to log whatever I want.

Now let's explain that a little bit.

- `tty -s` let you know if the script is run in a terminal or not. Or, to be more accurate, if the standard input (_stdin_) is connected to a terminal or not. If the script is run by hand, the answer is yes. If it's run by Cron, the answer is no.
- When run in a terminal, the logs are issued with `echo`, they appear in the console only. It means that when I work on the script, I won't pollute the logs.
- When run by Cron, logs are issued with `logger`, they go to the system log.

So you might notice that we do something slightly unusual here: we define functions conditionally. It's a little bit like macros in C.

If you feel uncomfortable with that, here is another way to achieve the same thing:


```
 log() { [[ -t 1 ]] && echo "$@" || logger -t $(basename $0) "$@"; }
 log_err() { [[ -t 2 ]] && >&2 echo "$@" || logger -t $(basename $0) -p user.err "$@"; }
 
```

Explanations again:

- `-t` is a bash builtin test that checks if a file descriptor is opened and refers to a terminal (`man bash` for more info).
- for normal logs that are supposed to go to _stdout_ (file descriptor number 1), we just check if stdout is connected to a terminal (`[[ -t 1 ]]`). If yes, we can use `echo`. Otherwise, we log to the system logs with `loggers`.
- for error logs, we check if _stderr_ is connected to a terminal (`[[ -t 2 ]]`).

OK, so in the end it just boils down to 2 smart lines. It's not much, is it?

Compared to simply piping the output of your script in the cron task, there are several advantages:

- Errors are logged as such, so they appear properly in the system logs (highlighted, or in bold, or in red or whatever).
- You may define more log functions, one for each log level (debug, warning and so on), the same way you use to do with C or Python, or whatever language you're used to. And as your script grows bigger, it's really handy.

## Credits

Piping the output of the script is not my idea, I got that from a ServerFault discussion:  
[Better logging for cronjobs? Send cron output to syslog?](http://serverfault.com/q/137468/323199)

There's a good thread that taught me how to define functions conditionnally, but it was burried deep within the net:  
[[Vm-dev] conditionally define a function in sh/bash?](http://lists.squeakfoundation.org/pipermail/vm-dev/2014-June/015945.html)

There's a stackoverflow discussion that taught me how to check for the terminal:  
[Can a bash script tell if it's being run via cron?](http://stackoverflow.com/q/3214935/776208)

I also read a lot of bullshit about testing the `$-` bash variable. `$-` can tell if you're in an interactive shell or not. So if you're wondering exactly what is an interactive shell, please read:  
[http://zsh.sourceforge.net/Guide/zshguide02.html#l7](http://zsh.sourceforge.net/Guide/zshguide02.html#l7)

Just to quote the most interesting part:

> when you are typing at a prompt and waiting for each command to run, the shell is interactive; in the other case, when the shell is reading commands from a file, it is, consequently, non-interactive.

Checking `$-` doesn't distinguish between a script run by hand and a script run by Cron: both are non-interactive. So it doesn't help at all, but I read plenty of discussions where people give it as a solution. Shame on them!