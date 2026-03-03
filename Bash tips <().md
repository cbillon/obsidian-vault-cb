---
tags:
  - bash
  - tips
---
## 6) >()

It s similar to $() in that the ouput of the command inside is re used

In this case, though, the output is treated as a file. This file can be used as an argument to commands that take files as an argument.

Here’s an example.
```
  $ grep somestring file1 > /tmp/a
  $ grep somestring file2 > /tmp/b
  $ diff /tmp/a /tmp/b
  
```
Have you ever done something like this?

That works, but instead you can write:

diff <(grep somestring file1) <(grep somestring file2)

## 7) Quoting

Quoting’s a knotty subject in bash, as it is in many software contexts.

Firstly, variables in quotes:

A='123'  
echo "$A" -> 123
echo '$A'  -> $A

Pretty simple – double quotes dereference variables, while single quotes go literal.
