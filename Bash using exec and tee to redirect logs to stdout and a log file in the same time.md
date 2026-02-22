---
tags:
  - bash
  - log
---
[Bash Ref Manual](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Process-Substitution)

Use [process substitution](https://www.gnu.org/software/bash/manual/bashref.html#Process-Substitution) with [`&` redirection](https://www.gnu.org/software/bash/manual/bashref.html#Redirecting-Standard-Output-and-Standard-Error) and [`exec`](https://www.gnu.org/software/bash/manual/bashref.html#index-exec):

```bash
exec &> >(tee -a "$log_file")
echo "This will be logged to the file and to the screen"
```

`$log_file` will contain the output of the script and any subprocesses, and the output will also be printed to the screen.

- `>(...)` starts the process `...` and returns a file representing its standard input.
    
- `exec &> ...` redirects both standard output and standard error into `...` for the remainder of the script (use just `exec > ...` for stdout only).
    
- `tee -a` appends its standard input to the file, and also prints it to the screen.

## 3.5.6 Process Substitution [¶](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Process-Substitution-1)

Process substitution allows a process’s input or output to be referred to using a filename. It takes the form of

<(list)

or

>(list)

The process list is run asynchronously, and its input or output appears as a filename. This filename is passed as an argument to the current command as the result of the expansion.

If the `>(list)` form is used, writing to the file provides input for list. If the `<(list)` form is used, reading the file obtains the output of list. Note that no space may appear between the `<` or `>` and the left parenthesis, otherwise the construct would be interpreted as a redirection.

Process substitution is supported on systems that support named pipes (FIFOs) or the /dev/fd method of naming open files.

When available, process substitution is performed simultaneously with parameter and variable expansion, command substitution, and arithmetic expansion.

