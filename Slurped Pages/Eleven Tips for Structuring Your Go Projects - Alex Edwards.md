---
link: https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects
byline: Alex Edwards
excerpt: "When working with Go, you have three main building blocks to help
  organize your code: files, packages and modules. But as Go developers, one of
  the common challenges we have is knowing how to best combine these building
  blocks to structure a codebase."
twitter: https://twitter.com/@ajmedwards
slurped: 2026-04-10T15:06
title: Eleven Tips for Structuring Your Go Projects - Alex Edwards
---

When working with Go, you have three main building blocks to help organize your code: _files_, _packages_ and _modules_. But as Go developers, one of the common challenges we have is _knowing how to best combine these building blocks_ to structure a codebase.

In this post, I'll share a mix of mindset tips and practical advice that I hope will help, especially if you're new to the language.

1. [Different projects, different structures](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#1-different-projects-different-structures)
2. [Aim for effective, not perfect](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#2-aim-for-effective-not-perfect)
3. [Forget conventions from other languages or frameworks](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#3-forget-conventions-from-other-languages-or-frameworks)
4. [Don't use directories just to organize files](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#4-dont-use-directories-just-to-organize-files)
5. [Use one of the standard layouts as a skeleton](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#5-use-one-of-the-standard-layouts-as-a-skeleton)
6. [And then let it evolve](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#6-then-let-it-evolve)
7. [If you're unsure, begin with two files](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#7-if-youre-unsure-begin-with-two-files)
8. [Keep related things close](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#8-keep-related-things-close)
9. [Big files aren't necessarily bad](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#9-big-files-arent-necessarily-bad)
10. [Create packages judiciously](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#10-create-packages-judiciously)
11. [Look out for warning signs](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#11-look-out-for-warning-signs)

## 1. Different projects, different structures

I'd like to start by emphasizing that there's no single "right" way to structure a Go codebase.

If you're using a specific framework or tool to scaffold your project, then you might be given a fixed directory structure to work with. But outside of that, there are relatively few conventions widely-adopted by the Go community, and the answer to "how should I structure my codebase?" is almost always "it depends".

It depends on what you're building, your business needs, your testing approach, your team, your dependencies or tooling, and any internal conventions you choose to follow.

Take a look at GitHub, and you'll find thousands of examples of successful Go projects — with quite different structures. For example, [mkcert](https://github.com/FiloSottile/mkcert) and [Kubernetes](https://github.com/kubernetes/kubernetes) are both excellent Go projects, but they differ significantly in scale and purpose. And these differences mean that their repository structures also look quite different.

A structure that works well for your current project might not be the same as the structures that you've used before or seen elsewhere — and that's perfectly fine.

## 2. Aim for effective, not perfect

If you're a perfectionist, this might be easier said than done, but try not to stress _too much_ about making your codebase structure perfect.

If you find yourself obsessing over the "perfect" way to organize your code, try to let go of that. Instead, aim for a structure that works _effectively enough_ for your specific project. By "effective enough," I mean that your code is easy to find and navigate, the logic is straightforward to follow, changes can be made with confidence, and you're not running into the kind of [warning signs](https://www.alexedwards.net/blog/11-tips-for-structuring-your-go-projects#12-look-out-for-warning-signs) I'll cover later in this post.

## 3. Forget conventions from other languages or frameworks

Don't feel guilty if your codebase structure doesn't follow the conventions or best practices you're used to from other languages or frameworks. If it works effectively for your Go project, that's what matters.

For example, if you're an experienced Ruby on Rails or Django developer building your first Go web application, you might be tempted to recreate the familiar directory structure from those frameworks. But while you probably could make it work if you tried, it's unlikely to be the most effective or simple solution for your Go project.

## 4. Don't use directories just to organize files

This is a subtle but important point, especially if you're new to Go.

You shouldn't create new directories _just_ to organize your `.go` files. In Go, creating a directory creates a new package, and placing a file in that directory makes it part of that package.

Create a directory only when you have a specific reason _to create a new package_ – not because you want a neater/cleaner/clearer directory structure for your files.

## 5. Use one of the standard layouts as a skeleton

The official Go documentation has a great article describing some [standard project layouts](https://go.dev/doc/modules/layout). I use one of these layouts as the high level "skeleton structure" in pretty much every Go project I work on nowadays, and recommend that you do too.

### Small projects

For small projects, consider using the [basic layout](https://go.dev/doc/modules/layout#basic-command) where you _just put everything in the project's root directory_, like this:

`├── main.go ├── foo.go ├── bar.go ├── go.mod └── README.md`

A couple of real-life examples of projects that use this layout are [mkcert](https://github.com/FiloSottile/mkcert) and [flow](https://github.com/alexedwards/flow).

### Small projects with supporting packages

For projects where you need to break out some code out into supporting packages, use the [supporting packages](https://go.dev/doc/modules/layout#package-or-command-with-supporting-packages) layout. In this pattern, the supporting packages live within an `internal` directory in the project root, and your `main` package files and other project assets continue to live in the root directory.

`├── internal │   └── foo │       └── foo.go ├── main.go ├── bar.go ├── go.mod └── README.md`

### Larger projects

For larger projects I generally recommend using the [server project](https://go.dev/doc/modules/layout#server-project) layout, especially if:

- Your project will have a lot non `.go` assets (like template files, SQL migrations, tool configurations and Makefiles); or
- Your project will contain more than one `main` package (e.g. `main` packages for a web application _and_ a CLI tool)

In this layout:

- Your executable `main` package files live in sub-directories under a `cmd` directory
- The rest of your Go packages live in an `internal` directory
- All other project assets remain in the root of the project directory

Like so:

`├── cmd │   └── foo │       ├── main.go │       └── bar.go ├── internal │   └── baz │       └── baz.go ├── go.mod ├── Makefile └── README.md`

For a more complete example, here's the [directory structure](https://gist.github.com/alexedwards/898083890d1e8f20c060b4fde9a50691) from a recent project I worked on – including `main` packages for a web server and CLI application, along with various non-Go assets.

## 6. And then let it evolve

Use one of the standard project layouts as your high-level 'skeleton', but beyond that, I recommend letting the rest of the structure _within that skeleton_ evolve naturally as development progresses.

In other words, don't decide your directory structure or what `.go` files you will have upfront, and then shoehorn in your Go code into that. Instead, _let the code you're writing guide the files and packages that you create_.

## 7. If you're unsure, begin with two files

If you're in any doubt, start with the basic layout and just a `go.mod` and `main.go` file in the root of your project directory. Then, as your project evolves, add additional files and packages as needed.

Starting this way is perfectly OK. Personally, about half of the new projects I work on begin with just these two files — and nothing more.

This one feels pretty obvious – especially if you're an experienced developer – but it's still worth saying. As a general rule, keep related things close to each other – in the same `.go` file or in the same package. Here are a few examples:

- Constants, variables, custom types and utility functions (which are not reused by multiple packages) should be declared close to the code they support, in the same `.go` file or package.
- It may make sense to group utility functions that are related to each other and used in multiple places into a single reusable package.
- If you have a custom struct type, define any methods for it directly below the struct declaration in the same `.go` file.
- In a web application or API, define all routing rules together in a single function or `.go` file.

There will probably be times when it makes sense for you to break the 'keep related things close' rule in your code, and that's OK, but it's a good principle to default to.

## 9. Big files aren't necessarily bad

So long as it doesn't cause you practical problems during development or maintenance, file size in Go doesn't matter. It's OK to have `.go` files that contain a couple of lines, or thousands. Neither of these things is automatically considered an anti-pattern in Go.

To give you an idea of some big files, the `runtime/proc.go` file from the Go standard library contains 6,548 lines of code. And `/pkg/apis/core/validation/validation.go` from the Kubernetes repository contains 8,606 lines of code (it's corresponding `_test.go` file also has over 26,000 lines).

I'm not saying that your `.go` files _should_ be big. More that if – on balance – it makes sense to have a big file… then it makes sense. Don't feel guilty about it, and don't feel like you need to break it into smaller files unless there's a good reason to.

## 10. Create packages judiciously

In a similar vein, big packages aren't necessarily bad. In fact, I'd say that one of the more common mistakes in Go is splitting up your code into _too many_ small packages.

The problem with having lots of small packages is that it can _add_ complexity to your application, especially when you need to share state, configuration, or dependencies across package boundaries. It also increases the likelihood of encountering import cycle problems.

As a rule of thumb, only create additional packages when you have a _demonstrable need_ or good reason to. For example:

- You have some code that you want to **reuse**. Putting the code in a standalone package facilitates this because you can then import the package and use it in different files throughout your project, or even copy-and-paste the package directory straight into another codebase.
- You want to **isolate or enforce a boundary** between the package code and the rest of your project. For example, you might use packages as an architectural tool to create lightweight decoupled 'layers' in your project code, or to isolate part of the codebase so it's easier for another person or team to work on separately.
- You have some code that acts as a 'black box' and moving it to a standalone package will **reduce cognitive overhead** and make your codebase clearer overall.

## 11. Look out for warning signs

It can be hard to know exactly when your project structure is working effectively… instead it's probably easier to spot the signs that it _isn't_ working effectively in practice. Some things to look out for are:

- You keep running into import cycle problems.
- It's hard to find things in the codebase, especially after time away or for new contributors.
- Relatively small changes often impact multiple packages or `.go` files.
- The flow of control is overly "jumpy" and hard to follow when debugging.
- There's a lot of duplication that's difficult to refactor out (note: [some duplication is not always bad.](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction))
- You're finding it difficult to manage errors appropriately.
- You feel like your are 'fighting the language', or you resort to using language features in a way that is not intended or idiomatic.
- It feels like a single file or package is doing _too much_ and that there isn't a clear separation of responsibilities within it, and this is having a negative effect on the clarity of your code.

If you spot these warning signs, it might be worth taking a step back and considering if tweaking the structure your codebase and packages will help to fix the problem.