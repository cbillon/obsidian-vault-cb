---
link: https://wendelladriel.com/blog/composition-over-inheritance-in-php
site: Wendell Adriel
excerpt: The debate between the advantages and disadvantages of both composition and inheritance has taken centre stage in the PHP development community recently. In this article, I'll talk about those two concepts and the advantages of using Composition over Inheritance.
twitter: https://twitter.com/wendell_adriel
tags:
  - php
  - architecture
  - composition
  - inheritance
slurped: 2025-09-13T19:50
title: Composition Over Inheritance in PHP
---

[

## Introduction

The debate between the **advantages** and **disadvantages** of both **Composition** and **Inheritance** has taken center stage in the PHP development community recently. In the world of object-oriented programming (OOP) techniques, there are a lot of powerful tools and concepts each with its strengths and weaknesses, designed to enhance code readability, modularization, and reusability.

Before we explore the pros of **Composition** over **Inheritance**, let's understand these two principles independently.

[

## Inheritance

It's a fundamental OOP concept in PHP, is a technique where an object or class is based on another object (prototypal inheritance) or class (class-based inheritance), using the same implementation and augmenting it with the ability to use, extend, and alter the behaviours defined in other classes. A typical example is a `Vehicle` class that has subclasses like `Car` and `Motorcycle` - where the latter classes inherit properties and methods from the `Vehicle` class. The power of inheritance lies in its simplicity and how it facilitates the reuse of code, yet if not used properly, it can lead to tightly coupled and inflexible code structures known as **Inheritance Hierarchies**.

[

## Composition

On the other hand, **Composition** is another OOP practice where a class is formed from multiple simpler classes or objects by combining and assembling them in ways that help to model more complex behaviours. Instead of creating monolithic classes with inherited attributes and methods, composition works by containing instances of other classes that implement the desired functionality.

Using the previous example, instead of having a `Car` class inherit from `Vehicle`, we might have a `Vehicle` class that contains instances of `Engine`, `Wheels`, and other necessary components. This organization provides a high degree of flexibility, allowing more precise control over the behaviours that our objects can perform.

[

## Composition Over Inheritance

The principle of **Composition over Inheritance** is a recommendation to use **Composition** instead of **Inheritance** as much as possible. The reason is that while inheritance allows you to define the behaviour of a class in terms of another's, it binds the lifecycle and access level of the parent to the child class, which can make the code more rigid. Composition allows objects to obtain behaviour and features through relationships and not via a rigid class structure, leading to better modularization and lower coupling between classes.

Leveraging on **Composition** over **Inheritance** offers a more robust solution when developing **complex applications** with **numerous moving parts and behaviours**. This approach gives us flexibility and scalability and leads to a design that is more aligned with the real world.

We will see below an example of how to refactor a code that's using **Inheritance** to use **Composition** instead. The goal is not to neglect inheritance but to show how composition can provide a more robust and flexible solution in certain scenarios.

[

## Refactoring from Inheritance to Composition

Imagine that we have a system to catalogue and categorise birds. We could have something like this:

```
// Inheritance Example

class Bird
{
    public function fly(): string
		{
		    return "I can fly!";
		}
}

final class Pigeon extends Bird
{
}

$pigeon = new Pigeon();
echo $pigeon->fly(); // I can fly!
```

In the example above, we have a basic `Bird` class that has a method `fly()`. The `Pigeon` class inherits from `Bird` and thus has access to the `fly()` method. However, this approach can become problematic if we want to add a `Penguin` class. Penguins technically are birds, but they don't fly.

```
final class Penguin extends Bird
{
}

$penguin = new Penguin();
echo $penguin->fly(); // I can fly!
```

That could be fixed by overriding the `fly()` method, but depending on the size and complexity of the project this can get out of hand. We can refactor this implementation to use **Composition** instead:

```
// Composition Example

final class Bird
{
    public function __construct(
		    protected BirdType $birdType,
		) {}

    public function fly(): string
		{
		    return $this->birdType->fly();
		}
}

interface BirdType
{
    public function fly(): string;
}

final class FlyingBird implements BirdType
{
    public function fly(): string
		{
		    return "I can fly!";
		}
}

final class NonFlyingBird implements BirdType
{
    public function fly(): string
		{
		    return "I can't fly!";
		}
}

$pigeon = new Bird(new FlyingBird());
echo $pigeon->fly(); // I can fly!

$penguin = new Bird(new NonFlyingBird());
echo $penguin->fly(); // I can't fly!
```

In the example above, we've refactored our code to use **Composition**. We now have a `BirdType` interface denoting whether a bird can fly or not, and the `Bird` class takes a `BirdType` object as an argument to its constructor. The `Bird` class now has a `fly()` method that points to the `fly()` method of the `BirdType` object. This approach translates into more control over individual behaviours, as each functionality is encapsulated within its own class. This change allows us more flexibility (now our penguin can't fly) and our `Bird` class has better scalability, making it easier to manage the unique behaviours of different types of birds.

Through **Composition**, each class handles its very own responsibility **(Single Responsibility Principle)**, and complex behaviours are better managed by splitting them into separate entities. The way classes interact with each other becomes more explicit, and we end up with a **code structure** that is both **easier to maintain and to extend**, avoiding the rigid structure of inheritance.

[

## Conclusion

In the realm of OOP, both concepts are vital tools in a developer's "toolbox". In this article I tried to show the power of **Composition** over **Inheritance**. While **Inheritance** is a really intuitive and elegant way to organize and reuse code, depending on the context and how it is implemented it can become restrictive and problematic when classes become tightly coupled and overreliant on a rigid hierarchical structure.

Real-world PHP application development typically involves complex classes with numerous behaviours. As shown in the example, employing **Composition** over **Inheritance** makes code more robust, scalable, and manageable. This leads to a design that is easier to extend and maintain, reinforcing modularity and encapsulation.

Remember that choosing the right technique or concept is heavily dependent on the particular context and overall project requirements. The goal of this article is not to ignore **Inheritance**, but rather to understand how **Composition** provides a flexible alternative in specific situations.

Embracing **Composition** as a first-class tool alongside **Inheritance** enables a more diverse range of solutions for **Software Design** problems, leading to robust and maintainable PHP applications.