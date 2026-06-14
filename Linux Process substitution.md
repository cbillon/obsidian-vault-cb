---
tags:
  - linux
  - bash
---
#### Process Substitution

This is another niche use case, but it comes in really handy when you need it.

Say that you have a command that expects a file as an argument, but the input you want to provide is the output of a command. Being able to provide that file input from a _command_ can be useful.

Here's an example: I sometimes need to quickly diff what may be running on remote systems. Using process substitution, you can diff output of `ssh` commands:

shell

```
diff <(ssh host1 systemctl list-unit-files) <(ssh host2 systemctl list-unit-files)
```

`diff` will operate on the output of those `ssh` commands as if they were files. This applies to any command that accepts files as arguments, so you can get really creative for a variety of use cases. For example, you can retrieve a json response from `curl` and forward it along to another server with `curl -XPOST` something like `-d @<(curl ...)`. In my experience, not all CLI utilities will transparently accept temporary file handles (which is what `<()` constructs), but the majority do.