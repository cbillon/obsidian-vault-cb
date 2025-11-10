---
link: https://www.freecodecamp.org/news/how-to-use-closures-in-go
byline: Gabor Koos
site: freeCodeCamp.org
date: 2025-10-27T20:35
excerpt: "If you've written code in JavaScript, Python, or Rust, you've probably heard the word closure before. The concept has subtle differences in each language, but the core idea is the same: a closure is a function that captures variables from its surroun..."
twitter: https://twitter.com/@gkoos430
slurped: 2025-11-09T09:32
title: How to Use Closures in Go
tags:
  - go
  - closures
---
In this article, we'll explore how closures work in Go, a statically typed language known for its simplicity and efficiency. We'll look at how to create closures, how they capture variables, and some practical use cases.

## What We'll Cover

- [Prerequisites](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-prerequisites)
    
- [What Closures Really Are in Go](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-what-closures-really-are-in-go)
    
    - [How Go Makes This Work](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-how-go-makes-this-work)
        
    - [Multiple Independent Closures](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-multiple-independent-closures)
        
- [The Classic Loop Trap](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-the-classic-loop-trap)
    
- [How to Create Closures in Go](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-how-to-create-closures-in-go)
    
    - [Returning Closures From Functions](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-returning-closures-from-functions)
        
    - [Named Inner Functions](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-named-inner-functions)
        
    - [Inline Closures in Loops or Goroutines](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-inline-closures-in-loops-or-goroutines)
        
    - [Closures With Parameters](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-closures-with-parameters)
        
    - [Capturing Multiple Variables](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-capturing-multiple-variables)
        
    - [Closures in Structs](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-closures-in-structs)
        
    - [Note on Method Receivers](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-note-on-method-receivers)
        
    - [Key Takeaways](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-key-takeaways)
        
- [Closures and Concurrency](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-closures-and-concurrency)
    
    - [Independent State Across Goroutines](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-independent-state-across-goroutines)
        
    - [Capturing Shared Variables Safely](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-capturing-shared-variables-safely)
        
- [Practical Patterns with Closures](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-practical-patterns-with-closures)
    
    - [Memoization / Caching](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-memoization-caching)
        
    - [Event Handlers / Callbacks](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-event-handlers-callbacks)
        
    - [Encapsulated Pipelines / Producers](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-encapsulated-pipelines-producers)
        
    - [Deferred Execution with Captured State](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-deferred-execution-with-captured-state)
        
    - [How to Implement Interfaces Dynamically](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-how-to-implement-interfaces-dynamically)
        
    - [Key Takeaways](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-key-takeaways-1)
        
- [How Closures Affect Memory and Performance](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-how-closures-affect-memory-and-performance)
    
    - [Variables May Live Longer Than Expected](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-variables-may-live-longer-than-expected)
        
    - [Many Closures Can Add Overhead](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-many-closures-can-add-overhead)
        
- [How to Test and Debug Closures](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-how-to-test-and-debug-closures)
    
    - [Isolate the Closure](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-isolate-the-closure)
        
    - [Check Captured Variables](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-check-captured-variables)
        
    - [Use Logging or Debug Prints](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-use-logging-or-debug-prints)
        
    - [Test Concurrency Carefully](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-test-concurrency-carefully)
        
    - [Key Takeaways](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-key-takeaways-2)
        
- [Best Practices and Takeaways for Using Closures in Go](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-best-practices-and-takeaways-for-using-closures-in-go)
    
