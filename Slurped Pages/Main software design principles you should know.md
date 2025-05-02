---
link: https://newsletter.techworld-with-milan.com/p/main-software-design-principles-you
byline: Dr Milan Milanović
site: Tech World With Milan Newsletter
date: 2023-10-12T15:00
excerpt: Main software design principles. The seven design smells of rotting
  software. Software development antipatterns. SOLID principles. Clean Code.
slurped: 2025-02-08T09:44
title: Main software design principles you should know
---

In this issue, we talk about:

- **Main software design principles**
    
- **The seven design smells of rotting software**
    
- **Software development antipatterns**
    
- **Bonus: What Every Programmer Should Know About Memory**
    

So, let’s dive in.

Software design principles guide developers in creating efficient, scalable, and maintainable software. By adhering to these principles, developers can create easier code to read, test, and extend, lowering the overall cost of ownership and making it easier for teams to collaborate effectively.

Here's a list of some of the most essential software design principles:

1. **Separation of concerns.** Your application should be divided into discrete features with little functional overlap. Reducing interaction locations is crucial to attaining strong cohesion and low coupling. Even though the enclosed functionality within a feature does not considerably overlap, separating functionality at improper boundaries might lead to excessive coupling and complexity across features.
    
2. **Object-Oriented Programming Principles**
    
    1. **Encapsulation** involves bundling the data with the methods that operate on that data. It restricts direct access to some of an object's components, preventing unintended interference and misuse of the data.
        
    2. **Abstraction**: Abstraction means using simple classes to represent complexity. It hides the complex reality while exposing only the essential parts.
        
    3. **Inheritance:** This allows a class (the child or subclass) to inherit properties and behaviors (methods) from another class (the parent or superclass)
        
    4. **Polymorphism**: This enables one entity to be treated as a general category, allowing it to take multiple forms. For example, a particular class can be treated as its parent class or one of its implemented interfaces.
        
3. **SOLID Principles** - design principles guide developers in creating maintainable, scalable, and efficient object-oriented software systems.
    
    1. **Single Responsibility Principle (SRP):** A class/service/api should have only one reason to change, meaning it should have only one job or responsibility.
        
    2. **Open/Closed Principle (OCP):** Software entities (classes, modules, functions, etc.) should be available for extension but closed for modification. This means you can add new functionality without changing the existing code.
        
    3. **[Liskov Substitution Principle (LSP)](https://en.wikipedia.org/wiki/Liskov_substitution_principle)** States that you should be able to use any subclass instead of a parent class and expect it to work correctly. This means a program using a base class type should still work if a derived (subclass) type is passed to it without knowing it.
        
    4. **Interface Segregation Principle** (ISP): A class should not be forced to implement interfaces it does not use. This means creating specific interfaces for each class rather than having one large, all-encompassing one.
        
    5. **Dependency Inversion Principle (DIP)**: High-level modules should not depend on low-level modules. Abstractions should be independent of details. Both should depend on abstractions. This means that you should rely on abstractions rather than on concrete implementations.
        
4. **Don’t repeat yourself (DRY)**: Avoid duplication in code, which can lead to inconsistencies and bugs. Reuse code instead of copying it. Yet, there are situations where copying is better.
    
5. **Keep it simple, stupid (KISS)**: Code should be kept as simple and straightforward as possible. Simple code is easier to understand and maintain and less prone to errors.
    
6. **YAGNI (You Aren't Gonna Need It)**: Avoid adding unnecessary complexity by adding functionality only when needed. In certain situations, if development costs are really expensive or there is a significant design failure, you might need to pay for upfront, thorough design and testing. Don't do much design work too soon if your application requirements are unclear or you anticipate the design changing over time.
    
7. **Law of Demeter (LoD)** or **Principle of least knowledge**: An object should only communicate with its immediate friends and not know other objects' inner workings.
    
8. **Composition over Inheritance**: Favoring object composition over class inheritance, as it's more flexible and helps to avoid problems associated with larger inheritance hierarchies.
    
9. **[The principle of least astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)** or the principle of least surprise - Suggests that a system should behave in a way that is least surprising or confusing to its users (i.e., it should act as most users expect). For example, if you have a user account service, updating user data should be done by a `UpdateUserData()` method. The wrong one would be a method named `RebuildUserData()`.
    

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb24035a2-f161-4ed6-bcf0-61bd4b58329c_1920x1080.png)

](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fb24035a2-f161-4ed6-bcf0-61bd4b58329c_1920x1080.png)

SOLID Principles

We all start designing and implementing our software with the best intentions. However, as software grows because it constantly evolves, the design and architecture grow large and complex. During this time, tech debt tends to pile up, making software maintenance difficult and error-prone.

What are some symptoms of bad architecture, according to Robert C. Martin:

1. **Rigidity**
    

Rigid software is challenging to adapt because even one modification will inevitably lead to the necessity for other changes. When we believe we are almost done, we discover additional code that needs to be updated, sending us down a rabbit hole with no apparent exit.

2. **Fragility**
    

Fragile software tends to break frequently as a result of a single change. Often, the issues arise in locations that don't relate to the altered environment. The probability that a change may cause unforeseen issues approaches certainty as a module's fragility grows.

