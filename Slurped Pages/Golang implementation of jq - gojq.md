---
link: https://itchyny.medium.com/golang-implementation-of-jq-gojq-ad5bd46a4af2
byline: itchyny
site: Medium
date: 2020-04-26T11:05
excerpt: "Golang implementation of jq: gojq I have been working on a pure Go
  language implementation of jq. The command gojq is highly compatible with jq
  and also capable to be embeded within Go products …"
twitter: https://twitter.com/@Medium
slurped: 2026-09-17T15:20
title: "Golang implementation of jq: gojq"
---

[

![itchyny](https://miro.medium.com/v2/resize:fill:64:64/0*nHkbr9nHkTlwXTSF.jpg)



](https://itchyny.medium.com/?source=post_page---byline--ad5bd46a4af2-----------------------------------------)

I have been working on a pure Go language implementation of [jq](https://stedolan.github.io/jq/). The command `gojq` is highly compatible with jq and also capable to be embeded within Go products. Creating a cli tool in Go language is much fun and learned various things about jq.

The jq command is a tool for querying and filtering against JSON. As a web engineer, the command is an essential tool while developing REST apis.

One of the major usage of jq is querying objects or arrays, like `.foo[0].bar`. But jq is actually much more powerful tool, with capability of scripting. It is actually a Turing-complete language so reimplementation of jq is a good practice for implementing a programming language.

My first goal of reimplementation was to understand jq deeply. I use jq for simple filtering tasks but got stuck when I want to write more complex queries. I thought it would be the best way to learn jq is to implement the language itself.

From my viewpoint of the semantics of jq, it is a language with JSON value iterators as the first citizen. Not JSON values. The `.[]` returns an iterator which emits the value of arrays and objects. Even a simple literal `true` is an iterator which emits one JSON value.

One of the interesting syntax of jq is the comma operator (`,`). Most of you think that `[0, 1, 2]` is an array syntax, but in jq, it is an array construction syntax `[...]` with comma operators `,` and three JSON value iterators. Once you know what comma operator and iterators are, you understand why `echo [0, 1, 2] | jq '[.[], .[0], ..]'` emits a flatten array.

 % jq -n '(1, 2) * (3, 4)'  
3  
6  
4  
8  
 % jq -n -c '[range(1;5) * range(1;5)]'  
[1,2,3,4,2,4,6,8,3,6,9,12,4,8,12,16]  
 % jq -n -c 'def f: 1, f; [limit(10; f)]'  
[1,1,1,1,1,1,1,1,1,1]  
 % jq -n -c '[[0,1] | while(.[0]<100; [.[1], .[0]+.[1]]) | .[0]]'  
[0,1,1,2,3,5,8,13,21,34,55,89]

Let’s explore some other syntaxes. There’s function declaration syntax in jq.

 % echo '[1,2]' | jq 'def f(a; b): a * 10 + b; f(.[]; .[])'  
11  
21  
12  
22  
 % jq -n -c 'def f($n): if $n == 0 then empty else $n, f($n - 1) end; [f(10)]'  
[10,9,8,7,6,5,4,3,2,1]

Note that every query has implicit inputs.

 % echo '1 2 3' | jq 'def ntimes($n): . * $n; ntimes(3)'  
3  
6  
9  
 % echo '[1, 3, 6]' | jq 'def mean: add / length; mean'  
3.3333333333333335

You might think you have never defined a function in jq queries so far, but you are likely to have used [jq functions](https://github.com/stedolan/jq/blob/ccc79e592cfe1172db5f2def5a24c2f7cfd418bf/src/builtin.jq).

def map(f): [.[] | f];def select(f): if f then . else empty end;

It is an important fact that many builtin functions are coded by jq language. It reduces many kinds of bugs due to writing the functions in C and these functions themselves are the test cases of the jq language interpreter. The `..` operator is implemented by the alias of the following `recurse` function. This definition includes function declaration in a function, function arguments, array/object value iterator syntax `.[]?`.

def recurse: recurse(.[]?);  
def recurse(f): def r: ., (f | r); r;

Implementing jq builtin filters is after all implementing the language interpreter. And improving the performance of the builtins means improving the interpreter itself.

When I started implementation of gojq, I started with emulating the iterator semantics using channels. The jq language is all about iterators. All the values, even a single number or string are iterators emitting a value.

func unitIterator(v interface{}) <-chan interface{} {  
    c := make(chan interface{}, 1)  
    defer func() {  
        defer close(c)  
        c <- v  
    }()  
    return c  
}

A binary operator like `*` in `(1, 2) * (3, 4)` composes two iterators and creates a new iterator.

func binopIterator(l <-chan interface{}, r <-chan interface{}, fn func(l, r interface{}) interface{}) <-chan interface{} {  
    d := make(chan interface{}, 1)  
    go func() {  
        defer close(d)  
        ls := reuseIterator(l)  
        for r := range r {  
            for l := range ls() {  
                d <- fn(l, r)  
            }  
        }  
    }()  
    return d  
}

Implementation using channels works mostly well. I can implement most of the syntaxes, including function declarations, `foreach` and `reduce`. I was satisfied that I could prove that Go channels fit to the semantics of jq.

But the serious problem is performance. The command `jq -n "range(10000)"` exits in less than 0.1 sec, but same command takes over 4 mins in my implementation. You may wonder why just counting up the numbers takes that too long, but note that `range` function is implemented as follows.

def range($n): 0 | while(. < $n; . + 1);def while(cond; update):  
  def _while: if cond then ., (update | _while) else empty end;  
  _while;

This `while` function is an important key to the jq language. Firstly, jq does not have the loop syntax but allows recursions, which makes the language to be Turing-complete. Secondly, improving the performance of this function requires tail call optimization.

My first intuition that jq is all about iterators and can be implemented using Go channels was almost correct. It was capable to implement many syntaxes of jq. However, improving the performance was quite difficult. Walking through the syntax tree while evaluation costs too much, looking up the variable using maps is too heavy. Also it is difficult to optimize tail call recursions. I got less motivated for weeks, was thinking whether giving up the implementation or not.

I decided to rewrite my implementation to stack-machine based. I noticed that without compiling to virtual machine instructions, it is too difficult to apply various optimizations including tail call recursions. Nobody will use the new implementation which is thousands times slower. Also I found it difficult implement the `path` function in my first iterator-based interpreter so I can’t create the assignment operators (e.x. `=`, `+=`, `|=`).

Let’s take a quick look into how a stack-machine based virtual machine works. Consider evaluation of `1 * 2 + 3 * 4`. The instruction list looks like as follows.

0: push 1  
1: push 2  
2: call multiply  
3: push 3  
4: push 4  
5: call multiply  
6: call add  
7: ret

When the machine evaluates this code, the stack grows as follows.

[]  
[ 1 ]         # 0: push 1  
[ 1, 2 ]      # 1: push 2  
[ 2 ]         # 2: call multiply  
[ 2, 3 ]      # 3: push 3  
[ 2, 3, 4 ]   # 4: push 4  
[ 2, 12 ]     # 5: call multiply  
[ 14 ]        # 6: call add  
[]            # 7: ret => output: 14

This is a simple example but any stack-machine based interpreter behaves like this. Ruby (YARV), Python (CPython), JVM, WebAssembly are all stack-machined interpreter.

The jq interpreter is also stack-machine based. The huge difference is that it can save and restore the entire stack state. Consider evaluation of `(1, 2) * (3, 4)`. The right hand side is evaluated at first and the comma operator is compiled to `fork` instruction.

0: fork 3  
1: push 3  
2: jump 4  
3: push 4  
4: fork 7  
5: push 1  
6: jump 8  
7: push 2  
8: call multiply  
9: ret

When jq evaluates this instruction list, the stack and fork list grows as follows.

[]  
[]          # 0: fork 3 => forks: [ (3, []) ]  
[ 3 ]       # 1: push 3  
[ 3 ]       # 2: jump 4  
[ 3 ]       # 4: fork 7 => forks: [ (3, []), (7, [3]) ]  
[ 3, 1 ]    # 5: push 1  
[ 3, 1 ]    # 6: jump 8  
[ 3 ]       # 8: call multiply  
[]          # 9: ret => output: 3  
[ 3 ]       # backtrack => restore stack ([3]) and pc (7), forks: [ (3, []) ]  
[ 3, 2 ]    # 7: push 2  
[ 6 ]       # 8: call multiply  
[]          # 9: ret => output: 6  
[]          # backtrack => restore stack ([]) and pc (3), forks: []  
[ 4 ]       # 3: push 4  
[ 4 ]       # 4: fork 7 => forks: [ (7, [4]) ]  
[ 4, 1 ]    # 5: push 1  
[ 4, 1 ]    # 6: jump 8  
[ 4 ]       # 8: call multiply  
[]          # 9: ret => output: 4  
[ 4 ]       # backtrack => restore stack ([4]) and pc (7), forks: []  
[ 4, 2 ]    # 7: push 2  
[ 8 ]       # 8: call multiply  
[]          # 9: ret => output: 8  
[]          # backtrack => no forks, terminate

The `fork` instruction has a program counter as its operand and when evaluated, the virtual machine stores the stack state and the operand. When it reaches at the end of the instruction list, the machine restores the stack state and the program counter, and then restart evaluation. When there is no states to restore, the program terminates.

The comma operator is compiled to the `fork` instruction, which makes the interpreter unique. This is why I think the comma operator is the most important operator in jq language.

The remain problem is how to save and restore the stack state. It is apparently less performant to deeply copy the whole stack. The solution is to use a singly-linked list. Implementing the stack on a singly-linked list allows us to create a fork-able stack efficiently.

After rewriting to stack-machine based interpreter and introducing the tail call optimization, gojq executes `"range(10000)"` in less than 0.2 sec and is now its performance is comparable with that of jq.

Currently in Apr 2020, gojq is already matured and implements most of the features of jq. You can use many options including `-R` (raw input), `-r` (raw output), `-c` (compact output), `-s` (slurp input), `--arg` and `--argjson`. Also modules feature with `-L` flag is implemented.

I also implemented arbitrary-precision integer calculation. This is not only important for keeping the precision of numeric IDs and filtering nanoseconds timestamps, but fun for solving some mathematical problems.

 % echo '{"foo": 4722366482869645213696}' | gojq .foo  
4722366482869645213696  
 % gojq -n '123456789 * 987654321'  
121932631112635269  
 % gojq -n '123456789123456789 * 987654321987654321'  
121932631356500531347203169112635269  
 % gojq -n '987654321987654321 % 123456789123456789'  
9000000009  
 % gojq -n -r 'def fact($n): if $n < 1 then 1 else $n * fact($n - 1) end; range(40;51) | "\(.)! = \(fact(.))"'  
40! = 815915283247897734345611269596115894272000000000  
41! = 33452526613163807108170062053440751665152000000000  
42! = 1405006117752879898543142606244511569936384000000000  
43! = 60415263063373835637355132068513997507264512000000000  
44! = 2658271574788448768043625811014615890319638528000000000  
45! = 119622220865480194561963161495657715064383733760000000000  
46! = 5502622159812088949850305428800254892961651752960000000000  
47! = 258623241511168180642964355153611979969197632389120000000000  
48! = 12413915592536072670862289047373375038521486354677760000000000  
49! = 608281864034267560872252163321295376887552831379210240000000000  
50! = 30414093201713378043612608166064768844377641568960512000000000000

You can use gojq to solve some Project Euler problems. For example, [Problem 20](https://projecteuler.net/problem=20) is the digit sum of a factorial number.

 % gojq -n 'def fact($n): if $n < 1 then 1 else $n * fact($n - 1) end;  
     fact(100) | tostring | explode | map(. - 48) | add'

[YAML](https://yaml.org/) is widely used these days. The gojq command supports `--yaml-input` flag to read a YAML file and `--yaml-output` to output in YAML format. For example, in order to lookup the mount volumes from the definition file of Docker Compose,

 % gojq --yaml-input --yaml-output ".services.app.volumes" docker-compose.yml  
- .:/app

You can also embed the gojq interpreter in a Go program. I assume the usage in the server and consider the security the big issue. The modules feature and reading the process environment variables is disabled by default (you can configure to use `os.Environ` if you want). You can also set the execution timeout in the `context` way.

I created a pure Go implementation of jq. My first motivation was to understanding the jq language completely. The language is not just for accessing the fields of a JSON, it is a Turing-complete language with iterator semantics. The comma operator `,` is compiled to the `fork` instruction and is the most important key to understand the language.

I have implemented almost all the functions of jq and now I find it easier to write a jq query than before. The `path` function is used for in assignment and modification operators. For example `LHS = V` is an alias for `reduce path(LHS) as $p (.; setpath($p; V))`, which reads `for each path from LHS, set the value to V`. You can use not only simple paths like `.foo` or `.[0]`, more complex paths like `..` or `.[]` combining with other filters. When you want to replace all the strings recursively,

 % echo '{"x":"foo","y":[0,"foo"],"z":{"w":"foofoo","q":"baz"}}' | \  
     jq '(.. | strings) |= sub("foo"; "bar"; "g")'  
{  
  "x": "bar",  
  "y": [  
    0,  
    "bar"  
  ],  
  "z": {  
    "w": "barbar",  
    "q": "baz"  
  }  
}

you can use this trick for filling a JSON template (where you can’t use `envsubst`).

Reimplementation is the best (but the hardest) way of understanding a software deeply. Creating language interpreters is so much fun. The instruction set characterizes the language. Specifically jq has an interesting `fork` instruction which makes the stack behavior interesting.

Thank you for reading!