- [Conclusion](https://www.freecodecamp.org/news/how-to-use-closures-in-go#heading-conclusion)
    

## Prerequisites

To follow along with this article, you should have a basic understanding of Go programming, including functions and variable scope. If you're new to Go, consider checking out the [official Go tour](https://tour.golang.org/) to get up to speed.

## What Closures Really Are in Go

At its simplest, a closure in Go is a function that **references variables defined outside of it**. That may sound abstract, so let's start with an example you can run right away:

```
package main

import "fmt"

func counter() func() int {
    n := 0
    return func() int {
        n++
        return n
    }
}

func main() {
    next := counter()
    fmt.Println(next()) // 1
    fmt.Println(next()) // 2
    fmt.Println(next()) // 3
}
```

When you call `counter()`, it returns another function, but that function **keeps access to the variable** `n` that lived inside counter.

Even though `counter()` has already finished running, `n` hasn't disappeared. Each time you call `next()`, it updates the same `n` that was created during the original `counter()` call.

This is the defining property of a closure:

> A closure "closes over" its environment, keeping the variables it needs alive for as long as the closure itself exists.

### How Go Makes This Work

Normally, local variables in Go live on the **stack**, which is cleared when a function returns.

But if a nested function needs to keep using one of those variables, Go's compiler performs what's called _escape analysis_: it sees that the variable will outlive the function call, so it moves that variable to the **heap**, where it can stay alive as long as something references it - in this case, the closure.

You can actually ask the compiler to show you this process:

```
go build -gcflags="-m" main.go
```

You might see output like:

```
./main.go:6:6: moved to heap: n
```

That tells you the variable `n` "escaped" the stack so the closure could use it safely later.

### Multiple Independent Closures

Each call to a function that returns a closure creates a new, independent environment:

```
a := counter()
b := counter()
fmt.Println(a()) // 1
fmt.Println(a()) // 2
fmt.Println(b()) // 1
```

Here, `a` and `b` are two separate closures, each with its own `n`. Calling `a()` increments its own `n`, while calling `b()` starts from its own separate `n`.

## The Classic Loop Trap

One of the most common surprises for Go developers comes when closures are used inside a loop. Even experienced programmers often fall into this trap.

Consider this example:

```
package main

import "fmt"

func main() {
    funcs := make([]func(), 0)
    for i := 0; i < 3; i++ {
        funcs = append(funcs, func() {
            fmt.Println(i)
        })
    }
    for _, f := range funcs {
        f()
    }
}
```

You might expect this to print `0`, `1`, and `2`, but it actually prints

```
3
3
3
```

**Why Does this Happen?**

Inside the loop, each function literal **captures the variable** `i` itself, not its value at that moment.

The loop reuses the same `i` variable for all iterations. By the time the loop finishes, `i` equals 3, and **all the closures see this same** `i` when they run later.

**How to Fix It**

There are two common idiomatic fixes:

1. Shadow the loop variable:

```
for i := 0; i < 3; i++ {
    i := i // new variable shadows the loop variable
    funcs = append(funcs, func() {
        fmt.Println(i)
    })
}
```

2. Pass the variable as a parameter to an inner function:

```
for i := 0; i < 3; i++ {
    funcs = append(funcs, func(x int) func() {
        return func() { fmt.Println(x) }
    }(i))
}
```

Both approaches create a new variable for each iteration, so each closure captures its own independent value.

There are a few different ways to create closures in Go. Let's explore some common patterns.

### Returning Closures From Functions

The most common pattern is having a function return a closure that keeps its own state:

```
func makeCounter() func() int {
    n := 0
    return func() int {
        n++
        return n
    }
}

c1 := makeCounter()
fmt.Println(c1()) // 1
fmt.Println(c1()) // 2
```

Each call to `makeCounter` creates a new closure with its own `n`, as we saw earlier.

### Named Inner Functions

You can also give a name to a function literal for readability or debugging:

```
func makeCounter() func() int {
    n := 0
    next := func incr() int {
        n++
        return n
    }
    return next
}
```

This works the same way but gives the inner function a name (`incr`), which can be helpful in stack traces. Other than that, it behaves just like an anonymous function.

### Inline Closures in Loops or Goroutines

Closures are often defined inline, especially for loops or goroutines:

```
for i := 0; i < 3; i++ {
    go func(x int) {
        fmt.Println(x)
    }(i)
}
```

Here, we pass `i` as a parameter to the closure, ensuring each goroutine gets its own copy of the value, avoiding the loop variable trap.

### Closures With Parameters

Closures can accept their own arguments:

```
func adder(base int) func(int) int {
    return func(x int) int {
        return base + x
    }
}

add5 := adder(5)
fmt.Println(add5(10)) // 15
```

Here, `adder` returns a closure that adds a fixed `base` value to whatever argument it receives.

### Capturing Multiple Variables

Closures can capture multiple outer variables:

```
func multiplier(factor int) func(int) int {
    offset := 2
    return func(x int) int {
        return x*factor + offset
    }
}

m := multiplier(3)
fmt.Println(m(4)) // 14
```

In this example, the closure captures both `factor` and `offset` from its surrounding scope - `factor` is a parameter, while `offset` is a local variable.

### Closures in Structs

Closures can also be stored in structs, just like any other function value. This is a useful pattern when you want objects with dynamic or stateful behavior.

```
type Counter struct {
    Next func() int
}

func NewCounter() Counter {
    n := 0
    return Counter{
        Next: func() int {
            n++
            return n
        },
    }
}

func main() {
    c := NewCounter()
    fmt.Println(c.Next()) // 1
    fmt.Println(c.Next()) // 2
}
```

Here, the `Next` field holds a closure that captures the variable `n`. Each instance of `Counter` has its own independent state, without needing a separate type or mutex.

This pattern shows how closures can act as lightweight objects: bundling behavior and state together.

### Note on Method Receivers

Closures in Go don't implicitly capture the method receiver like some languages do. If you want a closure to use the receiver inside a method, you typically assign it to a local variable:

```
type Counter struct {
    n int
}

func (c *Counter) MakeIncrementer() func() int {
    r := c // capture receiver explicitly
    return func() int {
        r.n++
        return r.n
    }
}
```

This ensures the closure references the intended receiver rather than introducing unexpected behavior.

Unlike JavaScript or Python, Go closures capture lexical variables, not the implicit `this` or `self`.

### Key Takeaways

- Closures can be returned from functions, named, inlined, or even stored in structs.
    
- They capture outer variables, not copies of their values.
    
- Used this way, closures can replace small types or interfaces for lightweight encapsulation.
    

## Closures and Concurrency

Closures are powerful in Go, but when you combine them with concurrency, their captured variables can act in unexpected ways if you're not careful.

### Independent State Across Goroutines

Each closure keeps its own captured variables alive, even when used in concurrent goroutines:

```
func makeWorker(start int) func() int {
    counter := start
    return func() int {
        counter++
        return counter
    }
}

func main() {
    worker1 := makeWorker(0)
    worker2 := makeWorker(100)

    go func() { fmt.Println(worker1()) }() // prints 1
    go func() { fmt.Println(worker2()) }() // prints 101
}
```

Here, `worker1` and `worker2` have independent `counter` variables, so they don't interfere with each other. Each closure maintains independent state, even in separate goroutines.

### Capturing Shared Variables Safely

When multiple closures share a variable, you must coordinate access. For example:

```
counter := 0
ch := make(chan int)

for i := 0; i < 3; i++ {
    go func() {
        // increments a shared variable
        ch <- 1
    }()
}

// aggregate safely
for i := 0; i < 3; i++ {
    counter += <-ch
}
fmt.Println(counter) // 3
```

The closure captures the outer variable `ch` (a channel), which is safe because channels serialize access. Using a buffered channel here wouldn't change the behavior of the closure: it still captures its own `n` and sends the values to the channel independently.

Closures themselves **don't synchronize shared state, you still need channels or mutexes**.

## Practical Patterns with Closures

Closures in Go aren't just a language curiosity, they're a powerful tool for writing stateful, reusable, and flexible code. Here are a few practical patterns that go beyond the basics.

### Memoization / Caching

Closures can capture an internal map or cache to store results of expensive computations:

```
func memoize(f func(int) int) func(int) int {
    cache := map[int]int{}
    return func(x int) int {
        if val, ok := cache[x]; ok {
            return val
        }
        result := f(x)
        cache[x] = result
        return result
    }
}

func main() {
    fib := memoize(func(n int) int {
        if n <= 1 {
            return n
        }
        return fib(n-1) + fib(n-2)
    })
    fmt.Println(fib(10)) // 55
}
```

Here, the `memoize` function returns a closure that caches results of the Fibonacci function, avoiding redundant calculations.

### Event Handlers / Callbacks

Closures are perfect for defining event handlers or callbacks that need to maintain state:

```
type Button struct {
    onClick func()
}

func (b *Button) Click() {
    if b.onClick != nil {
        b.onClick()
    }
}

func main() {
    count := 0
    button := Button{
        onClick: func() {
            count++
            fmt.Println("Button clicked", count, "times")
        },
    }

    button.Click() // Button clicked 1 times
    button.Click() // Button clicked 2 times
}
```

In this example, the closure captures the `count` variable, allowing the button to keep track of how many times it has been clicked.

### Encapsulated Pipelines / Producers

Closures can wrap stateful logic for channels and pipelines:

```
func producer(start int) func(chan int) {
    n := start
    return func(ch chan int) {
        for i := 0; i < 3; i++ {
            ch <- n
            n++
        }
    }
}

func main() {
    ch := make(chan int, 3)
    go producer(5)(ch)
    for i := 0; i < 3; i++ {
        fmt.Println(<-ch) // 5, 6, 7
    }
}
```

Here, the `producer` function returns a closure that sends a sequence of numbers to a channel, maintaining its own state with `n`.

### Deferred Execution with Captured State

Using a closure with `defer` lets you capture variables at the moment the defer statement is executed, which is especially useful in loops or resource cleanup:

```
func main() {
    for i := 0; i < 3; i++ {
        defer func(x int) {
            fmt.Println(x)
        }(i) // capture current i
    }
}
```

Output:

```
2
1
0
```

Here, each deferred closure captures the value of `i` at the time of the `defer` statement, so they print in reverse order when the function exits.

### How to Implement Interfaces Dynamically

Closures can also be used to implement interfaces without defining a full struct type. For example, a simple function can satisfy a single-method interface:

```
type Greeter interface {
    Greet() string
}

func MakeGreeter(name string) Greeter {
    return struct{ Greeter }{
        Greeter: func() string { return "Hello, " + name },
    }
}

func main() {
    g := MakeGreeter("Alice")
    fmt.Println(g.Greet()) // Hello, Alice
}
```

Here, the closure captures `name`, allowing the returned object to implement the `Greet` method dynamically.

### Key Takeaways

- Closures allow memoization and caching without extra structs.
    
- Storing closures in structs provides customizable behavior for objects.
    
- Closures can encapsulate stateful concurrent pipelines, keeping logic localized and safe.
    
- Closures with `defer` capture variables at the time of deferment, useful for cleanup or logging.
    
- They enable dynamic interface implementations without boilerplate types.
    

## How Closures Affect Memory and Performance

Closures are powerful, but capturing variables from outer scopes has memory and performance implications.

### Variables May Live Longer Than Expected

Because closures keep references to captured variables (and move them to the heap if necessary, as we saw earlier), these variables live as long as the closure itself, which can increase memory usage:

```
func main() {
    bigData := make([]byte, 10_000_000) // 10MB
    f := func() int { return len(bigData) }
    _ = f
}
```

In this example, `bigData` remains in memory as long as the closure `f` exists, even if `bigData` is no longer needed elsewhere.

### Many Closures Can Add Overhead

Each closure carries a small environment for its captured variables. Creating thousands of closures is usually fine, but in high-performance or memory-sensitive code, this can add measurable overhead.

- Captured variables may be heap-allocated.
    
- Each closure has a small hidden struct for its environment.
    

Alternatives include **structs** or **plain functions** when you need maximum efficiency.

## How to Test and Debug Closures

Closures can sometimes behave in unexpected ways when capturing variables or working with concurrency. Here are some tips to test and debug them effectively.

### Isolate the Closure

Test the closure independently of its outer function to verify its behavior:

```
func TestCounter(t *testing.T) {
    counter := makeCounter()
    if counter() != 1 {
        t.Error("expected 1")
    }
    if counter() != 2 {
        t.Error("expected 2")
    }
}
```

This ensures the closure maintains state correctly.

### Check Captured Variables

Remember: closures capture variables by reference, not value. Be mindful of loop variables or shared state:

```
for i := 0; i < 3; i++ {
    i := i // shadow loop variable
    t.Run(fmt.Sprintf("i=%d", i), func(t *testing.T) {
        if i != i { // simplified check
            t.Fail()
        }
    })
}
```

This helps avoid the loop trap in tests.

### Use Logging or Debug Prints

Printing internal closure state is often the fastest way to debug subtle behavior:

```
adder := func(base int) func(int) int {
    return func(x int) int {
        fmt.Printf("base=%d, x=%d\n", base, x)
        return base + x
    }
}
result := adder(5)(10) // logs: base=5, x=10
```

### Test Concurrency Carefully

When closures are used in goroutines, race conditions can creep in. Use the Go race detector:

```
go test -race ./...
```

This flags any shared variable access that isn’t properly synchronized.

### Key Takeaways

- Test closures independently to ensure captured state behaves as expected.
    
- Be cautious with loop variables and shared state.
    
- Use logging and the race detector to debug concurrency issues.
    

## Best Practices and Takeaways for Using Closures in Go

Closures are a versatile feature in Go, but like any tool, they work best when used thoughtfully. Here are some practical guidelines:

- **Encapsulate state cleanly**: Use closures to maintain private state without introducing extra structs or types. Counters, memoization caches, and small factories are common patterns.
    
- **Be careful in loops**: Always capture loop variables correctly to avoid the classic loop trap. Shadowing the variable or passing it as a parameter to the closure are idiomatic solutions.
    
- **Handle concurrency explicitly**: Closures can safely maintain independent state in goroutines, but they do not synchronize shared state automatically. When multiple closures share variables, coordinate access with channels or mutexes.
    
- **Mind memory usage**: Captured variables may escape to the heap, so long-lived closures can retain more memory than expected. Avoid capturing large objects unless necessary.
    
- **Leverage closures in structs**: Storing closures in struct fields allows objects to have dynamic or customizable behavior without extra boilerplate, making your code more flexible.
    

## Conclusion

Closures in Go allow functions to carry state, encapsulate behavior, and interact safely with concurrency patterns, all while keeping your code clean and expressive. By understanding how closures capture variables, how they behave in loops and goroutines, and their memory implications, you can use them confidently to write more idiomatic and maintainable Go code.

---

---

Learn to code for free. freeCodeCamp's open source curriculum has helped more than 40,000 people get jobs as developers. [Get started](https://www.freecodecamp.org/learn/)