---
link: https://rednafi.com/go/tool-directive/
byline: Redowan Delowar
site: Redowan's Reflections
date: 2025-04-13T01:00
excerpt: |-
  Go 1.24 added a new tool directive that makes it easier to manage your project&rsquo;s tooling.
  I used to rely on Make targets to install and run tools like stringer, mockgen, and linters like gofumpt, goimports, staticcheck, and errcheck. Problem is, these installations were global, and they&rsquo;d often clash between projects.
  Another big issue was frequent version mismatch. I ran into cases where people were formatting the same codebase differently because they had different versions of the tools installed. Then CI would yell at everyone because it was always installing the latest version of the tools before running them. Chaos!
tags:
  - go
  - tools
slurped: 2025-11-05T17:30
title: Go 1.24's "tool" directive
---

Go 1.24 added a new `tool` directive that makes it easier to manage your project’s tooling.

I used to rely on Make targets to install and run tools like `stringer`, `mockgen`, and linters like `gofumpt`, `goimports`, `staticcheck`, and `errcheck`. Problem is, these installations were global, and they’d often clash between projects.

Another big issue was frequent version mismatch. I ran into cases where people were formatting the same codebase differently because they had different versions of the tools installed. Then CI would yell at everyone because it was always installing the latest version of the tools before running them. Chaos!

To avoid this mess, the Go community came up with a convention where you’d pin your tool versions in a `tools.go` file. I’ve [written about this before](app://obsidian.md/go/omit_dev_dependencies_in_binaries). But the gist is, you’d have a `tools.go` file in your root directory that imports the tooling and assigns them to `_`:

```
//go:build tools

// tools.go
package tools

import (
    _ "github.com/golangci/golangci-lint/cmd/golangci-lint"
    _ "mvdan.cc/gofumpt"
)
```

Since these dependencies aren’t used directly in the codebase, the `//go:build tools` directive ensures they’re excluded from the main build.

Then running `go mod tidy` keeps things clean and includes these dev dependencies in the `go.mod` and `go.sum` files.

This works, but it always felt a bit clunky. You end up polluting your main `go.mod` with tooling-only dependencies. And sometimes, transitive dependencies of those tools clash with your app’s dependencies.

The new `tool` directive in Go 1.24 solves [some of these pain points](app://obsidian.md/go/tool_directive/#still-not-perfect).

With Go 1.24, you can now add tooling with the `-tool` flag when using `go get`:

```
go get -tool github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

This adds the dependency to your `go.mod` like this:

```
module github.com/rednafi/foo

go 1.24.2

tool github.com/golangci/golangci-lint/cmd/golangci-lint

// ... other transitive dependencies
```

Notice the `tool` directive clearly separates these from regular module dependencies.

Then you can run the tool with:

```
go tool golangci-lint run ./...
```

One thing to keep in mind: the first time you run a tool this way, it might take a second — Go needs to compile it before running if it isn’t already compiled. After that, it’s cached, so subsequent runs are fast.

## What about `go generate`?

This also plays nicely with `go generate`. I’ve started replacing direct tool calls with `go tool`, so contributors don’t need to install tools globally. Just run `go generate` and you’re done:

```
//go:generate go tool stringer -type=MyEnum
```

No further setup needed, no path issues, and it’s always using the version you pinned.

## Still not perfect

That said, one thing still bugs me: `go get -tool` adds these dev tools to the main `go.mod` file. That means your application and dev dependencies are still mixed together. Same problem the `tools.go` hack had.

There’s no built-in way to avoid this yet. So your options are:

- Accept that dev and app deps will live in the same `go.mod` file.
- Create a separate `tools` module to isolate your tooling. A bit clunky, but doable.

I went with the second option.

My layout looks like this:

```
.
├── go.mod
├── go.sum
└── tools
    └── go.mod
```

Then I install tools like this:

```
cd tools
go get -tool github.com/golangci/golangci-lint/cmd/golangci-lint@latest
```

And run them from the root directory as follows:

```
go tool -modfile tools/go.mod golangci-lint run ./...
```

The `go tool` command supports a `-modfile` flag that you can use to specify where to pull the tool version from. I _really_ wish `go get` supported `-modfile` too — that way you wouldn’t need to manage the dependencies in such a wonky manner. This was close to being perfect. Well, maybe in a future release.

Another limitation is that it only works with tools written in Go. So if you’re using stuff like `eslint`, `prettier`, or `jq`, you’re on your own. But for most of my projects, the dev tooling is written in Go anyway, so this setup has been working okay.

~~~

## Recent posts

- [Revisiting interface segregation in Go](https://rednafi.com/go/interface-segregation/)
- [Avoiding collisions in Go context keys](https://rednafi.com/go/avoid-context-key-collisions/)
- [Organizing Go tests](https://rednafi.com/go/organizing-tests/)
- [Subtest grouping in Go](https://rednafi.com/go/subtest-grouping/)
- [Let the domain guide your application structure](https://rednafi.com/go/app-structure/)
- [Test state, not interactions](https://rednafi.com/go/test-state-not-interactions/)
- [Early return and goroutine leak](https://rednafi.com/go/early-return-and-goroutine-leak/)
- [Lifecycle management in Go tests](https://rednafi.com/go/lifecycle-management-in-tests/)
- [Gateway pattern for external service calls](https://rednafi.com/go/gateway-pattern/)
- [Flags for discoverable test config in Go](https://rednafi.com/go/test-config-with-flags/)