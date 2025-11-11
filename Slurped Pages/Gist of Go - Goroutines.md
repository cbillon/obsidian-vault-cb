---
link: https://antonz.org/go-concurrency/goroutines/
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Let's write a concurrent program in Go!
twitter: https://twitter.com/@ohmypy
slurped: 2025-11-11T09:47
title: "Gist of Go: Goroutines"
---

_This is a chapter from my book on [Go concurrency](app://obsidian.md/go-concurrency), which teaches the topic from the ground up through interactive examples._

Let's skip the long talk about concurrency and parallelism and jump right into writing a concurrent program in Go!

- [Go-routine](#go-routine)
- [Dependent goroutines](#dependent-and-independent-goroutines)
- [Counting digits](#-counting-digits-in-words) ✎
- [Channels](#channels)
- [Result channel](#-result-channel) ✎
- [Generator](#generator)

- [Generator+goroutines](#-generator-with-goroutines) ✎
- [Reader/worker](#-reader-and-worker) ✎
- [Named goroutines](#-named-goroutines) ✎
- [Output channel](#output-channel)
- [Keep it up](#keep-it-up)

## Go-routine

Suppose we have a function that speaks a phrase word by word with some pauses:

```
// say prints each word of a phrase.
func say(phrase string) {
    for _, word := range strings.Fields(phrase) {
        fmt.Printf("Simon says: %s...\n", word)
        dur := time.Duration(rand.Intn(100)) * time.Millisecond
        time.Sleep(dur)
    }
}
```

We call it from the main function:

```
func main() {
    say("go is awesome")
}
```

```
Simon says: go...
Simon says: is...
Simon says: awesome...
```

Now let's create two talkers, each saying their own phrase:

```
// say prints each word of a phrase.
func say(id int, phrase string) {
    for _, word := range strings.Fields(phrase) {
        fmt.Printf("Worker #%d says: %s...\n", id, word)
        dur := time.Duration(rand.Intn(100)) * time.Millisecond
        time.Sleep(dur)
    }
}
```

Run the program:

```
func main() {
    say(1, "go is awesome")
    say(2, "cats are cute")
}
```

```
Worker #1 says: go...
Worker #1 says: is...
Worker #1 says: awesome...
Worker #2 says: cats...
Worker #2 says: are...
Worker #2 says: cute...
```

Not bad, but the functions speak one after the other. To make them speak at the same time, let's add `go` before calling the `say()` function:

```
func main() {
    go say(1, "go is awesome")
    go say(2, "cats are cute")
    time.Sleep(500 * time.Millisecond)
}
```

```
Worker #2 says: cats...
Worker #1 says: go...
Worker #2 says: are...
Worker #1 says: is...
Worker #2 says: cute...
Worker #1 says: awesome...
```

Now they really compete for our attention! When we write `go f()`, the function `f()` runs independently of the others.

> If you're familiar with concurrency in Python, JavaScript, or other languages with async/await, don't try to apply that experience to Go. Go has a very different approach to concurrency. Try to look at it with fresh eyes.

Functions that run with `go` are called _goroutines_. The Go runtime juggles these goroutines and distributes them among operating system threads running on CPU cores. Compared to OS threads, goroutines are lightweight, so you can create hundreds or thousands of them.

You may be wondering why we need `time.Sleep()` in the `main` function. Let's clarify that.

## Dependent and independent goroutines

Goroutines are completely independent. When we call `go say()`, the function runs on its own. `main` doesn't wait for it. So if we write `main` like this:

```
func main() {
    go say(1, "go is awesome")
    go say(2, "cats are cute")
}
```

— the program won't print anything. `main` finishes before our goroutines speak, and since the main is done, the whole program terminates.

The main function is also a goroutine, but it starts implicitly when the program starts. So we have three goroutines: `main`, `say(1)`, and `say(2)`, all of which are independent. The only catch is that when `main` ends, everything else ends too.

### Wait group

Using `time.Sleep()` to wait for goroutines is a bad idea because we can't predict how long they will take. A better approach is to use a _wait group_:

```
func main() {
    var wg sync.WaitGroup // (1)

    wg.Add(1)             // (2)
    go say(&wg, 1, "go is awesome")

    wg.Add(1)             // (2)
    go say(&wg, 2, "cats are cute")

    wg.Wait()             // (3)
}

// say prints each word of a phrase.
func say(wg *sync.WaitGroup, id int, phrase string) {
    for _, word := range strings.Fields(phrase) {
        fmt.Printf("Worker #%d says: %s...\n", id, word)
        dur := time.Duration(rand.Intn(100)) * time.Millisecond
        time.Sleep(dur)
    }
    wg.Done()             // (4)
}
```

```
Worker #2 says: cats...
Worker #1 says: go...
Worker #2 says: are...
Worker #1 says: is...
Worker #2 says: cute...
Worker #1 says: awesome...
```

`wg` ➊ has a counter inside. Calling `wg.Add(1)` ➋ increments it by one, while `wg.Done()` ➍ decrements it. `wg.Wait()` ➌ blocks the goroutine (in this case, `main`) until the counter reaches zero. This way, `main` waits for `say(1)` and `say(2)` to finish before it exits.

However, this approach mixes business logic (`say`) with concurrency logic (`wg`). As a result, we can't easily run `say` in regular, non-concurrent code.

In Go, it's common to separate concurrency logic from business logic. This is usually done with separate functions. In simple cases like ours, even anonymous functions will do:

```
func main() {
    var wg sync.WaitGroup
    wg.Add(2)

    go func() {
        defer wg.Done()
        say(1, "go is awesome")
    }()

    go func() {
        defer wg.Done()
        say(2, "cats are cute")
    }()

    wg.Wait()
}
```

```
Worker #2 says: cats...
Worker #1 says: go...
Worker #2 says: are...
Worker #1 says: is...
Worker #2 says: cute...
Worker #1 says: awesome...
```

Here's how it works:

- We know there will be two goroutines, so we call `wg.Add(2)` right away. Alternatively, we can call `wg.Add(1)` before starting each goroutine — the result will be the same.
- Anonymous functions are started with `go` just like regular ones.
- `defer wg.Done()` ensures the goroutine decrements the counter before exiting, even if `say` panics.
- `say` itself knows nothing about concurrency and just runs happily.

### WaitGroup.Go

The `WaitGroup.Go` method (Go 1.25+) automatically increments the wait group counter, runs a function in a goroutine, and decrements the counter when it's done. This means we can rewrite the example above without using `wg.Add()` and `wg.Done()`:

```
func main() {
    var wg sync.WaitGroup

    wg.Go(func() {
        fmt.Println("go is awesome")
    })

    wg.Go(func() {
        fmt.Println("cats are cute")
    })

    wg.Wait()
    fmt.Println("done")
}
```

```
cats are cute
go is awesome
done
```

The implementation uses `Add` and `Done` just like we did before:

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

I'll use both `Go` and `Add`+`Done` throughout the book, depending on the situation.

## ✎ Counting digits in words

_The ✎ symbol indicates exercises. They are an essential part of the book, so try not to skip them. Half of what you learn comes from the exercises._

Here's a function that counts the digits in a word:

```
// countDigits returns the number of digits in a string.
func countDigits(str string) int {
    count := 0
    for _, char := range str {
        if unicode.IsDigit(char) {
            count++
        }
    }
    return count
}
```

Create a function `countDigitsInWords` that takes an input string, splits it into words, and counts the digits in each word using `countDigits`. Be sure to do the counting for each word in a separate goroutine.

We haven't discussed how to modify shared data from different goroutines yet, so there is a ready-to-use variable called `syncStats` that you can safely access from goroutines.

**Important**: When submitting your solution, send only the code fragment marked with "solution start" and "solution end" comments. The full source code is available via the "Playground" link below.

```
// solution start

// countDigitsInWords counts the number of digits in the words of a phrase.
func countDigitsInWords(phrase string) counter {
    var wg sync.WaitGroup
    syncStats := new(sync.Map)
    words := strings.Fields(phrase)

    // Count the number of digits in words,
    // using a separate goroutine for each word.

    // To store the results of the count,
    // use syncStats.Store(word, count)

    // As a result, syncStats should contain words
    // and the number of digits in each.

    return asStats(syncStats)
}

// solution end
```

## Channels

Starting a bunch of goroutines is great, but how do they exchange data? In Go, goroutines can pass values to each other through _channels_. A channel is like a window where one goroutine can throw something and another can catch it.

```
┌─────────────┐    ┌─────────────┐
│ goroutine A │    │ goroutine B │
│             └────┘             │
│        X <-  chan  <- X        │
│             ┌────┐             │
│             │    │             │
└─────────────┘    └─────────────┘
```

_Goroutine B sends value X to goroutine A. Canvas, oil, circa 2024._

Here's how it works:

```
func main() {
    // To create a channel, use `make(chan type)`.
    // Channel can only accept values of the specified type:
    messages := make(chan string)

    // To send a value to a channel,
    // use the `channel <-` syntax.
    // Let's send "ping":
    go func() { messages <- "ping" }()

    // To receive a value from a channel,
    // use the `<-channel` syntax.
    // Let's receive "ping" and print it:
    msg := <-messages
    fmt.Println(msg)
}
```

When the program runs, the first goroutine (anonymous) sends a message to the second (`main`) through the `messages` channel.

Sending a value through a channel is a synchronous operation. When the sending goroutine writes a value to the channel (`messages <- "ping"`), it blocks and waits for someone to receive that value (`<-messages`). Only then does it continue:

```
func main() {
    messages := make(chan string)

    go func() {
        fmt.Println("B: Sending message...")
        messages <- "ping"                    // (1)
        fmt.Println("B: Message sent!")       // (2)
    }()

    fmt.Println("A: Doing some work...")
    time.Sleep(500 * time.Millisecond)
    fmt.Println("A: Ready to receive a message...")

    <-messages                               //  (3)

    fmt.Println("A: Messege received!")
    time.Sleep(100 * time.Millisecond)
}
```

```
A: Doing some work...
B: Sending message...
A: Ready to receive a message...
A: Messege received!
B: Message sent!
```

After sending the message to the channel ➊, goroutine B gets blocked. Only when goroutine A receives the message ➌ does goroutine B continue and print "message sent" ➋.

So, channels not only transfer data, but also help to synchronize independent goroutines. This will come in handy later.

## ✎ Result channel

We often see the "producer-consumer" pattern in programming:

- The producer supplies data.
- The consumer receives and processes it.

In this and the following exercises, we'll explore how producers and consumers can interact through channels.

We are working with a function that counts digits in words:

```
// counter stores the number of digits in each word.
// The key is the word, and the value is the number of digits.
type counter map[string]int

// countDigitsInWords counts the number of digits
// in the words of a phrase.
func countDigitsInWords(phrase string) counter {
    words := strings.Fields(phrase)
    // ...
    return stats
}
```

Do the following:

- Start a goroutine.
- In this goroutine, loop through the words, count the digits in each, and write to the `counted` channel (producer).
- In the outer function, read values from the channel and fill the `stats` counter (consumer).

In this and following exercises, don't use wait groups. We won't need them for quite a while.

Submit only the code fragment marked with "solution start" and "solution end" comments. The full source code is available via the "Playground" link below.

```
// solution start

// countDigitsInWords counts the number of digits in the words of a phrase.
func countDigitsInWords(phrase string) counter {
    words := strings.Fields(phrase)
    counted := make(chan int)

    go func() {
        // Loop through the words,
        // count the number of digits in each,
        // and write it to the counted channel.
    }()

    // Read values from the counted channel
    // and fill stats.

    // As a result, stats should contain words
    // and the number of digits in each.

    return stats
}

// solution end
```

> If you've worked with concurrency in Go, the solutions in this chapter may seem a bit... unorthodox, if not clunky. It's designed this way to avoid overwhelming readers who are new to concurrency. In a few chapters, we'll cover the core concepts, and the code will start to look idiomatic.

## Generator

So far, we've assumed that the `countDigitsInWords` function knows all the words in advance.

But we can't expect that luxury in real life. Data can come from a database or over the network, and the function has no idea how many words there will be.

Let's simulate this situation by passing a generator function `next` instead of a phrase. Each call to `next()` gives us the next word from the source. When there are no more words, it returns an empty string.

A sequential program would look like this:

```
// counter stores the number of digits in each word.
// The key is the word, and the value is the number of digits.
type counter map[string]int

// countDigitsInWords counts the number of digits in words,
// fetching the next word with next().
func countDigitsInWords(next func() string) counter {
    stats := counter{}

    for {
        word := next()
        if word == "" {
            break
        }
        count := countDigits(word)
        stats[word] = count
    }

    return stats
}

func main() {
    phrase := "0ne 1wo thr33 4068"
    next := wordGenerator(phrase)
    stats := countDigitsInWords(next)
    printStats(stats)
}
```

```
thr33: 2
4068: 4
0ne: 1
1wo: 1
```

Now let's add some concurrency.

## ✎ Generator with goroutines

We are working with a function that counts digits in words:

```
// countDigitsInWords counts the number of digits in words,
// fetching the next word with next().
func countDigitsInWords(next func() string) counter {
    // ...
    return stats
}
```

Do the following:

- Start a goroutine.
- In this goroutine, fetch words from the generator, count the digits in each, and write to the `counted` channel.
- In the outer function, read values from the channel and fill `stats`.

If you try to solve the exercise like the previous one, you'll run into a couple of problems:

```
func countDigitsInWords(next func() string) counter {
    counted := make(chan int)

    // count digits in words
    go func() {
        for {
            // should return when
            // there are no more words
            word := next()
            count := countDigits(word)
            counted <- count
        }
    }()

    // fill stats by words
    stats := counter{}
    for {
        count := <-counted
        // how to break from the loop
        // when there are no more words?
        if ... {
            break
        }
        // where should the word come from?
        stats[...] = count
    }

    return stats
}
```

Think about _what_ to send to the `counted` channel to solve both problems. Note the `pair` type.

Submit only the code fragment marked with "solution start" and "solution end" comments. The full source code is available via the "Playground" link below.

```
// solution start

// countDigitsInWords counts the number of digits in words,
// fetching the next word with next().
func countDigitsInWords(next func() string) counter {
    counted := make(chan ...)

    go func() {
        // Fetch words from the generator,
        // count the number of digits in each,
        // and write it to the counted channel.
    }()

    // Read values from the counted channel
    // and fill stats.

    // As a result, stats should contain words
    // and the number of digits in each.

    return stats
}

// solution end
```

## ✎ Reader and worker

We are working with a function that counts digits in words:

```
// countDigitsInWords counts the number of digits in words,
// fetching the next word with next().
func countDigitsInWords(next func() string) counter {
    // ...
    return stats
}
```

In the previous exercise, the goroutine went through the words and counted the digits, while the outer function received the counts and updated the counter:

```
┌───────────────┐
│ loops through │               ┌────────────────┐
│ words and     │ → (counted) → │ fills stats    │
│ counts digits │               └────────────────┘
└───────────────┘
  goroutine          channel      outer function
```

For more complex tasks, it's useful to have a goroutine for reading data (_reader_) and another for processing data (_worker_). Let's use this approach in our function:

```
┌───────────────┐               ┌───────────────┐
│ sends words   │               │ counts digits │               ┌────────────────┐
│ to be counted │ → (pending) → │ in words      │ → (counted) → │ fills stats    │
│               │               │               │               └────────────────┘
└───────────────┘               └───────────────┘
  reader             channel      worker             channel      outer function
```

Do the following:

- Start a goroutine that fetches words from the generator and sends them to the `pending` channel (reader).
- Start a second goroutine that reads from `pending`, counts the digits, and writes to the `counted` channel (worker).
- In the outer function, read from `counted` and update the final `stats` counter.

Submit only the code fragment marked with "solution start" and "solution end" comments. The full source code is available via the "Playground" link below.

```
// solution start

// countDigitsInWords counts the number of digits in words,
// fetching the next word with next().
func countDigitsInWords(next func() string) counter {
    pending := make(chan string)
    counted := make(chan pair)

    // sends words to be counted
    go func() {
        // Fetch words from the generator
        // and send them to the pending channel.
    }()

    // counts digits in words
    go func() {
        // Read the words from the pending channel,
        // count the number of digits in each word,
        // and send the results to the counted channel.
    }()

    // Read values from the counted channel
    // and fill stats.

    // As a result, stats should contain words
    // and the number of digits in each.

    return stats
}

// solution end
```

## ✎ Named goroutines

We are working with a function that counts digits in words:

```
// countDigitsInWords counts the number of digits in words,
// fetching the next word with next().
func countDigitsInWords(next func() string) counter {
    // ...
    return stats
}
```

After splitting the logic between the reader and the worker, the function became quite hefty:

```
func countDigitsInWords(next func() string) counter {
    // ...

    // sends words to be counted
    go func() {
        // ...
    }()

    // counts digits in words
    go func() {
        // ...
    }()

    // fills stats
    // ...

    return stats
}
```

There are clearly three logical blocks:

1. Send words to be counted.
2. Count digits in words.
3. Fill in the final results.

It would be convenient to extract these blocks into separate functions that exchange data through channels:

```
func countDigitsInWords(next func() string) counter {
    pending := make(chan string)
    go submitWords(next, pending)

    counted := make(chan pair)
    go countWords(pending, counted)

    return fillStats(counted)
}
```

Refactor the program to achieve this.

Submit only the code fragment marked with "solution start" and "solution end" comments. The full source code is available via the "Playground" link below.

```
// solution start

// submitWords sends words to be counted.

// countWords counts digits in words.

// fillStats prepares the final statistics.

// solution end
```

## Output channel

Here is the function we ended up with:

```
// countDigitsInWords counts the number of digits in words,
// fetching the next word with next().
func countDigitsInWords(next func() string) counter {
    pending := make(chan string)
    go submitWords(next, pending)

    counted := make(chan pair)
    go countWords(pending, counted)

    return fillStats(counted)
}
```

It looks fine, but there's still one thing we might change.

The `pending` channel is created in the parent function only to be passed to the child `submitWords` function. It'd be better to create the channel in `submitWords` and return it to the parent, so that `submitWords` owns it completely. The same goes for the `counted` channel and `countWords`.

Then `countDigitsInWords` will look like this:

```
func countDigitsInWords(next func() string) counter {
    pending := submitWords(next)
    counted := countWords(pending)
    return fillStats(counted)
}
```

Now the ownership is clear, and the program is easier to reason about. But where did all the goroutines go? We start them inside `submitWords` and `countWords` like this:

```
// submitWords sends words to be counted.
func submitWords(next func() string) chan string {
    out := make(chan string)
    go func() {
        for {
            word := next()
            out <- word
            if word == "" {
                break
            }
        }
    }()
    return out
}

// countWords counts digits in words.
func countWords(in chan string) chan pair {
    out := make(chan pair)
    go func() {
        for {
            word := <-in
            count := countDigits(word)
            out <- pair{word, count}
            if word == "" {
                break
            }
        }
    }()
    return out
}
```

Let's make sure the program still works as expected:

```
func main() {
    phrase := "0ne 1wo thr33 4068"
    next := wordGenerator(phrase)
    stats := countDigitsInWords(next)
    printStats(stats)
}
```

```
thr33: 2
4068: 4
0ne: 1
1wo: 1
```

> This approach has a downside: `submitWords` and `countWords` are now more complex and start goroutines. On the other hand, `countDigitsInWords` is simpler and more robust (especially if we make the channels directed, which we'll discuss later in the book). Which option you choose depends on your preferences, but you definitely shouldn't mix the two methods.

Returning an output channel from a function and filling it within an internal goroutine is a common pattern in Go.

## Keep it up

We learned about goroutines and channels in Go. There is still a long way to go, but we've made a start! In the next chapter we'll dive into [channels](app://obsidian.md/go-concurrency/channels/).

[Pre-order for $10](https://antonz.gumroad.com/l/go-concurrency)   or [read online](app://obsidian.md/go-concurrency/)

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.