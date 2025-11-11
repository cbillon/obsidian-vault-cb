---
link: https://antonz.org/go-concurrency/context/
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Safely cancel and timeout operations in a concurrent environment.
twitter: https://twitter.com/@ohmypy
slurped: 2025-11-11T09:51
title: "Gist of Go: Context"
---

_This is a chapter from my book on [Go concurrency](app://obsidian.md/go-concurrency), which teaches the topic from the ground up through interactive examples._

In programming, _context_ refers to information about the environment in which an object exists or a function executes. In Go, context usually refers to the `Context` interface from the `context` package. It was originally designed to make working with HTTP requests easier. However, contexts can also be used in regular concurrent code. Let's see how exactly.

- [Canceling with channel](#canceling-with-channel)
- [Canceling with context](#canceling-with-context)
- [Timeout](#timeout)
- [Parent and child timeouts](#parent-and-child-timeouts)
- [Deadline](#deadline)

- [Cancellation cause](#cancellation-cause)
- [context.AfterFunc](#contextafterfunc)
- [Context with values](#context-with-values)
- [Keep it up](#keep-it-up)

## Canceling with channel

Suppose we have an `execute()` function, which can run a given function and supports cancellation:

```
// execute runs fn in a separate goroutine
// and waits for the result unless canceled.
func execute(cancel <-chan struct{}, fn func() int) (int, error) {
    ch := make(chan int, 1)

    go func() {
        ch <- fn()
    }()

    select {
    case res := <-ch:
        return res, nil
    case <-cancel:
        return 0, errors.New("canceled")
    }
}
```

Everything is familiar here:

- The function takes a channel through which it can receive a cancellation signal.
- It runs `fn()` in a separate goroutine.
- It uses select to wait for `fn()` to complete or cancel, whichever occurs first..

Let's write a client that cancels operations with a 50% probability:

```
// work does something for 100 ms.
func work() int {
    time.Sleep(100 * time.Millisecond)
    fmt.Println("work done")
    return 42
}

// maybeCancel waits for 50 ms and cancels with 50% probability.
func maybeCancel(cancel chan struct{}) {
    time.Sleep(50 * time.Millisecond)
    if rand.Float32() < 0.5 {
        close(cancel)
    }
}

func main() {
    cancel := make(chan struct{})
    go maybeCancel(cancel)

    res, err := execute(cancel, work)
    fmt.Println(res, err)
}
```

Run it a few times:

No surprises here.

Now let's reimplement `execute` with context.

## Canceling with context

The main purpose of context in Go is to cancel operations.

Let's reimplement what we just did with a cancel channel – this time with a context. The `execute()` function accepts a context `ctx` instead of a `cancel` channel:

```
// execute runs fn in a separate goroutine
// and waits for the result unless canceled.
func execute(ctx context.Context, fn func() int) (int, error) {
    ch := make(chan int, 1)

    go func() {
        ch <- fn()
    }()

    select {
    case res := <-ch:
        return res, nil
    case <-ctx.Done():       // (1)
        return 0, ctx.Err()  // (2)
    }
}
```

The code has barely changed:

- ➊ Instead of the `cancel` channel, the cancellation signal comes from the `ctx.Done()` channel
- ➋ Instead of manually creating a "canceled" error, we return `ctx.Err()`

The client also changes slightly:

```
// maybeCancel waits for 50 ms and cancels with 50% probability.
func maybeCancel(cancel func()) {
    time.Sleep(50 * time.Millisecond)
    if rand.Float32() < 0.5 {
        cancel()
    }
}

func main() {
    ctx := context.Background()              // (1)
    ctx, cancel := context.WithCancel(ctx)   // (2)
    defer cancel()                           // (3)

    go maybeCancel(cancel)                   // (4)

    res, err := execute(ctx, work)           // (5)
    fmt.Println(res, err)
}
```

Here's what we do:

- ➊ Create an empty context with `context.Background()`.
- ➋ Create a manual cancel context based on the empty context with `context.WithCancel()`.
- ➌ Schedule a deferred cancel when `main()` exits.
- ➍ Cancel the context with a 50% probability.
- ➎ Pass the context to the `execute()` function.

`context.WithCancel()` returns the context itself and a `cancel` function to cancel it. Calling `cancel()` releases the resources occupied by the context and closes the `ctx.Done()` channel — we use this effect to interrupt `execute()`. If the context is canceled, `ctx.Err()` returns an error (in our case `context.Canceled`).

All in all, it works exactly the same as the previous version with the cancel channel:

A few nuances that were not present with the cancel channel:

**Context is layered**. A context object is immutable. To add new properties to a context, a new (child) context is created based on the old (parent) context. That's why we first created an empty context and then a cancel context based on it:

```
// parent context
ctx := context.Background()

// child context
ctx, cancel := context.WithCancel(ctx)
```

If the parent context is canceled, all child contexts are canceled as well (but not vice versa):

```
// parent context
parentCtx, parentCancel := context.WithCancel(context.Background())

// child context
childCtx, childCancel := context.WithCancel(parentCtx)

// parentCancel() cancels both parentCtx and childCtx.
// childCancel() cancels only childCtx.
```

**Multiple cancels are safe**. If you call `close()` on a channel twice, it will cause a panic. However, you can call `cancel()` on the context as many times as you want. The first cancel will work, and the rest will be ignored. This is convenient because you can schedule a deferred `cancel()` right after creating the context, and explicitly cancel the context if necessary (as we did in the `maybeCancel` function). This wouldn't be possible with a channel.

**✎ Exercise: Canceling the generator**

Practice is essential for turning knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of interactive exercises with automated tests — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you're okay with just reading for now, let's continue.

## Timeout

The real power of context is its ability to handle both manual cancellation and _timeouts_.

```
// execute runs fn in a separate goroutine
// and waits for the result unless canceled.
func execute(ctx context.Context, fn func() int) (int, error) {
    // remains unchanged
}

// work does something for 100 ms.
func work() int {
    // remains unchanged
}
```

With a timeout of 150 ms, `work()` completes on time:

```
func main() {
    timeout := 150 * time.Millisecond
    ctx, cancel := context.WithTimeout(context.Background(), timeout)  // (1)
    defer cancel()

    res, err := execute(ctx, work)
    fmt.Println(res, err)
}
```

With a timeout of 50 ms, execution gets canceled:

```
func main() {
    timeout := 50 * time.Millisecond
    ctx, cancel := context.WithTimeout(context.Background(), timeout)  // (1)
    defer cancel()

    res, err := execute(ctx, work)
    fmt.Println(res, err)
}
```

```
0 context deadline exceeded
```

The `execute()` function remains unchanged, but `context.WithCancel()` in main is now replaced with `context.WithTimeout()` ➊. This change causes `execute()` to fail with a timeout error (`context.DeadlineExceeded`) when `work()` doesn't finish on time.

Thanks to the context, the `execute()` function doesn't need to know whether the cancellation was triggered manually or by a timeout. All it needs to do is listen for the cancellation signal on the `ctx.Done()` channel.

Convenient!

## Parent and child timeouts

Let's say we have the same `execute()` function and two functions it can run — the faster `work()` and the slower `slow()`:

```
// execute runs fn in a separate goroutine
// and waits for the result unless canceled.
func execute(ctx context.Context, fn func() int) (int, error) {
    ch := make(chan int, 1)

    go func() {
        ch <- fn()
    }()

    select {
    case res := <-ch:
        return res, nil
    case <-ctx.Done():
        return 0, ctx.Err()
    }
}

// work does something for 100 ms.
func work() int {
    time.Sleep(100 * time.Millisecond)
    return 42
}

// slow does something for 300 ms.
func slow() int {
    time.Sleep(300 * time.Millisecond)
    return 13
}
```

Suppose the timeout is 200 ms. Then `work()` with will have enough time to complete:

```
func main() {
    const dur200ms = 200 * time.Millisecond
    ctx, cancel := context.WithTimeout(context.Background(), dur200ms)
    defer cancel()

    // completes in time
    res, err := execute(ctx, work)
    fmt.Println(res, err)
}
```

But `slow()` won't make it:

```
func main() {
    const dur200ms = 200 * time.Millisecond
    ctx, cancel := context.WithTimeout(context.Background(), dur200ms)
    defer cancel()

    // gets canceled
    res, err := execute(ctx, slow)
    fmt.Println(res, err)
}
```

```
0 context deadline exceeded
```

We can create a child context to set a stricter timeout. This will override the parent context:

```
func main() {
    // parent context with a 200 ms timeout
    const dur200ms = 200 * time.Millisecond
    parentCtx, cancel := context.WithTimeout(context.Background(), dur200ms)
    defer cancel()

    // child context with a 50 ms timeout
    const dur50ms = 50 * time.Millisecond
    childCtx, cancel := context.WithTimeout(parentCtx, dur50ms)
    defer cancel()

    // now the work gets canceled
    res, err := execute(childCtx, work)
    fmt.Println(res, err)
}
```

```
0 context deadline exceeded
```

Creating a child context with a softer restriction is pointless. The parent context's timeout will trigger first:

```
func main() {
    // parent context with a 200 ms timeout
    const dur200ms = 200 * time.Millisecond
    parentCtx, cancel := context.WithTimeout(context.Background(), dur200ms)
    defer cancel()

    // child context with a 500 ms timeout
    const dur500ms = 500 * time.Millisecond
    childCtx, cancel := context.WithTimeout(parentCtx, dur500ms)
    defer cancel()

    // slow still gets canceled
    res, err := execute(childCtx, slow)
    fmt.Println(res, err)
}
```

```
0 context deadline exceeded
```

To summarize:

- The shorter timeout between the parent and child contexts always wins.
- The child context can only shorten the parent's timeout, not extend it.

## Deadline

A context also supports a _deadline_, which cancels an operation at a specific time instead of after a set duration.

```
// execute runs fn in a separate goroutine
// and waits for the result unless canceled.
func execute(ctx context.Context, fn func() int) (int, error) {
    // remains unchanged
}

// work does something for 100 ms.
func work() int {
    // remains unchanged
}
```

With a deadline of +150 ms from now, `work()` completes on time:

```
func main() {
    deadline := time.Now().Add(150 * time.Millisecond)
    ctx, cancel := context.WithDeadline(context.Background(), deadline)  // (1)
    defer cancel()

    res, err := execute(ctx, work)
    fmt.Println(res, err)
}
```

With a deadline of +50 ms from now, execution gets canceled:

```
func main() {
    deadline := time.Now().Add(50 * time.Millisecond)
    ctx, cancel := context.WithDeadline(context.Background(), deadline)  // (1)
    defer cancel()

    res, err := execute(ctx, work)
    fmt.Println(res, err)
}
```

```
0 context deadline exceeded
```

`context.WithDeadline()` ➊ behaves just like `context.WithTimeout()`, but it takes a `time.Time` value instead of `time.Duration`.

Moreover, `WithTimeout()` is simply a wrapper around `WithDeadline()`:

```
// A snippet of standard library code.
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
    return WithDeadline(parent, time.Now().Add(timeout))
}
```

You can access the deadline within the context through the `Deadline()` method:

```
now := time.Now()
fmt.Println(now)
// 2009-11-10 23:00:00

ctx, _ := context.WithTimeout(context.Background(), 5*time.Second)
deadline, ok := ctx.Deadline()
fmt.Println(deadline, ok)
// 2009-11-10 23:00:05 true

plus5s := now.Add(5 * time.Second)
ctx, _ = context.WithDeadline(context.Background(), plus5s)
deadline, ok = ctx.Deadline()
fmt.Println(deadline, ok)
// 2009-11-10 23:00:05 true
```

```
2009-11-10 23:00:00
2009-11-10 23:00:05 true
2009-11-10 23:00:05 true
```

The second output of `Deadline()` indicates whether a deadline is set:

- For contexts created with `WithTimeout()` and `WithDeadline()`, it is `true`.
- For contexts created with `WithCancel()` and `Background()`, it is `false`.

```
ctx, _ := context.WithCancel(context.Background())
deadline, ok := ctx.Deadline()
fmt.Println(deadline, ok)
// 0001-01-01 00:00:00 false

ctx = context.Background()
deadline, ok = ctx.Deadline()
fmt.Println(deadline, ok)
// 0001-01-01 00:00:00 false
```

```
0001-01-01 00:00:00 false
0001-01-01 00:00:00 false
```

**✎ Exercise: Pipeline with context**

Practice is essential for turning knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of interactive exercises with automated tests — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you're okay with just reading for now, let's continue.

## Cancellation cause

As mentioned at the beginning of the chapter, a `context.Canceled` error occurs when the context is canceled:

```
ctx, cancel := context.WithCancel(context.Background())
cancel()

fmt.Println(errors.Is(ctx.Err(), context.Canceled))
// true
fmt.Println(ctx.Err())
// context canceled
```

The error indicates that the context was canceled, but it doesn't specify the exact reason.

In Go 1.20+, you can create a context using `context.WithCancelCause()`. The `cancel()` function will then accept a single parameter: the root _cause_ of the cancellation.

```
ctx, cancel := context.WithCancelCause(context.Background())
cancel(errors.New("the night is dark"))
```

Use `context.Cause()` to get the error's cause:

```
fmt.Println(ctx.Err())
// context canceled

fmt.Println(context.Cause(ctx))
// the night is dark
```

```
context canceled
the night is dark
```

In Go 1.21+, you can specify a custom cause for timeout (or deadline) cancellation when creating a context. This cause is accessible through `context.Cause()` when the context is canceled due to a timeout (or deadline):

```
cause := errors.New("the night is dark")
ctx, cancel := context.WithTimeoutCause(
    context.Background(), 10*time.Millisecond, cause,
)
defer cancel()

time.Sleep(50 * time.Millisecond)
fmt.Println(ctx.Err())
// context deadline exceeded
fmt.Println(context.Cause(ctx))
// the night is dark
```

```
context deadline exceeded
the night is dark
```

## context.AfterFunc

Suppose we're performing a long-running task with the option to cancel:

```
// work does something for 100 ms.
func work(ctx context.Context) {
    select {
    case <-time.After(100 * time.Millisecond):
    case <-ctx.Done():
    }
}
```

Let's set the timeout to 50 ms (expecting `work()` to be canceled):

```
// the context is canceled after a 50 ms timeout
ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
defer cancel()

start := time.Now()
work(ctx)

if ctx.Err() != nil {
    fmt.Println("canceled after", time.Since(start))
}
```

What should we do if `work()` has occupied resources that need to be freed upon cancellation?

```
// cleanup frees up occupied resources.
func cleanup() {
    fmt.Println("cleanup")
}
```

We can add the `cleanup()` call directly into `work()`:

```
func work(ctx context.Context) {
    select {
    case <-time.After(100 * time.Millisecond):
    case <-ctx.Done():
        cleanup()
    }
}
```

But Go 1.21+ offers a more flexible solution.

### Calling a function on context cancellation

You can register a function to execute when the context is canceled:

```
// the context is canceled after a 50 ms timeout
ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
defer cancel()

// cleanup is called after the context is canceled
context.AfterFunc(ctx, cleanup)

start := time.Now()
work(ctx)

if ctx.Err() != nil {
    fmt.Println("canceled after", time.Since(start))
}
```

```
cleanup
canceled after 50ms
```

In this version, `work()` doesn't need to know about `cleanup()`.

`context.AfterFunc()` offers the following:

- `cleanup()` runs in a separate goroutine.
- You can register multiple functions by calling `AfterFunc()` multiple times. Each function runs independently in a separate goroutine when the context is canceled.
- You can change your mind and "detach" a registered function.

### Registering a function after the context is canceled

If the context is already canceled when the function is registered, the function executes immediately:

```
// the context is canceled after a 50 ms timeout
ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
defer cancel()

start := time.Now()
work(ctx)

if ctx.Err() != nil {
    fmt.Println("canceled after", time.Since(start))
}

// cleanup is called immediately since the context is already canceled
context.AfterFunc(ctx, cleanup)
```

```
canceled after 50ms
cleanup
```

### Canceling the registration

Example of "changing one's mind":

```
// the context is canceled after a 50 ms timeout
ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
defer cancel()

// cleanup is called after the context is canceled
stopCleanup := context.AfterFunc(ctx, cleanup)  // (1)

// I changed my mind, let's not call cleanup
stopped := stopCleanup()                        // (2)
work(ctx)

fmt.Println("stopped cleanup:", stopped)
```

In ➊, we saved the "detach" function (returned by `context.AfterFunc()`) in the `stopCleanup` variable, and in ➋, we called it, detaching `cleanup()` from `ctx`. As a result, the context was canceled due to the timeout, but `cleanup()` did not execute.

If the context is canceled and the function has already started executing, you can't detach it:

```
// the context is canceled after a 50 ms timeout
ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
defer cancel()

// cleanup is called after the context is canceled
stopCleanup := context.AfterFunc(ctx, cleanup)

work(ctx)

// I changed my mind about calling cleanup, but it's already too late
stopped := stopCleanup()
fmt.Println("stopped cleanup:", stopped)
```

```
cleanup
stopped cleanup: false
```

Phew. `AfterFunc()` is not the most intuitive context-related feature.

**✎ Exercise: Cancelable worker**

Practice is essential for turning knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of interactive exercises with automated tests — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you're okay with just reading for now, let's continue.

## Context with values

The main purpose of context in Go is to cancel operations, either manually or by timeout/deadline. But it can also pass additional information about a call using `context.WithValue()`, which creates a context with a value for a specific key:

```
type contextKey string

// "id" and "user" keys
var idKey = contextKey("id")
var userKey = contextKey("user")

// work does something.
func work() int {
    return 42
}

func main() {
    {
        ctx := context.Background()
        // context with request ID
        ctx = context.WithValue(ctx, idKey, 1234)
        // and user
        ctx = context.WithValue(ctx, userKey, "admin")
        res := execute(ctx, work)
        fmt.Println(res)
    }

    {
        // empty context
        ctx := context.Background()
        res := execute(ctx, work)
        fmt.Println(res)
    }
}
```

We use values of a custom type `contextKey` as keys instead of strings or numbers. This prevents conflicts if two packages modify the same context and both add a value with the `id` or `user` key.

To retrieve a value by key, use the `Value()` method:

```
// execute runs fn with respect to ctx.
func execute(ctx context.Context, fn func() int) int {
    reqId := ctx.Value(idKey)
    if reqId != nil {
        fmt.Printf("Request ID = %d\n", reqId)
    } else {
        fmt.Println("Request ID unknown")
    }

    user := ctx.Value(userKey)
    if user != nil {
        fmt.Printf("Request user = %s\n", user)
    } else {
        fmt.Println("Request user unknown")
    }
    return fn()
}
```

```
Request ID = 1234
Request user = admin
42
Request ID unknown
Request user unknown
42
```

Both `context.WithValue()` and `Context.Value()` work with values of type `any` (they were added to the standard library long before generics):

```
func WithValue(parent Context, key, val any) Context

type Context interface {
    // ...
    Value(key any) any
}
```

I'm mentioning this feature for completeness, but it's generally better to avoid passing values in context. It's better to use explicit parameters or custom structs instead.

## Keep it up

Use context to safely cancel and timeout operations in a concurrent environment. It's a perfect fit for remote calls, pipelines, or other long-running operations.

Now you know how to:

- Manually cancel operations.
- Cancel on timeout or deadline.
- Use child contexts to restrict timeouts.
- Specify the cancellation reason.
- Register functions to execute when the context is canceled.

In the next chapter, we'll closely examine [wait groups](app://obsidian.md/wait-groups/).

[Pre-order for $10](https://antonz.gumroad.com/l/go-concurrency)   or [read online](app://obsidian.md/go-concurrency/)

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.