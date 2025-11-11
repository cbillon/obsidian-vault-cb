---
link: https://antonz.org/go-concurrency/race-conditions/
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Keep the system state correct by any means necessary.
twitter: https://twitter.com/@ohmypy
slurped: 2025-11-11T09:56
title: "Gist of Go: Race conditions"
---

_This is a chapter from my book on [Go concurrency](app://obsidian.md/go-concurrency), which teaches the topic from the ground up through interactive examples._

Preventing data races with mutexes may sound easy, but dealing with race conditions is a whole other matter. Let's learn how to handle these beasts!

[Race condition](#race-condition) • [Compare-and-set](#compare-and-set) • [Idempotence and atomicity](#idempotence-and-atomicity) • [Locker](#locker) • [TryLock](#trylock) • [Shared nothing](#shared-nothing) • [Keep it up](#keep-it-up)

## Race condition

Let's say we're keeping track of the money in users' accounts:

```
// Accounts - money in users' accounts.
type Accounts struct {
    bal map[string]int
    mu  sync.Mutex
}

// NewAccounts creates a new set of accounts.
func NewAccounts(bal map[string]int) *Accounts {
    return &Accounts{bal: maps.Clone(bal)}
}
```

We can check the balance by username or change the balance:

```
// Get returns the user's balance.
func (a *Accounts) Get(name string) int {
    a.mu.Lock()
    defer a.mu.Unlock()
    return a.bal[name]
}

// Set changes the user's balance.
func (a *Accounts) Set(name string, amount int) {
    a.mu.Lock()
    defer a.mu.Unlock()
    a.bal[name] = amount
}
```

Account operations — `Get` and `Set` — are concurrent-safe, thanks to the mutex.

There's also a store that sells Lego sets:

```
// A Lego set.
type LegoSet struct {
    name  string
    price int
}
```

Alice has 50 coins in her account. She wants to buy two sets: "Castle" for 40 coins and "Plants" for 20 coins:

```
func main() {
    acc := NewAccounts(map[string]int{
        "alice": 50,
    })
    castle := LegoSet{name: "Castle", price: 40}
    plants := LegoSet{name: "Plants", price: 20}

    var wg sync.WaitGroup

    // Alice buys a castle.
    wg.Go(func() {
        balance := acc.Get("alice")
        if balance < castle.price {
            return
        }
        time.Sleep(5 * time.Millisecond)
        acc.Set("alice", balance-castle.price)
        fmt.Println("Alice bought the castle")
    })

    // Alice buys plants.
    wg.Go(func() {
        balance := acc.Get("alice")
        if balance < plants.price {
            return
        }
        time.Sleep(10 * time.Millisecond)
        acc.Set("alice", balance-plants.price)
        fmt.Println("Alice bought the plants")
    })

    wg.Wait()

    balance := acc.Get("alice")
    fmt.Println("Alice's balance:", balance)
}
```

```
Alice bought the castle
Alice bought the plants
Alice's balance: 30
```

What a twist! Not only did Alice buy both sets for a total of 60 coins (even though she only had 50 coins), but she also ended up with 30 coins left! Great deal for Alice, not so great for us.

The problem is that checking and updating the balance is not an atomic operation:

```
// body of the second goroutine
balance := acc.Get("alice")             // (1)
if balance < plants.price {             // (2)
    return
}
time.Sleep(10 * time.Millisecond)
acc.Set("alice", balance-plants.price)  // (3)
```

At point ➊, we see a balance of 50 coins (the first goroutine hasn't done anything yet), so the check at ➋ passes. By point ➌, Alice has already bought the castle (the first goroutine has finished), so her actual balance is 10 coins. But we don't know this and still think her balance is 50 coins. So at point ➌, Alice buys the plants for 20 coins, and the balance becomes 30 coins (the "assumed" balance of 50 coins minus the 20 coins for the plants = 30 coins).

Individual actions on the balance are safe (there's no _data race_). However, balance reads/writes from different goroutines can get "mixed up", leading to an incorrect final balance. This situation is called a _race condition_.

You can't fully eliminate uncertainty in a concurrent environment. Events will happen in an unpredictable order — that's just how concurrency works. However, you can protect the system's state — in our case, the purchased sets and balance — so it stays correct no matter what order things happen in.

Let's check and update the balance in one atomic operation, protecting the entire purchase with a mutex. This way, purchases are processed strictly sequentially:

```
// Shared mutex.
var mu sync.Mutex

// Alice buys a castle.
wg.Go(func() {
    // Protect the entire purchase with a mutex.
    mu.Lock()
    defer mu.Unlock()

    balance := acc.Get("alice")
    if balance < castle.price {
        return
    }
    time.Sleep(5 * time.Millisecond)
    acc.Set("alice", balance-castle.price)
    fmt.Println("Alice bought the castle")
})

// Alice buys plants.
wg.Go(func() {
    // Protect the entire purchase with a mutex.
    mu.Lock()
    defer mu.Unlock()

    balance := acc.Get("alice")
    if balance < plants.price {
        return
    }
    time.Sleep(10 * time.Millisecond)
    acc.Set("alice", balance-plants.price)
    fmt.Println("Alice bought the plants")
})
```

```
Alice bought the plants
Alice's balance: 30
```

One of the goroutines will run first, lock the mutex, check and update the balance, then unlock the mutex. Only after that will the second goroutine be able to lock the mutex and make its purchase.

We still can't be sure which purchase will happen — it depends on the order the goroutines run. But now we are certain that Alice won't buy more than she's supposed to, and the final balance will be correct:

```
Alice bought the castle
Alice's balance: 10
```

Or:

```
Alice bought the plants
Alice's balance: 30
```

To reiterate:

- A _data race_ happens when multiple goroutines access shared data, and at least one of them modifies it. We need to protect the data from this kind of concurrent access.
- A _race condition_ happens when an unpredictable order of operations leads to an incorrect system state. In a concurrent environment, we can't control the exact order things happen. Still, we need to make sure that no matter the order, the system always ends up in the correct state.

Go's race detector can find data races, but it doesn't catch race conditions. It's always up to the programmer to prevent race conditions.

## Compare-and-set

Let's go back to the situation with the race condition before we added the mutex:

```
// Alice's balance = 50 coins.
// Castle price = 40 coins.
// Plants price = 20 coins.

// Alice buys a castle.
wg.Go(func() {
    balance := acc.Get("alice")
    if balance < castle.price {
        return
    }
    time.Sleep(5 * time.Millisecond)
    acc.Set("alice", balance-castle.price)
    fmt.Println("Alice bought the castle")
})

// Alice buys plants.
wg.Go(func() {
    balance := acc.Get("alice")
    if balance < plants.price {
        return
    }
    time.Sleep(10 * time.Millisecond)
    acc.Set("alice", balance-plants.price)
    fmt.Println("Alice bought the plants")
})
```

```
Alice bought the castle
Alice bought the plants
Alice's balance: 30
```

As we discussed, the reason for the incorrect final state is that buying a set (checking and updating the balance) is not an atomic operation:

```
// body of the second goroutine
balance := acc.Get("alice")             // (1)
if balance < plants.price {             // (2)
    return
}
time.Sleep(10 * time.Millisecond)
acc.Set("alice", balance-plants.price)  // (3)
```

At point ➊, we see a balance of 50 coins, so the check at ➋ passes. By point ➌, Alice has already bought the castle, so her actual balance is 10 coins. But we don't know this and still think her balance is 50 coins. So at point ➌, Alice buys the plants for 20 coins, and the balance becomes 30 coins (the "assumed" balance of 50 coins minus the 20 coins for the plants = 30 coins).

To solve the problem, we can protect the entire purchase with a mutex, just like we did before. But there's another way to handle it.

We can keep two separate operations (checking and updating the balance), but instead of a regular update (set), use an atomic _compare-and-set_ operation:

```
// Get returns the user's balance.
func (a *Accounts) Get(name string) int {
    a.mu.Lock()
    defer a.mu.Unlock()
    return a.bal[name]
}

// CompareAndSet changes the user's balance to new
// if the current value equals old. Returns false otherwise.
func (a *Accounts) CompareAndSet(name string, old, new int) bool {
    a.mu.Lock()
    defer a.mu.Unlock()
    if a.bal[name] != old {
        return false
    }
    a.bal[name] = new
    return true
}
```

`CompareAndSet` first checks if the balance has changed compared to the old value provided by the caller (the "assumed" balance). If the balance has changed (the assumed balance doesn't match the actual balance), it doesn't set the new value and returns `false`. If the balance hasn't changed (the assumed balance matches the actual balance), it sets the new value and returns `true`.

Now we can safely sell Lego:

```
// Alice buys a castle.
wg.Go(func() {
    balance := acc.Get("alice")
    if balance < castle.price {
        return
    }
    time.Sleep(5 * time.Millisecond)
    if acc.CompareAndSet("alice", balance, balance-castle.price) {
        fmt.Println("Alice bought the castle")
    }
})

// Alice buys plants.
wg.Go(func() {
    balance := acc.Get("alice")
    if balance < plants.price {
        return
    }
    time.Sleep(10 * time.Millisecond)
    if acc.CompareAndSet("alice", balance, balance-plants.price) {
        fmt.Println("Alice bought the plants")
    }
})
```

```
Alice bought the castle
Alice's balance: 10
```

We no longer use a mutex to protect the entire purchase. Individual operations from different goroutines can get mixed up, but using compare-and-set instead of a regular update protects us from the race condition. If the actual account state doesn't match what we expect (because another goroutine made a change), the update won't happen.

**CAS with retry**

In practice, a failed compare-and-set is often followed by a retry. In our example, we would re-read the balance with `acc.Get` and, if there is still enough money, retried the purchase with `acc.CompareAndSet`:

```
// Alice buys plants (with retries).
wg.Go(func() {
    for { // Start of retry loop.
        balance := acc.Get("alice")
        if balance < plants.price {
            return // Not enough money, exit loop.
        }
        // Attempt the purchase.
        if acc.CompareAndSet("alice", balance, balance-plants.price) {
            fmt.Println("Alice bought the plants")
            return // Success, exit loop.
        }
        // Wait a bit before trying again.
        time.Sleep(time.Millisecond)
        // It's also a good idea to limit the number of retries
        // (not shown here for simplicity) - you don't want
        // the program to get stuck in an infinite loop.
    }
})
```

This approach helps the program handle occasional goroutine conflicts over a shared resource.

Compare-and-set is often used in concurrent programming. It comes in different flavors, such as:

```
// CompareAndSet changes the value to new if the current value equals old.
// Returns true if the value was changed.
CompareAndSet(old, new any) bool

// CompareAndSwap changes the value to new if the current value equals old.
// Returns the old value.
CompareAndSwap(old, new any) any

// CompareAndDelete deletes the value if the current value equals old.
// Returns true if the value was deleted.
CompareAndDelete(old any) bool

// etc
```

The idea is always the same:

- Check if the assumed (old) state matches reality.
- If it does, change the state to new.
- If not, do nothing.

> Go's standard library provides compare-and-swap operations for basic types like `bool` and `int64`. We'll cover these in another chapter.

If compare-and-set doesn't work for your situation, you can always use a regular shared mutex instead. It can protect any sequence of operations, no matter how complex.

**✎ Exercise: Concurrent map + 1 more**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Idempotence and atomicity

Idempotence means that calling an operation on an object multiple times doesn't cause any changes or errors after the first time.

Let's say we have a resource that need to be released after use:

```
// resource that needs to be freed after use.
type resource struct {
    // ...
}

// free releases the resource.
// Panics if the resource has already been released.
func (r *resource) free() {
    // ...
}
```

And there's a worker that frees up resources when closed:

```
// Worker does some work and
// frees resources when closed.
type Worker struct {
    res resource
}

// Work performs some work.
func (w *Worker) Work() {
    // ...
}

// Close frees resources
// and shuts down the worker.
func (w *Worker) Close() {
    w.res.free()
}
```

Everything works fine until we call `Close` twice:

```
func main() {
    w := new(Worker)
    w.Work()
    w.Close()
    w.Close()
    fmt.Println("worker closed")
}
```

```
panic: resource is already freed
```

Let's see how to make `Close` idempotent so we can safely release the resources.

### Boolean flag

Let's add a `closed` flag to the worker and check it in the `Close` method:

```
// Worker does some work and
// frees resources when closed.
type Worker struct {
    res    resource
    closed bool
}

// Close frees resources
// and shuts down the worker.
func (w *Worker) Close() {
    // Ignore repeated calls.
    if w.closed {    // (1)
        return
    }

    w.res.free()     // (2)
    w.closed = true  // (3)
}
```

Now, calling `Close` multiple times works fine:

```
func main() {
    w := new(Worker)
    w.Close()
    w.Close()
    fmt.Println("worker closed")
}
```

But what happens if we call `Close` simultaneously from different goroutines?

```
func main() {
    w := new(Worker)

    var wg sync.WaitGroup
    wg.Go(w.Close)
    wg.Go(w.Close)
    wg.Wait()

    fmt.Println("worker closed")
}
// panic: resource is already freed
```

```
panic: resource is already freed
```

> You won't see panic here often. But it's still there, lurking in the dark.

Panic! You already know why this happens — the check for `w.closed` ➊ and the following resource cleanup ➋ ➌ aren't atomic. Since goroutines run concurrently, they both pass the if statement and call `w.res.free()`. Then, one of the goroutines panics.

We need a structure that ensures atomicity and idempotence in a concurrent environment.

### select

Why don't we use a `closed` channel instead of a boolean flag, and use `select` to close the channel only once?

```
// Worker does some work and
// frees resources when closed.
type Worker struct {
    res    resource
    closed chan struct{}
}

// NewWorker creates a new worker.
// We need an explicit NewWorker constructor instead of
// new(Worker), because the default value for a channel
// is nil, which doesn't work for our needs.
func NewWorker() *Worker {
    return &Worker{closed: make(chan struct{})}
}

// Close frees resources
// and shuts down the worker.
func (w *Worker) Close() {
    select {
    case <-w.closed:
        // Ignore repeated calls.
        return
    default:
        w.res.free()
        close(w.closed)
    }
}
```

We call `Close` from two goroutines and get "worker closed". Everything works fine. Then we deploy this code to production, and a week later we get a bug report saying the app sometimes crashes with the "resource is already freed" panic. What's wrong?

The thing is, select _does not_ protect against freeing resources more than once. As we know, select is safe for concurrent use. So, if two goroutines call `Close` at the same time, they both enter the select and have to pick a case. Since the `closed` channel isn't closed yet, both goroutines choose the default case. Both call `w.res.free()`, and one of them panics.

The chances of this happening are pretty low. You could call `Close` 999 times without any issues, but on the thousandth try, the stars will align just right, both goroutines will hit the default case, and you will get a panic.

It's especially frustrating that the race detector doesn't always catch this kind of issues. In the example above, the race detector might not find anything (depends on the Go version), especially if you comment out `w.res.free()`. But the race condition is still there, and closing the channel more than once will eventually cause a panic.

Select is not atomic. Choosing a select case and running its body are separate actions, not a single atomic operation. So, if the code inside the case changes shared data, select can cause a race condition. It's important to keep this in mind.

### Mutex

Let's go back to the `closed` boolean flag, but this time protect it with a mutex:

```
// Worker does some work and
// frees resources when closed.
type Worker struct {
    res    resource
    mu     sync.Mutex
    closed bool
}

// Close frees resources
// and shuts down the worker.
func (w *Worker) Close() {
    // Make the method atomic.
    w.mu.Lock()
    defer w.mu.Unlock()

    // Ignore repeated calls.
    if w.closed {
        return
    }

    w.res.free()
    w.closed = true
}
```

The mutex stays locked for the entire scope of the `Close` method. This ensures that no matter how many goroutines call `Close` simultaneously, only one can execute its body at a time.

Checking the `closed` flag, freeing resources, and updating the `closed` state all happen as one atomic operation. This makes `Close` idempotent, so you won't get any panics when releasing resources.

A mutex (or something similar) is the only way to make sure a complex operation is atomic in a concurrent environment. Do not rely on select in these situations.

> Speaking of "something similar", for this specific use case — making sure something happens exactly once — Go's standard library provides a handy `sync.Once` type. We'll cover it in another chapter.

**✎ Exercise: Spot the race**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

## Locker

Here are the mutex methods we learned about in the last chapter:

```
type Mutex struct {
    // internal state
}

func (m *Mutex) Lock()
func (m *Mutex) Unlock()
```

```
type RWMutex struct {
    // internal state
}

func (rw *RWMutex) Lock()
func (rw *RWMutex) Unlock()

func (rw *RWMutex) RLock()
func (rw *RWMutex) RUnlock()
```

As you can see, both types have the same `Lock` and `Unlock` methods. Go's standard library provides a common interface for them:

```
// A Locker represents an object that
// can be locked and unlocked.
type Locker interface {
    Lock()
    Unlock()
}
```

If the "locking mechanism" is defined by the client, and your code just needs to lock or unlock access to shared data, use `Locker` instead of a specific type:

```
// ArrayList is a concurrent-safe dynamic array.
type ArrayList struct {
    vals []any
    lock sync.Locker
}

// NewArrayList creates a new empty array.
func NewArrayList(lock sync.Locker) *ArrayList {
    if lock == nil {
        panic("NewArrayList: lock cannot be nil")
    }
    return &ArrayList{vals: []any{}, lock: lock}
}

// Len returns the length of the array.
func (al *ArrayList) Len() int {
    al.lock.Lock()
    defer al.lock.Unlock()
    return len(al.vals)
}

// Append adds an element to the array.
func (al *ArrayList) Append(val any) {
    al.lock.Lock()
    defer al.lock.Unlock()
    al.vals = append(al.vals, val)
}
```

This way, the client can use `Mutex`, `RWMutex`, or any other implementation they prefer:

```
func main() {
    var lock sync.Mutex
    list := NewArrayList(&lock)

    // Add 400 elements to the array using 4 goroutines.
    var wg sync.WaitGroup
    for range 4 {
        wg.Go(func() {
            for range 100 {
                list.Append(rand.IntN(100))
                time.Sleep(time.Millisecond)
            }
        })
    }
    wg.Wait()

    fmt.Println("list length =", list.Len())
}
```

By using `sync.Locker`, you can build components that don't depend on a specific lock implementation. This lets the client decide which lock to use. While you probably won't need this very often, it's a useful feature to be aware of.

## TryLock

Let's say our program needs to call a legacy system, represented by the `External` type. This system is so ancient that it can handle no more than one call at a time. That's why we protect it with a mutex:

```
// External is a client for an external system.
type External struct {
    lock sync.Mutex
}

// Call calls the external system.
func (e *External) Call() {
    e.lock.Lock()
    defer e.lock.Unlock()
    // Simulate a remote call.
    time.Sleep(100 * time.Millisecond)
}
```

Now, no matter how many goroutines try to access the external system at the same time, they'll have to take turns:

```
func main() {
    const nCalls = 4
    ex := new(External)
    start := time.Now()

    var wg sync.WaitGroup
    for range nCalls {
        wg.Go(func() {
            ex.Call()
            fmt.Println("success")
        })
    }
    wg.Wait()

    fmt.Printf(
        "%d calls took %d ms\n",
        nCalls, time.Since(start).Milliseconds(),
    )
}
```

```
success
success
success
success
4 calls took 400 ms
```

Everything looks good on paper. But in reality, if there are a lot of these goroutines, the external system will be constantly busy handling all these sequential calls. This could end up being too much for it. So, let's change the approach:

- If a goroutine needs to call an external system,
- and that system is already busy,
- then the goroutine shouldn't wait,
- but should immediately return an error.

We can use the `TryLock` method of a mutex to implement this logic:

```
// External is a client for an external system.
type External struct {
	lock sync.Mutex
}

// Call calls the external system.
func (e *External) Call() error {
	if !e.lock.TryLock() {
		return errors.New("busy")  // (1)
	}
	defer e.lock.Unlock()
	// Simulate a remote call.
	time.Sleep(100 * time.Millisecond)
	return nil
}
```

`TryLock` tries to lock the mutex, just like a regular `Lock`. But if it can't, it returns `false` right away instead of blocking the goroutine. This way, we can immediately return an error at ➊ instead of waiting for the system to become available.

Now, out of four simultaneous calls, only one will go through. The others will get a "busy" error:

```
func main() {
	const nCalls = 4
    ex := new(External)
	start := time.Now()

	var wg sync.WaitGroup
	for range nCalls {
		wg.Go(func() {
			err := ex.Call()
			if err != nil {
				fmt.Println(err)
			} else {
				fmt.Println("success")
			}
		})
	}
	wg.Wait()

	fmt.Printf(
		"%d calls took %d ms\n",
		nCalls, time.Since(start).Milliseconds(),
	)
}
```

```
busy
busy
busy
success
4 calls took 100 ms
```

According to the standard library docs, `TryLock` is rarely needed. In fact, using it might mean there's a problem with your program's design. For example, if you're calling `TryLock` in a busy-wait loop ("keep trying until the resource is free") — that's usually a bad sign:

```
for {
    if mutex.TryLock() {
        // Use the shared resource.
        mutex.Unlock()
        break
    }
}
```

This code will keep one CPU core at 100% usage until the mutex is unlocked. It's much better to use a regular `Lock` so the scheduler can take the blocked goroutine off the CPU.

**✎ Exercise: Rate limiter**

Practice is crucial in turning abstract knowledge into skills, making theory alone insufficient. The full version of the book contains a lot of exercises — that's why I recommend [getting it](https://antonz.gumroad.com/l/go-concurrency).

If you are okay with just theory for now, let's continue.

Let's go back one last time to Alice and the Lego sets we started the chapter with.

We manage user accounts:

```
// Accounts - money in users' accounts.
type Accounts struct {
	bal map[string]int
	mu  sync.Mutex
}

// NewAccounts creates a new set of accounts.
func NewAccounts(bal map[string]int) *Accounts {
	return &Accounts{bal: maps.Clone(bal)}
}

// Get returns the user's balance.
func (a *Accounts) Get(name string) int {
	a.mu.Lock()          // (1)
	defer a.mu.Unlock()
	return a.bal[name]
}

// Set changes the user's balance.
func (a *Accounts) Set(name string, amount int) {
	a.mu.Lock()          // (2)
	defer a.mu.Unlock()
	a.bal[name] = amount
}
```

And handle purchases:

```
acc := NewAccounts(map[string]int{
    "alice": 50,
})
castle := LegoSet{name: "Castle", price: 40}
plants := LegoSet{name: "Plants", price: 20}

// Shared mutex.
var mu sync.Mutex

// Alice buys a castle.
wg.Go(func() {
    // Protect the entire purchase with a mutex.
    mu.Lock()            // (3)
    defer mu.Unlock()

    // Check and update the balance.
})

// Alice buys plants.
wg.Go(func() {
    // Protect the entire purchase with a mutex.
    mu.Lock()            // (4)
    defer mu.Unlock()

    // Check and update the balance.
})
```

This isn't a very complex use case — I'm sure you've seen worse. Still, we had to put in some effort:

- Protect the balance with a mutex to prevent a data race ➊ ➋.
- Protect the entire purchase operation with a mutex (or use compare-and-set) to make sure the final state is correct ➌ ➍.

We were lucky to notice and prevent the race condition during a purchase. What if we had missed it?

There's another approach to achieving safe concurrency: instead of protecting shared state when working with multiple goroutines, we can avoid shared state altogether. Channels can help us do this.

Here's the idea: we'll create a `Processor` function that accepts purchase requests through an input channel, processes them, and sends the results back through an output channel:

```
// A purchase request.
type Request struct {
    buyer string
    set   LegoSet
}

// A purchase result.
type Purchase struct {
    buyer   string
    set     LegoSet
    succeed bool
    balance int // balance after purchase
}

// Processor handles purchases.
func Processor(acc map[string]int) (chan<- Request, <-chan Purchase) {
    // ...
}
```

Buyer goroutines will send requests to the processor's input channel and receive results (successful or failed purchases) from the output channel:

```
func main() {
    const buyer = "Alice"
    acc := map[string]int{buyer: 50}

    wishlist := []LegoSet{
        {name: "Castle", price: 40},
        {name: "Plants", price: 20},
    }

    reqs, purs := Processor(acc)

    // Alice buys stuff.
    var wg sync.WaitGroup
    for _, set := range wishlist {
        wg.Go(func() {
            reqs <- Request{buyer: buyer, set: set}
            pur := <-purs
            if pur.succeed {
                fmt.Printf("%s bought the %s\n", pur.buyer, pur.set.name)
                fmt.Printf("%s's balance: %d\n", buyer, pur.balance)
            }
        })
    }
    wg.Wait()
}
```

```
Alice bought the Plants
Alice's balance: 30
```

This approach offers several benefits:

- Buyer goroutines send their requests and get results without worrying about how the purchase is done.
- All the buying logic is handled inside the processor goroutine.
- No need for mutexes.

All that's left is to implement the processor. How about this:

```
// Processor handles purchases.
func Processor(acc map[string]int) (chan<- Request, <-chan Purchase) {
    in := make(chan Request)
    out := make(chan Purchase)
    acc = maps.Clone(acc)

    go func() {
        for {
            // Receive the purchase request.
            req := <-in

            // Handle the purchase.
            balance := acc[req.buyer]
            pur := Purchase{buyer: req.buyer, set: req.set, balance: balance}
            if balance >= req.set.price {
                pur.balance -= req.set.price
                pur.succeed = true
                acc[req.buyer] = pur.balance
            } else {
                pur.succeed = false
            }

            // Send the result.
            out <- pur
        }
    }()

    return in, out
}
```

> It would have been a good idea to add a way to stop the processor using context, but I decided not to do it to keep the code simple.

The processor clones the original account states and works with its own copy. This approach makes sure there is no concurrent access to the accounts, so there are no races. Of course, we should avoid running two processors at the same time, or we could end up with two different versions of the truth.

It's not always easy to structure a program in a way that avoids shared state. But if you can, it's a good option.

## Keep it up

Now you know how to protect shared data (from data races) and sequences of operations (from race conditions) in a concurrent environment using mutexes. Be careful with them and always test your code thoroughly with the race detector enabled.

Use code reviews, because the race detector doesn't catch every data race and can't detect race conditions at all. Having someone else look over your code can be really helpful.

In the next chapter, we'll talk about [semaphores](app://obsidian.md/semaphores/).

[Pre-order for $10](https://antonz.gumroad.com/l/go-concurrency)   or [read online](app://obsidian.md/go-concurrency/)

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.