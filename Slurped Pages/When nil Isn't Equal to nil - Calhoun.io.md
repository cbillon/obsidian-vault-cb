---
link: https://www.calhoun.io/when-nil-isnt-equal-to-nil/
byline: Jon Calhoun
site: Calhoun.io
excerpt: It is easy to get confused by when and why different variants of nil will be equal and when they won't be in Go. In this article we explore why this happens so that you know what to expect when writing your code.
twitter: https://twitter.com/@joncalhoun
tags:
  - go
  - nil
slurped: 2025-11-09T19:27
title: When nil Isn't Equal to nil - Calhoun.io
---

In this article we are going to explore a few situations where variables that appear to be equal to the developer will evaluate to false when we compare them using Go’s `==` operation. We will also discuss why it happens, which will hopefully make it easier to avoid running into this problem in your own code.

First let’s look at an example of what I mean. Imagine we had two variables, each with their own type, but each are assigned the hard-coded value of `nil`.

```
var a *int = nil
var b interface{} = nil
```

What do you expect the following evaluations to be?

```

fmt.Println("a == nil:", a == nil)
fmt.Println("b == nil:", b == nil)
fmt.Println("a == b:", a == b)
```

If you want to run the code on the Go playground to verify you can, but here is what the actual output will be:

a == nil: true
b == nil: true
a == b: false

Now let’s quickly look at another example that is very similar, but slightly different. We are going to change the initial value of `b`.

```

var a *int = nil
// This is the only change - we assign b to a instead
// of the hard-coded nil value.
var b interface{} = a

fmt.Println("a == nil:", a == nil)
fmt.Println("b == nil:", b == nil)
fmt.Println("a == b:", a == b)
}
```

Again, what do you expect the output to be? Below is the updated output to check your answer.

a == nil: true
b == nil: false
a == b: true

## What in the world is going on?

This phenomena is somewhat hard/weird to explain, but no, it is not a bug in the language, and no, it isn’t random/magical/whatever else. There are some clear rules being applied, we just need to take some time to understand them. After that this will all make sense and you will understand why you occasionally see people write code like this:

Instead of just assigning `b` to `a`.

The first thing we need to understand is that every pointer in Go has two basic pieces of information; the type of the pointer, and the value it points to. We will represent these as a pair like `(type, value)` moving forward.

The fact that every pointer variable needs a type is why we can’t have a nil value assigned to a variable without declaring the type as well. That is, the following code will NOT compile.

```
// This does not work because we do not know the type
n := nil
```

In order to make this code compile we must use a typed pointer and assign it to the value nil:

```
var a *int = nil
var b interface{} = nil
```

Now both of these variables have types. We can even use the `fmt.Printf` function to print out those types and see what they are.

```

var a *int = nil
var b interface{} = nil

fmt.Printf("a=(%T, ...)\n", a)
fmt.Printf("b=(%T, ...)\n", b)
```

The output to this code is shown below.

a=(*int, ...)
b=(<nil>, ...)

It appears that when we assign `nil`, the hard-coded value, to the `*int` variable `a` the type will be set to the same type used when defining the `a` variable - `*int`. That makes sense.

