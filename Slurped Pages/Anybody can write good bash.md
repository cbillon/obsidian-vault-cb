---
link: https://blog.yossarian.net/2020/01/23/Anybody-can-write-good-bash-with-a-little-effort
excerpt: |-
  Jan 23, 2020

         

      
        
          Tags:
          
          
            programming,
          
            rant
slurped: 2026-01-01T09:56
title: Anybody can write good bash (with a little effort)
tags:
  - bash
  - best_pratices
---


#### What this post is

A gentle admonishment to use shell scripts where appropriate accept that shell scripts will appear in your codebases and to lean _heavily_ on automated tools, modern features, safety rails, and best practices whenever possible.

---

Shell programming is a popular and predictable target of ire in programming communities[1](https://blog.yossarian.net/2020/01/23/Anybody-can-write-good-bash-with-a-little-effort#fn:1): virtually everybody has a horror story about a vintage, broken, or monstrous shell script underpinning a critical component of their development environment or project.

Personal favorites include:

- The ominous `run.sh`, which regularly:
    1. Runs _something_, _somewhere_
    2. Lacks the executable bit
    3. Doesn’t specify its shell with a [shebang](https://en.wikipedia.org/wiki/Shebang_\(Unix\))
    4. Expects to be run as a particular user, or _runs itself again_ in a different context
    5. Does very bad things if run from the wrong directory
    6. May or may not `fork`
    7. May or may not write a pidfile correctly, or at all
    8. May or may not _check_ that pidfile, and subsequently clobber itself on the next run
    9. All of the above
- `make.sh` (or `build.sh`, or `compile.sh`, or …), which:
    1. Doesn’t understand `CC`, `CXX`, `CFLAGS` or any other standard build environment variable
    2. Gets clever and tries to implement its own preprocessor
    3. Contains a half-baked `-j` implementation
    4. Contains a half-baked `make` implementation, including (broken) `install` and `clean` targets
    5. Decides that it knows better than you where it should be installed
    6. Fails if _anything, anywhere_ has a space in it
    7. Leaves the build in an indeterminate state if interrupted
    8. Happily chugs along after a command fails, leaving the build undiagnosable
    9. All of the above
- `test.sh`, which:
    1. Expects to be run in some kind of virtual environment (a `venv`, a container, a folder containing a `bundle`, &c)
    2. …tries to install and/or configure and/or load that virtual environment if not run inside it correctly, usually breaking more things
    3. Incorrectly detects that is or isn’t in the environment it wants, and tries to do the wrong thing
    4. Masks and/or ignores the exit codes of the test runner(s) it invokes internally
    5. Swallows and/or clobbers the output of the runner(s) it invokes internally
    6. Contains a half-baked unit test implementation that doesn’t clean up intermediates or handle signals correctly
    7. Gets really clever with colored output and doesn’t bother to check [`isatty`](https://linux.die.net/man/3/isatty)
    8. All of the above
- `env.sh`, which:
    1. May or may not actually be a shell script
    2. May or may `eval`ed into a shell process of indeterminate privilege and state _somewhere_ in your stack
    3. May or may not just be split on `=` in Python by your burnt-out DevOps person
    4. All of the above, at different stages and on different machines

I’ve experienced all of these, and am personally guilty of a (slight) majority of them[2](https://blog.yossarian.net/2020/01/23/Anybody-can-write-good-bash-with-a-little-effort#fn:2). Despite that (and perhaps because of it) I continue to believe that shell scripts[3](https://blog.yossarian.net/2020/01/23/Anybody-can-write-good-bash-with-a-little-effort#fn:3) have an important (and irreplaceable) niche in my development cycle, _and should occupy that same niche in yours_.

I’ll go through the steps I take to write (reliable, composable) `bash` below.

## Basics

A `bash` _script_ (i.e., a `bash` file that’s meant to be run directly) doesn’t end up in my codebases unless it:

- Has the executable bit
- Has a shebang _and_ that shebang is `#!/usr/bin/env bash`
    
    Explanation: not all systems have a (good) version of GNU bash at `/bin/bash`: macOS infamously supplies an ancient version at that path, and other platforms may use other paths.
    
- Has a top level comment that _briefly_ explains its functionality
- Has `set -e` (and ideally `set -euo pipefail`)[4](https://blog.yossarian.net/2020/01/23/Anybody-can-write-good-bash-with-a-little-effort#fn:4)
    
    Explanation: `set -e`, while not perfect, catches and makes fatal many types of (otherwise silent) failure. `set -u` makes expansions of undefined variables fatal, which catches the classic case of `rm -rf "${PERFIX}/usr/bin"`. `set -o pipefail` extends `-e` by making any failure _anywhere_ in a pipeline fatal, rather than just the last command.
    
- Can either:
    - Be run from _any_ directory, or
    - Fail immediately and loudly if it isn’t run from the correct directory

I also put two functions in (almost) every script:

```
function installed {
  cmd=$(command -v "${1}")

  [[ -n "${cmd}" ]] && [[ -f "${cmd}" ]]
  return ${?}
}

function die {
  >&2 echo "Fatal: ${@}"
  exit 1
}
```

_Edit_: [a Redditor has pointed out](https://www.reddit.com/r/programming/comments/esu8gu/anybody_can_write_good_bash_with_a_little_effort/ffdk2pl/) that this `installed` function is unnecessarily cautious and verbose.

These compose nicely with `bash`’s conditional tests and operators (and each other) to give me easy sanity checks at the top of my scripts:

```
[[ "${BASH_VERSINFO[0]}" -lt 4 ]] && die "Bash >=4 required"

deps=(curl nc dig)
for dep in "${deps[@]}"; do
  installed "${dep}" || die "Missing '${dep}'"
done
```

Some other niceties:

- I use `shopt -s extglob` and `shopt -s globstar` in some of my scripts, slightly preferring it over (simple) `find` invocations. Compare this `find` invocation:
    
    ```
    1
      items=$(find . -name 'foo*' -o -name 'bar*')
    ```
    
    to the shorter (and process-spawn-free):
    
    Linux Journal has a nice extended globbing reference [here](https://www.linuxjournal.com/content/bash-extended-globbing); `globstar` is explained in the GNU `shopt` documentation [here](https://www.gnu.org/software/bash/manual/html_node/The-Shopt-Builtin.html).
    

## Automated linting and formatting

In terms of popularity and functionality, [`shellcheck`](https://github.com/koalaman/shellcheck) reigns supreme. Going by [its changelog](https://github.com/koalaman/shellcheck/blob/master/CHANGELOG.md#v010---2013-07-23), `shellcheck` has been around for a little under 7 years. It’s also available in [just about every package manager](https://repology.org/project/shellcheck/versions).

As of 0.7.0, `shellcheck` can even auto-generate (unified-format) patches for some problems:

```
shellcheck -f diff my_script.sh | patch
```

And includes a (sadly optional) check for my personal bugbear: non-mandatory variable braces:

```
# Bad
foo="$bar"
stuff="$# $? $$ $_"

# Good
foo="${bar}"
stuff="${#} ${?} ${$} ${_}"
```

`shellcheck` also doesn’t complain about usage of `[` (instead of `[[`), even when the shell is explicitly GNU bash[5](https://blog.yossarian.net/2020/01/23/Anybody-can-write-good-bash-with-a-little-effort#fn:5).

There’s also [`bashate`](https://github.com/openstack/bashate) and [`mvdan/sh`](https://github.com/mvdan/sh), neither of which I’ve used.

## Environment variables, not flags

In the past, I’ve used the `shift` and `getopt` builtins (sometimes at the same time) to do flag parsing. I’ve mostly given up on that, and have switched to the following pattern:

- Boolean and trivial flags are passed via environment variables:
    
    ```
      VERBOSE=1 STAMP=$(date +%s) frobulate-website
    ```
    
    I find this substantially easier to read and remember than flags (did I use `-v` or `-V` for verbose in this script?), and allows me to use this nice syntax for defaults:
    
    ```
      VERBOSE=${VERBOSE:-0}
      STAMP=${STAMP:-$(date +%s)}
    ```
    
- Where possible `stdin`, `stdout`, and `stderr` are used instead of dedicated positional files:
    
    ```
      VERBOSE=1 DEBUG=1 frobulate-website < /var/www/index.html > /var/www/index2.html
    ```
    
- The only parameters are positional ones, and should _generally_ conform to a variable-argument pattern (i.e., `program <arg> [arg ...]`).
    
- `-h` and `-v` are only added if the program has non-trivial argument handling _and_ is expected to be (substantially) revised in the future.
    - I _generally_ prefer not to implement `-v` at all, favoring a line in the header of `-h`’s output instead.
    - Running the command with no arguments is treated as equivalent to `-h`.
- All other kinds of flags, inputs, and mechanisms (including `getopt`) are only used as a last resort.

## Compose liberally

Don’t be afraid of composing pipes and subshells:

```
# Combine the outputs of two `stage-run` invocations for
# a single pipeline into `stage-two`
(stage-one foo && stage-one bar) | stage-two
```

Or of using code blocks to group operations:

```
# Code blocks aren't subshells, so `exit` works as expected
risky-thing || { >&2 echo "risky-thing didn't work!"; exit 1; }
```

Subshells and blocks can be used in many of the same contexts; which one you use should depend on whether you need an independent temporary shell or not:

```
# Both of these work, but the latter preserves the variables

(read line1 && read line2 && echo "${line1} vs. ${line2}") < "${some_input}"
# line1 and line2 are undefined

{ read line1 && read line2 && echo "${line1} vs. ${line2}"; } < "${some_input}"
# line1 and line2 are defined and contain their last values
```

Note the slight syntactic differences: blocks require spacing and a final semicolon (when on a single line).

Use process substitution to avoid temporary file creation and management:

_Bad_:

```
function cleanup {
  rm -f /tmp/foo-*
}

output=$(mktemp -t foo-XXXXXX)
trap cleanup EXIT

first-stage output
second-stage --some-annoying-input-flag output
```

_Good_:

```
second-stage --some-annoying-input-flag <(first-stage)
```

You can also use them to cleanly process `stderr`:

```
1
2
# Drop `big-task`'s stdout and redirect its stderr to a substituted process
(big-task > /dev/null) 2> >(sed -ne '/^EMERG: /p')
```

## Roundup

The shell is a particularly _bad_ programming language that is particularly easy to write (unsafe, unreadable) code in.

It’s also a particularly _effective_ language with idioms and primitives that are hard to (tersely, faithfully) reproduce in objectively better languages.

It’s also not going anywhere anytime soon: according to [`sloccount`](https://linux.die.net/man/1/sloccount), [`kubernetes@e41bb32`](https://github.com/kubernetes/kubernetes/tree/e41bb325c2453fc373826f5cd2b8d9b106038d2f) has 28055 lines of shell in it[6](https://blog.yossarian.net/2020/01/23/Anybody-can-write-good-bash-with-a-little-effort#fn:6).

The moral of the story: shell is going to sneak into your projects. You should be prepared with good practices and good tooling for when it does.

If you somehow manage to keep it out of your projects[7](https://blog.yossarian.net/2020/01/23/Anybody-can-write-good-bash-with-a-little-effort#fn:7), people will use shell to deploy your projects or to integrate it into _their_ projects. You should be prepared to justify your project’s behavior and (non-)conformity to the (again, objectively bad) status quo of UNIX-like environments for when they come knocking.

---

---