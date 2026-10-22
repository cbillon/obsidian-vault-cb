---
link: https://www.amazingcto.com/go-build-tools-make-bazel-just-xc-taskfile-mage/
byline: Stephan Schmidt
site: Amazing CTO
date: 2025-08-14T00:00
excerpt: Start with go build, add tools when needed. Makefile works for most
  teams. Mage, Taskfile, and goreleaser for more power. Here's how to choose.
twitter: https://twitter.com/@KingOfCoders
slurped: 2026-10-20T20:56
title: Build Tools for Go
---

Programming languages have their build tools. Java has Maven, Scala has SBT, and Javascript has NPM beside many web build tools. Newcomers to Golang wonder about the build tool for Go. Starting is easy, the go compiler is easier to use than comparable alternatives, to start no build tool is necessary.

All of these

```
go build
go run
go install
```

do the job.

When you have more than Go code to build, but tests to run, code to lint etc. the newcomer to Golang asks what now?

I’ve started with `go build`, added `bash` scripts for linting and deployment and for now I’ve settled with `Makefile` and `make`—replacing all [bash scripts](app://obsidian.md/simplicity-of-golang-systemd-deployments/) with make targets.

```
build:  git-hash
  templ generate ./...
  go build -o bin/ ./...
```

Here we a have a `build` target, that depends on another `git-hash` target. The target needs to generate `a-h/templ` templates and then build the application. For most people `Makefile` is enough, it’s easy to use for basic tasks and available on many systems.

**Asking around [r/Golang](https://www.reddit.com/r/golang/), more options popped up:**

### [bazel](https://github.com/bazelbuild/bazel)

bazel was [built by Google](app://obsidian.md/act-like-meta/) for large monorepos with many different languages. But works for small projects just fine, and the Go integration seems to be the best language integration in bazel.

### [goreleaser](https://goreleaser.com/)

Not a build tool for all the project tasks you might have used `Makefile` for, but a build tool to build Go for many different platforms and operating system targets. It can cross compile, build for Linux packaging like deb and release to GitHub, beside many more targets.

### [just](https://github.com/casey/just)

just is inspired by `make` but tries to make it easier and better.

### [Taskfile](https://taskfile.dev/)

Another one for a better `make`: _“Task is a task runner / build tool that aims to be simpler and easier to use than, for example, GNU Make.”_

It is different because it is using YAML:

```
version: '3'
tasks:
  hello:
    cmds:
      - echo 'Hello World from Task!'
      silent: true
```

### [mage](https://github.com/magefile/mage)

mage uses Go to define build files

```
//go:build mage
package main

import (
  "github.com/magefile/mage/sh"
)

// Runs go mod download and then installs the binary.
func Build() error {
  if err := sh.Run("go", "mod", "download"); err != nil {
    return err
  }
  return sh.Run("go", "install", "./...")
}
```

### [XC](https://github.com/joerdav/xc)

_“The problem `xc` is intended to solve is scripts maintained separately from their documentation.”_ So with XC the build file contains its own documentation in Markdown

```
## tag

Deploys a new tag for the repo.

Requires: test

'''
export VERSION=`git rev-list --count HEAD`
echo Adding git tag with version v0.0.${VERSION}
git tag v0.0.${VERSION}
git push origin v0.0.${VERSION}
'''
```

### [script](https://github.com/bitfield/script)

_"`script` is a Go library for doing the kind of tasks that shell scripts are good at: reading files, executing subprocesses, counting lines, matching strings, and so on."_

Can be used together with `mage`

_So there are many different options for build tools with Go. Start with `go build`, then use the one that suits you best—might be plain old `make`._

**About me:** Hey, I'm Stephan, I help [CTOs with Coaching](app://obsidian.md/cto-coaching-from-experienced-cto/), with 40+ years of software development and 25+ years of engineering management experience. I've coached and mentored 100+ CTOs and founders in more than 1200 sparring and coaching sessions. I've founded 3 startups. 1 nice exit. I help CTOs and engineering leaders grow, scale their teams, gain clarity, lead with confidence and navigate the challenges of fast-growing companies.

Most of the CTOs I coach didn't know CTO coaching was a thing until they were already drowning. It is a thing - here's [what it is](app://obsidian.md/what-is-cto-coaching/).