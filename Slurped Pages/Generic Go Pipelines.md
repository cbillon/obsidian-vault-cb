---
link: https://preslav.me/2021/09/04/generic-golang-pipelines/
byline: Preslav Rachev
site: Preslav Rachev
date: 2021-09-04T02:00
excerpt: Generic Go Pipelines
slurped: 2026-01-26T19:46
title: Generic Go Pipelines
tags:
  - golang
  - pipeline
---

1. [Preslav Rachev /](app://obsidian.md/)[Programming](app://obsidian.md/categories/programming) /
2. [My Writings /](app://obsidian.md/posts/)[Programming](app://obsidian.md/categories/programming) /
3. [Generic Go Pipelines /](app://obsidian.md/2021/09/04/generic-golang-pipelines/)[Programming](app://obsidian.md/categories/programming) /

[My journey](https://preslav.me/2021/08/17/generic-programming-in-golang-time-for-a-restart/) with generic type parameters in Go continues. This time, I’d like to explore an idea I picked up from functional programming (FP) - _function pipelines_.

In FP languages, code execution is the result of calling long function chains, passing the result of one function to the next:

```
res = foo(bar(baz(some_data)))
```

While not necessarily bad, the added level of nesting makes code harder to edit and reason about. That is why FP languages like Elixir, Clojure, F#, Haskell, and ML before that have developed a variation of a concept we can generally call a function pipeline. Using a dedicated operator (`|>` in Elixir and F#’s case), one can un-nest the function call and place all functions in an easy to read and maintain chain of calls:

```
res = some_data
	|> baz() 
	|> bar() 
	|> foo()
```

Having all functions aligned on the same level allows developers to describe processes similar to how they would write a checklist:

```
- data
	 - first do this
	 - then do that
	 - if an error occurs, skip the rest
	 - are we still here?
	 - finally, do the following
```

Let’s take this concept and see if we can apply it to Go. One nice bonus if we succeed will be the graceful handling and propagation of errors in one single place.

Do you like my writing? Don’t forget to [follow me](https://twitter.com/preslavrachev) on Twitter.

## From Elixir to Go [#](#from-elixir-to-go)

When writing Go code, I discourage myself from separating my code into too many granular functions. The reason for this is error handling. If a complex piece of logic has, for example, even if I extract them in their respective functions, I would still have to do all the error handling in the place where I call them.

```
res, err := foo()
if err != nil {
	// handle the error
}

res, err = bar(res)
if err != nil {
	// handle the error
}

res, err = baz(res)
if err != nil {
	// handle the error
}
```

Before [generic type parameters](https://go.dev/blog/generics-proposal), I would not dare come up with patterns that streamline the process. In the best case, I would end up having to type-assert a bunch of `interface{}` values which is not optimal.

### Enter generics [#](#enter-generics)

Now that we know for sure that generic type parameters are going to make it into the language, I decided to dust off an old construct I had played with. I call this construct a `Pipe` (one can play with it [here](https://go2goplay.golang.org/p/QHgN2CwCqwo)):

```
type Pipe[T any] struct {
	chain []action[T]
}
```

At the bottom of it, a `Pipe` is simply a container for a chain of generic functions. I have abstracted away the function signature in its own type:

```
type action[T any] func(input T) (T, error)
```

This resembles the idea of a pure function in FP languages: it gets some input (T), which it manipulates and ideally, returns a manipulated copy of the manipulated input or an error. Additionally, the `Pipe` has two methods - `Next` for adding more steps to the chain, and `Do` for executing the chain.

```
// Next simply stores the chain of action steps
func (p *Pipe[T]) Next(f action[T]) *Pipe[T] {
	p.chain = append(p.chain, f)
	return p
}
```

```
// Do executes the chain, or cuts it early in case of an error 
func (p *Pipe[T]) Do() (T, error) {
	var res T
	var err error
	for _, fn := range p.chain {
		res, err = fn(res)
		if err != nil {
			break
		}
	}

	return res, err
}
```

That’s it. Add in a simple constructor function and we are ready:

### Using the pipe [#](#using-the-pipe)

Before we use the pipe, we will need to declare a type for the object we are going to pass around. As the author of many great names in my code, I will simply call it `InputOutput`, but I am sure you can come up with a better name for it.

```
type InputOutput struct {
	Answer int // the one and only attribute we are going to manipulate
}
```

Finally, here is the working example using our pipe:

```
func main() {
	res, err := NewPipe[InputOutput]().
		Next(foo).
		Next(bar).
		Next(baz). 
		Do()

	if err != nil {
		log.Panic(err)
	}

	log.Printf("%+v", res)
}

func foo(p InputOutput) (InputOutput, error) {
	p.Answer = 42
	return p, nil
}

func bar(p InputOutput) (InputOutput, error) {
	if p.Answer == 42 {
		return p, errors.New("wrong value")
	}

	return p, nil
}

func baz(p InputOutput) (InputOutput, error) {
	p.Answer = 3
	return p, nil
}
```

In [this example](https://go2goplay.golang.org/p/QHgN2CwCqwo), the pipeline will fail early. Comment `bar` out and it will reach the end.

## Conclusion [#](#conclusion)

This is only a simple example of creating a pipeline of Go functions. It is not “idiomatic” and might never be. Depending on the use case, however, it can help break down complex logic and lay it out in a manner easy to comprehend.

Let me know what you think, using the comment options above.


[![](app://obsidian.md/img/preslav.jpg)](app://obsidian.md/2021/11/18/generic-golang-optionals/)

[![](app://obsidian.md/img/preslav.jpg)](app://obsidian.md/2026/01/08/golang-structs-vs-pointers-pointer-first/)