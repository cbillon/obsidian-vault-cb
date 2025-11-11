---
link: https://antonz.org/go-concurrency/atomics/
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Concurrent-safe operations without explicit synchronization.
twitter: https://twitter.com/@ohmypy
slurped: 2025-11-11T09:58
title: "Gist of Go: Atomics"
---

_This is a chapter from my book on [Go concurrency](app://obsidian.md/go-concurrency), which teaches the topic from the ground up through interactive examples._

Some concurrent operations don't require explicit synchronization. We can use these to create lock-free types and functions that are safe to use from multiple goroutines. Let's dive into the topic!

[Non-atomic increment](#non-atomic-increment) • [Atomic operations](#atomic-operations) • [Composition](#atomics-composition) • [Atomic vs. mutex](#atomic-instead-of-mutex) • [Keep it up](#keep-it-up)

## Non-atomic increment

Suppose multiple goroutines increment a shared counter:

```
total := 0

var wg sync.WaitGroup
for range 5 {
    wg.Go(func() {
        for range 10000 {
            total++
        }
    })
}
wg.Wait()

fmt.Println("total", total)
```

There are 5 goroutines, and each one increments `total` 10,000 times, so the final result should be 50,000. But it's usually less. Let's run the code a few more times:

```
total 26775
total 22978
total 30357
```

The race detector is reporting a problem:

```
$ go run -race total.go
==================
WARNING: DATA RACE
...
==================
total 33274
Found 1 data race(s)
```

This might seem strange — shouldn't the `total++` operation be atomic? Actually, it's not. It involves three steps (read-modify-write):

1. Read the current value of `total`.
2. Add one to it.
3. Write the new value back to `total`.

If two goroutines both read the value `42`, then each increments it and writes it back, the new `total` will be `43` instead of `44` like it should be. As a result, some increments to the counter will be lost, and the final value will be less than 50,000.

As we talked about in the [Race conditions](app://obsidian.md/go-concurrency/race-conditions/#race-condition) chapter, you can make an operation atomic by using mutexes or other synchronization tools. But for this chapter, let's agree not to use them. Here, when I say "atomic operation", I mean an operation that doesn't require the caller to use explicit locks, but is still safe to use in a concurrent environment.

## Atomic operations

An operation without synchronization can only be truly atomic if it translates to a single processor instruction. Such operations don't need locks and won't cause issues when called concurrently (even the write operations).

In a perfect world, every operation would be atomic, and we wouldn't have to deal with mutexes. But in reality, there are only a few atomics, and they're all found in the `sync/atomic` package. This package provides a set of atomic types:

- `Bool` — a boolean value;
- `Int32`/`Int64` — a 4- or 8-byte integer;
- `Uint32`/`Uint64` — a 4- or 8-byte unsigned integer;
- `Value` — a value of `any` type;
- `Pointer` — a pointer to a value of type `T` (generic).

Each atomic type provides the following methods:

`Load` reads the value of a variable, `Store` sets a new value:

```
var n atomic.Int32
n.Store(10)
fmt.Println("Store", n.Load())
```

`Swap` sets a new value (like `Store`) and returns the old one:

```
var n atomic.Int32
n.Store(10)
old := n.Swap(42)
fmt.Println("Swap", old, "->", n.Load())
```

`CompareAndSwap` sets a new value only if the current value is still what you expect it to be:

```
var n atomic.Int32
n.Store(10)
swapped := n.CompareAndSwap(10, 42)
fmt.Println("CompareAndSwap 10 -> 42:", swapped)
fmt.Println("n =", n.Load())
```

```
CompareAndSwap 10 -> 42: true
n = 42
```

```
var n atomic.Int32
n.Store(10)
swapped := n.CompareAndSwap(33, 42)
fmt.Println("CompareAndSwap 33 -> 42:", swapped)
fmt.Println("n =", n.Load())
```

```
CompareAndSwap 33 -> 42: false
n = 10
```

Numeric types also provide an `Add` method that increments the value by the specified amount:

```
var n atomic.Int32
n.Store(10)
n.Add(32)
fmt.Println("Add 32:", n.Load())
```

And the `And`/`Or` methods for bitwise operations (Go 1.23+):

```
const (
    modeRead  = 0b100
    modeWrite = 0b010
    modeExec  = 0b001
)

var mode atomic.Int32
mode.Store(modeRead)
old := mode.Or(modeWrite)

fmt.Printf("mode: %b -> %b\n", old, mode.Load())
```

All methods are translated to a single CPU instruction, so they are safe for concurrent calls.

> Strictly speaking, this isn't always true. Not all processors support the full set of concurrent operations, so sometimes more than one instruction is needed. But we don't have to worry about that — Go guarantees the atomicity of `sync/atomic` operations for the caller. It uses low-level mechanisms specific to each processor architecture to do this.

Like other synchronization primitives, each atomic variable has its own internal state. So, you should only pass it as a pointer, not by value, to avoid accidentally copying the state.

When using `atomic.Value`, all loads and stores should use the same concrete type. The following code will cause a panic:

```
var v atomic.Value
v.Store(10)
v.Store("hi")
```

```
panic: sync/atomic: store of inconsistently typed value into Value
```

Now, let's go back to the counter program:

```
total := 0

var wg sync.WaitGroup
for range 5 {
    wg.Go(func() {
        for range 10000 {
            total++
        }
    })
}
wg.Wait()

fmt.Println("total", total)
```

And rewrite it to use an atomic counter:

```
var total atomic.Int32

var wg sync.WaitGroup
for range 5 {
    wg.Go(func() {
        for range 10000 {
            total.Add(1)
        }
    })
}
wg.Wait()

fmt.Println("total", total.Load())
```

Much better!

**✎ Exercise: Atomic counter +1 more**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Atomics composition

An atomic operation in a concurrent program is a great thing. Such operation usually transforms into a single processor instruction, and it does not require locks. You can safely call it from different goroutines and receive a predictable result.

But what happens if you combine atomic operations? Let's find out.

### Atomicity

Let's look at a function that increments a counter:

```
var counter int32

// increment increases the counter value by two.
func increment() {
    counter += 1
    sleep(10)
    counter += 1
}

// sleep pauses the current goroutine for up to maxMs ms.
func sleep(maxMs int) {
    dur := time.Duration(rand.IntN(maxMs)) * time.Millisecond
    time.Sleep(dur)
}
```

As you already know, `increment` isn't safe to call from multiple goroutines because `counter += 1` causes a data race.

Now I will try to fix the problem and propose several options. In each case, answer the question: if you call `increment` from 100 goroutines, is the final value of the `counter` guaranteed?

Example 1:

```
var counter atomic.Int32

func increment() {
    counter.Add(1)
    sleep(10)
    counter.Add(1)
}
```

Is the `counter` value guaranteed?

Answer

It is guaranteed.

Example 2:

```
var counter atomic.Int32

func increment() {
    if counter.Load()%2 == 0 {
        sleep(10)
        counter.Add(1)
    } else {
        sleep(10)
        counter.Add(2)
    }
}
```

Is the `counter` value guaranteed?

Answer

It's not guaranteed.

Example 3:

```
var delta atomic.Int32
var counter atomic.Int32

func increment() {
    delta.Add(1)
    sleep(10)
    counter.Add(delta.Load())
}
```

Is the `counter` value guaranteed?

Answer

It's not guaranteed.

### Composition

People sometimes think that the composition of atomic operations also magically becomes an atomic operation. But it doesn't.

For example, the second of the above examples:

```
var counter atomic.Int32

func increment() {
    if counter.Load()%2 == 0 {
        sleep(10)
        counter.Add(1)
    } else {
        sleep(10)
        counter.Add(2)
    }
}
```

Call `increment` 100 times from different goroutines:

```
var wg sync.WaitGroup
for range 100 {
    wg.Go(increment)
}
wg.Wait()
fmt.Println("counter =", counter.Load())
```

Run the program with the `-race` flag — there are no races:

```
% go run atomic-2.go
192
% go run atomic-2.go
191
% go run atomic-2.go
189
```

But can we be sure what the final value of `counter` will be? Nope. `counter.Load` and `counter.Add` calls are interleaved from different goroutines. This causes a race condition (not to be confused with a data race) and leads to an unpredictable `counter` value.

Check yourself by answering the question: in which example is `increment` an atomic operation?

Answer

In none of them.

### Sequence independence

In all examples, `increment` is not an atomic operation. The composition of atomics is always non-atomic.

The first example, however, guarantees the final value of the `counter` in a concurrent environment:

```
var counter atomic.Int32

func increment() {
    counter.Add(1)
    sleep(10)
    counter.Add(1)
}
```

If we run 100 goroutines, the `counter` will ultimately equal 200.

The reason is that `Add` is a sequence-independent operation. The runtime can perform such operations in any order, and the result will not change.

The second and third examples use sequence-dependent operations. When we run 100 goroutines, the order of operations is different each time. Therefore, the result is also different.

A bulletproof way to make a composite operation atomic and prevent race conditions is to use a mutex:

```
var delta int32
var counter int32
var mu sync.Mutex

func increment() {
    mu.Lock()
    delta += 1
    sleep(10)
    counter += delta
    mu.Unlock()
}

// After 100 concurrent increments, the final value is guaranteed:
// counter = 1+2+...+100 = 5050
```

But sometimes an atomic variable with `CompareAndSwap` is all you need. Let's look at an example.

**✎ Exercise: Concurrent-safe stack**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Atomic instead of mutex

Let's say we have a gate that needs to be closed:

```
type Gate struct {
    closed bool // gate state
}

func (g *Gate) Close() {
    if g.closed {
        return // ignore repeated calls
    }
    g.closed = true
    // free resources
}
```

```
func work() {
	var g Gate
	defer g.Close()
	// do something while the gate is open
}
```

In a concurrent environment, there are data races on the `closed` field. We can fix this with a mutex:

```
type Gate struct {
    closed bool
    mu sync.Mutex // protects the state
}

func (g *Gate) Close() {
    g.mu.Lock()
    defer g.mu.Unlock()
    if g.closed {
        return // ignore repeated calls
    }
    g.closed = true
    // free resources
}
```

Alternatively, we can use `CompareAndSwap` on an atomic `Bool` instead of a mutex:

```
type Gate struct {
    closed atomic.Bool
}

func (g *Gate) Close() {
    if !g.closed.CompareAndSwap(false, true) {
        return // ignore repeated calls
    }
    // The gate is closed.
    // We can free resources now.
}
```

The `Gate` type is now more compact and simple.

This isn't a very common use case — we usually want a goroutine to wait on a locked mutex and continue once it's unlocked. But for "early exit" situations, it's perfect.

## Keep it up

Atomics are a specialized but useful tool. You can use them for simple counters and flags, but be very careful when using them for more complex operations. You can also use them instead of mutexes to exit early.

In the next chapter, we'll talk about testing concurrent code (coming soon).

[Pre-order for $10](https://antonz.gumroad.com/l/go-concurrency)   or [read online](app://obsidian.md/go-concurrency/)

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.