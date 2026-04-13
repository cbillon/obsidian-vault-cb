---
link: https://rednafi.com/misc/shell-redirection/
byline: Redowan Delowar
site: Redowan Delowar
date: 2024-09-12T02:00
excerpt: Complete guide to Bash redirection. Master stdout, stderr, pipes, tee,
  file descriptors, and shorthand syntax with practical examples for every case.
twitter: https://twitter.com/@rednafi
tags:
  - slurp/Shell
  - slurp/Unix
slurped: 2026-04-10T15:22
title: Shell redirection syntax soup
---

September 12, 2024

Table of contents

- [Redirecting stdout and stderr](#redirecting-stdout-and-stderr)
    - [Redirect stdout to a file](#redirect-stdout-to-a-file)
    - [Redirect stderr to a file](#redirect-stderr-to-a-file)
    - [Redirect both stdout and stderr to a file](#redirect-both-stdout-and-stderr-to-a-file)
    - [Append instead of overwriting](#append-instead-of-overwriting)
- [Piping output](#piping-output)
    - [Pipe stdout to another command](#pipe-stdout-to-another-command)
    - [Pipe both stdout and stderr](#pipe-both-stdout-and-stderr)
    - [Shorthand for piping both stdout and stderr (`|&`)](#shorthand-for-piping-both-stdout-and-stderr-)
- [Redirecting file descriptors](#redirecting-file-descriptors)
    - [Custom file descriptors](#custom-file-descriptors)
    - [Redirect stderr to a file descriptor](#redirect-stderr-to-a-file-descriptor)
    - [Redirect both stdout and stderr to a file descriptor](#redirect-both-stdout-and-stderr-to-a-file-descriptor)
- [Discarding output](#discarding-output)
    - [Send stdout and stderr to /dev/null](#send-stdout-and-stderr-to-devnull)
- [At a glance](#at-a-glance)

I always struggle with the syntax for redirecting multiple streams to another command or a file. LLMs do help, but beyond the most obvious cases, it takes a few prompts to get the syntax right. When I know exactly what I’m after, scanning a quick post is much faster than wrestling with a non-deterministic kraken. So, here’s a list of the redirection and piping syntax I use the most, with real examples.

## Redirecting stdout and stderr [#](#redirecting-stdout-and-stderr)

### Redirect stdout to a file [#](#redirect-stdout-to-a-file)

- Standard way:
    
    This replaces the content of `file` with the stdout of `command`. For example:
    
    ```
    echo "Hello, world!" > hello.txt
    ```
    
- Print and redirect to file:
    
    Example:
    
    ```
    echo "Hello, world!" | tee hello.txt
    ```
    
    This prints “Hello, world!” to the terminal and also writes it to `hello.txt`.
    

### Redirect stderr to a file [#](#redirect-stderr-to-a-file)

- Standard way:
    
    Sends all errors (stderr) to `file`. For example:
    
    ```
    ls non_existing_file 2> error.log
    ```
    
- Print and redirect stderr to file:
    
    Example:
    
    ```
    ls non_existing_file 2> >(tee error.log)
    ```
    

### Redirect both stdout and stderr to a file [#](#redirect-both-stdout-and-stderr-to-a-file)

- Common approach:
    
    Combines stdout and stderr into one stream and saves them to `file`. For example:
    
    ```
    ls non_existing_file existing_file > output.log 2>&1
    ```
    
- Print and redirect both to file:
    
    Example:
    
    ```
    ls non_existing_file existing_file 2>&1 | tee output.log
    ```
    
- Convenient shorthand:
    
    Example:
    
    ```
    ls non_existing_file existing_file &> output.log
    ```
    

### Append instead of overwriting [#](#append-instead-of-overwriting)

- Append stdout to a file:
    
    Example:
    
    ```
    echo "Appending line" >> hello.txt
    ```
    
- Print and append stdout to file:
    
    Example:
    
    ```
    echo "Appending line" | tee -a hello.txt
    ```
    
- Append both stdout and stderr (explicit):
    
    Example:
    
    ```
    ls non_existing_file existing_file >> output.log 2>&1
    ```
    
- Print and append both stdout and stderr to file:
    
    ```
    command 2>&1 | tee -a file
    ```
    
    Example:
    
    ```
    ls non_existing_file existing_file 2>&1 | tee -a output.log
    ```
    
- Convenient shorthand for appending both:
    
    Example:
    
    ```
    ls non_existing_file existing_file &>> output.log
    ```
    

## Piping output [#](#piping-output)

### Pipe stdout to another command [#](#pipe-stdout-to-another-command)

- Basic usage:
    
    This sends the stdout of `command1` to the input of `command2`. For example:
    
    ```
    echo "Hello, world!" | grep "Hello"
    ```
    
- Print and redirect piped stdout to file:
    
    ```
    command1 | tee file | command2
    ```
    
    Example:
    
    ```
    echo "Hello, world!" | tee output.txt | grep "Hello"
    ```
    

### Pipe both stdout and stderr [#](#pipe-both-stdout-and-stderr)

- Common way:
    
    Combines stdout and stderr, then pipes the combined stream to `command2`. For example:
    
    ```
    ls non_existing_file existing_file 2>&1 | grep "No"
    ```
    
- Print and redirect both stdout and stderr to file:
    
    ```
    command1 2>&1 | tee file | command2
    ```
    
    Example:
    
    ```
    ls non_existing_file existing_file 2>&1 | tee output.txt | grep "No"
    ```
    

### Shorthand for piping both stdout and stderr (`|&`) [#](#shorthand-for-piping-both-stdout-and-stderr-)

- Shorthand syntax:
    
    This is equivalent to `command1 2>&1 | command2`, combining stdout and stderr. For example:
    
    ```
    ls non_existing_file existing_file |& grep "No"
    ```
    
- Print and redirect both stdout and stderr using `|&`:
    
    ```
    command1 |& tee file | command2
    ```
    
    Example:
    
    ```
    ls non_existing_file existing_file |& tee output.txt | grep "No"
    ```
    

## Redirecting file descriptors [#](#redirecting-file-descriptors)

### Custom file descriptors [#](#custom-file-descriptors)

- Create a new file descriptor (e.g., `3`) and redirect stdout to it:
    
    ```
    exec 3> outputfile
    command >&3
    ```
    
    This sends the stdout of `command` to file descriptor `3`, which points to `outputfile`. For example:
    
    ```
    exec 3> custom_output.txt
    echo "Using FD 3" >&3
    ```
    
- Print and redirect stdout to custom file descriptor:
    
    ```
    exec 3> custom_output.txt
    echo "Using FD 3" | tee /dev/tty > /dev/fd/3
    ```
    
    This prints “Using FD 3” to the terminal and simultaneously writes it to `custom_output.txt`.
    

### Redirect stderr to a file descriptor [#](#redirect-stderr-to-a-file-descriptor)

- Common case:
    
    Redirects stderr to file descriptor `3`. For example:
    
    ```
    exec 3> error_output.txt
    ls non_existing_file 2>&3
    ```
    
- Print and redirect stderr to custom file descriptor:
    
    ```
    command 2> >(tee >(cat > /dev/fd/3))
    ```
    
    Example:
    
    ```
    ls non_existing_file 2> >(tee >(cat > /dev/fd/3))
    ```
    

### Redirect both stdout and stderr to a file descriptor [#](#redirect-both-stdout-and-stderr-to-a-file-descriptor)

- Common way:
    
    Combines stdout and stderr, and redirects them to file descriptor `3`.
    
    **Note**: There’s no shorthand equivalent for redirecting both stdout and stderr to a file descriptor. You need to use the full syntax. For example:
    
    ```
    exec 3> combined_output.txt
    ls non_existing_file existing_file > /dev/fd/3 2>&1
    ```
    

## Discarding output [#](#discarding-output)

### Send stdout and stderr to /dev/null [#](#send-stdout-and-stderr-to-devnull)

- Common:
    
    Silences all output (stdout and stderr). For example:
    
    ```
    ls non_existing_file > /dev/null 2>&1
    ```
    
- Print and discard stdout and stderr (not sure why you’d ever need this):
    
    Example:
    
    ```
    ls non_existing_file | tee /dev/null
    ```
    
- Convenient shorthand:
    
    Example:
    
    ```
    ls non_existing_file &>/dev/null
    ```
    

## At a glance [#](#at-a-glance)

- **Redirect stdout**: `command > file`
    
- **Redirect stderr**: `command 2> file`
    
- **Redirect both stdout and stderr**:
    
    - Standard: `command > file 2>&1`
    - Shorthand: `command &> file`
- **Append stdout**: `command >> file`
    
- **Append both stdout and stderr**:
    
    - Standard: `command >> file 2>&1`
    - Shorthand: `command &>> file`
- **Pipe stdout**: `command1 | command2`
    
- **Pipe both stdout and stderr**:
    
    - Standard: `command1 2>&1 | command2`
    - Shorthand: `command1 |& command2`
- **Custom file descriptors**:
    
    - Create and redirect stdout: `exec 3> file; command >&3`
    - Redirect stderr: `command 2>&3`
    - Redirect both stdout and stderr: `command > /dev/fd/3 2>&1` (no shorthand available)
- **Discard stdout and stderr**:
    
    - Standard: `command > /dev/null 2>&1`
    - Shorthand: `command &>/dev/null`

- [Shell](https://rednafi.com/tags/shell/)
- [Unix](https://rednafi.com/tags/unix/)

§