---
link: https://blog.rohitlokhande.in/structs-and-interfaces-in-go
byline: Rohit Lokhande
site: Tech Blog
date: 2025-11-08T18:24
excerpt: Explore how Go utilizes structs and interfaces to foster clean, flexible programs through its unique composition over inheritance approach
slurped: 2025-11-09T09:06
title: "Structs & Interfaces: Organizing Data in Go"
tags:
  - go
  - structs
  - interface
---

When learning Go, one thing becomes clear early on: Go doesn’t follow traditional Object-Oriented Programming (OOP) patterns like classes and inheritance.  
Instead, it gives us two powerful tools: Structs and Interfaces.

These two concepts are at the heart of Go’s design philosophy — **composition over inheritance**.  
Let’s break them down and see how they work together to build clean and flexible Go programs.

## What Are Structs?

A struct in Go is a collection of fields, basically a way to group data together. Think of it as Go’s version of an “object” or “class,” but simpler and without inheritance.

```
type Person struct {
    Name string
    Age int
}
```

Now we can create and use a `Person`:

```
func main() {
    p1 := Person{Name: "Rohit",Age:25}
    fmt.Println(p1.Name, p1.Age)
}
```

Structs help you define custom data types that represent real-world entities like users, orders, products, and more.

## Creating Structs in Different Ways

```
// Using var
var p1 Person
p1.Name = "Rohit"
p1.Age = 25

// Using shorthand
p2 := Person{"Alex",30}

// Using pointers
p2 := &Person{Name: "John", Age: 30}
```

> Pointers to structs are often used to avoid copying large structs and to modify their values directly.

## Nested and Anonymous Structs

You can also define structs inside structs or even anonymous structs.

```
type Address struct {
    City  string
    State string
}

type Employee struct {
    Name    string
    Address Address
}
```

And access nested fields like this:

```
fmt.Println(emp.Address.City)
```

Anonymous structs are useful when you need something quick and temporary.

```
person := struct {
    Name string
    Age  int
}{"Rohit", 25}
```

## Methods on Structs

Go doesn't have "classes," but it allows you to attach methods to structs.

```
func (p Person) Greet() {
    fmt.Println("Hello,", p.Name)
}
```

## Value Receiver vs Pointer Receiver

When defining methods, you’ll often see two styles:

```
func (p Person) Greet()        // value receiver
func (p *Person) HaveBirthday() // pointer receiver
```

- Value receiver: Works on a copy; doesn’t change the original data.
    
- Pointer receiver: Works on the original struct; can modify its data.
    

## What is a Method Set?

A method set is simply the collection of methods that belong to a type in Go.  
It defines which methods can be called on a value or a pointer of that type.

Go differentiates between methods with value receivers and methods with pointer receivers.

Let’s see an example first:

```
package main
import "fmt"

type Person struct {
  name string
}

// Value receiver
func (p Person) SayHello() {
    fmt.Println("Hello, my name is", p.name)
}

// Pointer receiver
func (p *Person) ChangeName(newName string) {
    p.name = newName
}

func main() {
    p := Person{name: "Rohit"}
    p.SayHello()
    p.ChangeName("Raj") // Go automatically take the address here

}
```

## Struct Embedding (also known as Composition over Inheritance)

Go doesn't have inheritance like OOP languages; instead, it uses embedding.  
You can "embed" one structure inside another to reuse its fields and methods.

Let's understand this with a code example:

```
package main
import "fmt"

type Person struct {
    Name string
    Age int
}

type Employee struct {
    Person // embedded struct
    Position string
}

func main() {
    e := Employee{
        Person: Person{Name: "Rohit", Age: 25},
        Position: "Software Engineer",
    }

    // Access Person fields directly
    fmt.Println(e.Name) // "Rohit"
    fmt.Println(e.Person.Age) // also valid
}
```

### Why use embedding?

- It's a simple way to combine types.
    
- You achieve an "inheritance-like" behavior without the complexity.
    

## Interfaces — Defining Behavior, Not Data

While structs define what something is, interfaces define what something does.

For example

```
type Greeter interface {
  Greet()
}
```

If a struct has a Greet() method, it automatically satisfies the Greeter interface. There's no need for explicit "implements" keywords like in Java or C#.

Example:

```
type Greeter interface {
  Greet()
}
type Dog struct{}
func (d Dog) Greet() {
    fmt.Println("Woof! 🐶")
}

type Person struct {
    Name string
}
func (p Person) Greet() {
    fmt.Println("Hi, I’m", p.Name)
}

func SayHello(g Greeter) {
    g.Greet()
}

func main() {
    SayHello(Person{Name: "Rohit"})
    SayHello(Dog{})
}
```

Output

```
Hi, I’m Rohit
Woof! 🐶
```

In this, both Person and Dog satisfy the Greeter interface—no explicit declaration needed.

## Interface Embedding

Similar to struct embedding, you can also embed interfaces.

```
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type ReadWriter interfce {
    Reader
    Writer
}
```

This way, ReadWriter automatically includes both the Read() and Write() methods.

## Nil Interface vs Nil Concrete Value

This is one of the trickiest Go interface concepts.

Example:

```
var err error = nil
fmt.Println(err == nil) // true
```

But for below code the output is different

```
var p *int = nil
var any interface{} = p
fmt.Println(any == nil) // false
```

Why?  
Because an interface internally holds (type, value).  
Here any has (type=*int, value=nil) → so it’s not fully nil

> Rule:  
> An interface is nil only if both its type and value are nil.

## Final Thoughts

Structs and interfaces are fundamental concepts in Go that highlight its unique approach to programming. They may seem simple at first glance, but they offer a powerful way to structure and organize code. By understanding how structs and interfaces function, you can appreciate how Go's design philosophy emphasizes clarity, flexibility, and simplicity in software development.

Structs allow you to define complex data types by grouping together variables, known as fields, under a single name. This makes it easier to manage and manipulate related data. Interfaces, on the other hand, define a set of method signatures that a type must implement, enabling polymorphism. This means you can write functions that operate on different types as long as they implement the same interface, promoting code reusability and flexibility.

Once you become familiar with these concepts, you'll notice how Go encourages you to write clear and maintainable code. The language's simplicity doesn't sacrifice power; instead, it provides a robust foundation for building scalable and efficient applications. Embracing Go's approach to structs and interfaces can lead to more intuitive and effective programming, allowing you to focus on solving problems rather than wrestling with complex language features.