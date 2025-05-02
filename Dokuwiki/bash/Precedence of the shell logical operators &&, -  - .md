---
link: https://unix.stackexchange.com/questions/88850/precedence-of-the-shell-logical-operators
byline: |-
  Joseph R.Joseph R.
          
              40k88 gold badges111111 silver badges145145 bronze badges
site: Unix & Linux Stack Exchange
excerpt: |-
  I am trying to understand how the logical operator precedence works in bash. For example, I would have expected, that the following command does not echo anything.

  true || echo aaa && echo...
slurped: 2024-10-11T08:30
title: Precedence of the shell logical operators &&, ||
tags:
  - bash
  - operator
  - precedence
---

The `&&` and `||` operators are **_not_** exact inline replacements for if-then-else. Though if used carefully, they can accomplish much the same thing.

A single test is straightforward and unambiguous...

```
[[ A == A ]]  && echo TRUE                          # TRUE
[[ A == B ]]  && echo TRUE                          # 
[[ A == A ]]  || echo FALSE                         # 
[[ A == B ]]  || echo FALSE                         # FALSE
```

However, attempting to add multiple tests may yield unexpected results...

```
[[ A == A ]]  && echo TRUE   || echo FALSE          # TRUE  (as expected)
[[ A == B ]]  && echo TRUE   || echo FALSE          # FALSE (as expected)
[[ A == A ]]  || echo FALSE  && echo TRUE           # TRUE  (as expected)
[[ A == B ]]  || echo FALSE  && echo TRUE           # FALSE TRUE   (huh?)
```

Why are both FALSE _and_ TRUE echoed?

What's happening here is that we've not realized that `&&` and `||` are overloaded operators that act differently inside conditional test brackets `[[ ]]` than they do in the AND and OR (conditional execution) list we have here.

From the bash manpage (edited)...

> Lists
> 
> A list is a sequence of one or more pipelines separated by one of the operators ;, &, &&, or ││, and optionally terminated by one of ;, &, or . Of these list operators, && and ││ have equal precedence, followed by ; and &, which have equal precedence.
> 
> A sequence of one or more newlines may appear in a list instead of a semicolon to delimit commands.
> 
> If a command is terminated by the control operator &, the shell executes the command in the background in a subshell. The shell does not wait for the command to finish, and the return status is 0. Commands separated by a ; are executed sequentially; the shell waits for each command to terminate in turn. The return status is the exit status of the last command executed.
> 
> AND and OR lists are sequences of one of more pipelines separated by the && and ││ control operators, respectively. AND and OR lists are executed with left associativity.
> 
> An AND list has the form ...  
> `command1 && command2`  
> Command2 is executed if, and only if, command1 returns an exit status of zero.
> 
> An OR list has the form ...  
> `command1 ││ command2`  
> Command2 is executed if and only if command1 returns a non-zero exit status.
> 
> The return status of AND and OR lists is the exit status of the last command executed in the list.

Returning to our last example...

```
[[ A == B ]]  || echo FALSE  && echo TRUE

[[ A == B ]]  is false

     ||       Does NOT mean OR! It means...
              'execute next command if last command return code(rc) was false'

 echo FALSE   The 'echo' command rc is always true
              (i.e. it successfully echoed the word "FALSE")

     &&       Execute next command if last command rc was true

 echo TRUE    Since the 'echo FALSE' rc was true, then echo "TRUE"
```

Okay. If that's correct, then why does the next to last example echo anything at all?

```
[[ A == A ]]  || echo FALSE  && echo TRUE


[[ A == A ]]  is true

     ||       execute next command if last command rc was false.

 echo FALSE   Since last rc was true, shouldn't it have stopped before this?
                Nope. Instead, it skips the 'echo FALSE', does not even try to
                execute it, and continues looking for a `&&` clause.

     &&       ... which it finds here

 echo TRUE    ... so, since `[[ A == A ]]` is true, then it echos "TRUE"
```

The risk of logic errors when using more than one `&&` or `||` in a command list is quite high.

**Recommendations**

A single `&&` or `||` in a command list works as expected so is pretty safe. If it's a situation where you don't need an else clause, something like the following can be clearer to follow (the curly braces are required to group the last 2 commands) ...

```
[[ $1 == --help ]]  && { echo "$HELP"; exit; }
```

Multiple `&&` and `||` operators, where each command except for the last is a test (i.e. inside brackets `[[ ]]`), are usually also safe as all but the last operator behave as expected. The last operator acts more like a `then` or `else` clause.