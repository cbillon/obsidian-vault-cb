---
link: https://blog.kowalczyk.info/a-d9qh/explaining-nil-interface-gotcha-in-go.html
site: "@kjk"
excerpt: I explain nil interface{} gotcha in Go and fail to come up with a better design
twitter: https://twitter.com/@kjk
slurped: 2025-11-05T18:44
title: Explaining nil interface{} gotcha in Go
tags:
  - go
  - interface
---

In Go empty interface is an interface without any methods, typed as `interface{}`.

A zero value of `interface{}` is `nil`:

```
var v interface{} // compiler sets this to nil, you could explicitly write = nil
if v == nil {
  fmt.Printf("v is nil\n")
} else {
  fmt.Printf("v is NOT nil\n")
}
```

This prints: `v is nil`.

However, this sometimes trips people up:

```
type Foo struct {
}

var v interface{}
var nilFoo *Foo // implicilty initialized by compiler to nil
if nilFoo == nil {
    fmt.Printf("nilFoo is nil.")
} else {
    fmt.Printf("nilFoo is NOT nil.")
}
v = nilFoo
if v == nil {
    fmt.Printf("v is nil\n")
} else {
    fmt.Printf("v is NOT nil\n")
}
```

This prints: `nilFoo is nil. v is NOT nil`.

On surface level, this is wrong: `t` is a `nil`. We assigned a `nil` to `v` but it doesn’t equal to `nil`?

How to check if `interface{}` is `nil` of any pointer type?

```
func isNilPointer(i interface{}) bool {
    if i == nil {
        return false // interface itself is nil
    }
    
    v := reflect.ValueOf(i)
    return v.Kind() == reflect.Ptr && 
           v.IsNil()
}

	type Foo struct {
	}

	var pf *Foo
	var v interface{} = pf

	if isNilPointer(v) {
		fmt.Printf("v is nil pointer\n")
	} else {
		fmt.Printf("v is NOT nil pointer\n")
	}
```

## 

Why

[](https://blog.kowalczyk.info/a-d9qh/explaining-nil-interface-gotcha-in-go.html#why)

There’s a reason for this perplexing behavior.

`nil` is an abstract value. If you come from C/C++ or Java/C#, you might think that this is equivalent of NULL pointer or null reference.

It isn’t. `nil` is a symbol that represents a zero value of pointers, channels, maps, slices.

Logically `interface{}` combines type and value. You can think of it as a tuple `(type, value)`.

An uninitialized value of `interface{}` is a tuple without a type and value `(no type, no value)`.

In Go `uninitialized value` is zero value and since `nil` is an abstract value representing zero value for several types, it makes sense to use it for zero value of `interface{}`.

So: zero value of `interface{}` is `nil` which is `(no type, no value)`.

When we assigned `nilFoo` to `v`, the value is `(*Foo, nil)`.

Are you surprised that `(no type, no value)` is not the same as `(*Foo, nil)`?

To understand this gotcha, you have to understand two things.

One: `nil` is an abstract value that only has a meaning in context.

Consider this:

```
var ch chan (bool)
var m map[string]bool
if ch == m {
    fmt.Printf("ch is equal to m\n")
}
```

This snippet doesn’t even compile: `Error:./prog.go:8:11: invalid operation: ch == m (mismatched types chan bool and map[string]bool)`.

Both `ch` and `m` are `nil` but you can’t compare them because they are of different types. `nil != nil` because nil is an abstract concept, not an actual value.

Two: `nil` value of `interface{}` is `(no type, no value)`.

Once you understand the above, you’ll understand why `nil` doesn’t compare to `(type, nil)` e.g. `(*Foo, nil)` or `(map[string]bool, nil)` or `(int, 0)` or `(string, "")`.

## 

Bad design or inevitable consequence of previous decisions?

[](https://blog.kowalczyk.info/a-d9qh/explaining-nil-interface-gotcha-in-go.html#bad-design-or-inevitable-consequence-of-previous-decisions)

Many claim it’s a bad design.

No-one describes what a better design would look like.

Let’s play act a Go language designer.

You’ve already designed concrete types, you came up with notion of zero value and created nil to denote zero value for pointers, channels, maps, slices.

You’re now designing `interface{}` as a logical tuple of `(type, value)`.

The zero value is obviously `(no type, no value)`.

You have to figure how to represent the zero value.

### 

A different symbol for interface{} zero value

[](https://blog.kowalczyk.info/a-d9qh/explaining-nil-interface-gotcha-in-go.html#a-different-symbol-for-interface-zero-value)

Instead of using `nil` you could create a different symbol e.g. `zeroInteface`.

You could then write:

```
var v interface{}
var v2 interface{} = &Foo{nil}
var v3 interface{} = int(0)

if v == zeroInteface {
  // this is true
}

if v2 == nil {
  // tihs is true
}

if v3 == nil {
  // is it true or not?
}
```

Is this a better design?

I don’t think so.

We don’t have `zeroPointer`, `zeroMap`, `zeroChanel` etc. so this breaks consistency. It sticks out like a sore `zeroInterface`.

And `v == nil` is subtle. Not all values wrapped in an `interface{}` have zero value of `nil`. What should happen if you compare to `(int, 0)` given that `0` is zero value of `int`?

### 

Damn the consistency, let’s do what user expects

[](https://blog.kowalczyk.info/a-d9qh/explaining-nil-interface-gotcha-in-go.html#damn-the-consistency-let-s-do-what-user-expects)

You could ditch the strict logic of `nil` values and special case the `if v == nil` for `interface{}` to do what people superficially expect to happen.

You then have to answer the question below: what happens when you do `if (int, 0) == nil`?

The biggest issue is that you’ve lost ability to distinguish between `(no type, no value)` and `(type, nil)`.

They both compare to `nil` so how would you test for `(no type, no value)` but not `(type, nil)`?

It doesn’t seem like a better design either.

### 

Your proposal

[](https://blog.kowalczyk.info/a-d9qh/explaining-nil-interface-gotcha-in-go.html#your-proposal)

Now that you understand the problem and seen two ideas for how to fix it, it’s your turn to design a better solution.

I tried and the above 2 are the only ideas I had.

We are boxed by existing notions of zero values and using nil to represent them.

We could explore designs that re-think those assumptions but would that be Go anymore?

It’s easy to complain that something is a bad design.

It’s much harder, often impossible, to design something better.