---
link: https://medium.com/@andreiboar/7-common-interface-mistakes-in-go-1d3f8e58be60
byline: Andrei Boar
site: Medium
date: 2024-06-24T07:16
excerpt: 7 Common Interface Mistakes in Go Go is still a new language, and if you’re working with it, chances are that this is not your first programming language. Coming from a different language, you …
twitter: https://twitter.com/@zuzuleinen
slurped: 2025-11-05T12:27
title: 7 Common Interface Mistakes in Go
tags:
  - go
  - interface
---
Go is still a new language, and if you’re working with it, chances are that this is not your first programming language.

Coming from a different language, you bring forth both your experience as well as your biases. Things you used to do in your previous language might not be a good idea in Go.

Learning Go is not only about learning a new syntax. It’s also about learning a new way of thinking about your programs:

> Go is a language for writing Go programs, not Java programs or Haskell programs or any other language’s programs. You need to think a different way to write good Go programs. But that takes time and effort, more than most will invest. So the usual story is to translate one program from another language into Go and see how it turns out. But translation misses idiom. A first attempt to write, for example, some Java construct in Go will likely fail, while a different Go-specific approach might succeed and illuminate. After 10 years of Java programming and 10 minutes of Go programming, any comparison of the language’s capabilities is unlikely to generate insight, yet here come the results, because that’s a modern programmer’s job.  
> **Rob Pike** — [Esmerelda’s Imagination](https://commandcenter.blogspot.com/2011/12/esmereldas-imagination.html)

As Rob Pike suggests, if you want to improve your Go skills, you need to invest time and effort in learning the idioms of the language.

Go does several things differently than other traditional languages, and in this article, I will focus on one of them: **interfaces**.

Below is a list of common mistakes people make when writing Go interfaces. These might not be mistakes in other languages, but in Go, you need to unlearn them. Or at least, give a chance to not work with them for a while and see where that leads you.

## **But before a bit of theory**

Here is a list of things to keep in mind while reading this article. Feel free to skip if you’re already familiar with them.

**Interface segregation principle**: A client should never be forced to implement an interface that it doesn’t use, or clients shouldn’t be forced to depend on methods they do not use.

**Polymorphism**: a piece of code changes its behavior based on the concrete data it receives.

**Liskov Substitution Principle**: If your code depends upon an abstraction, one implementation can be replaced by another without changing your code.

> The purpose of abstracting is not to be vague, but to create a new semantic level in which one can be absolutely precise. — E.W.Dijkstra

Interfaces are concepts that precisely contain an idea used to compose your programs.

Using interfaces the right way leads to simplicity, readability, and organic code design.

Organic code is code that grows in response to the behaviors you need at a point in time. It doesn’t force you to think ahead about your types and the relationship between them, because it’s quite probable you won’t get them right.

That’s why Go is said to favor **composition over inheritance**. You have a small set of behaviors from which you can compose anything you want, as opposed to predefining types inherited by other types and hoping they will fit the problem domain.

Rob Pike [explained](https://groups.google.com/g/golang-nuts/c/mg-8_jMyasY/m/lo-kDuEd540J) this approach in the [golang-nuts](https://groups.google.com/g/golang-nuts) forum:

Press enter or click to view image in full size

Go’s interfaces aren’t a variant on Java or C# interfaces, they’re much more. They are a key to large-scale programming and adaptable, evolutionary design.

Anyway, enough theory let’s get to the most common mistakes:

## 1. You create way too many interfaces

The term for having too many interfaces is called [interface pollution](https://www.ardanlabs.com/blog/2016/10/avoid-interface-pollution.html). It happens when you start abstracting before writing your concrete types. Since you can’t foresee what abstractions you’ll need, it’s very easy to write too many interfaces that, later on, are either wrong or useless.

Rob Pike has a great guideline that helps us avoid interface pollution:

> Don’t design with interfaces, discover them.  
> **Rob Pike**

What Rob is pointing out here is that you don’t need to think ahead about what abstractions you need. You can start the design with concrete structs and create an interface **only when the design requires it**. By doing this, your code grows organically to the expected design.

I still see people creating interfaces in advance because they think they might need more than one implementation in the future.

To them, I say:

Press enter or click to view image in full size

Be lazy but in a good way. The perfect time to create an interface is when you actually need it, not when you predict you’ll need it. Here’s an [example of an interface](https://github.com/benchkram/bob/issues/115) created by thinking ahead and where that led.

Useless interfaces tend to have only one implementation. They just add an extra level of indirection, forcing programmers to always pass through them when they actually want to go to the implementation.

An interface has a cost: it’s a new concept you need to remember when reasoning about your code. As Djikstra said, an ideal interface has to be _“a new semantic level in which one can be absolutely precise.”_

If your code requires the idea of a `Box`, an extra interface called `Container` implemented only by `Box` brings nothing to the table, except confusion.

So ask yourself before creating an interface: Do you **have** multiple implementations of the interface? I emphasized _have_ because _will have_ assumes that you can predict the future, which you can’t.

## **2. You have way too many methods**

It’s quite typical in a PHP project to see 10-method interfaces. In Go, interfaces are small, the average number of methods on all the interfaces from the standard library is 2.

**The bigger the interface the weaker the abstraction** is actually one of the [Go Proverbs](https://go-proverbs.github.io/). As Rob Pike says, this is the most important thing about interfaces, which implies that **the smaller the interface, the more useful it is**.

The more implementations an interface can have, the more universal it is. If you have an interface with a large set of methods, it’s hard to have multiple implementations of it. The more methods you have, the more specific the interface becomes. The more specific it is, the lower the chances that different types could display the same behavior.

A good example of useful interfaces are [io.Reader](https://pkg.go.dev/io#Reader) and [io.Writer](https://pkg.go.dev/io#Writer) that have hundreds of implementations. Or the [error](https://pkg.go.dev/builtin#error) interface which is so powerful that it enables the whole error handling in Go.

Remember that you can compose an interface from other interfaces later on. Here’s for example the `ReadWriteCloser` composed of 3 smaller interfaces:

type ReadWriteCloser interface {  
	Reader  
	Writer  
	Closer  
}

## 3. You don’t write behavior-driven interfaces

In traditional languages, noun interfaces such as `User`, `Request`, and so on are quite common. In Go, most interfaces have an`er`suffix: `Reader`, `Writer`, `Closer`, etc. That’s because, in Go, interfaces expose behavior, and their names point to that behavior.

As I previously wrote in [Fundamentals of IO](https://medium.com/@andreiboar/fundamentals-of-i-o-in-go-c893d3714deb), when defining interfaces in Go, you don’t define what _something is but what it provides —_ **behavior**, not things! That’s why there’s no `File` interface in Go, but a `Reader` and a `Writer`: these are behaviors, and `File` is a thing implementing `Reader` and `Writer`.

The same idea is mentioned in [Effective Go](https://go.dev/doc/effective_go), an official guide on writing idiomatic Go:

> Interfaces in Go provide a way to specify the behavior of an object: if something can do _this_, then it can be used _here_.

When writing interfaces, try to think about actions or behaviors. If you define an interface called `Thing` ask yourself why that `Thing` is not a struct.

## 4. You write the interface on the producer side

I often see this in code reviews: people define interfaces in the same package where they write their concrete implementation.

Press enter or click to view image in full size

Interface defined on the producer side

But, maybe a client doesn’t want to use all the methods from the producer interface. Remember from the Interface Segregation Principle that a _“client should not be forced to implement methods it doesn’t use”_. Here’s an example:

package main

// ====== producer side

// This interface is not needed  
type UsersRepository interface {  
    GetAllUsers()  
    GetUser(id string)  
}

type UserRepository struct {  
}

func (UserRepository) GetAllUsers()      {}  
func (UserRepository) GetUser(id string) {}

// ====== client side

// Client only needs GetUser and  
// can create this interface implicitly implemented  
// by concrete UserRepository on his side   
type UserGetter interface {  
    GetUser(id string)  
}

If a client wants to use all the methods from producer, he can use the concrete struct. The behavior is already available by the struct methods.

Even if a client wants to decouple its code and use multiple implementations, he can still create an interface with all the methods on his side:

Press enter or click to view image in full size

Interface defined on the client side

These things are enabled by the fact that interfaces in Go are satisfied implicitly. Client code no longer needs to import some interface and write `implements` because there is no such keyword in Go. If `Implementation` has the same methods as `Interface`, then `Implementation` already satisfies that interface and can be used in client code.

## **5. You are returning interfaces**

If a method returns an interface instead of a concrete struct, all the clients calling that method are forced to work with the same abstraction. You need to let clients decide what abstractions they need because the code is their courtyard.

It’s annoying when you want to use something from a struct but can’t because the interface doesn’t expose it. There could be reasons for that limit, but not always. Here’s a contrived example:

package main

import "math"

type Shape interface {  
    Area() float64  
    Perimeter() float64  
}

type Circle struct {  
    Radius float64  
}

func (c Circle) Area() float64 {  
    return math.Pi * c.Radius * c.Radius  
}

func (c Circle) Perimeter() float64 {  
    return 2 * math.Pi * c.Radius  
}

// NewCircle returns an interface instead of struct  
func NewCircle(radius float64) Shape {  
    return Circle{Radius: radius}  
}

func main() {  
    circle := NewCircle(5)

    // we lose access to circle.Radius  
}

In above example, not only we lose access to `circle.Radius`, but we need to fill our code with type assertions every time we want to access it:

shape := NewCircle(5)

if circle, ok := shape.(Circle); ok {  
    fmt.Println(circle.Radius)  
}

To follow Postel’s Law, _“be conservative in what you send, be liberal in what you accept”_, return concrete structs from your methods, and opt to accept interfaces.

In Practical Go by Dave Cheney, there is [a nice write-up](https://dave.cheney.net/practical-go/presentations/qcon-china.html#_let_functions_define_the_behaviour_they_requires) why the following code:

// Save writes the contents of doc to the file f.  
func Save(f *os.File, doc *Document) error

is improved by accepting an interface:

// Save writes the contents of doc to the supplied  
// Writer.  
func Save(w io.Writer, doc *Document) error

**later edit:** Do not think returning an interface is a bad idea(thank ThreeFactorAuth):

Press enter or click to view image in full size

## **6. You create interfaces purely for testing**

Here’s another cause for interface pollution: creating an interface with only one implementation just because you want to mock the implementation.

If you abuse interfaces by creating many mocks, you end up testing mocks that are never used in production instead of the actual logic of your application. And in your real code, you now have 2 concepts(_semantic levels as Djikstra said_) where one will do. And this was just for a thing you want to test. Do you want to double your semantic levels every time you create a new test?

You can always use [testcontainers](https://testcontainers.com/) instead of mocking your database for example. Or just have your own container if `testcontainers` doesn't support it already.

Or maybe you need to mock something but not the whole thing. If you have a struct with 10 methods for example, maybe you don’t need to mock the whole struct. Maybe you can mock only a small part, and you can use your concrete struct in your tests. Mocking the whole struct is a very lazy solution for testing.

If you write an API, you don’t need to provide an interface to your clients so they can use it for mocking. If they want to write mocks, they can do that themselves by specifying interfaces on their side(see Point 4).

## **7. You don’t verify interface compliance**

Let’s say you have a package that exports a type called `User`, and you implement the `Stringer` interface because, for some reason, when you print it, you don’t want the e-mail displayed:

package users

type User struct {  
    Name  string  
    Email string  
}

func (u User) String() string {  
    return u.Name  
}

The client has the following code:

package main

import (  
    "fmt"

    "pkg/users"  
)

func main() {  
    u := users.User{  
       Name:  "John Doe",  
       Email: "john.doe@gmail.com",  
    }  
    fmt.Printf("%s", u)  
}

This will correctly output: `John Doe`.  
Now let’s say you refactor and, by mistake, you remove or comment the `String()` implementation and your code looks like this:

package users

type User struct {  
    Name  string  
    Email string  
}

In this case, your code will still compile and run, but the output will now be `{John Doe [john.doe@gmail.com](mailto:john.doe@gmail.com)}` There was no feedback enforcing your previous intent.

The compiler helps you when you have methods that accept a `User`, but in cases such as above, it won’t.

To enforce the fact that a certain type implements an interface, we can do the following:

package users

import "fmt"

type User struct {  
    Name  string  
    Email string  
}

var _ fmt.Stringer = User{} // User implements the fmt.Stringer

func (u User) String() string {  
    return u.Name  
}

Now, if we remove the `String()` method, we get the following on build:

cannot use User{} (value of type User) as fmt.Stringer value in variable declaration: User does not implement fmt.Stringer (missing method String)

What we did in that line was that we tried to assign an empty `User{}`, to a variable of type `fmt.Stringer`. Since the `User{}` stopped implementing `fmt.Stringer` we received a complaint. We used an `_`for the variable name because we don’t really use it so no allocations will be performed.

Above we have the `User` implementing the interface. `User` and `*User` are different types. So if you want the `*User` to implement it you do something like this:

var _ fmt.Stringer = (*User)(nil) // *User implements the fmt.Stringer

I also like when doing this that my Goland IDE will show me an I_mplement missing methods_ option:

Press enter or click to view image in full size

To learn more check this [article from Mat Ryer](https://medium.com/@matryer/golang-tip-compile-time-checks-to-ensure-your-type-satisfies-an-interface-c167afed3aae) or the guide from [Uber Go Style](https://github.com/uber-go/guide/blob/master/style.md#verify-interface-compliance) guide.

While this is a cool trick, you don’t need to do it for every type that implements an interface because if we have functions that require an interface, the compiler will already complain if you try to use types that don’t implement them. I myself had to think for a while to come up with an example for this article, so really, it’s a rare case.

As we are warned in [Effective Go](https://go.dev/doc/effective_go#interfaces:~:text=Don%27t%20do%20this%20for%20every%20type%20that%20satisfies%20an%20interface%2C%20though.%20By%20convention%2C%20such%20declarations%20are%20only%20used%20when%20there%20are%20no%20static%20conversions%20already%20present%20in%20the%20code%2C%20which%20is%20a%20rare%20event.):

> The appearance of the blank identifier in this construct indicates that the declaration exists only for the type checking, not to create a variable. Don’t do this for every type that satisfies an interface, though. By convention, such declarations are only used when there are no static conversions already present in the code, which is a rare event.

That was it! I hope you enjoyed this article. If you have something to add or if something is unclear, feel free to leave a comment. You can also [connect with me on LinkedIn](https://www.linkedin.com/in/andrei-boar/).

If you agree and want to spread the word, feel free to share this article or give it a 👏.

And remember: take any advice with a pinch of common sense. I wrote this article to explain alternatives and idiomatic ways of designing your Go code and show the costs of the decisions you make, not to list some things you should never do.

Be nice in your code reviews. We’re humans, after all, and constantly learning!

**later edit:** This article had some controversial takes and many things I didn’t manage to properly explain in one article. For a full discussion and other points of view check the [Reddit thread](https://www.reddit.com/r/golang/comments/1dnk96p/7_common_interface_mistakes_in_go/), there are some valuable points there from other users.