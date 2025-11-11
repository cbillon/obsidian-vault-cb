---
link: https://antonz.org/go-concurrency/data-races/
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Two goroutines racing for the same data is a recipe for disaster.
twitter: https://twitter.com/@ohmypy
slurped: 2025-11-11T09:56
title: "Gist of Go: Data races"
---

_This is a chapter from my book on [Go concurrency](app://obsidian.md/go-concurrency), which teaches the topic from the ground up through interactive examples._

What happens if multiple goroutines modify the same data structure? Sadly, nothing good. Let's learn more about it.

- [Concurrent modification](#concurrent-modification)
- [Data race](#data-race)
- [Sequential modification](#sequential-modification)
- [Mutex](#mutex)

- [Read-write mutex](#read-write-mutex)
- [Channel as mutex](#channel-as-mutex)
- [Keep it up](#keep-it-up)

## Concurrent modification

So far, our goroutines haven't gotten in each other's way. They've used channels to exchange data, which is safe. But what happens if several goroutines try to access the same object at the same time? Let's find out.

Let's write a program that counts word frequencies:

```
func main() {
    // generate creates 100 words, each 3 letters long,
    // and sends them to the channel.
    in := generate(100, 3)

    var wg sync.WaitGroup
    wg.Add(2)

    // count reads words from the input channel
    // and counts how often each one appears.
    count := func(counter map[string]int) {
        defer wg.Done()
        for word := range in {
            counter[word]++
        }
    }

    counter := map[string]int{}
    go count(counter)
    go count(counter)
    wg.Wait()

    fmt.Println(counter)
}
```

```
fatal error: concurrent map writes

goroutine 1 [sync.WaitGroup.Wait]:
sync.runtime_SemacquireWaitGroup(0x140000021c0?)

goroutine 34 [chan send]:
main.generate.func1()

goroutine 35 [running]:
internal/runtime/maps.fatal({0x104b4039e?, 0x14000038a08?})

goroutine 36 [runnable]:
internal/runtime/maps.newTable(0x104b78340, 0x80, 0x0, 0x0)
```

What is generate

```
// generate creates nWords words, each wordLen letters long,
// and sends them to the channel.
func generate(nWords, wordLen int) <-chan string {
    out := make(chan string)
    go func() {
        defer close(out)
        for ; nWords > 0; nWords-- {
            out <- randomWord(wordLen)
        }
    }()
    return out
}

// randomWord returns a random word with n letters.
func randomWord(n int) string {
    const vowels = "eaiou"
    const consonants = "rtnslcdpm"
    chars := make([]byte, n)
    for i := 0; i < n; i += 2 {
        chars[i] = consonants[rand.IntN(len(consonants))]
    }
    for i := 1; i < n; i += 2 {
        chars[i] = vowels[rand.IntN(len(vowels))]
    }
    return string(chars)
}
```

`generate()` generates words and sends them to the `in` channel. `main()` creates an empty map called `counter` and passes it to two `count()` goroutines. `count()` reads from the `in` channel and fills the map with word counts. In the end, `counter` should contain the frequency of each word.

Let's run it:

```
map[cec:1 ... nol:2 not:3 ... tut:1]
```

And once again, just in case:

```
fatal error: concurrent map writes

goroutine 1 [sync.WaitGroup.Wait]:
sync.runtime_SemacquireWaitGroup(0x140000021c0?)

goroutine 34 [chan send]:
main.generate.func1()

goroutine 35 [running]:
internal/runtime/maps.fatal({0x104b4039e?, 0x14000038a08?})

goroutine 36 [runnable]:
internal/runtime/maps.newTable(0x104b78340, 0x80, 0x0, 0x0)
```

Panic!

Go doesn't let multiple goroutines write to a map at the same time. At first, this might seem odd. Here's the only operation that the `count()` goroutine does with the map:

Looks like an atomic action. Why not perform it from multiple goroutines?

The problem is that the action only seems atomic. The operation "increase the key value in the map" actually involves several smaller steps. If one goroutine does some of these steps and another goroutine does the rest, the map can get messed up. That's what the runtime is warning us about.

## Data race

When multiple goroutines access the same variable at the same time, and at least one of them changes it, it's called a _data race_. Concurrent map modification in the previous section is an example of a data race.

A data race doesn't always cause a runtime panic (the map example in the previous section is a nice exception: Go's map implementation has built-in runtime checks that can catch some data races). That's why Go provides a special tool called the _race detector_. You can turn it on with the `race` flag, which works with the `test`, `run`, `build`, and `install` commands.

> To use Go's race detector, you'll need to install [gcc](https://gcc.gnu.org/wiki/InstallingGCC), the C compiler.

For example, take this program:

```
func main() {
    var total int
    var wg sync.WaitGroup

    wg.Go(func() {
        total++
    })
    wg.Go(func() {
        total++
    })

    wg.Wait()
    fmt.Println(total)
}
```

At first glance, it seems to work correctly. But actually, it has a data race:

```
==================
WARNING: DATA RACE
Read at 0x00c000112038 by goroutine 6:
  main.main.func1()
      race.go:16 +0x74

Previous write at 0x00c000112038 by goroutine 7:
  main.main.func2()
      race.go:21 +0x84

Goroutine 6 (running) created at:
  main.main()
      race.go:14 +0x104

Goroutine 7 (finished) created at:
  main.main()
      race.go:19 +0x1a4
==================
2
Found 1 data race(s)
```

> If you're wondering why a data race is a problem for a simple operation like `total++` — we'll cover it later in the chapter on atomic operations.

Channels, on the other hand, are safe for concurrent reading and writing, and they don't cause data races:

```
func main() {
    ch := make(chan int, 2)
    var wg sync.WaitGroup

    wg.Go(func() {
        ch <- 1
    })
    wg.Go(func() {
        ch <- 1
    })

    wg.Wait()
    fmt.Println(<-ch + <-ch)
}
```

Data races are dangerous because they're hard to spot. Your program might work fine a hundred times, but on the hundred and first try, it could give the wrong result. Always check your code with a race detector.

**✎ Exercise: Spot the race**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Sequential modification

You can often rewrite a program to avoid concurrent modifications. Here is a possible approach for our word frequency program:

- Each `count()` goroutine counts frequencies in its own map.
- A separate `merge()` function goes through the frequency maps and builds the final map.

```
func main() {
    // generate creates 100 words, each 3 letters long,
    // and sends them to the channel.
    in := generate(100, 3)

    var wg sync.WaitGroup
    wg.Add(2)

    // count reads words from the input channel
    // and counts how often each one appears.
    count := func(counters []map[string]int, idx int) {
        defer wg.Done()
        counter := map[string]int{}
        for word := range in {
            counter[word]++
        }
        counters[idx] = counter
    }

    counters := make([]map[string]int, 2)
    go count(counters, 0)
    go count(counters, 1)
    wg.Wait()

    // merge combines frequency maps.
    counter := merge(counters...)
    fmt.Println(counter)
}

// merge combines frequency maps into a single map.
func merge(counters ...map[string]int) map[string]int {
    merged := map[string]int{}
    for _, counter := range counters {
        for word, freq := range counter {
            merged[word] += freq
        }
    }
    return merged
}
```

```
map[cec:1 ... nol:2 not:3 ... tut:1]
```

Technically, we're still using shared data — the `counters` slice in `count()`. But the `idx` parameter makes sure the first `count()` only works with the first element of the slice, and the second one only works with the second element. Go allows this kind of concurrent slice access.

> It's important that multiple goroutines don't try to change the same element of the slice or call `append()`.

Even if Go didn't allow concurrent access to slices, we could still solve the problem. We would just use a channel of maps instead of a slice:

- Each `count()` goroutine counts frequencies in its own map and sends it to a shared channel.
- A separate `merge()` function in the `main` goroutine reads frequency maps from the shared channel and builds the final map.

Either way, using a separate `merge()` step works, but it's not always convenient. Sometimes, we want to modify the same data from multiple goroutines. As is our right.

Let's see how we can do it.

## Mutex

The `sync` package has a special tool called a _mutex_. It protects shared data and parts of your code (_critical sections_) from being accessed concurrently:

```
func main() {
    // generate creates 100 words, each 3 letters long,
    // and sends them to the channel.
    in := generate(100, 3)

    var wg sync.WaitGroup
    wg.Add(2)

    // count reads words from the input channel
    // and counts how often each one appears.
    count := func(lock *sync.Mutex, counter map[string]int) {
        defer wg.Done()
        for word := range in {
            lock.Lock()       // (2)
            counter[word]++
            lock.Unlock()     // (3)
        }
    }

    var lock sync.Mutex       // (1)
    counter := map[string]int{}
    go count(&lock, counter)
    go count(&lock, counter)
    wg.Wait()

    fmt.Println(counter)
}
```

```
map[cec:1 ... nol:2 not:3 ... tut:1]
```

The mutex guarantees that only one goroutine can run the code between `Lock()` and `Unlock()` at a time. Here's how it works:

1. In ➊, we create a mutex and pass it to both `count()` goroutines.
2. In ➋, the first goroutine _locks_ the mutex, then runs `counter[word]++`.
3. If the second goroutine reaches ➋ at this time, it will _block_ because the mutex is locked.
4. In ➌, the first goroutine _unlocks_ the mutex.
5. Now the second goroutine is _unblocked_. It then locks the mutex and runs `counter[word]++`.

This way, `counter[word]++` can't be run by multiple goroutines at the same time. Now the map won't get corrupted:

```
$ go run -race counter.go
map[cec:1 ... nol:2 not:3 ... tut:1]
```

A mutex is used in these situations:

- When multiple goroutines are changing the same data.
- When one goroutine is changing data and others are reading it.

If all goroutines are only reading the data, you don't need a mutex.

Unlike some other languages, a mutex in Go is not reentrant. If a goroutine calls `Lock()` on a mutex it already holds, it will block itself:

```
func main() {
    var lock sync.Mutex

    lock.Lock()
    // ok

    lock.Lock()
    // fatal error: all goroutines are asleep - deadlock!
}
```

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [sync.Mutex.Lock]:
internal/sync.runtime_SemacquireMutex(0x4641f9?, 0x80?, 0xc00006cf40?)
    /usr/local/go/src/runtime/sema.go:95 +0x25
internal/sync.(*Mutex).lockSlow(0xc000010060)
    /usr/local/go/src/internal/sync/mutex.go:149 +0x15d
internal/sync.(*Mutex).Lock(...)
    /usr/local/go/src/internal/sync/mutex.go:70
sync.(*Mutex).Lock(...)
    /usr/local/go/src/sync/mutex.go:46
main.main()
    /sandbox/src/main.go:45 +0x5f (exit status 2)
```

This makes things harder for people who like to use mutexes in recursive functions (which isn't a great idea anyway).

Like a wait group, a mutex has internal state, so you should only pass it as a pointer.

**✎ Exercise: Concurrent-safe counter**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Read-write mutex

A regular mutex doesn't distinguish between read and write access: if one goroutine locks the mutex, others can't access the protected code. This isn't always necessary.

Here's the situation:

- One `writer` goroutine writes data.
- Four `reader` goroutines read that same data.

```
var wg sync.WaitGroup
wg.Add(5)

var lock sync.Mutex

// writer fills in the word frequency map.
writer := func(counter map[string]int, nWrites int) {
    defer wg.Done()
    for ; nWrites > 0; nWrites-- {
        word := randomWord(3)
        lock.Lock()
        counter[word]++
        time.Sleep(time.Millisecond)
        lock.Unlock()
    }
}

// reader looks up random words in the frequency map.
reader := func(counter map[string]int, nReads int) {
    defer wg.Done()
    for ; nReads > 0; nReads-- {
        word := randomWord(3)
        lock.Lock()
        _ = counter[word]
        time.Sleep(time.Millisecond)
        lock.Unlock()
    }
}

start := time.Now()

counter := map[string]int{}
go writer(counter, 100)
go reader(counter, 100)
go reader(counter, 100)
go reader(counter, 100)
go reader(counter, 100)
wg.Wait()

fmt.Println("Took", time.Since(start))
```

Even though we started 4 reader goroutines, they run sequentially because of the mutex. This isn't really necessary. It makes sense for readers to wait while the writer is updating the map. But why can't the readers run in parallel? They're not changing any data.

The `sync` package includes a `sync.RWMutex` that separates readers and writers. It provides two sets of methods:

- `Lock` / `Unlock` lock and unlock the mutex for both reading and writing.
- `RLock` / `RUnlock` lock and unlock the mutex for reading only.

Here's how it works:

- If a goroutine locks the mutex with `Lock()`, other goroutines will be blocked if they try to use `Lock()` or `RLock()`.
- If a goroutine locks the mutex with `RLock()`, other goroutines can also lock it with `RLock()` without being blocked.
- If at least one goroutine has locked the mutex with `RLock()`, other goroutines will be blocked if they try to use `Lock()`.

This creates a "single writer, multiple readers" setup. Let's verify it:

```
var wg sync.WaitGroup
wg.Add(5)

var lock sync.RWMutex          // (1)

// writer fills in the word frequency map.
writer := func(counter map[string]int, nWrites int) {
    // Not changed.
    defer wg.Done()
    for ; nWrites > 0; nWrites-- {
        word := randomWord(3)
        lock.Lock()
        counter[word]++
        time.Sleep(time.Millisecond)
        lock.Unlock()
    }
}

// reader looks up random words in the frequency map.
reader := func(counter map[string]int, nReads int) {
    defer wg.Done()
    for ; nReads > 0; nReads-- {
        word := randomWord(3)
        lock.RLock()           // (2)
        _ = counter[word]
        time.Sleep(time.Millisecond)
        lock.RUnlock()         // (3)
    }
}

start := time.Now()

counter := map[string]int{}
go writer(counter, 100)
go reader(counter, 100)
go reader(counter, 100)
go reader(counter, 100)
go reader(counter, 100)
wg.Wait()

fmt.Println("Took", time.Since(start))
```

The mutex type ➊ has changed, so have the locking ➋ and unlocking ➌ methods in the reader. Now, readers run concurrently, but they always wait while the writer updates the map. That's exactly what we need!

**✎ Exercise: Counter with RWMutex**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Channel as mutex

Let's go back to the program that counts word frequencies:

```
func main() {
    // generate creates 100 words, each 3 letters long,
    // and sends them to the channel.
    in := generate(100, 3)

    var wg sync.WaitGroup
    wg.Add(2)

    // count reads words from the input channel
    // and counts how often each one appears.
    count := func(lock *sync.Mutex, counter map[string]int) {
        defer wg.Done()
        for word := range in {
            lock.Lock()       // (2)
            counter[word]++
            lock.Unlock()     // (3)
        }
    }

    var lock sync.Mutex       // (1)
    counter := map[string]int{}
    go count(&lock, counter)
    go count(&lock, counter)
    wg.Wait()

    fmt.Println(counter)
}
```

We created the `lock` mutex ➊ and used it to protect access to the shared `counter` map ➋ ➌. This way, the `count()` goroutines don't cause data races, and the final `counter[word]` value is correct.

We can also use a channel instead of a mutex to protect shared data:

```
type token struct{}

func main() {
    // generate creates 100 words, each 3 letters long,
    // and sends them to the channel.
    in := generate(100, 3)

    var wg sync.WaitGroup
    wg.Add(2)

    // count reads words from the input channel
    // and counts how often each one appears.
    count := func(lock chan token, counter map[string]int) {
        defer wg.Done()
        for word := range in {
            lock <- token{}     // (2)
            counter[word]++
            <-lock              // (3)
        }
    }

    lock := make(chan token, 1) // (1)

    counter := map[string]int{}
    go count(lock, counter)
    go count(lock, counter)
    wg.Wait()

    fmt.Println(counter)
}
```

```
map[cec:1 ... nol:2 not:3 ... tut:1]
```

We created a `lock` channel with a one-element buffer ➊ and used it to protect access to the shared `counter` map ➋ ➌.

Two `count()` goroutines run concurrently. However, in each loop iteration, only one of them can put a token into the `lock` channel (like locking a mutex), update the counter, and take the token back out (like unlocking a mutex). So, even though the goroutines run in parallel, changes to the map happen sequentially.

As a result, the `count()` goroutines don't cause data races, and the final `counter[word]` value is correct — just like when we used a mutex.

Go's channels are a versatile concurrency tool. Often, you can use a channel instead of lower-level synchronization primitives. Sometimes, using a channel is unnecessary, as in the example above. Other times, however, it makes your code simpler and helps prevent mistakes. You'll see this idea come up again throughout the book.

## Keep it up

Now you know how to safely change shared data from multiple goroutines using mutexes. Be careful not to overuse them — it's easy to make mistakes and cause data races or deadlocks.

In the next chapter, we'll talk about [race conditions](app://obsidian.md/race-conditions/).

[Pre-order for $10](https://antonz.gumroad.com/l/go-concurrency)   or [read online](app://obsidian.md/go-concurrency/)

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.