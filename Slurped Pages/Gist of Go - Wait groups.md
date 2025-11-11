---
link: https://antonz.org/go-concurrency/wait-groups/
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Waiting for goroutines to finish.
twitter: https://twitter.com/@ohmypy
slurped: 2025-11-11T09:48
title: "Gist of Go: Wait groups"
---

_This is a chapter from my book on [Go concurrency](app://obsidian.md/go-concurrency), which teaches the topic from the ground up through interactive examples._

Channels are a multi-purpose concurrency tool in Go. In [Part 1](app://obsidian.md/go-concurrency/#contents) of the book, we covered their main use cases:

- Transferring data between goroutines.
- Synchronizing goroutines (the _done_ channel).
- Canceling goroutines (the _cancel_ channel).

Transferring data is what channels were designed for, and they excel at it. For canceling goroutines, there is a special tool besides channels — a _context_ (which we've also discussed). For synchronizing goroutines, there is also a special tool — a _wait group_. Let's talk about it.

- [Wait group](#wait-group)
- [Inner world](#inner-world)
- [Value vs. pointer](#value-vs-pointer)
- [Encapsulation](#encapsulation)

- [Add after Wait](#add-after-wait)
- [Multiple Waits](#multiple-waits)
- [Panic](#panic)
- [Keep it up](#keep-it-up)

## Wait group

A wait group lets you wait for one or more goroutines to finish. We started with a wait group in the very first chapter on goroutines, and now we'll go into more detail.

Suppose we want to start a goroutine and wait for it to complete. Here's how to do it with a done channel:

```
func main() {
    done := make(chan struct{}, 1)

    go func() {
        time.Sleep(50 * time.Millisecond)
        fmt.Print(".")
        done <- struct{}{}
    }()

    <-done
    fmt.Println("done")
}
```

And here's how to do it with a wait group:

```
func main() {
    var wg sync.WaitGroup

    wg.Add(1)
    go func() {
        time.Sleep(50 * time.Millisecond)
        fmt.Print(".")
        wg.Done()
    }()

    wg.Wait()
    fmt.Println("done")
}
```

Interestingly, a `WaitGroup` doesn't know anything about the goroutines it manages. It works with an internal counter. Calling `wg.Add(1)` increments the counter by one, while `wg.Done()` decrements it. `wg.Wait()` blocks the calling goroutine (in this case, `main`) until the counter reaches zero. So, `main()` waits for the called goroutine to finish before exiting.

The `WaitGroup.Go` method (Go 1.25+) automatically increments the wait group counter, runs a function in a goroutine, and decrements the counter when it's done. This means we can rewrite the example above without using `wg.Add()` and `wg.Done()`:

```
func main() {
    var wg sync.WaitGroup

    wg.Go(func() {
        time.Sleep(50 * time.Millisecond)
        fmt.Print(".")
    })

    wg.Wait()
    fmt.Println("done")
}
```

In short, you can wait for a goroutine to finish using these methods:

- Done channel.
- Wait group `Add`+`Done`+`Wait`.
- Wait group `Go`+`Wait`.

Typically, if you just need to wait for goroutines to complete without needing a result from them, you use a wait group instead of a done channel. The default choice should be the `Go` method, but in this chapter, I'll use `Add`+`Done` a lot because they do a better job of showing how things work internally.

**✎ Exercise: From channel to wait group**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Inner world

As we discussed, the wait group knows nothing about goroutines and works with a counter instead. This simplifies the implementation a lot. Conceptually, you can think of the wait group like this:

```
// A WaitGroup waits for a collection of goroutines to finish.
type WaitGroup struct {
    n int
}

// Add adds delta to the WaitGroup counter.
func (wg *WaitGroup) Add(delta int) {
    wg.n += delta
    if wg.n < 0 {
        panic("negative counter")
    }
}

// Done decrements the WaitGroup counter by one.
func (wg *WaitGroup) Done() {
    wg.Add(-1)
}

// Wait blocks until the WaitGroup counter is zero.
func (wg *WaitGroup) Wait() {
    for wg.n > 0 {}
}
```

```
func main() {
    var wg WaitGroup

    wg.Add(1)
    go func() {
        time.Sleep(50 * time.Millisecond)
        fmt.Print(".")
        wg.Done()
    }()

    wg.Wait()
    fmt.Println("done")
}
```

Of course, in practice it's more complicated :

- All methods can be called concurrently from multiple goroutines. Modifying the shared variable `n` from multiple goroutines is unsafe — concurrent access can corrupt data (we'll talk more about this in the chapter on data races).
- A loop-based `Wait` implementation will max out a CPU core until the loop finishes (this type of waiting is also known as _busy waiting_). Such code is strongly discouraged in production.

However, our naive implementation shows the properties of a wait group that are also present in the actual `sync.WaitGroup`:

- `Add` increments or decrements (if `delta < 0`) the counter. Positive deltas are much more common, but technically nothing prevents you from calling `Add(-1)`.
- `Wait` blocks execution until the counter reaches 0. So if you call `Wait` before the first `Add`, the goroutine won't block.
- After `Wait` completes, the wait group returns to its initial state (counter is 0). You can then reuse it.

As for the `Go` method, it's a simple wrapper that combines `Add` and `Done`. Here's the complete implementation taken directly from the standard library code:

```
// https://github.com/golang/go/blob/master/src/sync/waitgroup.go
func (wg *WaitGroup) Go(f func()) {
    wg.Add(1)
    go func() {
        defer wg.Done()
        f()
    }()
}
```

Try changing the [example above](#example-02) from `Add`+`Done` to `Go` and see if it works.

## Value vs. pointer

Another important implementation nuance: you should pass the wait group as a pointer (`*WaitGroup`), not as a value (`WaitGroup`). Otherwise, each recipient will get its own copy with a duplicate counter, and synchronization won't work.

Here's an example of passing a value:

```
func runWork(wg sync.WaitGroup) {
    wg.Add(1)
    go func() {
        time.Sleep(50 * time.Millisecond)
        fmt.Println("work done")
        wg.Done()
    }()
}

func main() {
    var wg sync.WaitGroup
    runWork(wg)
    wg.Wait()
    fmt.Println("all done")
}
```

`runWork` got a copy of the group and increased its counter with `Add`. Meanwhile, `main` has its own copy with a zero counter, so `Wait` didn't block execution. As a result, `main` finished without waiting for the `runWork` goroutine to complete.

Here's an example of passing a pointer:

```
func runWork(wg *sync.WaitGroup) {
    wg.Add(1)
    go func() {
        time.Sleep(50 * time.Millisecond)
        fmt.Println("work done")
        wg.Done()
    }()
}

func main() {
    var wg sync.WaitGroup
    runWork(&wg)
    wg.Wait()
    fmt.Println("all done")
}
```

Now `runWork` and `main` share the same instance of the group, so everything works as it should.

An even better approach would be not to pass the wait group around at all. Instead, we can encapsulate it in a separate type that hides the implementation details and provides a nice interface. Let's see how to do that.

## Encapsulation

In Go, it's considered a good practice to hide synchronization details from clients calling your code. Fellow developers won't thank you for forcing them to deal with wait groups. It's better to encapsulate the synchronization logic in a separate function or type, and provide a convenient interface.

### Wrapper functions

Let's say I wrote a function called `RunConc` that runs a set of given functions concurrently:

```
// RunConc executes functions concurrently.
func RunConc(wg *sync.WaitGroup, funcs ...func()) {
    wg.Add(len(funcs))
    for _, fn := range funcs {
        go func() {
            defer wg.Done()
            fn()
        }()
    }
}
```

And I suggest calling it this way:

```
func main() {
    work := func() {
        time.Sleep(50 * time.Millisecond)
        fmt.Print(".")
    }

    start := time.Now()

    var wg sync.WaitGroup
    RunConc(&wg, work, work, work)
    wg.Wait()

    elapsed := time.Now().Sub(start).Milliseconds()
    fmt.Printf("took %d ms\n", elapsed)
}
```

Is this convenient, given that the client just wants to run functions concurrently and wait for them to finish? Not really.

It's better to hide the wait group inside a function:

```
// RunConc executes functions concurrently and waits for them to finish.
func RunConc(funcs ...func()) {
    var wg sync.WaitGroup
    wg.Add(len(funcs))
    for _, fn := range funcs {
        go func() {
            defer wg.Done()
            fn()
        }()
    }
    wg.Wait()
}
```

Now you can call it like this:

```
func main() {
    work := func() {
        time.Sleep(50 * time.Millisecond)
        fmt.Print(".")
    }

    start := time.Now()
    RunConc(work, work, work)
    elapsed := time.Now().Sub(start).Milliseconds()
    fmt.Printf("took %d ms\n", elapsed)
}
```

The client doesn't need to know how `RunConc` does its job. It just works, and that's great.

### Wrapper types

Suppose other developers tried `RunConc` and didn't like it. They say they prefer to add functions one at a time and then run them all together later. They also want to run a set of functions multiple times.

Okay, I'll rewrite `RunConc` as a `ConcRunner` type:

```
// ConcRunner executes functions concurrently.
type ConcRunner struct {
    wg    sync.WaitGroup
    funcs []func()
}

// NewConcRunner creates a new ConcRunner instance.
func NewConcRunner() *ConcRunner {
    return &ConcRunner{}
}

// Add adds a function without executing it.
func (cg *ConcRunner) Add(fn func()) {
    cg.funcs = append(cg.funcs, fn)
}

// Run executes functions concurrently and waits for them to finish.
func (cg *ConcRunner) Run() {
    cg.wg.Add(len(cg.funcs))
    for _, fn := range cg.funcs {
        go func() {
            defer cg.wg.Done()
            fn()
        }()
    }
    cg.wg.Wait()
}
```

> You might ask: why is the `wg` field in `ConcRunner` defined as a `WaitGroup` value instead of a `*WaitGroup` pointer? It's because `ConcRunner` itself is used as a pointer: the constructor returns a `*ConcRunner`, and methods are defined on it. So the methods use the same `wg` value, avoiding counter issues.

The wait group is hidden in the type's fields, while the client still has a clean interface without the messy details:

```
func main() {
    cr := NewConcRunner()

    // add functions to the runner
    cr.Add(work)
    cr.Add(work)
    cr.Add(work)

    // run the functions concurrently
    timeit(cr)

    // and again
    timeit(cr)
}

// timeit executes the runner and measures
// how long it takes to finish.
func timeit(cg *ConcRunner) {
    start := time.Now()
    cg.Run()
    elapsed := time.Now().Sub(start).Milliseconds()
    fmt.Printf("took %d ms\n", elapsed)
}
```

```
...took 50 ms
...took 50 ms
```

In rare cases, a client may want to explicitly access your code's synchronization machinery. But usually it's better to encapsulate the synchronization logic.

**✎ Exercise: Concurrent group**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Add after Wait

Normally, all `Add` calls happen before `Wait`. But technically, there's nothing stopping us from doing some of the `Add` calls before `Wait` and some after (from another goroutine).

Let's say we have a function `runWork` that does its job in a separate goroutine:

```
// runWork performs work in a goroutine.
func runWork(wg *sync.WaitGroup) {
    wg.Add(1)
    fmt.Println("starting work...")
    go func() {
        time.Sleep(50 * time.Millisecond)
        fmt.Println("work done")
        wg.Done()
    }()
}
```

We'll do the following:

- Start a `runWork` goroutine (worker);
- Start another goroutine to wait for the work to finish (waiter);
- Start two more workers;
- When all three workers have finished, the waiter will wake up and signal completion to the `main` function.

```
func main() {
    // main wait group
    var wgMain sync.WaitGroup

    // worker wait group
    var wgWork sync.WaitGroup

    // run the first worker
    runWork(&wgWork)

    // the waiter goroutine waits for all workers to finish,
    // and then completes the main wait group
    wgMain.Add(1)
    go func() {
        fmt.Println("waiting for work to be done...")
        wgWork.Wait()
        fmt.Println("all work done")
        wgMain.Done()
    }()

    // run two more workers after a while
    time.Sleep(10 * time.Millisecond)
    runWork(&wgWork)
    runWork(&wgWork)

    // executes when the waiter goroutine finishes
    wgMain.Wait()
}
```

```
starting work...
waiting for work to be done...
starting work...
starting work...
work done
work done
work done
all work done
```

This is rarely used in practice.

## Multiple Waits

Another not-so-popular `WaitGroup` feature: you can call `Wait` from multiple goroutines. They will all block until the group's counter reaches zero.

For example, we can start one worker and three waiters:

```
func main() {
    var wg sync.WaitGroup

    // worker
    wg.Add(1)
    go func() {
        // do stuff
        time.Sleep(50 * time.Millisecond)
        fmt.Println("work done")
        wg.Done()
    }()

    // first waiter
    go func() {
        wg.Wait()
        fmt.Println("waiter 1 done")
    }()

    // second waiter
    go func() {
        wg.Wait()
        fmt.Println("waiter 2 done")
    }()

    // main waiter
    wg.Wait()
    fmt.Println("main waiter done")
}
```

```
work done
waiter 1 done
waiter 2 done
main waiter done
```

All waiters unblock after the worker calls `wg.Done()`. But the order in which this happens is not guaranteed. Could be this:

```
work done
waiter 1 done
waiter 2 done
main waiter done
```

Or this:

```
work done
waiter 1 done
main waiter done
waiter 2 done
```

Or even this:

```
work done
main waiter done
```

In the last case, the main waiter finished first, and then `main` exited before the other waiters could even print anything.

We'll see another use case for multiple `Wait`s in the chapter on semaphores.

**✎ Exercise: Waiting for worker**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Panic

If multiple goroutines are involved in the wait group, there are multiple possible panic sources.

Let's say there's a `work` function that panics on even numbers:

```
func work() {
    if n := rand.IntN(9) + 1; n%2 == 0 {
        panic(fmt.Errorf("bad number: %d", n))
    }
    // do stuff
}
```

We start four `work` goroutines:

```
func main() {
    var wg sync.WaitGroup

    for range 4 {
        wg.Go(work)
    }

    wg.Wait()
    fmt.Println("work done")
}
```

```
panic: bad number: 8

goroutine 9 [running]:
main.work()
	/sandbox/src/main.go:19 +0x76
sync.(*WaitGroup).Go.func1()
	/usr/local/go/src/sync/waitgroup.go:239 +0x4a
created by sync.(*WaitGroup).Go in goroutine 1
	/usr/local/go/src/sync/waitgroup.go:237 +0x73 (exit status 2)
```

And we face a panic (unless we are very lucky).

### Shared recover

Let's add `recover` to catch the panic and run the program again:

```
func main() {
    defer func() {
        val := recover()
        if val == nil {
            fmt.Println("work done")
        } else {
            fmt.Println("panicked!")
        }
    }()

    var wg sync.WaitGroup
    for range 4 {
        wg.Go(work)
    }
    wg.Wait()
}
```

```
panic: bad number: 6

goroutine 10 [running]:
main.work()
	/sandbox/src/main.go:19 +0x76
sync.(*WaitGroup).Go.func1()
	/usr/local/go/src/sync/waitgroup.go:239 +0x4a
created by sync.(*WaitGroup).Go in goroutine 1
	/usr/local/go/src/sync/waitgroup.go:237 +0x73 (exit status 2)
```

Nope. You might expect `recover` to catch the panic and print "panicked". But instead we get the same unhandled panic as before.

The problem is that `recover` has an important limitation: it only works within the _same goroutine_ that caused the panic. In our case, the panic comes from the `work` goroutines, while `recover` runs in the `main` goroutine — so it doesn't catch the panic. Goroutines are completely independent, remember? You can only catch the panic happening in those goroutines themselves.

### Per-goroutine recover

Let's move `recover` inside the `work` goroutines:

```
func main() {
    var wg sync.WaitGroup
    panicked := false

    catchPanic := func() {
        err := recover()
        if err != nil {
            panicked = true
        }
    }

    for range 4 {
        wg.Go(func() {
            defer catchPanic()
            work()
        })
    }

    wg.Wait()
    if !panicked {
        fmt.Println("work done")
    } else {
        fmt.Println("panicked!")
    }
}
```

Now, the panic is caught in its own goroutine, which then sets the `panicked` flag in the `main` goroutine. Now the program works fine and prints "panicked" as we expected.

> Here we are modifying the shared `panicked` variable from multiple goroutines. In general, this is not a good practice because it leads to data races (we'll talk about them in the next chapter). But in this particular case, there's no real harm from races.

Key takeaway: you cannot catch a panic from "child" goroutines in the "parent" goroutine. If you want to catch a panic, do it in the goroutine where it happens.

**✎ Exercise: Concurrent group with panic handling**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Keep it up

The wait group is used to wait for goroutines to finish. Now you understand how it works and how to apply it. In the next chapter, we'll talk about [data races](app://obsidian.md/data-races/).

[Pre-order for $10](https://antonz.gumroad.com/l/go-concurrency)   or [read online](app://obsidian.md/go-concurrency/)

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.