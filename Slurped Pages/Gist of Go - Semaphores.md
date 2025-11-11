---
link: https://antonz.org/go-concurrency/semaphores/
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Limiting the concurrency and waiting for the peers.
twitter: https://twitter.com/@ohmypy
slurped: 2025-11-11T09:57
title: "Gist of Go: Semaphores"
---

_This is a chapter from my book on [Go concurrency](app://obsidian.md/go-concurrency), which teaches the topic from the ground up through interactive examples._

Having the full power of multi-core hardware is great, but sometimes we prefer to limit concurrency in certain parts of a system. Semaphores are a great way to do this. Let's learn more about them!

[Mutex](#mutex-one-goroutine-at-a-time) • [Semaphore](#semaphore--n-goroutines-at-a-time) • [Implementing a semaphore](#-implementing-a-semaphore) ✎ • [Rendezvous](#rendezvous) • [Synchronization barrier](#synchronization-barrier) • [Keep it up](#keep-it-up)

## Mutex: one goroutine at a time

Let's say our program needs to call a legacy system, represented by the `External` type. This system is so ancient that it can handle no more than one call at a time. That's why we protect it with a mutex:

```
// External is a adapter for an external system.
type External struct {
    lock sync.Mutex
}

// Call calls the external system.
func (e *External) Call() {
    e.lock.Lock()
    defer e.lock.Unlock()
    // Simulate a remote call.
    time.Sleep(10 * time.Millisecond)
}
```

Now, no matter how many goroutines try to access the external system at the same time, they'll have to take turns:

```
func main() {
    const nCalls = 12
    ex := new(External)
    start := time.Now()

    var wg sync.WaitGroup
    for range nCalls {
        wg.Go(func() {
            ex.Call()
            fmt.Print(".")
        })
    }
    wg.Wait()

    fmt.Printf(
        "\n%d calls took %d ms\n",
        nCalls, time.Since(start).Milliseconds(),
    )
}
```

```
............
12 calls took 120 ms
```

Suppose the developers of the legacy system made some changes and now they say we can make up to four simultaneous calls. In this case, our approach with the mutex stops working, because it blocks all goroutines except the one that managed to lock the mutex.

It would be great to use a tool that allows several goroutines to run at the same time, but no more than N. Luckily for us, such a tool already exists.

## Semaphore: ≤ N goroutines at a time

So, we want to make sure that no more than 4 goroutines access the external system at the same time. To do this, we'll use a _semaphore_. You can think of a semaphore as a container with N available slots and two operations: _acquire_ to take a slot and _release_ to free a slot.

Here are the semaphore rules:

- Calling `Acquire` takes a free slot.
- If there are no free slots, `Acquire` blocks the goroutine that called it.
- Calling `Release` frees up a previously taken slot.
- If there are any goroutines blocked on `Acquire` when `Release` is called, one of them will immediately take the freed slot and unblock.

Let's see how this works. To keep things simple, let's assume that someone has already implemented a `Semaphore` type for us, and we can just use it.

We add a semaphore to the external system adapter:

```
type External struct {
    sema Semaphore
}

func NewExternal(maxConc int) *External {
    // maxConc sets the maximum allowed
    // number of concurrent calls.
    return &External{NewSemaphore(maxConc)}
}
```

We acquire a spot in the semaphore before calling the external system. After the call, we release it:

```
func (e *External) Call() {
    e.sema.Acquire()
    defer e.sema.Release()
    // Simulate a remote call.
    time.Sleep(10 * time.Millisecond)
}
```

Now let's allow 4 concurrent calls and perform a total of 12 calls. The client code doesn't change:

```
func main() {
    const nCalls = 12
    const nConc = 4

    ex := NewExternal(nConc)
    start := time.Now()

    var wg sync.WaitGroup
    for range nCalls {
        wg.Go(func() {
            ex.Call()
            fmt.Print(".")
        })
    }
    wg.Wait()

    fmt.Printf(
        "\n%d calls took %d ms\n",
        nCalls, time.Since(start).Milliseconds(),
    )
}
```

```
............
12 calls took 30 ms
```

12 calls were completed in three steps (each step = 4 concurrent calls). Each step took 10 ms, so the total time was 30 ms.

You might have noticed a downside to this approach: even though only 4 goroutines (`nConc`) run concurrently, we actually start all 12 (`nCalls`) right away. With small numbers, this isn't a big deal, but if `nCalls` is large, the waiting goroutines will use up memory for no good reason.

We can modify the program so that there are never more than `nConc` goroutines at any given time. To do this, we add the `Acquire` and `Release` methods directly to `External` and remove them from `Call`:

```
type External struct {
    // Semaphore is embedded into External so clients
    // can call Acquire and Release directly on External.
    Semaphore
}

func NewExternal(maxConc int) *External {
    return &External{NewSemaphore(maxConc)}
}

func (e *External) Call() {
    // Simulate a remote call.
    time.Sleep(10 * time.Millisecond)
}
```

The client calls `Acquire` before starting each new goroutine in the loop, and calls `Release` when it's finished:

```
func main() {
    const nCalls = 12
    const nConc = 4

    ex := NewExternal(nConc)
    start := time.Now()

    var wg sync.WaitGroup
    for range nCalls {
        ex.Acquire()
        wg.Go(func() {
            defer ex.Release()
            ex.Call()
            fmt.Print(".")
        })
    }
    wg.Wait()

    fmt.Printf(
        "\n%d calls took %d ms\n",
        nCalls, time.Since(start).Milliseconds(),
    )
}
```

```
............
12 calls took 30 ms
```

Now there are never more than 4 goroutines at any time (not counting the main goroutine, of course).

In summary, the semaphore helped us solve the problem of limited concurrency:

- goroutines are allowed to run concurrently,
- but no more than N at the same time.

Unfortunately, the standard library doesn't include a `Semaphore` type. So in the next step, we'll implement it ourselves!

> There is a semaphore available in the `golang.org/x/sync/semaphore` package. But for simple cases like ours, it's perfectly fine to use your own implementation.

## ✎ Implementing a semaphore

_The ✎ symbol indicates exercises. They're usually only available in the paid version of the book, but this one is an exception._

Here's a `Semaphore` type that I'm not really proud of:

```
// A synchronization semaphore.
type Semaphore struct {
    n   int
    val int
}

// NewSemaphore creates a new semaphore with the given capacity.
func NewSemaphore(n int) *Semaphore {
    return &Semaphore{n: n}
}

// Acquire takes a spot in the semaphore if one is available.
// Otherwise, it blocks the calling goroutine.
func (s *Semaphore) Acquire() {
    if s.val < s.n {
        s.val += 1
        return
    }
    for s.val >= s.n {
    }
}

// Release frees up a spot in the semaphore and unblocks
// one of the blocked goroutines (if there are any).
func (s *Semaphore) Release() {
    s.val -= 1
}
```

There are a few problems with it:

- There's a data race when changing the `val` counter.
- The waiting in `Acquire` is done with an infinite loop (busy-waiting).
- The semaphore doesn't work 😁

Create your own `Semaphore` that doesn't have these issues.

Hint

Use a channel. The solution will be very straightforward.

Submit only the code fragment marked with "solution start" and "solution end" comments. The full source code is available via the "Playground" link below.

```
// solution start

// A synchronization semaphore.
type Semaphore

// NewSemaphore creates a new semaphore with the given capacity.
func NewSemaphore(n int) *Semaphore {
    // ...
}

// Acquire takes a spot in the semaphore if one is available.
// Otherwise, it blocks the calling goroutine.
func (s *Semaphore) Acquire() {
    // ...
}

// Release frees up a spot in the semaphore and unblocks
// one of the blocked goroutines (if there are any).
func (s *Semaphore) Release() {
    // ...
}

// solution end
```

**✎ Exercise: Implementing TryAcquire**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Rendezvous

Imagine you and your friend agreed to meet at a certain place before going to a cafe together. You arrive at the meeting spot, but your friend isn't there yet. As planned, you don't go anywhere; you just wait for them, and then you both proceed to drink tea (or whatever you planned) together. If your friend had arrived first, they would have waited for you the same way before heading to the cafe.

This kind of logic in concurrent programs is called a _rendezvous_:

- There are two goroutines — G1 and G2 — and each one can signal that it's ready.
- If G1 signals but G2 hasn't yet, G1 blocks and waits.
- If G2 signals but G1 hasn't yet, G2 blocks and waits.
- When both have signaled, they both unblock and continue running.

Let's say there are two goroutines, and each one performs two steps. After the first step, each goroutine wants to wait for the other. Here's what their execution looks like without a rendezvous:

```
var wg sync.WaitGroup

wg.Go(func() {
    fmt.Println("1: started")
    time.Sleep(10 * time.Millisecond)
    fmt.Println("1: reached the sync point")

    // Sync point: how do I wait for the second goroutine?

    fmt.Println("1: going further")
    time.Sleep(20 * time.Millisecond)
    fmt.Println("1: done")
})

time.Sleep(20 * time.Millisecond)

wg.Go(func() {
    fmt.Println("2: started")
    time.Sleep(20 * time.Millisecond)
    fmt.Println("2: reached the sync point")

    // Sync point: how do I wait for the first goroutine?

    fmt.Println("2: going further")
    time.Sleep(10 * time.Millisecond)
    fmt.Println("2: done")
})

wg.Wait()
```

```
1: started
1: reached the sync point
1: going further
2: started
1: done
2: reached the sync point
2: going further
2: done
```

As you can see, the second goroutine is just getting started, while the first one is already finished. No one is waiting for anyone else.

Let's set up a rendezvous for them:

```
var wg sync.WaitGroup

ready1 := make(chan struct{})
ready2 := make(chan struct{})

wg.Go(func() {
    fmt.Println("1: started")
    time.Sleep(10 * time.Millisecond)
    fmt.Println("1: reached the sync point")

    // Sync point.
    close(ready1)
    <-ready2

    fmt.Println("1: going further")
    time.Sleep(20 * time.Millisecond)
    fmt.Println("1: done")
})

time.Sleep(20 * time.Millisecond)

wg.Go(func() {
    fmt.Println("2: started")
    time.Sleep(20 * time.Millisecond)
    fmt.Println("2: reached the sync point")

    // Sync point.
    close(ready2)
    <-ready1

    fmt.Println("2: going further")
    time.Sleep(10 * time.Millisecond)
    fmt.Println("2: done")
})

wg.Wait()
```

```
1: started
1: reached the sync point
2: started
2: reached the sync point
2: going further
1: going further
2: done
1: done
```

Now everything works fine: the goroutines waited for each other at the sync point before moving on.

Here's how it works:

- G1 closes its own channel when it's ready and then blocks on the other channel. G2 does the same thing.
- When G1's channel is closed, it unblocks G2, and when G2's channel is closed, it unblocks G1.
- As a result, both goroutines are unblocked at the same time.

Here, we close the channel to signal an event. We've done this before:

- With a done channel in the [Channels](app://obsidian.md/channels/) chapter (the goroutine signals the caller that the work is finished).
- With a cancel channel in the [Pipelines](app://obsidian.md/pipelines/) chapter (the caller signals the goroutine to stop working).

**Caution: Using Print for debugging**

Since printing uses a single output device (stdout), goroutines that print concurrently have to synchronize access to it. So, using print statements adds a synchronization point to your program. This can cause unexpected results that are different from what actually happens in production (where there is no printing).

In the book, I use print statements only because it's much harder to understand the material without them.

**✎ Exercise: Implementing a rendezvous**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Synchronization barrier

Imagine you walk up to a crosswalk with a traffic light and see a button that's supposed to turn the light green. You press the button, but nothing happens. Another person comes up behind you and presses the button too, but still nothing changes. Two more people arrive, both press the button, but the light stays red. Now the four of you are just standing there, not sure what to do. Finally, a fifth person comes, presses the button, and the light turns green. All five of you cross the street together.

This kind of logic in concurrent programs is called a _synchronization barrier_:

- The barrier has a counter (starting at 0) and a threshold N.
- Each goroutine that reaches the barrier increases the counter by 1.
- The barrier blocks any goroutine that reaches it.
- Once the counter reaches N, the barrier unblocks all waiting goroutines.

Let's say there are N goroutines. Each one first does a preparation step, then the main step. Here's what their execution looks like without a barrier:

```
const nWorkers = 4
start := time.Now()

var wg sync.WaitGroup
for i := range nWorkers {
    wg.Go(func() {
        // Simulate the preparation step.
        dur := time.Duration((i+1)*10) * time.Millisecond
        time.Sleep(dur)
        fmt.Printf("ready to go after %d ms\n", dur.Milliseconds())

        // Simulate the main step.
        fmt.Println("go!")
    })
}

wg.Wait()
fmt.Printf("all done in %d ms\n", time.Since(start).Milliseconds())
```

```
ready to go after 10 ms
go!
ready to go after 20 ms
go!
ready to go after 30 ms
go!
ready to go after 40 ms
go!
all done in 40 ms
```

Each goroutine proceeds to the main step as soon as it's ready, without waiting for the others.

Let's say we want the goroutines to wait for each other before moving on to the main step. To do this, we just need to add a barrier after the preparation step:

```
const nWorkers = 4
start := time.Now()

var wg sync.WaitGroup
b := NewBarrier(nWorkers)
for i := range nWorkers {
    wg.Go(func() {
        // Simulate the preparation step.
        dur := time.Duration((i+1)*10) * time.Millisecond
        time.Sleep(dur)
        fmt.Printf("ready to go after %d ms\n", dur.Milliseconds())

        // Wait for all goroutines to reach the barrier.
        b.Touch()

        // Simulate the main step.
        fmt.Println("go!")
    })
}

wg.Wait()
fmt.Printf("all done in %d ms\n", time.Since(start).Milliseconds())
```

```
ready to go after 10 ms
ready to go after 20 ms
ready to go after 30 ms
ready to go after 40 ms
go!
go!
go!
go!
all done in 40 ms
```

Now the faster goroutines waited at the barrier for the slower ones, and only after that did they all move on to the main step together.

Here are some examples of when a synchronization barrier can be useful:

- _Parallel computing_. If you're sorting in parallel and then merging the results, the sorting steps must finish before the merging starts. If you merge too soon, you'll get the wrong results.
- _Multiplayer applications_. If a duel in the game involves N players, all resources for those players need to be fully prepared before the duel begins. Otherwise, some players might be at a disadvantage.
- _Distributed systems_. To create a backup, you need to wait until all nodes in the system reach a consistent state (a checkpoint). Otherwise, the backup's integrity could be compromised.

The standard library doesn't have a barrier, so now is a great time to make one yourself!

**✎ Exercise: Implementing a barrier**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Keep it up

You've learned the classic synchronization tools — mutexes, semaphores, rendezvous, and barriers. Be careful when using them. Try to avoid complicated setups, and always write tests for tricky concurrent situations.

In the next chapter, we'll talk about [signaling](app://obsidian.md/signaling/).

[Pre-order for $10](https://antonz.gumroad.com/l/go-concurrency)   or [read online](app://obsidian.md/go-concurrency/)

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.