---
link: https://jerrynsh.com/3-easy-ways-to-add-version-flag-in-go/
byline: Jerry Ng
site: Jerry Ng
date: 2024-10-08T01:00
excerpt: |-
  One of my favorite things about Go is its distribution process using the go install command. With go install, I don’t have to deal with the trouble of setting up brew, npm, pip, or any other package manager separately like I had to with some languages. It just works out of the box!

  Now a while back, I built a command-line (CLI) app in Go. I wanted to add a simple way for users to check the app's version after installing it via go install. This led me to discover different ways to add version fl
twitter: https://twitter.com/@jerrynsh
slurped: 2025-11-05T09:01
title: 3 Easy Ways To Add Version Flag in Go
tags:
  - go
  - version
---

One of my favorite things about Go is its distribution process using the [`go install` command](https://pkg.go.dev/cmd/go#hdr-Compile_and_install_packages_and_dependencies). With `go install`, I don’t have to deal with the trouble of setting up `brew`, `npm`, `pip`, or any other package manager separately like I had to with some languages. It just works out of the box!

Now a while back, [I built a command-line (CLI) app in Go](https://jerrynsh.com/how-i-scraped-michelin-guide-using-golang/). I wanted to add a simple way for users to check the app's version after installing it via `go install`. This led me to discover different ways to add version flags in Go.

## What I Wanted to Do

Let's say we have a Go CLI app named `mym`. My goals were straightforward:

1. This should work right after they install the app with `go install`
2. I wanted users to type `mym -version` or `mym -v` and see the app's current version:

```
❯ go install github.com/ngshiheng/michelin-my-maps/v2/cmd/[email protected]
❯ mym --version
Version: v2.6.1
```

This led me to explore different ways to do this in Go. Here's what I’ve gathered:

## Method 1: Build time injection

If you've done some digging online, you've probably come across this common solution online. It involves using the `-ldflags` switch with the build command to set version information into the binary during the build process. Here are the steps:

### Step 1: Define the version variable

First, you define a version variable in your `main` package:

```
package main

// Version is set during build time using -ldflags
var Version = "unknown"

// printVersion prints the application version
func printVersion() {
	fmt.Printf("Version: %s\n", Version)
}

func main() {
    versionFlag := flag.Bool("version", false, "print version information")
    flag.Parse()

    if *versionFlag {
        printVersion()
        return
	}

    // ...
}
```

### Step 2: Build with the version flag

Then, you add the `-ldflags` flag to your `go build` command to set the version dynamically:

```
❯ go build -ldflags "-X 'main.Version=v2.6.0'" cmd/mym/mym.go
❯ ./mym --version
Version: v2.6.0
```

### Step 3: Host the binary somewhere

Once you've built your Go binary with the version stamped using `-ldflags`, the next step is to host the binary on a platform where users can download, e.g. on GitHub release, AWS S3, or your own server.

Users who download this exact binary would then be able to run the command and get the same version information:

```
# (... download binary directly)
❯ mym --version
Version: v2.6.0
```

### Downside

While this approach works, I think it has some major drawbacks for both the developer and users:

1. It requires a few extra steps
2. You can't expect users to build the app themselves with a specific `ldflag`
3. Users can’t pass `ldflags` with `go install` (even if they could, expecting users to install your app with a long, complex command is poor UX)

### What about CI/CD?

Here's the thing: I already have a [CI in place that handles automatic releases](https://github.com/ngshiheng/michelin-my-maps/blob/3dadd302e607efd6ad31a95922f316316bf8fe69/.github/workflows/release.yml#L17). Implementing this method would require either:

1. Setting up another workflow, or
2. Modifying the existing workflow to include steps for building and uploading binaries with version flags.

This adds an extra layer of build process complexity. I thought we could do better!

## Method 2: Read from runtime build info

Git tags play an important role in [publishing and versioning Go modules](https://go.dev/blog/publishing-go-modules).

Each version tag in Git corresponds to a specific release of the module. When you push a new tag to your repository, it creates a new version of your module.

Then it hit me:

> Since my Go CLI app is a Go module, the version info is already available in the Git tag, right?
> 
> Why not reuse it by reading it at runtime?

### Here's how we can implement this approach

```
package main

import (
    "fmt"
    "runtime/debug"
)

// printVersion prints the application version
func printVersion() {
	buildInfo, ok := debug.ReadBuildInfo()
	if !ok {
		fmt.Println("Unable to determine version information.")
		return
	}

	if buildInfo.Main.Version != "" {
		fmt.Printf("Version: %s\n", buildInfo.Main.Version)
	} else {
		fmt.Println("Version: unknown")
	}
}

func main() {
    versionFlag := flag.Bool("version", false, "print version information")
    flag.Parse()

    if *versionFlag {
        printVersion()
        os.Exit(0)
    }

    // ...
}
```

NOTE: If the binary wasn’t built using `go install`, the version will show up as `"(devel)"`

From the end user’s perspective, all they needed to do was:

```
❯ go install github.com/ngshiheng/michelin-my-maps/v2/cmd/[email protected]
❯ mym --version
Version: v2.6.1
```

This means I don't have to modify my existing CI/CD workflow, and I get versioning out of the box from our automated release (`git tag`)!

## Method 3: Use the `versioninfo` module

As I was writing this, I came across [this blog post](https://blog.carlana.net/post/2023/golang-git-hash-how-to/) where someone else had the same issue and [created a Go package for this](https://github.com/earthboundkid/versioninfo). The simple package allows you to add a version flag to your CLI with just two lines of code!

```
package main

import (
    "flag"
    "fmt"

    "github.com/carlmjohnson/versioninfo" // 1. import
)

func main() {
    versioninfo.AddFlag(nil) // 2. add flag
    flag.Parse()
}
```

However, if you really don't want to add another module to your already long list of dependencies, you should probably stick to Method 2.

## Closing Thoughts

Adding version information to a Go CLI/app turned out to be more interesting than I initially thought.

While all three options are valid, Method 2 hit the sweet spot for me. It doesn't require any special build steps and works with `go install`. As a bonus point, it’s dependency-free, and won’t leave you scrambling if a [package disappears](https://en.wikipedia.org/wiki/Npm_left-pad_incident).