3. **Immobility**
    

When a design has components that could be used in different systems but would require too much work and danger, it is said to be immobile. Unfortunately, this is a relatively typical occurrence.

4. **Viscosity**
    

At this point, maintaining the software's architecture can be challenging. It's more difficult to do the right thing than the wrong thing (breaking the architecture). It should be simple to maintain the design of the software architecture.

5. **Needless Complexity**
    

This might be the most obvious evidence of poor design. Developers incorporate a variety of abstractions and make provisions for anticipated future changes in an ardent effort to avoid the other six scents. Good software is lightweight, adaptable, simple to read and comprehend, and above all, simple to alter, so you don't have to consider future modifications (YAGNI principle). Cyclomatic complexity metrics can help diagnose this problem.

6. **Needles Repetition**
    

This occurs when an architecture has code structures that should be unified under a single abstraction but are instead duplicated, typically by copying and pasting. The task of modifying software becomes problematic when there is redundant code present. When a flaw in repetitive code is discovered, it must be corrected in every instance. Each iteration, though, can be a little different.

7. **Opacity**
    

Opacity is a module's tendency to be challenging to understand. Code can be written in two ways: either clearly and expressively or opaquely and cryptically. Code that changes over time typically gets harder to know as it does. We usually need to dive into implementation to understand what they do. Code reviews can help in this situation.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4f1d4643-efba-4344-8ffc-43031215995e_1280x720.png)

](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F4f1d4643-efba-4344-8ffc-43031215995e_1280x720.png)

We are all familiar with **[GoF Design Patterns](https://en.wikipedia.org/wiki/Design_Patterns),** which represent a general reusable solution to some occurring problems in software design. They are templates for how to solve a problem that can be used many times, and they represent best practices we can use to solve common problems when designing applications.

However, there are also some **software development anti-patterns**. They represent a pattern that is commonly used, but it is ineffective or counterproductive in practice. We use those anti-patterns as we are probably inexperienced or unaware that these patterns are anti-patterns, but they can also produce bad designs or performances.

Here is the list of some of the most common anti-patterns:

- **God object:** It performs many functions which would be better separated into different objects. A typical god object takes many responsibilities and a lot of dependencies. It controls many other classes, and it validates the Single Responsibility principle.
    
- **Singleton:** This is one of the most used design patterns. However, it is an anti-pattern. They violate information hiding and usually behave as static data in your applications, violating sound OO principles.
    
- **Blob**: Most other objects in a procedural-style design are data holders or basic process implementers, leaving one object to handle the lion's share of the duties.
    
- **Spaghetti Code** or Big Ball of Mud: One of the most common anti-patterns. It is a code with little or no structure or modules. There are usually random files in random directories, and the whole flow is difficult to follow. Also, everything is connected.
    
- **Golden hammer, Silver Bullet, or Cargo-cult: A** familiar technology applied to many software problems. The solution entails exposing developers to alternative technologies and methods by broadening their knowledge through education, training, and book study groups.
    
- **Gold plating** or YAGNI: Continuing to work on a task or project well past the point when extra effort adds value.
    
- **Boat Anchor** and Dead code: This concept is similar to the previous one, but with the difference that developers leave some code because it might need it later. This creates nightmares with maintenance because code contains obsolete code deployed to production.
    
- **Abstraction inversion**: Here, we don't expose the implemented functionality required by callers of a function or method, so the caller needs to re-implement the same functionality. E.g., using a Vector in Java to implement a fixed-size list instead of an array (and Vector uses an array internally).
    
- **Interface bloat:** Making an interface so large is extremely difficult to implement and violates Single Responsibility. Example: implement IList in C#: IsReadOnly, Count, Add, Remove, IndexOf.
    
- **Lava Flow**: It refers to a concept where developers need to deal with redundant or low-quality code, but usually that code works, so removing it is risky. Such issues typically arise from legacy code written in the initial phase, so there needed to be more time to optimize or refactor code.
    

Also, there are **Code Smells** in different groups, such as Bloaters, OO Abusers, Change Preventers, Dispensables, and Couplers.

> "Prefer duplication over the wrong abstraction" - Sandi Metz.

[

![](https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7bccd2e0-251d-4a01-9152-0eb7640bcc13_1280x720.png)

](https://substackcdn.com/image/fetch/f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F7bccd2e0-251d-4a01-9152-0eb7640bcc13_1280x720.png)

To learn more about these anti-patterns, check the following links:

- [Software Development AntiPatterns](https://sourcemaking.com/antipatterns/software-development-antipatterns)
    
- [Code Smells](https://refactoring.guru/refactoring/smells)
    
- [AntiPatterns: Refactoring Software, Architectures, and Projects in Crisis book](https://amzn.to/3rw6vl3)
    
- [Anti-pattern in Wikipedia](https://en.wikipedia.org/wiki/Anti-pattern)
    

Check out this **[great manuscript by Ulrich Drepper](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf)**, which delves deep into computer memory systems and their implications for software developers. It talks about:

- **CPU Caches**
    
- **Virtual Memory**
    
- **Memory Access Patterns**
    
- **Performance Tools**
    
- **Locks**
    
- **Multi-threaded Programming and Memory**
    
- **And more**