The second variable, `b`, is a little more confusing. Its type is `interface{}` (the [empty interface](https://www.calhoun.io/how-do-interfaces-work-in-go/)), but the type given when we print out the nil value is `<nil>`. What is going on?

The short version is that because we use the empty interface, any type will satisfy. The `<nil>` type is technically a type, and it satisfies the empty interface, so it is used when no other type information can be determined by the compiler.

Okay, so we know all pointers have a `(type, value)` associated with them and we have seen what happens when we assign `nil` to each variable. Now let’s look at what those types are when we assign `b` to nil using the `a` variable rather than a hard-coded `nil`.

```

var a *int = nil
var b interface{} = a

fmt.Printf("a=(%T, ...)\n", a)
fmt.Printf("b=(%T, ...)\n", b)
```

Huh… It looks like `b` has a new type.

Before when we assigned `b` to the hard-coded nil value we had no type information. Now that we are assigning `b` to the value `a` that is no longer true. We know exactly what type information to use - whatever was used by the `a` variable!

## What happens when we check for equality?

Now that we understand how these types are being determined, let’s look at what happens when we check for equality in our code.

We will start with both `a` and `b` being assigned to hard-coded `nil` values. After that we will look at a similar snippet where `b` is assigned to the variable `a`.

```

var a *int = nil
var b interface{} = nil

// We will print out both type and value here
fmt.Printf("a=(%T, %v)\n", a, a)
fmt.Printf("b=(%T, %v)\n", b, b)
fmt.Println()
fmt.Println("a == nil:", a == nil)
fmt.Println("b == nil:", b == nil)
fmt.Println("a == b:", a == b)
```

The output to this program is shown below.

a=(*int, <nil>)
b=(<nil>, <nil>)

a == nil: true
b == nil: true
a == b: false

The obviously weird part here is that `a` does NOT equal `b`. This seems incredibly weird, because at first glance it looks like we are saying that `a == nil` and `b == nil` but `a != b` which is not logically possible.

The reality of the situation is that what we have written - eg `a == nil` - is not a true representation of what is actually being compared. What we are really comparing is both the values AND the types. That is, we are not just comparing the value stored in `a` with the `nil` value; we are also comparing their types. This is shown more explicitly below.

```
a == nil: (*int, <nil>) == (*int*, <nil>)
b == nil: (<nil>, <nil>) == (<nil>, <nil>)
# Notice that these two are clearly not equal
# once we add in the type information.
a == b: (*int, <nil>) == (<nil>, <nil>)
```

When written the comparisons this way it becomes pretty clear that the two are not equal - they have different types - but this is all information not explicitly clear in our code which is what unfortunately leads to this common misunderstanding.

Now let’s look at what happens when we assign `b` to the `a` variable and perform the same comparisons.

```

var a *int = nil
var b interface{} = a // <- the change

fmt.Printf("a=(%T, %v)\n", a, a)
fmt.Printf("b=(%T, %v)\n", b, b)
fmt.Println()
fmt.Println("a == nil:", a == nil)
fmt.Println("b == nil:", b == nil)
fmt.Println("a == b:", a == b)
```

The output of this program is…

a=(*int, <nil>)
b=(*int, <nil>)

a == nil: true
b == nil: false
a == b: true

Now the odd man out is the second line, `b == nil`.

This one is a little less obvious, but when we compare the variable `b` to a hard-coded `nil` our compiler once again needs to determine what type to give that `nil` value. When this happens the compiler makes the same decision it would make if assigning `nil` to the `b` variable - that is it sets the right hand side of our equation to be `(<nil>, <nil>)` - and if we look at the output for `b` it clearly has a different type: `(*int, <nil>)`.

A common thought at this point is that this is confusing, and the language should just handle this detail for us. Unfortunately that isn’t possible at compile-time because the actual type of the `b` variable can change as the program runs.

```

var a *int = nil
var b interface{} = a
var c *string = nil

fmt.Printf("b=(%T, %v)\n", b, b)
fmt.Println("b == nil:", b == nil)

b = c
fmt.Printf("b=(%T, %v)\n", b, b)
fmt.Println("b == nil:", b == nil)

b = nil
fmt.Printf("b=(%T, %v)\n", b, b)
fmt.Println("b == nil:", b == nil)
```

In this program our `b` variable has its type change three times. It starts out as `(*int, <nil>)`, then becomes `(*string, <nil>)`, and finally ends as `(<nil>, <nil>)`.

There is no way for the compiler each of these types at compile-time, which means that this could only be handled automatically in Go if it became a runtime decision, which would have its own set of unique complications that likely aren’t worth introducing.

## Compiler type decisions can also be demonstrated with numbers

We saw how `nil` can be coerced into the correct type by the compiler, but this isn’t actually the only situation where the compiler makes type decisions like this. For instance, when you assign variables to a hard-coded number the compiler will decide which type should be used based on the context of the program.

The obvious situation is when the type is declared along with the variable (eg `var a int = 12`), but this can also happen when we pass a hard-coded value into a function or when we just assign a variable to a number. All of these situations are shown below.

```

package main

import "fmt"

func main() {
	var a int = 12
	var b float64 = 12
	var c interface{} = a
	d := 12 // will be an int

	fmt.Printf("a=(%T,%v)\n", a, a)
	fmt.Printf("b=(%T,%v)\n", b, b)
	fmt.Printf("c=(%T,%v)\n", c, c)
	fmt.Printf("d=(%T,%v)\n", d, d)
	useInt(12)
	useFloat(12)
}

func useInt(n int) {
	fmt.Printf("useInt=(%T,%v)\n", n, n)
}

func useFloat(n float64) {
	fmt.Printf("useFloat=(%T,%v)\n", n, n)
}
```

We can even demonstrate some of the comparison confusion using numbers.

```

var a int = 12
var b float64 = 12
var c interface{} = a

fmt.Println("a==12:", a == 12) // true
fmt.Println("b==12:", b == 12) // true
fmt.Println("c==12:", c == 12) // true
fmt.Println("a==c:", a == c) // true
fmt.Println("b==c:", b == c) // false
// We cannot compare a and b because their types
// are clearly never going to match.
```

So `a == 12`, `b == 12` and `c == 12`, but if we compare `b == c` we get back false. What?!?!

Again, it goes back to the underlying types:

```
a=(int,12)
b=(float64,12)
c=(int,12)
```

`a` and `c` have the `int` type. `b` has a `float64` type, so when we compare them with hard coded values like `12` those need coerced into a type before the comparison can happen.

Another interesting note specific to numbers is that when comparing `12` to an interface, the compiler will always coerce it into an `int`. This is similar to how `nil` gets coerced into `(<nil>, <nil>)` when compared to an interface, and we can demonstrate this by changing our last code snippet to instead be:

```

var b float64 = 12
var c interface{} = b

fmt.Println("c==12:", c == 12)
fmt.Printf("c=(%T,%v)\n", c, c)
fmt.Printf("hard-coded=(%T,%v)\n", 12, 12)
```

Which has the following output:

c==12: false
c=(float64,12)
hard-coded=(int,12)

Now `c == 12` returns false because `(float64, 12)` is not the same as the hard-coded `(int, 12)` because it has a different type.

## In summary…

When we compare hard-coded values with variables the compiler has to assume they have some specific type and follows some set of rules to make this happen. Sometimes this can be confusing, but over time you get used to it.

If you find yourself working with various types that can all be assigned to nil, one common technique for avoiding issue is to explicitly assign things to `nil`. That is, instead of `a = b` write:

```
var a *int = nil
var b interface{}

if a == nil {
  b = nil
}
```

Now when we later compare `b` to a hard-coded `nil` we will get the results we expected. It is a little more work, but it leads to much better overall results in almost all circumstances.