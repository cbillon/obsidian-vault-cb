---
tags:
  - bash
  - debug
---
[ici](https://mywiki.wooledge.org/BashGuide/Practices](https://mywiki.wooledge.org/BashGuide/Practices)
## Activate Bash's Debug Mode

If you still don't see the error of your ways, Bash's debugging mode might help you see the problem through the code.

When Bash runs with the x option turned on, it prints out every command it executes before executing it (to standard error). That is, **after any expansions have been applied**. As a result, you can see exactly what's happening as each line in your code is executed. Pay very close attention to the quoting used. Bash uses quotes to show you exactly which strings are passed as a single argument.

There are three ways of turning on this mode.

- Run the script with bash -x:
    
    $ bash -x ./mybrokenscript
    
- Modify your script's header:
    
    #!/bin/bash -x
    [.. script ..]
    
- Or:
    
    #!/usr/bin/env bash
    set -x
    
- Or add set -x somewhere in your code to turn on this mode for only a specific block of your code:
    
    #!/usr/bin/env bash
    [..irrelevant code..]
    set -x
    [..relevant code..]
    set +x
    [..irrelevant code..]
    

Because the debugging output goes to stderr, you will generally see it on the screen, if you are running the script in a terminal. If you would like to log it to a file, you can tell Bash to send all stderr to a file:

exec 2>> /path/to/my.logfile
set -x

A nice feature of bash version >= 4.1 is the variable BASH_XTRACEFD. This allows you to specify the file descriptor to write the set -x debugging output to. In older versions of bash, this output always goes to stderr, and it is difficult if not impossible to keep it separate from normal output (especially if you are logging stderr to a file, but you need to see it on the screen to operate the program). Here's a nice way to use it:

[Afficher/masquer les numéros de lignes](https://mywiki.wooledge.org/BashGuide/Practices#)

   [1](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_1) # dump set -x data to a file
   [2](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_2) # turns on with a filename as $1
   [3](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_3) # turns off with no params
   [4](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_4) setx_output()
   [5](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_5) {
   [6](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_6)     if [[ $1 ]]; then
   [7](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_7)         exec {BASH_XTRACEFD}>>"$1"
   [8](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_8)         set -x
   [9](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_9)     else
  [10](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_10)         set +x
  [11](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_11)         unset -v BASH_XTRACEFD
  [12](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_12)     fi
  [13](https://mywiki.wooledge.org/BashGuide/Practices#CA-79c0ba7091f2499321f60ea7a14cfd4e25a0f478_13) }

If you have a complicated mess of scripts, you might find it helpful to change PS4 before setting -x. If the value assigned to PS4 is surrounded by double quotes it will be expanded during variable assignment, which is probably not what you want; with single quotes, the value will be expanded when the PS4 prompt is displayed.

PS4='+$BASH_SOURCE:$LINENO:$FUNCNAME: '