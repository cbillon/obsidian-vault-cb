---
link: https://laurentsv.com/blog/2024/10/19/no-nonsense-go-package-layout.html
excerpt: "It’s a recurring question on gopher slack and discord: «How should I set up my go project repository?». Unfortunately, there are a lot of both outdated and o..."
slurped: 2026-01-26T19:21
title: No nonsense guide to Go projects layout
tags:
  - golang
  - guide
---

It’s a recurring question on gopher slack and discord: «How should I set up my go project repository?». Unfortunately, there are a _lot_ of both outdated and overly complicated blogs, sample repositories and old projects out there.

One key philosophy of Go, is keeping things simple. More code writing, less structuring ahead of time (before it’s actually needed).

### Official

The official guide on this topic is [go.dev/doc/modules/layout](https://go.dev/doc/modules/layout) - which you should read if you haven’t already, then come back here. Despite that reference (and maybe in part because of it) people still tend to over complicate. For instance it says

> **Larger** packages or commands **may** benefit from splitting off some functionality into supporting packages. **Initially,** it’s recommended placing such packages into a directory named internal; […] Since other projects cannot import code from our internal directory, we’re free to refactor its API and generally move things around without breaking external users.

Emphasis on _larger_ and _may_ is mine. And seems to be missed by many. 99% of people do not need `internal/` when, be real, most of what gets written will never be reused much and even if it is, don’t worry about over publishing. And the _initially_ is, in my opinion wrong. Initially, make something useful and don’t worry that it’ll get such great adoption that you’re stuck into maintaining your early layout forever, you’re not… (see semver point below too).

Likewise, you don’t need to use `cmd/` when you only have 1 binary (or even a few and no library).

In general because a feature exists, doesn’t mean to have to use it (e.g `internal/`). Or because some people have a convention doesn’t mean you should follow (like your mom probably used to say “if they jump off a bridge… will you?”, remember? ).

By the way, stick to 0.x semver (another pet peeve of mine is the v2/v3/… in go) for as long as you can and document what you change and why, rather than under-publishing and forcing people to fork to access what they really need and you didn’t foresee.

### The bad

So one infamous example of the nonsense is the fake “golang-standards” - see [github.com/golang-standards/project-layout/issues/117](https://github.com/golang-standards/project-layout/issues/117) and other closed issues for why it’s _not_ a standard. This led folks to come up with a counter point at [github.com/go-standard/project-layout](https://github.com/go-standard/project-layout) (which I contributed a few additions/corrections) but also got flak for having the word “standard” in it… so here you go, below is _my_ **opinion** (based on decades of professional experience in many languages, including 7 years of go at top companies and OSS projects) - Some of the text comes from there:

### My Take

#### Where should main go?

Main package in the root is great, it makes for the shortest and cleanest install/run:

```
go install github.com/you/project@latest
```

Which is typically the priority if you do have a CLI or server to ship. Pure libraries obviously have no main at the root.

If you do have both, you could move the main out of the way in a child directory (people like `cmd/execname/` but the `cmd/` part is actually not needed) or move the library into a directory that will make importing and using it readable. Consider also you may have more than 1 library package to publish in which case `lib1/` and `lib2/` makes total sense and keep the top level for the main binary.

For many small to medium-sized projects, these are the only packages you need.

#### internal/

Just in case you do not know already, as a refresher, `internal/` is not a convention, it’s a feature of the go tool chain: code in or below such directory is importable only by code in the directory tree rooted at the parent of `internal/`.

But you don’t need that, unless you’re shipping code to _a lot_ of third party users and have a lot of code that can’t just be lower case not exported, yet used in many exported packages, you don’t need to hide your code… it’s not so precious or such a huge commitment.

#### pkg/

Any package you put into a pkg directory can be moved to the root. `pkg` is a very outdated convention from before `internal/` (and you saw what I think about internal/).

Some people think that having packages at the top, “clutters”, the directory. First what does get cluttered if you add unnecessary directory layers in every single import in dozen of files and possibly hundreds of dependencies… think about them! Second, feel free to move the various docs, ancillary files, etc to some subdirectory. Lastly what is ‘clutter’ really, navigation through the code probably comes through your “godoc” (the excellent [pkg.go.dev](https://pkg.go.dev/), or your self hosted equivalent for private repositories).

Btw don’t get me started with repos that have dozens of packages and files and where to see what is going on without an IDE, you have to go: main.go -> some other file -> some init file -> some start function -> some other files… while all this could be in 1 or 2 places, easier to follow and find.

#### util/

Those functions, types, and methods which you think belong in a separate `util/` (or `common/`, or `shared/`, or `lib/`) package simply don’t. Try to put them where they belong, probably near where they are used, or in their own meaningfully named package (e.g applog for your application logging utils).

But Laurent, I’m more comfortable with…

Let me guess, you have more experience writing Java, Python, Ruby, JavaScript, TypeScript, or C# than you do writing C, C++, or Perl. Please consider that package structuring is more an art than hard rules.

#### Too many packages

You can have more than 1 file in a package/directory in go. You don’t need to make a new package each time to separate code/types/… And the more packages the more chances to create a loop or having to later move code around. This being said, moving code around is good and an easy way to grow your code base - later, when needed.

Also please don’t make many files with only a few lines in them either, it makes it painful to look at your code.

#### Examples

There are lots of example of overly complicated repos. I’d give my own repos but they also evolved organically and aren’t actually that good/or simple, but here we go anyway:

Repositories in [github.com/fortio/](http://github.com/fortio/)

Server example: [github.com/fortio/proxy](https://github.com/fortio/proxy)

CLI example: [github.com/fortio/multicurl](https://github.com/fortio/multicurl)

Server + libraries + CLI: [github.com/fortio/fortio](https://github.com/fortio/fortio)

Library + CLIs: that one is somewhat oddly set up as it has 2 libraries, one at the root where it started and a newer one, so not exactly best practices [github.com/fortio/terminal](https://github.com/fortio/terminal)

Language with CLI, libraries and wasm target: [github.com/grol-io/grol](https://github.com/grol-io/grol)

### But…

This post is partially in response to what I see is over complication in many small projects, which isn’t to say it’s always right or applicable either, but if it gets folks to pause and consider, even to disagree. I think it’s mission accomplished.

Ping me on gopher slack (Laurent Demailly) or discord (_dl) if you disagree, have comments etc… or open an [issue](https://github.com/ldemailly/laurentsv/issues) or comment directly below if you’re on facebook.

### In the same vein

You might like [simple effective 7 lines Dockerfile and minimal sized images](https://laurentsv.com/blog/2024/06/25/stop-the-go-and-docker-madness.html) and [my other thoughts on go](https://laurentsv.com/blog/2024/09/26/golang-7-years-later.html).