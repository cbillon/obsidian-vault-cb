---
link: https://antonz.org/go-concurrency/signaling/
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Sending events between goroutines.
twitter: https://twitter.com/@ohmypy
slurped: 2025-11-11T09:57
title: "Gist of Go: Signaling"
---

_This is a chapter from my book on [Go concurrency](app://obsidian.md/go-concurrency), which teaches the topic from the ground up through interactive examples._

The main way goroutines communicate in Go is through channels. But channels aren't the only way for goroutines to signal each other. Let's try a different approach!

[Signaling](#signaling) • [One-time subscription](#one-time-subscription) • [Broadcasting](#broadcasting) • [Broadcasting w/channels](#broadcasting-with-channels) • [Publish/subscribe](#publishsubscribe) • [Run once](#run-once) • [Once-functions](#once-functions) • [Object pool](#object-pool) • [Keep it up](#keep-it-up)

## Signaling

Let's say we have a goroutine that generates a random number between 1 and 100:

```
num := 0
go func() {
    num = 1 + rand.IntN(100)
}()
```

And the second one checks if the number is lucky or not:

```
go func() {
    if num%7 == 0 {
        fmt.Printf("Lucky number %d!\n", num)
    } else {
        fmt.Printf("Unlucky number %d...\n", num)
    }
}()
```

The second goroutine will only work correctly if the first one has already set the number. So, we need to find a way to synchronize them. For example, we can make `num` a channel:

```
var wg sync.WaitGroup
num := make(chan int, 1)

// Generates a random number from 1 to 100.
wg.Go(func() {
    num <- 1 + rand.IntN(100)
})

// Checks if the number is lucky.
wg.Go(func() {
    n := <-num
    if n%7 == 0 {
        fmt.Printf("Lucky number %d!\n", n)
    } else {
        fmt.Printf("Unlucky number %d...\n", n)
    }
})

wg.Wait()
```

But what if we want `num` to be a regular number, and channels are not an option?

We can make the generator goroutine signal when a number is ready, and have the checker goroutine wait for that signal. In Go, we can do this using a _condition variable_, which is implemented with the `sync.Cond` type.

A `Cond` has a mutex inside it:

```
cond := sync.NewCond(&sync.Mutex{})
// The mutex is available through the cond.L field.
fmt.Printf("%#v\n", cond.L)
```

```
&sync.Mutex{state:0, sema:0x0}
```

A `Cond` has two methods — `Wait` and `Signal`:

- `Wait` unlocks the mutex and suspends the goroutine until it receives a signal.
- `Signal` wakes the goroutine that is waiting on `Wait`.
- When `Wait` wakes up, it locks the mutex again.

If there are multiple waiting goroutines when `Signal` is called, only one of them will be resumed. If there are no waiting goroutines, `Signal` does nothing.

To see why `Cond` needs to go through all this mutex trouble, check out this example:

```
cond := sync.NewCond(&sync.Mutex{})
num := 0

// Generates a random number from 1 to 100.
go func() {
    time.Sleep(10 * time.Millisecond)
    cond.L.Lock()    // (1)
    num = 1 + rand.IntN(100)
    cond.Signal()    // (2)
    cond.L.Unlock()
}()

// Checks if the number is lucky.
go func() {
    cond.L.Lock()    // (3)
    if num == 0 {
        cond.Wait()  // (4)
    }
    if num%7 == 0 {
        fmt.Printf("Lucky number %d!\n", num)
    } else {
        fmt.Printf("Unlucky number %d...\n", num)
    }
    cond.L.Unlock()
}()
```

Both goroutines use the shared `num` variable, so we need to protect it with a mutex.

The checker goroutine starts by locking the `cond.L` mutex ➌. If the generator hasn't run yet (meaning `num` is 0), the goroutine calls `cond.Wait()` ➍ and blocks. If `Wait` only blocked the goroutine, the mutex would stay locked, and the generator couldn't change `num` ➊. That's why `Wait` unlocks the mutex before blocking.

The generator goroutine also starts by locking the mutex ➊. After setting the `num` value, the generator calls `cond.Signal()` ➋ to let the checker know it's ready, and then unlocks the mutex. Now, if resumed `Wait` ➍ did nothing, the checker goroutine would continue running. But the mutex would stay unlocked, so working with `num` wouldn't be safe. That's why `Wait` locks the mutex again after receiving the signal.

In theory, everything should work. Here's the output:

Everything seems fine, but there's a subtle bug. When the checker goroutine wakes up after receiving a signal, the mutex is unlocked for a brief moment before `Wait` locks it again. Theoretically, in that short time, another goroutine could sneak in and set `num` to 0. The checker goroutine wouldn't notice this and would keep running, even though it's supposed to wait if `num` is zero.

That's why, in practice, `Wait` is always called inside a for loop, not inside an if statement.

Not like this:

```
if num == 0 {
    cond.Wait()
}
```

But like this (note that the condition is the same as in the if statement):

```
for num == 0 {
    cond.Wait()
}
```

In most cases, this for loop will work just like an if statement:

1. The goroutine receives a signal and wakes up.
2. It locks the mutex.
3. On the next loop iteration, it checks the `num` value.
4. Since `num` is not 0, it exits the loop and continues.

But if another goroutine intervenes between ➊ and ➋ and sets `num` to zero, the goroutine will notice this at ➌ and go back to waiting. This way, it will never keep running when `num` is zero — which is exactly what we want.

Here's the complete example:

```
var wg sync.WaitGroup
cond := sync.NewCond(&sync.Mutex{})
num := 0

// Generates a random number from 1 to 100.
wg.Go(func() {
    time.Sleep(10 * time.Millisecond)
    cond.L.Lock()
    num = 1 + rand.IntN(100)
    cond.Signal()
    cond.L.Unlock()
})

// Checks if the number is lucky.
wg.Go(func() {
    cond.L.Lock()
    for num == 0 {
        cond.Wait()
    }
    if num%7 == 0 {
        fmt.Printf("Lucky number %d!\n", num)
    } else {
        fmt.Printf("Unlucky number %d...\n", num)
    }
    cond.L.Unlock()
})

wg.Wait()
```

Like other synchronization primitives, a condition variable has its own internal state. So, you should only pass it as a pointer, not by value. Even better, don't pass it at all — wrap it inside a type instead. We'll do this in the next step.

**✎ Exercise: Blocking queue**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## One-time subscription

Let's go back to the lucky numbers example:

```
// Generates a random number from 1 to 100.
go func() {
    // ...
}()

// Checks if the number is lucky.
go func() {
    // ...
}()
```

Let's refactor the code and create a `Lucky` type with `Guess` and `Wait` methods:

```
// Guess generates a random number and notifies
// the subscriber who's waiting with Wait.
Guess()

// Wait waits for a notification about a new number,
// then calls the subscriber's callback function.
Wait(callback func(int))
```

Here's the implementation:

```
// Lucky generates a random number.
type Lucky struct {
    cond *sync.Cond
    num  int
}

// NewLucky creates a new Lucky.
func NewLucky() *Lucky {
    l := &Lucky{}
    l.cond = sync.NewCond(&sync.Mutex{})
    return l
}

// Guess generates a random number and notifies
// the subscriber who's waiting with Wait.
func (l *Lucky) Guess() {
    l.cond.L.Lock()
    defer l.cond.L.Unlock()
    l.num = 1 + rand.IntN(100)
    l.cond.Signal()
}

// Wait waits for a notification about a new number,
// then calls the subscriber's callback function.
func (l *Lucky) Wait(callback func(int)) {
    l.cond.L.Lock()
    // Wait for a signal about a new number.
    for l.num == 0 {
        l.cond.Wait()
    }

    // Make a copy of the number to avoid holding
    // the lock while calling the callback.
    num := l.num
    l.cond.L.Unlock()

    // Call the subscriber's callback function.
    callback(num)
}
```

Example usage:

```
func main() {
    var wg sync.WaitGroup
    lucky := NewLucky()

    wg.Add(1)
    go lucky.Wait(func(num int) {
        if num%7 == 0 {
            fmt.Printf("Lucky number %d!\n", num)
        } else {
            fmt.Printf("Unlucky number %d...\n", num)
        }
        wg.Done()
    })

    lucky.Guess()
    wg.Wait()
}
```

> Note that this is a one-time signaling, not a long-term subscription. Once a subscriber goroutine receives the generated number, it is no longer subscribed to `Lucky`. We'll look at an example of a long-term subscription later in the chapter.

Everything works, but there's still a problem. If you call `Wait` from N goroutines, you can set up N subscribers, but `Guess` only notifies one of them. We'll figure out how to notify all of them in the next step.

## Broadcasting

Notifying all subscribers instead of just one is called _broadcasting_. To do this in `Lucky`, we only need to change one line in the `Guess` method:

```
// Guess generates a random number and notifies
// the subscribers who are waiting with Wait.
func (l *Lucky) Guess() {
    l.cond.L.Lock()
    defer l.cond.L.Unlock()
    l.num = 1 + rand.IntN(100)
    // Using Broadcast instead of Signal.
    l.cond.Broadcast()
}
```

The `Signal` method wakes up _one_ goroutine that's waiting on `Cond.Wait`, while the `Broadcast` method wakes up _all_ goroutines waiting on `Cond.Wait`. This is exactly what we need.

Here's a usage example:

```
// checkLucky checks if num is a lucky number.
func checkLucky(num, divisor int) {
    if num%divisor == 0 {
        fmt.Printf("mod %d: lucky number %d!\n", divisor, num)
    } else {
        fmt.Printf("mod %d: unlucky number %d...\n", divisor, num)
    }
}

func main() {
    var wg sync.WaitGroup
    wg.Add(3)

    lucky := NewLucky()
    go lucky.Wait(func(num int) {
        checkLucky(num, 3)
        wg.Done()
    })
    go lucky.Wait(func(num int) {
        checkLucky(num, 7)
        wg.Done()
    })
    go lucky.Wait(func(num int) {
        checkLucky(num, 13)
        wg.Done()
    })

    lucky.Guess()
    wg.Wait()
}
```

```
mod 3: unlucky number 98...
mod 13: unlucky number 98...
mod 7: lucky number 98!
```

A typical way to use a condition variable looks like this:

1. There is a publisher goroutine and one or more subscriber goroutines. They all use some shared state protected by a condition variable `c`.
    
2. The publisher goroutine changes the shared state and notifies either one subscriber (`Signal`) or all subscribers (`Broadcast`):
    

```
c.L.Lock()
// ... change the shared state
c.Signal() // or c.Broadcast()
c.L.Unlock()
```

3. The subscriber goroutine waits for an event in a loop, then works with the new shared state:

```
c.L.Lock()
for !condition() {
    c.Wait()
}
// ... use the changed shared state
c.L.Unlock()
```

Note that this is a one-time notification, not a long-term subscription. Once a subscriber goroutine receives the signal, it is no longer subscribed to the publisher. We'll look at an example of a long-term subscription later in the chapter.

**✎ Exercise: Barrier using a condition variable**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Broadcasting with channels

As we discussed, it's easy to implement signaling using a channel:

```
// Common state.
num := make(chan int, 1)

// Sender goroutine.
go func() {
    // Send a signal.
    num <- 1 + rand.Intn(100)
}()

// Receiver goroutine.
go func() {
    // Receive a signal.
    n := <-num
    // ...
}()
```

This approach only works with one receiver. If we subscribe multiple goroutines to the `num` channel, only one of them will get the generated number.

For broadcasting, we can just close the channel:

```
var wg sync.WaitGroup

// Common state and mutex to protect it.
var mu sync.Mutex
var num int

// Broadcasting channel.
broadcast := make(chan struct{})

// Sender goroutine.
wg.Go(func() {
    // Generate a number.
    mu.Lock()
    num = 1 + rand.IntN(100)
    mu.Unlock()
    // Send a broadcast.
    close(broadcast)
})

// Receiver goroutine.
wg.Go(func() {
    // Receive a signal.
    <-broadcast
    mu.Lock()
    // Use the generated number.
    fmt.Println("received", num)
    mu.Unlock()
})

// Receiver goroutine.
wg.Go(func() {
    // Receive a signal.
    <-broadcast
    mu.Lock()
    // Use the generated number.
    fmt.Println("received", num)
    mu.Unlock()
})

wg.Wait()
```

However, we can only broadcast the _fact_ that the state has changed (the `broadcast` channel is closed), not the actual _state_ (the value of `num`). So, we still need to protect the state with a mutex. This goes against the idea of using channels to pass data between goroutines. Also, we can't send a second broadcast notification because a channel can only be closed once.

## Publish/subscribe

We have two problems with our "broadcast by closing a channel" approach:

- The broadcast only sends a signal, not the actual state.
- The broadcast only works once.

Let's solve both problems and create a simple publish/subscribe system:

- The `Lucky` type generates random numbers (publisher).
- Multiple goroutines want to receive these numbers (subscribers).
- Each subscriber has its own channel registered with `Lucky`.
- After generating a number, `Lucky` sends it to each subscriber's channel.

Here's the `Lucky` type:

```
// Lucky generates random numbers
// and notifies subscribers.
type Lucky struct {
    // sbox stores a slice of subscription channels.
    // It lets us safely work with the slice
    // without needing a mutex.
    sbox chan []chan int
}

// NewLucky creates a new Lucky.
func NewLucky() *Lucky {
    sbox := make(chan []chan int, 1)
    sbox <- nil // no subscribers initially
    return &Lucky{sbox: sbox}
}
```

The `Subscribe` method adds a subscriber and returns the channel where random numbers will be sent:

```
// Subscribe adds a new subscriber and returns
// a channel to receive lucky numbers.
func (l *Lucky) Subscribe() <-chan int {
    subs := <-l.sbox
    sub := make(chan int, 1)
    subs = append(subs, sub)
    l.sbox <- subs
    return sub
}
```

Note that we use `l.sbox` as a mutex here to protect access to the shared list of subscription channels. Without it, concurrent `Subscribe` calls would cause a data race on `subs`. Alternatively, we can use a regular channel slice `l.subs` and protect is with a mutex `l.mu`:

```
// Lucky generates random numbers
// and notifies subscribers.
type Lucky struct {
    subs []chan int
    mu   sync.Mutex
}

// Subscribe adds a new subscriber and returns
// a channel to receive lucky numbers.
func (l *Lucky) Subscribe() <-chan int {
    l.mu.Lock()
    defer l.mu.Unlock()
    sub := make(chan int, 1)
    l.subs = append(l.subs, sub)
    return sub
}
```

The `Guess` number generates a number and sends it to each subscriber:

```
// Guess generates a random number
// and notifies subscribers.
func (l *Lucky) Guess() {
    subs := <-l.sbox
    num := 1 + rand.IntN(100)
    for _, sub := range subs {
        select {
        case sub <- num:
        default:
            // Subscriber is not ready
            // to receive, drop the number.
        }
    }
    l.sbox <- subs
}
```

Our `Guess` implementation drops the message if the subscriber hasn't processed the previous one. So, `Guess` always works quickly and doesn't block, but slower subscribers might miss some data. Alternatively, we can use a blocking `sub <- num` without select to make sure everyone gets all the data, but this means the whole system will only run as fast as the slowest subscriber.

The `Stop` method terminates all subscriptions:

```
// Stop unsubscribes all subscribers.
func (l *Lucky) Stop() {
    subs := <-l.sbox
    for _, sub := range subs {
        close(sub)
    }
    l.sbox <- nil
}
```

Here's an example with three subscribers. Each one gets three random numbers:

```
func main() {
    var wg sync.WaitGroup

    lucky := NewLucky()
    sub1 := lucky.Subscribe()
    sub2 := lucky.Subscribe()
    sub3 := lucky.Subscribe()

    wg.Go(func() {
        for num := range sub1 {
            checkLucky(num, 3)
        }
    })
    wg.Go(func() {
        for num := range sub2 {
            checkLucky(num, 7)
        }
    })
    wg.Go(func() {
        for num := range sub3 {
            checkLucky(num, 13)
        }
    })

    lucky.Guess()
    time.Sleep(10 * time.Millisecond)

    lucky.Guess()
    time.Sleep(10 * time.Millisecond)

    lucky.Guess()
    time.Sleep(10 * time.Millisecond)

    lucky.Stop()
    wg.Wait()
}
```

```
mod 13: unlucky number 42...
mod 3: lucky number 42!
mod 7: lucky number 42!
mod 3: lucky number 36!
mod 13: unlucky number 36...
mod 7: unlucky number 36...
mod 13: unlucky number 9...
mod 3: lucky number 9!
mod 7: unlucky number 9...
```

That's it for signals and broadcasting! Now let's look at a couple more tools from the `sync` package.

## Run once

Let's say we have a currency converter:

```
// CurrencyConverter converts money
// amounts between currencies.
type CurrencyConverter struct {
    rates map[string]float64
}

// Converts an amount from one currency to another.
func (c *CurrencyConverter) Convert(amount float64, from, to string) float64 {
    // Skipping validation for simplicity.
    return amount * c.rates[to] / c.rates[from]
}
```

Exchange rates are loaded from an external API, so we decided to fetch them lazily the first time `Convert` is called:

```
// init loads exchange rates from an external source.
func (c *CurrencyConverter) init() {
    // Simulate a network delay.
    time.Sleep(100 * time.Millisecond)
    c.rates = map[string]float64{"USD": 1.0, "EUR": 0.86}
}

// Converts an amount from one currency to another.
func (c *CurrencyConverter) Convert(amount float64, from, to string) float64 {
    if c.rates == nil {
        c.init()
    }
    return amount * c.rates[to] / c.rates[from]
}
```

Unfortunately, this creates a data race on `c.rates` when used in a concurrent environment:

```
func main() {
    cc := new(CurrencyConverter)

    var wg sync.WaitGroup
    for i := range 4 {
        wg.Go(func() {
            usd := 100.0 * float64(i+1)
            eur := cc.Convert(usd, "USD", "EUR")
            fmt.Printf("%v USD = %v EUR\n", usd, eur)
        })
    }
    wg.Wait()
}
```

```
==================
WARNING: DATA RACE
Write at 0x00c000058028 by goroutine 9:
  ...

Previous write at 0x00c000058028 by goroutine 6:
  ...
==================
```

We could protect the `rates` field with a mutex. Or we could use the `sync.Once` type. It guarantees that a function called with `Once.Do()` runs only once:

```
// CurrencyConverter converts money
// amounts between currencies.
type CurrencyConverter struct {
    rates map[string]float64
    once  sync.Once
}

// init loads exchange rates from an external source.
func (c *CurrencyConverter) init() {
    time.Sleep(100 * time.Millisecond)
    c.rates = map[string]float64{"USD": 1.0, "EUR": 0.86}
}

// Converts an amount from one currency to another.
func (c *CurrencyConverter) Convert(amount float64, from, to string) float64 {
    c.once.Do(c.init)
    return amount * c.rates[to] / c.rates[from]
}
```

```
400 USD = 344 EUR
200 USD = 172 EUR
100 USD = 86 EUR
300 USD = 258 EUR
```

`Once.Do` makes sure that the given function runs only once. If multiple goroutines call `Do` at the same time, only one will run the function, while the others will wait until it returns. This way, all calls to `Convert` are guaranteed to proceed only after the `rates` map has been filled.

`sync.Once` is perfect for one-time initialization or cleanup in a concurrent environment. No need to worry about data races!

## Once-functions

Besides the `Once` type, the `sync` package also includes three convenience once-functions that you might find useful.

Let's say we have the `randomN` function that returns a random number:

```
// randomN returns a random number from 1 to 10.
func randomN() int {
    return 1 + rand.IntN(10)
}
```

And the `initN` function sets the `n` variable to a random number:

```
n := 0
initN := func() {
    if n != 0 {
        panic("n is already initialized")
    }
    n = randomN()
}
```

It's clear that calling `initN` more than once will cause a panic (I'm keeping it simple and not using goroutines here):

```
for range 10 {
    initN()
}
fmt.Println(n)
```

```
panic: n is already initialized
```

We can fix this by wrapping `initN` in `sync.OnceFunc`. It returns a function that makes sure the code runs only once:

```
initOnce := sync.OnceFunc(initN)

for range 10 {
    initOnce()
}
fmt.Println(n)
```

`sync.OnceValue` wraps a function that returns a single value (like our `randomN`). The first time you call the function, it runs and calculates a value. After that, every time you call it, it just returns the same value from the first call:

```
initN := sync.OnceValue(randomN)

for range 4 {
    fmt.Print(initN(), " ")
}
fmt.Println()
```

`sync.OnceValues` does the same thing for a function that returns two values:

```
initNM := sync.OnceValues(func() (int, int) {
    return randomN(), randomN()
})

for range 4 {
    n, m := initNM()
    fmt.Printf("(%d,%d) ", n, m)
}
fmt.Println()
```

Here are the signatures of all the once-functions side by side for clarity:

```
// Calls f only once.
func (o *Once) Do(f func())

// Returns a function that calls f only once.
func OnceFunc(f func()) func()

// Returns a function that calls f only once
// and returns the value from that first call.
func OnceValue[T any](f func() T) func() T

// Returns a function that calls f only once
// and returns the pair of values from that first call.
func OnceValues[T1, T2 any](f func() (T1, T2)) func() (T1, T2)
```

The functions `OnceFunc`, `OnceValue`, and `OnceValues` are shortcuts for common ways to use the `Once` type. You can use them if they fit your situation, or use `Once` directly if they don't.

**✎ Exercise: Guess the average**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Object pool

The last tool we'll cover is `sync.Pool`. It helps reuse memory instead of allocating it every time, which reduces the load on the garbage collector.

Let's say we have a program that:

1. Allocates 1024 bytes.
2. Does something with that memory.
3. Goes back to step 1 and repeats this process many times.

It looks something like this:

```
func runAlloc() {
    // 4 goroutines, each allocating
    // and freeing 1000 buffers.
    var wg sync.WaitGroup
    for range 4 {
        wg.Go(func() {
            for range 1000 {
                buf := make([]byte, 1024)
                rand.Read(buf)
            }
        })
    }
    wg.Wait()
}
```

If we run the benchmark:

```
func BenchmarkAlloc(b *testing.B) {
    for b.Loop() {
        runAlloc()
    }
}
```

Here's what we'll see:

```
BenchmarkAlloc-8    219     5392291 ns/op   4096215 B/op    4005 allocs/op
```

Since we're allocating a new buffer on each loop iteration, we end up with 4000 memory allocations, using a total of 4 MB of memory. Even though the garbage collector eventually frees all this memory, it's quite inefficient. Ideally, we should only need 4 buffers instead of 4000 — one for each goroutine.

That's where `sync.Pool` comes in handy:

```
func runPool() {
    // Pool with 1 KB buffers.
    pool := sync.Pool{
        New: func() any {                    // (1)
            // Allocate a 1 KB buffer.
            buf := make([]byte, 1024)
            return &buf
        },
    }

    // 4 goroutines, each allocating
    // and freeing 1000 buffers.
    var wg sync.WaitGroup
    for range 4 {
        wg.Go(func() {
            for range 1000 {
                buf := pool.Get().(*[]byte)  // (2)
                rand.Read(*buf)
                pool.Put(buf)                // (3)
            }
        })
    }
    wg.Wait()
}
```

`pool.Get` ➋ takes an item from the pool. If there are no available items, it creates a new one using `pool.New` ➊ (which we have to define ourselves, since the pool doesn't know anything about the items it creates). `pool.Put` ➌ returns an item back to the pool.

When the first goroutine calls `Get` during the first iteration, the pool is empty, so it creates a new buffer using `New`. In the same way, each of the other goroutines create three more buffers. These four buffers are enough for the whole program.

Let's benchmark:

```
func BenchmarkPool(b *testing.B) {
    for b.Loop() {
        runPool()
    }
}
```

```
BenchmarkPool-8     206     5266199 ns/op      5770 B/op      15 allocs/op
BenchmarkAlloc-8    219     5392291 ns/op   4096215 B/op    4005 allocs/op
```

The difference in memory usage is clear. Thanks to the pool, the number of allocations has dropped by two orders of magnitude. As a result, the program uses less memory and puts minimal pressure on the garbage collector.

Things to keep in mind:

- `New` should return a pointer, not a value, to reduce memory copying and avoid extra allocations.
- The pool has no size limit. If you start 1000 more goroutines that all call `Get` at the same time, 1000 more buffers will be allocated.
- After an item is returned to the pool with `Put`, you shouldn't use it anymore (since another goroutine might already have taken and started using it).

`sync.Pool` is a pretty niche tool that isn't used very often. However, if your program works with temporary objects that can be reused (like in our example), it might come in handy.

## Keep it up

We've covered some of the lesser-known tools in the `sync` package — condition variables (`sync.Cond`), one-time execution (`sync.Once`), and pools (`sync.Pool`):

- A condition variable notifies one or more waiting goroutines about an event. You can often use a channel instead, and that's usually the better choice.
- A one-time execution guarantees that a function runs exactly once, no matter how many goroutines call it at the same time.
- A pool lets you reuse temporary objects so you don't have to allocate memory every time.

Don't use these tools just because you know they exist. Rely on common sense.

In the next chapter, we'll talk about [atomics](app://obsidian.md/atomics/).

[Pre-order for $10](https://antonz.gumroad.com/l/go-concurrency)   or [read online](app://obsidian.md/go-concurrency/)

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.