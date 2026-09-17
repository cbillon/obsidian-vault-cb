---
link: https://will-keleher.com/posts/more-shell-patterns-i-use-weekly/
byline: Will Keleher
site: will keleher
date: 2026-02-27T01:00
excerpt: A few quick notes on string parameter expansion (`${file/.md/.html}`),
  adding colors to `printf`, `/tmp` and `mktemp`, and using `awk` for filtering
twitter: https://twitter.com/@WKeleher
slurped: 2026-09-17T09:12
title: More Shell Patterns I Use Weekly
---

💡

Covers find-and-replace with `sed` and `xargs`, `while` loops until success/failure, parallelizing commands using `wait`, and `$SECONDS`.

I’m not a particularly strong shell programmer. I’m incapable of writing a `for` loop in Bash that handles input with spaces in it. Or writing a `trap`. Shell arrays? Not a chance without checking the docs. Whenever I need to do anything more advanced, I normally switch to `python` or `zx`.

But I’m still strong enough on the command line that I’m able to accomplish the things that I want to accomplish day-to-day and build quick little utilities that serve my needs well. Here are a few quick notes on patterns I find myself using regularly.

### String parameter expansion

```
for file in **/*.md; do
    pandoc $file -o "${file/.md/.html}"
done
```

I’m terrible with shell string manipulation, but I can still remember `${variable/pattern/replacement}`. There are two ways of referring to a variable in the shell: `$x` and `${x}`. `${x}` mode gives you a few useful things:

1. `${x}y` will print the variable stored in x followed by the character y. `$xy` will print nothing unless `xy` is set
2. `${x:-string_fallback}` lets you create fallbacks if a variable is unset: `echo ${x:-hello, world}`
3. `x=page.html; ${x%%.html}` will remove `.html` from the end of the string (but I just do `${x/.html/}` because I can’t remember `%` and `#`)
4. And the aforementioned `${x/pattern/replacement}` syntax I already mentioned.

The `**/*.md` syntax is supported in zsh (and in Bash with `shopt -s globstar`): [You probably don’t need `find`](https://will-keleher.com/posts/you-probably-dont-need-find/).

In addition to `pandoc`, I’ll often write `for` loops with renames with tools like `magick` and `ffmpeg`.

### printf / chalk

[Urjit](https://urjit.io/) introduced me to the pattern of including a copy-pastable `print` colors block to scripts.

```
# ----- Colours easy def: Usage: printf "%s\n" "${red}foobar${end}"
red=$'\e[1;31m'
green=$'\e[1;32m'
yellow=$'\e[1;33m'
blue=$'\e[1;34m'
mag=$'\e[1;35m'
cyn=$'\e[1;36m'
end=$'\e[0m'

printf "\n\t%s\n" "${red}Helpful error message that we want to highlight.${end}"
```

A teeny bit of effort spent making script output more legible can make hacked-together shell scripts nicer to work with.

In general though, once I start adding error messages to a shell script, that’s about the point where I want to switch it over to a language like JS (or zx) or python. In JS, [chalk](https://www.npmjs.com/package/chalk) makes it quick to augment console output.

### Using `mktemp` and `/tmp` heavily

This isn’t a command per se, but I’ve started using the `/tmp` directory more. Having a command that spits out a ton of output that I’ll only care about if it fails is a bummer, so it’s nice to have a spot to stash that data that will automatically get cleaned up.

Your operating system will (generally) take care of keeping `/tmp` clean of old files.

```
output_log_file=$(mktemp)
if ! some_build_command_with_tons_of_output &> "$output_log_file"; then
    printf "\n%s\n" "${red}some_build_command_with_tons_of_output FAILED${end}"
    printf "\t%s\n" "${red}Full log output available at ${output_log_file}${end}"
    printf "\t%s\n\n" "${red}Last ten lines of output:${end}"
    tail -n10 "$output_log_file"
    exit 1
fi
```

This is a helpful pattern to use with LLMs because it avoids polluting the context window with random success messages.[1](https://will-keleher.com/posts/more-shell-patterns-i-use-weekly/#fn:1) Humans have a context window too, and if something is working properly, I generally don’t care to see its logs.

This particular script isn’t great – it sticks together stdout and stderr and it loses the exact error code from the failing command – but it illustrates the general pattern.

💡

In the past, I’ve used scripts similar to [allelify](https://www.npmjs.com/package/allelify) in JS to parallelize command runs while limiting output. I don’t recommend using this setup today – it’s not DAG-aware and isn’t built for an LLM-heavy world – but the general approach of storing logs in `/tmp` unless things fail is useful.

I’ll also occasionally add things like `| tee /tmp/just-in-case` to commands I run. Being able to use `less` to navigate around failed commands feels more ergonomic than having to run a heavy command again to get its output.

It’s also worth mentioning that `mktemp` can create directories as well as files if you need a temporary directory to do some quick work in.

### Using `$()` in simple commands rather than assigning to an intermediate variable

With more familiarity with the shell, I’ve stopped using intermediate variables when writing quick scripts in the shell. I think this is a less readable style than having explicit named variables, but I find it more ergonomic, especially because it keeps the command as a one-liner that I can edit and replay:

```
x=$(command | grep foo)
do_something_with_x "$x"

# vs.

do_something_with_x "$(command | grep foo)"
```

- `mocha $(rg -l pattern)`: run the tests that match the search. (On a large codebase, this will be much faster than `mocha -g` because `mocha -g` needs to load all of the test files to decide which ones to run)
- `hx $(which command)` or `hx $(rg -l pattern | fzf)`: edit the shell script behind `command` (for stuff I’ve written) or edit a particular file after tracking it down

If you’re not yet comfortable with `$()` and `<()`, it probably makes sense to continue using intermediate variables.

### Filtering with `awk` rather than `grep`

- `awk '!seen[$0]++'`: removes duplicates. This is alternative to `| sort | uniq`. `awk` is 1-indexed, and `$0` refers to the full line. `seen` (or any other variable) will be created when referenced. `awk` will print the full line when provided truthy values.
- `awk '$5 == "keyword"'`: avoids false positives with `grep keyword` because it’s only looking for the fifth element
- `awk '$2 ~ "pattern"'`: is similarly more focused than a similar grep command

I’ve occasionally started using `grep` for a filter, made a complex search, and then realized that I should have just started with `awk`.