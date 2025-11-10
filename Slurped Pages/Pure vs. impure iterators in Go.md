---
link: https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/
date: 2025-05-29T08:00
excerpt: |-
  TL;DR ¶ Go has now standardised iterators. Iterators are powerful. Being functions under the hood, iterators can be closures. The classification of iterators suggested by the documentation is ambiguous. Dividing iterators into two categories, “pure” and “impure”, seems to me preferrable. Whether iterators should be designed as “pure” whenever possible is unclear. The advent of iterators in Go ¶ The iterator pattern was popularised by the classic “Gang of Four” book as
  [providing] a way to access the elements of an aggregate object sequentially without exposing its underlying representation.
tags:
  - go
  - iterator
slurped: 2025-11-06T17:50
title: Pure vs. impure iterators in Go
---

## TL;DR [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#tldr)

- Go has now standardised iterators.
- Iterators are powerful.
- Being functions under the hood, iterators can be closures.
- The classification of iterators suggested by the documentation is ambiguous.
- Dividing iterators into two categories, “pure” and “impure”, seems to me preferrable.
- Whether iterators should be designed as “pure” whenever possible is unclear.

## The advent of iterators in Go [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#the-advent-of-iterators-in-go)

The iterator pattern was popularised by [the classic “Gang of Four” book](https://www.informit.com/store/design-patterns-elements-of-reusable-object-oriented-9780201633610) as

> [providing] a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

Until recently, the data structures over which you could iterate via [a `for`-`range` loop](https://go.dev/ref/spec#For_range) were limited to arrays (either directly or through a pointer), slices, strings, maps, channels, and integers. However, in the wake of [Go 1.18](https://go.dev/doc/go1.18#generics)’s support for [parametric polymorphism](https://en.wikipedia.org/wiki/Parametric_polymorphism) (a.k.a. “generics”), [Go 1.23](https://go.dev/doc/go1.23#iterators) standardised the way of defining custom iterators, saw the addition of the [iter](https://pkg.go.dev/iter) package in the standard library, and welcomed a couple of _iterator factories_ (i.e. functions or methods that return an iterator) in the [slices](https://pkg.go.dev/slices) and [maps](https://pkg.go.dev/maps) packages. And [Go 1.24](https://go.dev/doc/go1.24#minor_library_changes) marked the inception in the standard library of even more iterator factories, such as [`strings.SplitSeq`](https://pkg.go.dev/strings#SplitSeq):

> ```
> // SplitSeq returns an iterator over all substrings of s separated by sep.
> // The iterator yields the same strings that would be returned by Split(s, sep),
> // but without constructing the slice.
> // It returns a single-use iterator.
> func SplitSeq(s, sep string) iter.Seq[string]
> ```

---

If you’re not familiar with the syntax and semantics of iterators in Go 1.23+, I recommend you peruse [Ian Lance Taylor](https://www.airs.com/blog/)’s [introductory post](https://go.dev/blog/range-functions) published on the Go blog.

Once you get past the first impression of bewilderment (“Why so many `func`s?!”), it’s pretty smooth sailing. Moreover, callers of iterators typically are isolated from whatever complexity is involved in their implementation.

---

As a first example, consider the program below ([playground](https://go.dev/play/p/tlFDOBh2jyD)), which features a factory function named `fib0` that produces an iterator over [the sequence of Fibonacci numbers](https://en.wikipedia.org/wiki/Fibonacci_sequence):

```
package main

import (
	"fmt"
	"iter"
)

func fib0() iter.Seq[int] {
	return func(yield func(int) bool) {
		for a, b := 0, 1; yield(a); a, b = b, a+b {
			// deliberately empty
		}
	}
}

func main() {
	for n := range fib0() {
		if n > 100 {
			break
		}
		fmt.Printf("%d ", n)
	}
}
```

As you can expect, the program prints the Fibonacci numbers less than 100 in increasing order:

```
0 1 1 2 3 5 8 13 21 34 55 89 
```

## The power of iterators [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#the-power-of-iterators)

The standardisation of iterators is a welcome addition to the Go language. Iterators indeed provide tangible benefits:

- They promote **flexibility** and **separation of concerns**: callers can remain oblivious about how the sequence is produced and can instead focus on what to do with the data; [an iterator that abstracts the paginated consumption of GitHub’s API](https://blog.thibaut-rousseau.com/blog/writing-testing-a-paginated-api-iterator/) is a good example.
- They encourage **encapsulation** insofar as they expose data as sequences that cannot be mutated like slices and maps can.
- They have the potential to boost **performance**: instead of materialising a data structure containing the entire body of data, they dole out elements one by one only as required by the caller, thereby promising, in many ([though not all](https://go.dev/play/p/rG47tuwY_Ik)) situations, a lower latency and reduced heap allocations; they’re also more performant than iterator implementations relying on channels.
- They allow for **infinite sequences** (e.g. the sequence of prime numbers), an affordance that finite data structures like slices and maps never could provide.

Because iterators are so powerful, they’re likely to mushroom in libraries even beyond Go’s standard library. Therefore, to forestall any confusion in the discourse about iterators, the terminology surrounding them should be as precise as possible.

## Official guidance on iterator classification [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#official-guidance-on-iterator-classification)

The phrase “single-use iterator” may have caught your eye in `strings.SplitSeq`’s documentation. It is explained in [a section](https://pkg.go.dev/iter#hdr-Single_Use_Iterators) of the [iter](https://pkg.go.dev/iter) package:

> Most iterators provide the ability to walk an entire sequence: when called, the iterator does any setup necessary to start the sequence, then calls yield on successive elements of the sequence, and then cleans up before returning. Calling the iterator again walks the sequence again.
> 
> Some iterators break that convention, providing the ability to walk a sequence only once. These “single-use iterators” typically report values from a data stream that cannot be rewound to start over. Calling the iterator again after stopping early may continue the stream, but calling it again after the sequence is finished will yield no values at all. Doc comments for functions or methods that return single-use iterators should document this fact:
> 
> ```
> // Lines returns an iterator over lines read from r.
> // It returns a single-use iterator.
> func (r *Reader) Lines() iter.Seq[string]
> ```

This passage of the documentation seemingly divides iterators into two categories. I’ll attempt to elucidate them through a couple of examples.

### An example of a “pure” iterator [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#an-example-of-a-pure-iterator)

The result of my `fib0` function clearly exemplifies the first category:

> Most iterators provide the ability to walk an entire sequence: when called, the iterator does any setup necessary to start the sequence, then calls yield on successive elements of the sequence, and then cleans up before returning. Calling the iterator again walks the sequence again.

If we assign `fib0`’s result to a variable and then range over that iterator multiple times, we indeed walk the sequence of Fibonacci numbers _from the beginning_ each time:

```
package main

import (
	"fmt"
	"iter"
)

func fib0() iter.Seq[int] {
	return func(yield func(int) bool) {
		for a, b := 0, 1; yield(a); a, b = b, a+b {
			// deliberately empty
		}
	}
}

func main() {
	seq := fib0()
	for n := range seq {
		if n > 10 {
			break
		}
		fmt.Printf("%d ", n)
	}
	fmt.Println()
	for n := range seq {
		if n > 100 {
			break
		}
		fmt.Printf("%d ", n)
	}
	fmt.Println()
	for n := range seq {
		if n > 1000 {
			break
		}
		fmt.Printf("%d ", n)
	}
}
```

Output:

```
0 1 1 2 3 5 8 
0 1 1 2 3 5 8 13 21 34 55 89 
0 1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 
```

In my interpretation of the documentation, the first category of iterators corresponds to _externally [pure functions](https://en.wikipedia.org/wiki/Pure_function)_, i.e. iterators that may internally rely on mutation but display no externally observable side effects. Therefore, “pure” seems to me an apt qualifier for such iterators; “stateless” comes a close second.

## Example of a “single-use” iterator [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#example-of-a-single-use-iterator)

What about the second category?

> Some iterators break that convention, providing the ability to walk a sequence only once. These “single-use iterators” typically report values from a data stream that cannot be rewound to start over. Calling the iterator again after stopping early may continue the stream, but calling it again after the sequence is finished will yield no values at all.

Consider the example below:

```
func fib1() iter.Seq[int] {
	a, b := 0, 1
	return func(yield func(int) bool) {
		for ; yield(a); a, b = b, a+b {
			// deliberately empty body
		}
	}
}
```

At a glance, iterator factory `fib1` seems almost identical to `fib0`, and you would be forgiven to dismiss their difference in implementation as unimportant; in fact, you’d be in [very good company](https://github.com/golang/go/issues/73524#issuecomment-2886195765) if you did. However, the difference between `fib1` and `fib0`, far from being merely cosmetic, actually changes the semantics of their results. To understand why, you need some familiarity with [_closures_](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29).

---

If necessary, refer to the [appendix](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#appendix-refresher-on-closures) to this post for a refresher on closures.

---

Recall that, in Go, iterators are functions; as such, iterators can be closures! In `fib0`, variables `a` and `b` are declared within the resulting iterator. By contrast, in `fib1`, variables `a` and `b` are [_free variables_](https://en.wikipedia.org/wiki/Free_variables_and_bound_variables) of the resulting iterator, which also happens to mutate those variables. Therefore, if we repeatedly range over `fib1`’s result ([playground](https://go.dev/play/p/blZwSxciBqV)) as we did over `fib0`’s result earlier, we observe a very different behaviour:

```
package main

import (
	"fmt"
	"iter"
)

func fib1() iter.Seq[int] {
	a, b := 0, 1
	return func(yield func(int) bool) {
		for ; yield(a); a, b = b, a+b {
			// deliberately empty
		}
	}
}

func main() {
	seq := fib1()
	for n := range seq {
		if n > 10 {
			break
		}
		fmt.Printf("%d ", n)
	}
	fmt.Println()
	for n := range seq {
		if n > 100 {
			break
		}
		fmt.Printf("%d ", n)
	}
	fmt.Println()
	for n := range seq {
		if n > 1000 {
			break
		}
		fmt.Printf("%d ", n)
	}
}
```

Output:

```
0 1 1 2 3 5 8 
13 21 34 55 89 
144 233 377 610 987
```

In plain terms, the iterator “remembers” where in the sequence of Fibonacci numbers the caller last stopped and, when the caller resumes ranging over it, the iterator picks up right where it left off; because the sequence produced by this iterator cannot be rewound but can be resumed, the iterator could perhaps be described as “single-use” but “resumable”.

## A problematic classification [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#a-problematic-classification)

The classification of iterators suggested in the documentation is not ideal, in my opinion.

First, its lack of a term for iterators that I described earlier as “pure” is puzzling. Among all iterators, “pure” iterators arguably distinguish themselves as the easiest ones to reason about; as such, don’t they deserve their own qualifier? A parallel with [dynamical systems](https://go.dev/play/p/usL33VcLdPg) comes to mind: [linear dynamical systems](https://en.wikipedia.org/wiki/Linear_dynamical_system) are singled out from nonlinear ones precisely because they’re regarded as simpler and much easier to apprehend.

Second, I find the description of the second category confusingly imprecise and ambiguous. In particular, whether the term “single-use” is intended to encompass only some or _all_ “impure” iterators is unclear to me… and to other Gophers too, if the results of [an informal poll](https://gophers.slack.com/archives/C029RQSEE/p1747742475549899) that I recently ran on [Gophers Slack](https://gophers.slack.com/) are to be believed. If the “single-use” qualifier is intended for only a subcategory of “impure” iterators, I’m not sure why this subcategory should deserve a particular focus in the documentation. And if the “single-use” qualifier is intended for all “impure” iterators, it strikes me as a misnomer; after all, many forms of “impure” iterators are possible, not all of which could reasonably be described as “single-use”, as we shall see.

### A wondrous zoo of “impure” iterators [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#a-wondrous-zoo-of-impure-iterators)

In the program below, the iterator resulting from `fib2` ([playground](https://go.dev/play/p/lPuFqQz7RgR)) could reasonably be described as “usable twice” and “non-resumable” (or, perhaps, [_restarting_](https://github.com/golang/go/issues/61897#issuecomment-2182788027)):

```
package main

import (
	"fmt"
	"iter"
)

func fib2() iter.Seq[int] {
	var n int
	return func(yield func(n int) bool) {
		if n > 1 {
			return
		}
		for a, b := 0, 1; yield(a); a, b = b, a+b {
			// deliberately empty body
		}
		n++
	}
}

func main() {
	seq := fib2()
	for n := range seq {
		if n > 10 {
			break
		}
		fmt.Printf("%d ", n)
	}
	fmt.Println()
	for n := range seq {
		if n > 100 {
			break
		}
		fmt.Printf("%d ", n)
	}
	fmt.Println()
	for n := range seq {
		if n > 1000 {
			break
		}
		fmt.Printf("%d ", n)
	}
}
```

Output:

```
0 1 1 2 3 5 8 
0 1 1 2 3 5 8 13 21 34 55 89 
```

And, in the program below, the iterator resulting from `fib3` ([playground](https://go.dev/play/p/rQnBcNTN2VR)) could reasonably be described as “usable twice” and “resumable”:

```
package main

import (
	"fmt"
	"iter"
)

func fib3() iter.Seq[int] {
	var n int
	a, b := 0, 1
	return func(yield func(n int) bool) {
		if n > 1 {
			return
		}
		for ; yield(a); a, b = b, a+b {
			// deliberately empty body
		}
		n++
	}
}

func main() {
	seq := fib3()
	for n := range seq {
		if n > 10 {
			break
		}
		fmt.Printf("%d ", n)
	}
	fmt.Println()
	for n := range seq {
		if n > 100 {
			break
		}
		fmt.Printf("%d ", n)
	}
	fmt.Println()
	for n := range seq {
		if n > 1000 {
			break
		}
		fmt.Printf("%d ", n)
	}
}
```

Output:

```
0 1 1 2 3 5 8 
13 21 34 55 89 
```

I’m sure you could think of more variations around this theme. As soon as function purity goes out the window, the possibilities are endless.

## Should iterators be “pure” whenever possible? [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#should-iterators-be-pure-whenever-possible)

Another question arises: if “pure” iterators are easier to reason about than “impure” ones are, shouldn’t iterators be designed as “pure” whenever possible?

### Performance considerations [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#performance-considerations)

Perhaps they should, at least when we emphasise performance as a design criterion. “Pure” iterators indeed tend to incur fewer heap allocations than their “impure” counterparts do. Consider [`strings.Lines`](https://pkg.go.dev/strings#Lines) as a case study; shown below is [its source code in Go 1.24.3](https://cs.opensource.google/go/go/+/refs/tags/go1.24.3:src/strings/iter.go;l=18):

```
// Lines returns an iterator over the newline-terminated lines in the string s.
// The lines yielded by the iterator include their terminating newlines.
// If s is empty, the iterator yields no lines at all.
// If s does not end in a newline, the final yielded line will not end in a newline.
// It returns a single-use iterator.
func Lines(s string) iter.Seq[string] {
	return func(yield func(string) bool) {
		for len(s) > 0 {
			var line string
			if i := IndexByte(s, '\n'); i >= 0 {
				line, s = s[:i+1], s[i+1:]
			} else {
				line, s = s, ""
			}
			if !yield(line) {
				return
			}
		}
		return
	}
}
```

The resulting iterator is “impure” because it mutates `s`, which happens to be its only free variable. As a consequence of [escape analysis](https://github.com/golang/go/blob/master/src/cmd/compile/README.md), `s` escapes to the heap:

```
$ go build -gcflags='-m=2' ./strings
-snip-
strings/iter.go:18:12: s escapes to heap:
-snip-
```

But `strings.Lines` could instead have been designed to produce a “pure” iterator:

```
 func Lines(s string) iter.Seq[string] {
        return func(yield func(string) bool) {
+               s := s // local copy!
                for len(s) > 0 {
                        var line string
                        if i := strings.IndexByte(s, '\n'); i >= 0 {
                                line, s = s[:i+1], s[i+1:]
                        } else {
                                line, s = s, ""
                        }
                        if !yield(line) {
                                return
                        }
                }
                return
        }
 }
```

In this alternative version of `strings.Lines`, its `s` parameter remains a free variable of the resulting iterator, but the iterator does not mutate that variable; instead, the iterator exclusively operates on a local copy of the source string. With this simple change, escape analysis no longer concludes that variable `s` needs escape to the heap, and one fewer memory allocation is required. See [this related comment](https://github.com/golang/go/issues/73524#issuecomment-2837185259) by [Alan Donovan](https://github.com/adonovan) on GitHub.

However, performance is only one possible design criterion: [as Ian Lance Taylor shrewdly pointed out to me](https://github.com/golang/go/issues/61901#issuecomment-2364733597), consistency in behaviour with closely linked iterator factories is another. For example, [package `bytes`](https://pkg.go.dev/bytes) provides [a function named `Lines`](https://pkg.go.dev/bytes#Lines) that’s analogous to `strings.Lines`; shown below is [its source code in Go 1.24.3](https://cs.opensource.google/go/go/+/refs/tags/go1.24.3:src/bytes/iter.go;l=18):

```
// Lines returns an iterator over the newline-terminated lines in the byte slice s.
// The lines yielded by the iterator include their terminating newlines.
// If s is empty, the iterator yields no lines at all.
// If s does not end in a newline, the final yielded line will not end in a newline.
// It returns a single-use iterator.
func Lines(s []byte) iter.Seq[[]byte] {
	return func(yield func([]byte) bool) {
		for len(s) > 0 {
			var line []byte
			if i := IndexByte(s, '\n'); i >= 0 {
				line, s = s[:i+1], s[i+1:]
			} else {
				line, s = s, nil
			}
			if !yield(line[:len(line):len(line)]) {
				return
			}
		}
		return
	}
}
```

As much as I’d like to redesign `bytes.Lines` so as to make its product “pure”, I can’t think of an efficient way to do so. Contrary to strings, slices are mutable; moreover, multiples slices can reference the same underlying array. Therefore, even if the resulting iterator were modified to operate on a local copy of the source slice, it still wouldn’t be “pure”, as demonstrated by the following program ([playground](https://go.dev/play/p/aBiBsiw2xvq)):

```
package main

import (
	"bytes"
	"fmt"
	"iter"
	"os"
)

func main() {
	src := []byte("foo\nbar\nbaz\n")
	seq := Lines(src)
	for s := range seq {
		os.Stdout.Write(s)
	}
	for s := range seq { // pass that mutates the source slice
		if len(s) == 0 {
			continue
		}
		s[0] = 'a'
	}
	fmt.Println()
	for s := range seq {
		os.Stdout.Write(s)
	}
}

func Lines(s []byte) iter.Seq[[]byte] {
	return func(yield func([]byte) bool) {
		s := s // local copy
		for len(s) > 0 {
			var line []byte
			if i := bytes.IndexByte(s, '\n'); i >= 0 {
				line, s = s[:i+1], s[i+1:]
			} else {
				line, s = s, nil
			}
			if !yield(line[:len(line):len(line)]) {
				return
			}
		}
		return
	}
}
```

Output:

For `bytes.Lines` to produce a pure iterator, a “deep” copy of the source slice (i.e. a slice that would reference a clone of the source slice’s backing array) would be required ([playground](https://go.dev/play/p/hj_NKtR482P)); and such an approach seems to me like it would defeat the purpose of iterators.

If `bytes.Lines` can’t easily be designed to produce a “pure” iterator, `strings.Lines` should arguably still return an “impure” iterator as well. In general, should iterator “purity” be pursued at all costs, or should consistency with related iterators be a factor? The jury is still out.

## Conclusion [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#conclusion)

As [Austin Clements](https://github.com/aclements) [recently wrote](https://github.com/golang/go/issues/71901):

> Iterators are still young in Go […]

Conventions around iterators have not yet been firmly established, let alone percolated through the Go community; there is still room for experimentation and time to iron the terminology kinks out. The sooner, the better, though, if you ask me.

## Acknowledgments [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#acknowledgments)

Thanks to [Valentin Deleplace](https://hachyderm.io/@deleplace) for [spurring me to write this post](https://bsky.app/profile/jub0bs.com/post/3lchwvaets22s) and for reviewing an early draft of it.

## Appendix: refresher on closures [¶](https://jub0bs.com/posts/2025-05-29-pure-vs-impure-iterators-in-go/#appendix-refresher-on-closures)

A [closure](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29) is a function that _captures_ variables declared in an outer lexical scope. Such variables are called _free variables_ of the function in question. In the following example, anonymous function `f` reads a variable named `truth` that is declared in the `main` function:

```
package main

import "fmt"

func main() {
	truth := true
	f := func() bool { return truth }
    fmt.Println(f())
}
```

Output:

So far, nothing to write home about. However, the power of closures lies in their ability to _update_ their free variables! Such closures essentially are _stateful_ functions: functions that maintain a state (composed of their free variables) from one invocation to the next.

In the following example ([playground](https://go.dev/play/p/kyfFfndk6mB)), factory function `falseAfterN` returns a closure that captures a local variable named `truth`, updates that variable during each invocation, and varies its boolean results on the basis of the free variable’s value:

```
package main

import "fmt"

func falseAfterN(n uint) func() bool {
	var count uint
	return func() bool {
		if count < n {
			count++
			return true
		}
		return false
	}
}

func main() {
	const n = 4
	f := falseAfterN(n)
	for i := range 2 * n {
		fmt.Printf("%d %t\n", i, f())
	}
}
```

If you’ve been writing Go for some time, you’ve likely used closures, perhaps even without realising that you were. In the classic example of a concurrent program ([playground](https://go.dev/play/p/usL33VcLdPg)) shown below, the anonymous function is a closure, since it captures (and acts on) variables `i` and `wg`, which are both declared in an outer scope:

```
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	for i := range 10 {
		wg.Add(1)
		go func() {
			defer wg.Done()
			fmt.Println(i)
		}()
	}
	wg.Wait()
}
```