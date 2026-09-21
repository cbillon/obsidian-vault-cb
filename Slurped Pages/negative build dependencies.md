---
link: https://jyn.dev/negative-build-dependencies/
byline: jyn
excerpt: A build system needs to know not just which files exist, but which
  shouldn't exist.
slurped: 2026-09-18T09:02
title: negative build dependencies
---

This post is part 2/4 of [a series about build systems](https://jyn.dev/four-posts-about-build-systems/). The next post is [I want a better action graph serialization](https://jyn.dev/i-want-a-better-action-graph-serialization/).

---

This post is about a limitation of the dependencies you can express in a build file. It uses Ninja just because it's simple and I'm very familiar with it, but the problem exists in most build systems.

Say you have a C project with two different include paths: `dependency/include` and `src`:

```
.
├── build.ninja
├── dependency
│   └── include
│       └── interface.h
└── src
    └── main.c
```

and the following build.ninja:[1](https://jyn.dev/negative-build-dependencies/#fn-depfiles)

```
rule cc
  command = cc -I dependency/include -I src $in -o $out

build src/main: cc src/main.c | dependency/include/interface.h
```

and some small implementations, just so we can see that it actually works:

```
// src/main.c
#include <stdio.h>
#include "interface.h"

int main(void) {
        printf("%d\n", interface_version);
}

// dependency/include/interface.h
const static short interface_version = 1;
```

This works ok on our first build, and when `dependency/include/interface.h` changes:

```
$ ninja
[1/1] cc -I dependency/include -I src src/main.c -o src/main
```

But say now that we add our own `src/interface.h` file:

```
$ cat src/interface.h
const static char *version = "1.0";
$ ninja
ninja: no work to do.
```

Now we have a problem. If we remove our build artifacts, ninja _does_ rerun, and shows us what that problem is:

```
[1/1] cc -I dependency/include -I src src/main.c -o src/main
FAILED: [code=1] src/main
cc -I dependency/include -I src src/main.c -o src/main
src/main.c: In function ‘main’:
src/main.c:5:24: error: ‘interface_version’ undeclared (first use in this function)
    5 |         printf("%d\n", interface_version);
      |                        ^~~~~~~~~~~~~~~~~
src/main.c:5:24: note: each undeclared identifier is reported only once for each function it appears in
ninja: build stopped: subcommand failed.
```

We switched out what `interface.h` resolved to, but ninja didn't know about the changed dependency.

What we want is a way to tell it to rebuild if the file `src/interface.h` is created. To my knowledge, ninja has no way of expressing this kind of negative dependency edge.

Thanks to David Chisnall and Ben Boeckel (**@mathstuf**) for making me aware of this issue.

1. People familiar with ninja might say this looks odd and it should use a `depfile` with `cc -M` so dependencies are tracked automatically. That makes your files shorter and means you need to run the configure step less often, but it doesn't actually solve the problem of negative dependencies. Ninja still doesn't know when it needs to regenerate the depfile. [↩](https://jyn.dev/negative-build-dependencies/#fr-depfiles-1)