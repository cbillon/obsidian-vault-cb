---
link: https://www.dolthub.com/blog/2025-10-24-go-switch/
byline: Angela Xie
site: DoltHub
date: 2025-10-24T00:00
excerpt: Switch statements in Go have unique features that make it easy to write complex flow controls. Read this blog to see what makes them so special.
twitter: https://twitter.com/DoltHub
slurped: 2025-11-27T11:29
title: Switch Statements in Go
tags:
  - go
  - tips
---

Here at DoltHub, we write a lot of Go code — both [Dolt](https://github.com/dolthub/dolt) (the world’s first version-controlled database) and [go-mysql-server](https://github.com/dolthub/go-mysql-server/) (the SQL query engine that powers Dolt) are written in Go. We regularly blog about features and nuances we’ve observed in the language, and in this blog post, we’ll be discussing `switch` statements and what makes them special in Go.

If you’ve written C, C++, or Java before, you’re already familiar with `switch` statements and how useful they are for flow control. `switch` statements in Go are pretty similar but differ in that breaking is automatic, cases can be expressions, and a control variable is not needed. `switch` statements in Go also allow switching based on a control variable’s type. These features allow for more complex and easier-to-read conditional branching without relying on a bunch of `if/else` statements.

## Automatic Breaking[#](https://www.dolthub.com/blog/2025-10-24-go-switch/#automatic-breaking)

In C, C++, and Java, cases in `switch` statements fall through by default. This means once a matching case is executed, the control flow will automatically execute all subsequent cases until a `break` is encountered. Let’s look at an example in Java:

```
switch (color) { // color is an enum
  case BLUE:
    System.out.print("blue");
    break;
  case PINK:
    System.out.print("pink");
  case RED:
    System.out.print("red");
    break;
  case YELLOW:
    System.out.print("yellow");
    break;
}
```

Suppose `color` is `PINK`. We match the case `PINK` and print `pink`. Then, we move on to the following case, `RED`, print `red`, and break. We end up with the output `pinkred`.

In Go, cases do not fall through by default. Once a case is matched and executed, the switch is automatically exited. In other words, every case ends with an implicit `break`. But if you do want to continue to the next case, you can explicitly use the `fallthrough` keyword. Let’s recreate the same control flow from before in Go:

```
switch (color) { // color is an int and the cases are constants
case Blue:
  fmt.Print("blue")
case Pink:
  fmt.Print("pink")
  fallthrough
case Red:
  fmt.Print("red")
case Yellow:
  fmt.Print("yellow")
}
```

As you can see, the automatic breaking results in much cleaner code and avoids bugs caused by forgetting to `break` statements.

## Expression-based Cases[#](https://www.dolthub.com/blog/2025-10-24-go-switch/#expression-based-cases)

In Go, the cases of a `switch` statement don’t need to be constants and can instead be expressions that evaluate to the same type as the condition variable. If multiple case expressions evaluate to the same value as the condition variable, then the first one listed is executed. This can be useful when you want to match your condition variable to a value based on other variables or a function call. Let’s take a look at an example:

```
switch (x) {
case m:
  fmt.Println("x is the same value as m")
case m * 2:
  fmt.Println("x is twice the value of m")
case n:
  fmt.Println("x is the same value as n")
case n / 2:
  fmt.Println("x is half the value of n")
case m + n:
  fmt.Println("x is the sum of m and n")
}
```

## Switching without a Condition Variable[#](https://www.dolthub.com/blog/2025-10-24-go-switch/#switching-without-a-condition-variable)

You can also write a `switch` statement without a condition variable when each case is an expression that evaluates to a boolean; the first true case will be executed. This can be convenient when your control flow is dependent on ranges, multiple variables, or other more complex conditions. The following is an example of a `switch` statement without a condition variable:

```
switch {
case x == 0 || y == 0:
  fmt.Println("the product of x and y is 0")
case x > 0 && y > 0, x < 0 && y < 0:
  fmt.Println("the product of x and y is positive")
case x > 0 && y < 0, x < 0 && y > 0:
  fmt.Println("the product of x and y is negative")
}
```

If you want to see an example of this in our codebase, you can find it [here](https://github.com/dolthub/go-mysql-server/blob/70b084ecf8af6cdb294020e61956b97d4fae7f21/sql/memo/coster.go#L90) in the coster for join planning. Instead of switching on `jp.Op` and enumerating every possible `JoinType` ([37 total](https://github.com/dolthub/go-mysql-server/blob/70b084ecf8af6cdb294020e61956b97d4fae7f21/sql/plan/join.go#L29)), we instead call functions that return whether a `JoinType` belongs to a specified group of join types, allowing us to write more succinct code.

## Type-based Switching[#](https://www.dolthub.com/blog/2025-10-24-go-switch/#type-based-switching)

Something particularly unique and useful in Go is type-based switching. Type-based switching allows for control flow based on an interface’s dynamic type at runtime without doing a bunch of type checking.

So suppose we have the following types, and a `Listen` function that takes an interface as input and uses type-based switching to determine what to do.

```
type Animal interface {
  Sound() string
}

type Cat struct {}

// Sound implements Animal
func (c Cat) Sound() string {
  return "meow"
}

func (c Cat) Purr() string {
  return "purr"
}

type Dog struct {}

// Sound implements Animal
func (d Dog) Sound() string {
  return "woof"
}

func (d Dog) Growl() string {
  return "grrr"
}

func Listen(i interface{}) {
  switch x := i.(type) {
  case Cat:
    fmt.Println(x.Purr())
  case Animal:
    fmt.Println(x.Sound())
  case Dog:
    fmt.Println(x.Growl())
  default:
    fmt.Println("I didn't hear anything")
  }
}
```

Type-based switching will execute the code in the first case that matches the control variable’s type. So if `i` is of type `Cat` and we call `Listen(i)`, we’ll get `purr` as the output. But if `i` is of type `Dog`, we’ll get `woof` as the output; this is because the `Animal` case comes before the `Dog` case.

Since we use interfaces extensively in our codebase, we make use of type-based switching all the time. [Here](https://github.com/dolthub/go-mysql-server/blob/70b084ecf8af6cdb294020e61956b97d4fae7f21/sql/analyzer/fix_exec_indexes.go#L681) is an example where we take advantage of type-based switching executing the first matched type — `*plan.SetOp` implements `plan.TableIdNode` so we list the `*plan.SetOp` case before the `plan.TableIdNode` to ensure that it matches first.

## Conclusion[#](https://www.dolthub.com/blog/2025-10-24-go-switch/#conclusion)

`switch` statements in Go have robust features that allow for writing complex yet easy-to-read control flows. We make use of them all over the place in our codebase.