---
link: https://blog.jetbrains.com/go/2025/10/16/the-10x-commandments-of-highly-effective-go/
byline: Anna Protsenko
site: The JetBrains Blog
excerpt: What makes Go developers truly effective? In this guest post, John Arundel shares ten practical “commandments” of Go excellence – timeless lessons for writing cleaner, safer, and more maintainable Go code.
twitter: https://twitter.com/@jetbrains
slurped: 2025-11-05T18:16
title: The “10x” Commandments of Highly Effective Go | The GoLand Blog
tags:
  - go
  - tips
---

[GoLand](https://blog.jetbrains.com/go/category/goland/)

## The “10x” Commandments of Highly Effective Go

_This is a guest post from John Arundel of Bitfield Consulting, a Go trainer and writer who runs a_ [_free newsletter for Go learners_](https://bitfieldconsulting.com/subscribe)_. His most recent book is_ [_The Deeper Love of Go_](https://bitfieldconsulting.com/books/deeper).

Ever wondered if there’s a software engineer, somewhere, who actually knows what they’re doing? Well, I finally found the one serene, omnicompetent guru who writes perfect code. I can’t disclose the location of her mountain hermitage, but I _can_ share her ten mantras of Go excellence. Let’s meditate on them together.

## 1. Write packages, not programs

The [standard library](https://pkg.go.dev/std) is great, but the [universal library](https://pkg.go.dev/) of free, open-source software is Go’s biggest asset. Return the favour by writing not just programs, but [packages](https://bitfieldconsulting.com/posts/packages) that others can use too.

Your `main` function’s only job should be parsing flags and arguments, and handling errors and cleanup, while your imported “domain” package does the real work.  
Flexible packages return data instead of printing, and return errors rather than calling `panic` or `os.Exit`. Keep your [module structure](https://go.dev/doc/modules/layout) simple: ideally, one package.

**Tip:** Use _Structure view_ (Cmd-F12) for a high-level picture of your module.

![](https://blog.jetbrains.com/wp-content/uploads/2025/10/01_write_packages_not_programs.png)

## 2. Test everything

Writing tests helps you [dogfood](https://en.wikipedia.org/wiki/Eating_your_own_dog_food) your packages: awkward names and inconvenient APIs are obvious when you use them yourself.

[Test names should be sentences](https://bitfieldconsulting.com/posts/test-names). Focus tests on small units of user-visible behaviour. Add [integration tests](https://en.wikipedia.org/wiki/Integration_testing) for end-to-end checks. Test binaries with [`testscript`](https://bitfieldconsulting.com/posts/test-scripts).

**Tip:** Use GoLand’s “generate tests” feature to add tests for existing code. _Run with coverage_ can identify untested code. Use the debugger to analyse test failures.

## 3. Write code for reading

Ask a co-worker to read your code line by line and tell you what it does. Their stumbles will show you where your speed-bumps are: flatten them out and reduce [cognitive load](https://medium.com/@egonelbre/psychology-of-code-readability-d23b1ff1258a) by refactoring. Read other people’s code and notice where _you_ stumble—why?

Use [consistent naming](https://go.dev/talks/2014/names.slide#1) to maximise [glanceability](https://jarosz.dev/code/do-not-overload-your-brain-go-function-tips/): `err` for errors, `data` for arbitrary `[]byte`s, `buf` for buffers, `file` for `*os.File` pointers, `path` for pathnames, `i` for index values, `req` for requests, `resp` for responses, `ctx` for contexts, and so on.

Good names make code read naturally. [Design the architecture, name the components, document the details](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=1099s). Simplify wordy functions by moving low-level “paperwork” into smaller functions with informative names (`createRequest`, `parseResponse`).

**Tip:** In GoLand, use the _Extract method_ refactoring to shorten long functions. Use _Rename_ to rename an identifier everywhere.

![](https://blog.jetbrains.com/wp-content/uploads/2025/10/03_write_code_for_reading.png)

## 4. Be safe by default

Use “always valid values” in your programs, and design types so that users can’t accidentally create values that won’t work. [Make the zero value useful](https://go-proverbs.github.io/) for literals, or write a validating constructor that guarantees a valid, usable object with default settings. Add configuration using _WithX_ methods:

widget := NewWidget().WithTimeout(time.Second)

Use named [constants](https://go.dev/blog/constants) instead of magic values. [`http.StatusOK`](https://pkg.go.dev/net/http#pkg-constants) is self-explanatory; `200` isn’t. Define your own constants so IDEs like GoLand can auto-complete them, preventing typos. Use `[iota](https://bitfieldconsulting.com/posts/iota)` to auto-assign arbitrary values:

const (
    Planet = iota // 0
    Star          // 1
    Comet         // 2
    // ...
)

Prevent security holes by using [`os.Root`](https://pkg.go.dev/os#Root) instead of `os.Open`, eliminating [path traversal attacks](https://go.dev/blog/osroot):

root, err := os.OpenRoot("/var/www/assets")
if err != nil {
    return err
}
defer root.Close()
file, err := root.Open("../../../etc/passwd")
// Error: 'openat ../../../etc/passwd: path escapes from parent'

Don’t require your program to run as `root` or in [setuid](https://en.wikipedia.org/wiki/Setuid) mode; let users configure the minimal permissions and capabilities they need.

**Tip:** Use Goland’s _Generate constructor_ and _Generate getter and setter_ functions to help you create always valid struct types.

![](https://blog.jetbrains.com/wp-content/uploads/2025/10/04_be_safe_by_default.png)

## 5. Wrap errors, don’t flatten

Don’t type-assert errors or [compare error values directly](https://bitfieldconsulting.com/posts/comparing-errors) with `==`, define named “sentinel” values that users can match errors against:

var ErrOutOfCheese = errors.New("++?????++ Out of Cheese Error. Redo From Start.")

Don’t inspect the string values of errors to find out what they are; this is fragile. Instead, use [errors.Is](https://pkg.go.dev/errors#Is):

if errors.Is(err, ErrOutOfCheese) {

To add run-time information or context to an error, don’t flatten it into a string. Use the `%w` verb with `fmt.Errorf` to create a [_wrapped_](https://bitfieldconsulting.com/posts/wrapping-errors) error:

return fmt.Errorf("GNU Terry Pratchett: %w", ErrOutOfCheese)

This way, `errors.Is` can still match the wrapped error against your sentinel value, even though it contains extra information.

**Tip**: GoLand will warn you against comparing or type-asserting error values.

![](https://blog.jetbrains.com/wp-content/uploads/2025/10/05_handle_errors_dont_flatten.png)

## 6. Avoid mutable global state

Package-level variables can cause [data races](https://go.dev/doc/articles/race_detector): reading a variable from one goroutine while writing it from another can crash your program. Instead, use a `sync.Mutex` to prevent concurrent access, or allow access to the data only in a single “guard” goroutine that takes read or write requests via a channel.

Don’t use global objects like [`http.DefaultServeMux`](https://pkg.go.dev/net/http#DefaultServeMux) or [`DefaultClient`](https://pkg.go.dev/net/http#DefaultClient);  packages you import might invisibly change these objects, maliciously or otherwise. Instead, create a new instance with [`http.NewServeMux`](https://pkg.go.dev/net/http#NewServeMux) (for example) and configure it how you want.

**Tip:** Use GoLand’s _Run/Debug Configurations_ settings to enable the Go race detector for testing concurrent code.

![](https://blog.jetbrains.com/wp-content/uploads/2025/10/06_avoid_mutable_global_state.png)

## 7. Use (structured) concurrency sparingly

Concurrent programming is a minefield: it’s easy to trigger crashes or race conditions. Don’t introduce concurrency to a program unless it’s unavoidable. When you do use [goroutines](https://bitfieldconsulting.com/posts/goroutines) and channels, keep them strictly confined: once they escape the scope where they’re created, it’s hard to follow the flow of control. “Global” goroutines, like global variables, can lead to hard-to-find bugs.

Make sure any goroutines you create will terminate before the enclosing function exits, using a [context](https://go.dev/blog/context) or [waitgroup](https://pkg.go.dev/sync#WaitGroup):

var wg sync.WaitGroup
wg.Go(task1)
wg.Go(task2)
wg.Wait()

The `Wait` call ensures that both tasks have completed before we move on, making control flow easy to understand, and preventing resource leaks.

Use [errgroups](https://pkg.go.dev/golang.org/x/sync/errgroup) to catch the first error from a number of parallel tasks, and [terminate](https://go.dev/play/p/2YOGE29kYsE) all the others:

var eg errgroup.Group
eg.Go(task1)
eg.Go(task2)
err := eg.Wait()
if err != nil {
	fmt.Printf("error %v: all other tasks cancelled", err)
} else {
	fmt.Println("all tasks completed successfully")
}

When you take a [channel](https://go.dev/ref/spec#ChannelType) as the parameter to a function, take either its send or receive aspect, but not both. This prevents a common kind of deadlock where the function tries to send and receive on the same channel concurrently.

func produce(ch chan<- Event) {
	// can send on `ch` but not receive
}

func consume(ch <-chan Event) {
	// can receive on `ch` but not send
}

**Tip:** Use GoLand’s profiler and debugger to analyse the behaviour of your goroutines, eliminate leaks, and solve deadlocks.

![](https://blog.jetbrains.com/wp-content/uploads/2025/10/07_use_structured_concurrency_sparingly.png)

## 8. Decouple code from environment

Don’t depend on OS or environment-specific details. Don’t use [`os.Getenv`](https://pkg.go.dev/os#Getenv) or [`os.Args`](https://pkg.go.dev/os#Args) deep in your package: only `main` should access environment variables or command-line arguments. Instead of taking choices away from users of your package, let them configure it however they want.

Single binaries are easier for users to install, update, and manage; don’t distribute config files. If necessary, create your config file at run time using defaults.

Use [`go:embed`](https://pkg.go.dev/embed) to bundle static data, such as images or certificates, into your binary:

import _ "embed"

//go:embed hello.txt
var s string

fmt.Println(s) // `s` now has the contents of 'hello.txt'

Use [`xdg`](https://pkg.go.dev/github.com/adrg/xdg) instead of hard-coding paths. Don’t assume `$HOME` exists. Don’t assume any disk storage exists, or is writable.

Go is popular in constrained environments, so be frugal with memory. Don’t read all your data at once; handle one chunk at a time, re-using the same buffer. This will keep your memory footprint small and reduce [garbage collection](https://go.dev/doc/gc-guide) cycles.

**Tip:** Use GoLand’s profiler to optimise your memory usage and eliminate leaks.

![](https://blog.jetbrains.com/wp-content/uploads/2025/10/08_decouple_code_from_environment.png)

## 9. Design for errors

Always check errors, and handle them if possible, retrying where appropriate. Report run-time errors to the user and exit gracefully, reserving `panic` for [internal program errors](https://www.alexedwards.net/blog/when-is-it-ok-to-panic-in-go). Don’t ignore errors using `_`: this leads to obscure bugs.

Show usage hints for incorrect arguments, don’t crash. Rather than prompting users interactively, let them customise behaviour with flags or config.

**Tip:** GoLand will warn you about unchecked or ignored errors, and offer to generate the handling code for you.

![](https://blog.jetbrains.com/wp-content/uploads/2025/10/09_design_for_errors.png)

## 10. Log only actionable information

[Logorrhea](https://en.wikipedia.org/wiki/Logorrhea_\(psychology\)) is irritating, so don’t spam the user with trivia. If you log at all, log only actionable errors that someone needs to fix. Don’t use fancy loggers, just print to the console, and let users redirect that output where they need it. Never log [secrets](https://pkg.go.dev/log/slog#example-LogValuer-Secret) or personal data.

Use [`slog`](https://pkg.go.dev/log/slog) to generate machine-readable JSON:

logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
logger.Error("oh no", "user", os.Getenv("USER"))
// Output:
// {"time":"...","level":"ERROR","msg":"oh no",
// "user":"bitfield"}

Logging is not for request-scoped troubleshooting: use [tracing](https://opentelemetry.io/docs/concepts/signals/traces/) instead. Don’t log performance data or statistics: that’s what [metrics](https://opentelemetry.io/docs/concepts/signals/metrics/) are for.

**Tip:** Instead of logging, use GoLand’s debugger with [non-suspending logging breakpoints](https://www.jetbrains.com/help/go/using-breakpoints.html) to gather troubleshooting information.

![](https://blog.jetbrains.com/wp-content/uploads/2025/10/10_log_only_actionable_information.gif)

## Guru meditation

My mountain-dwelling guru also says, “Make it work first, then make it right. Draft a quick [walking skeleton](https://wiki.c2.com/?WalkingSkeleton), using [shameless green](https://bitfieldconsulting.com/posts/tdd-shameless-green), and try it out on real users. Solve their problems first, and only then focus on code quality.”

Software takes more time to maintain than it does to write, so invest an extra 10% effort in refactoring, simplifying, and improving code while you still remember how it works. Making your programs better makes you a better programmer

#### Subscribe to GoLang Blog updates

![image description](https://blog.jetbrains.com/wp-content/themes/jetbrains/assets/img/img-form.svg)