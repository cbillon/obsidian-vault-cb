---
link: https://loup-vaillant.fr/articles/solid-bull
excerpt: The so called SOLID
slurped: 2026-01-15T12:42
title: A SOLID Load of Bull
tags:
  - solid
---

12/2025

“SOLID” is an acronym used by the famous advocate Robert C. Martin, to popularise what is now known as the _SOLID principles_. There are five of them: one good, one obsolete, and three invented by Martin himself.

Robert Martin is not your average Joe. I’ve watched him, he’s a very good speaker: articulate, driven, and entertaining. I’ve read his prose, he knows his rhetoric, how to deflect criticism, and how to play with the audience. I’ve seen his code, or at least the samples he carefully selected to be educational material, and _holy crap that’s bad!_

Not to beat on the _Clean Code_ dead horse, but the code examples alone should have been enough to make any competent programmer sceptical of anything Martin has to say about programming. Unfortunately that hasn’t stopped his ideas from getting traction, and I’m getting sick of repeating myself about SOLID on programming forums.

So let’s do this once and for all.

## Liskov substitution principle

> The Liskov substitution principle (LSP) is a particular definition of a subtyping relation, called strong behavioral subtyping, that was initially introduced by Barbara Liskov in a 1987 conference keynote address titled Data abstraction and hierarchy. It is based on the concept of “substitutability” – a principle in object-oriented programming stating that an object (such as a class) may be replaced by a sub-object (such as a class that extends the first class) without breaking the program. […] Barbara Liskov and Jeannette Wing described the principle succinctly in a 1994 paper as follows:
> 
> > Subtype Requirement: Let ϕ(x) be a property provable about objects ⁠ x of type T. Then ϕ(y) should be true for objects y⁠ of type S where S is a subtype of T.

_[Wikipedia](https://en.wikipedia.org/wiki/Liskov_substitution_principle)_

The one good principle. And I now I can see why: it was invented by people who knew their maths — type theory in this case. In lay terms it’s simple: anything you can observe about the base class, remains true of its derived classes. That way you can pretend instances of the derived classes are also instances of the base class, without any nasty surprise.

On the one hand, _duh_: that’s just subtyping. But on the other hand, very few type systems enforce it — Java and C++ do not. That makes it very easy to make a dumb mistake like overriding a stable `sort()` method, and make it _not_ stable for the derived class.

Haskell programmers are keenly aware of this: when they devise a _type class_, they explicitly specify a number of “laws”, that must be true of all types that are instances of that class. Those laws aren’t enforced by the compiler, but they _are_ sometimes used by the optimiser to, for instance, fuse loops together. Stuff like

```
map f . map g  -- before optimisation
map (f . g)    -- after  optimisation
```

where instead of building 2 lists (one for `f` and one for `g`) we build only one (for the composition of `f` and `g`).

If I were trying really hard to be negative about the Liskov substitution principle, I would stress that it only applies when inheritance is involved, and inheritance is strongly discouraged anyway. (Abstract interfaces are _not_ discouraged, but since they cannot be instantiated, the principle holds trivially for them.)

But that would bad faith: first, the applicability of the principle is very well defined, and I myself haven’t stumbled upon a single exception to the rule Liskov and Wing stated in 1994. Second, it had yet to be clear in 2000, when Martin came up with SOLID, that inheritance ought to be avoided in the first place.

## Open-closed principle

> In object-oriented programming, the open–closed principle (OCP) states “software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification”; that is, such an entity can allow its behaviour to be extended without modifying its source code.

[Wikipedia](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)

I believe that when Bertrand Meyer came up with the principle in 1988, he mostly cared about backward compatibility. For instance, when we add an argument to a function, all call sites must be updated to provide that additional argument. That’s a _modification_. But when that additional argument has a default value, as it can have in C++ or Python for instance, no update is necessary, and new call sites may specify a value for the additional argument at their leisure. That’s an _extension_.

So the thing goes: do not _modify_ your interfaces, because that would break users, and [WE DO NOT BREAK USERSPACE!](https://linuxreviews.org/WE_DO_NOT_BREAK_USERSPACE "Angry Linus") But do feel free to _extend_ your interfaces, since that wouldn’t break anyone.

Put like that, this is a very reasonable principle. At least as far as external users are concerned. Breaking compatibility is a major disservice we do to all our users, that can only be justified when the benefits are even greater, _and_ impossible to get without the break.

The Wikipedia notes that at the time, Bertrand Meyer worked in systems where merely adding a field to a [record](https://en.wikipedia.org/wiki/Record_\(computer_science\) "Wikipedia") broke users of that record. It is especially salient in languages that did not have explicit support for such records, including machine and assembly languages. In those, records where denoted by their starting address, and each field was at a fixed offset from there:

```
struct foo {
    uint32_t a;  // address == &foo + 0
    uint16_t b;  // address == &foo + 4
    uint16_t c;  // address == &foo + 6
};

struct foo f;
uint32_t *a = &f + 0;
uint16_t *b = &f + 4;
uint16_t *c = &f + 6;
```

Adding a field in the middle of the record then displaces all subsequent fields:

```
struct foo {
    uint32_t a;
    float    x;  // New field
    uint16_t b;  // Displaced!!
    uint16_t c;  // Displaced!!
};

struct foo f;
uint32_t *a = &f +  0;  // No change
uint16_t *b = &f +  8;  // It was 4!!
uint16_t *c = &f + 10;  // It was 6!!
```

The same goes when we remove a field that would otherwise no longer be needed. Some [projects](https://en.wikipedia.org/wiki/Sketchpad "Sketchpad (Wikipedia)") even left some fields blank and wasted what little precious memory they had, because updating the offsets everywhere in the program was such a major, error prone hassle.

A similar problem goes for subroutines: if your assembly routine starts requiring one additional argument from the stack, you’d have to examine the whole code base to make sure that you provide that additional argument before jumping to that modified routine. In K&R C, type checking was so primitive the compiler would not even check that function calls matched the function definition. But at least you could search for the function name.

Add to those the longer iteration times and the relative lack of version control, and you can understand why Meyer’s open-closed principle was so important: any breaking change was _risky_. It’s no wonder extension was so often preferable to modification at the time.

But that was then, and this is now.

Modern languages, such as 37 year old ANSI C, take care of offsets for us, and we can now write this (before modification):

```
struct foo {
    uint32_t a;  // Don't care about the offset
    uint16_t b;  // Don't care about the offset
    uint16_t c;  // Don't care about the offset
};

struct foo f;
uint32_t *a = &f.a;
uint16_t *b = &f.b;
uint16_t *c = &f.c;
```

And that (after modification):

```
struct foo {
    uint32_t a;
    float    x;  // New field
    uint16_t b;  // Displaced.  Meh.
    uint16_t c;  // Displaced.  Meh.
};

struct foo f;
uint32_t *a = &f.a;
uint16_t *b = &f.b;  // No change!
uint16_t *c = &f.c;  // No change!
```

The only “hassle” here is recompiling the affected parts of the code. Granted, even that is sometimes off the table, when for instance we’re updating a dynamically linked library for external users. But for internal interfaces that is no problem at all.

Likewise for function calls. In statically typed languages (which is most OOP languages these days), even adding an argument to a function is not such a big deal: sure it will break all call sites, but the compiler will tell you about every last one of them, and you can use its report to eyeball the impact of your change. And if you decide to go through with it, it’s easy to make sure you didn’t forget anything by just squashing all compile errors one by one. You might still want to re-run your test suite of course, but even without one, changes are much less risky than they used to be.

What’s left of Meyer’s open-closed principle now is little more than Linus Torvalds’ “we don’t break users”. An important principle for sure, but a much narrower one, and no longer deserving the name of “open-closed”. But that’s when it got reinterpreted. Here’s [Martin’s original article](https://web.archive.org/web/20060822033314/http://www.objectmentor.com/resources/articles/ocp.pdf "The Open-Closed Principle, Robert Martin"):

> Modules that conform to the open-closed principle have two primary attributes.
> 
> 1. They are “Open For Extension”.  
>     This means that the behavior of the module can be extended. That we can make the module behave in new and different ways as the requirements of the application change, or to meet the needs of new applications.
>     
> 2. They are “Closed for Modiﬁcation”.  
>     The source code of such a module is inviolate. No one is allowed to make source code changes to it.
>     

This goes well beyond “do not make breaking changes”, which I understand was the spirit behinds Meyer’s original principle. Now we’re not even allowed to touch a single character of the source code of our modules. Except during initial writing or to fix bugs I presume.

To deal with this made up constraint, Martin recommended an even more radical solution than Meyer’s: don’t refer to concrete classes at all, only refer to abstract _interfaces_ instead.

Instead of writing this:

```
class Server { /* ... */ };

class Client {
public:
    Client(const &Server);
    // ...
}
```

He recommends we write something like that:

```
class Abstract_server                 { /* ... */ };
class Server : public Abstract_server { /* ... */ };

class Client {
public:
    Client(const &Abstract_server);
    // ...
}
```

His justification?

> There is no guarantee that the member functions of the `Server` class are virtual.

Robert. Can I call you Robert? Your recommendation is clearly addressed at the authors of the `Server` class. How on _Earth_ can they not guarantee that its member functions are virtual? Surely if we can type a whole abstract interface full of `virtual my_method() = 0;` declarations, we can instead just type `virtual` in the `Server` class instead?

Hem.

Taking for granted the need to change which kind of server the client depends on, without changing the source code of either `Client` nor `Server`; we just need to make sure `Server` is _open for extension_, virtual methods and all, and then just write a `Server_child` that inherits from it.

I hear you object already, that inheritance is a bad feature, that’s why we need the abstract interface. But that’s not the justification Martin gives in his article. Even if he was right about how to properly inject a dependency, he was for the wrong reason, and that invites further scrutiny.

Starting with, why are we injecting the dependency to begin with? What’s wrong with a concrete class depending on another concrete class?

> If we wish for a `Client` object to use a different server object, then the `Client` class must be changed to name the new server class.

Thank you Robert. There’s just one problem: if `Client` needs to use a different server object, that means the requirements for `Client` just changed, right? What’s wrong with changing its source code to match the new requirements?

Obviously nothing. We don’t inject dependencies just to avoid renaming a couple things when requirements change. That would be premature architecture, and unless you have reason to anticipate a _specific_ kind of change, it is less risky to stick to the problem you know of right now. So that when unanticipated changes do come, you’ll have a simpler program to modify.

In my experience, the only valid justification for dependency injection, is when your `Client` needs to work with several kinds of servers in the same program. It can also make some kind of tests more convenient, but even then, tests should use the real thing when they can — mocking should not be the default, that’s how you end up testing nothing. If the problem is portability (say you use different servers on different platforms), the build system ought to be able to swap dependencies — no need to inject them at the source code level.

I don’t want to dissect Martin’s entire article here, so let me just close with this quote:

> It should be clear that no signiﬁcant program can be 100% closed. […] In general, no matter how “closed” a module is, there will always be some kind of change against which it is not closed.
> 
> Since closure cannot be complete, it must be strategic. That is, the designer must choose the kinds of changes against which to close his design. This takes a certain amount of prescience derived from experience.

Okay, so the “principle” is now a _judgement call_. But wait, there’s more:

> The experienced designer knows the users and the industry well enough to judge the probability of different kinds of changes. He then makes sure that the open-closed principle is invoked for the most probable changes.

In other words, do _not_ invoke open-closed, except for the “most probable changes”. A “principle”, that doesn’t apply most of the time. You heard it from Martin himself.

## Single-responsibility principle

> The single-responsibility principle (SRP) is a computer programming principle that states that “A module should be responsible to one, and only one, actor.” The term actor refers to a group (consisting of one or more stakeholders or users) that requires a change in the module.
> 
> Robert C. Martin, the originator of the term, expresses the principle as, “A class should have only one reason to change”.

[Wikipedia](https://en.wikipedia.org/wiki/Single-responsibility_principle)

The SRP is a cute little heuristic, but ultimately focused on the wrong thing. My suspicion here is that Martin just wanted a letter to promote high cohesion and low coupling, and “Single Responsibility” provided a much needed “S” (that could have come from “Segregated Interfaces”, but then he would have needed this one to be an “I”).

One way to achieve high cohesion and low coupling, is to try and focus each module on one single thing. Most of the time it works pretty well, but sometimes two “things” go so well together that fusing them into the same module ends up yielding a smaller API than if it was two separate modules, making the fused module quite a bit deeper and easier to use.

But that’s not SRP as stated in the Wikipedia. They say a module should be responsible to only one _actor_, defined as one group of stakeholders or users. But such groups tend to have _lots_ of requirements! Does that mean we can write classes that address all the concerns of any particular group?

Probably not, if we go by Martin’s “only one reason to change”. But that one isn’t much better: even if a module focuses on one thing, that one thing may have several reasons to change, even though it is ostensibly a single, coherent “thing”. Ultimately this would lead us to cut our programs in too many little pieces.

Maybe tiny pieces are where Martin was really getting at though: after all, he’s fond of small functions to an unreasonable degree. His code examples from _Clean Code_ (and I’ve heard, the second edition as well) are littered with functions so small some take more characters to call than to copy & paste. That’s a mistake. We don’t want our functions (or modules) small, we want them _deep_: small interface, with a significant implementation behind it. _That_ yields high cohesion and low coupling.

My advice: forget about being responsible for a single actor, or having only one reason to change. Use the following heuristic instead: “it is generally a good idea to focus your module on a single purpose each.” It’s neither a rule nor a principle. Just a handy heuristic. The real North Star remains depth. That’s how you’ll know for sure you have achieved high cohesion and low coupling.

Also note that I said “purpose”, not “thing”. Most of the time you don’t want to decompose your program around the entities of the world: mushrooms, turtles, Mario… Instead you want your module boundaries around your data transforms: input gathering, rendering, collision detection…

## Interface segregation principle

> In the field of software engineering, the _interface segregation principle (ISP)_ states that no code should be forced to depend on methods it does not use. ISP splits interfaces that are very large into smaller and more specific ones so that clients will only have to know about the methods that are of interest to them. Such shrunken interfaces are also called _role interfaces_.

[Wikipedia](https://en.wikipedia.org/wiki/Interface_segregation_principle)

As of 2025/12/30, the text of the Wikipedia does not match one of its citations. Robert Martin wrote in his [C++ report article](https://drive.google.com/file/d/0BwhCYaYDn8EgOTViYjJhYzMtMzYxMC00MzFjLWJjMzYtOGJiMDc5N2JkYmJi/view?resourcekey=0-3H2Ld4l-dIZZVRmHSpNLcA "The Interface Segregation Principle, Robert Martin"):

> Clients should not be forced to depend upon interfaces that they do not use.

_Interfaces_, not methods. And the example Martin gives is almost reasonable: given the following interfaces,

```
class Timer {
public:
    virtual void timeout() = 0;
};

class Door {
public:
    virtual void lock()    = 0;
    virtual void unlock()  = 0;
    virtual bool is_open() = 0;
};
```

we need to implement a `Timed_door`, that sounds an alarm when kept open for too long. Obviously the `Timed_door` is to implement the above interfaces. He then show what is supposed to be a common solution to this problem: have one interface inherit from the other, so the `Timed_door` can implement both:

```
class Timer {
public:
    virtual void timeout() = 0;
};

class Door: public Timer { // we're extending Timer!!
public:
    virtual void lock()    = 0;
    virtual void unlock()  = 0;
    virtual bool is_open() = 0;
};

class Timed_door : public Door {
public:
    // ...
};
```

Of course, he demolishes the stupid solution, and asserts that the ISP would have avoided the mistake. He presents two solutions, one of which uses multiple inheritance:

```
class Timer { /* ... */ };
class Door  { /* ... */ };

class Timed_door : public Door, public Timer {
public:
    // ...
};
```

Like, duh. And you will note this is possible both in C++ _and_ Java, since both `Timer` and `Door` are abstract interfaces, and in Java a class can implement any number of interfaces. So I’m not sure what the fuss was all about. Especially considering his example was also violating the Liskov substitution principle, though only in spirit (remember, the LSP trivially holds when we extend or implement an abstract interface).

Anyway, it would seem something got lost between Martin’s original article and Wikipedia. The latter reads:

> no code should be forced to depend on methods it does not use.

This is a different understanding of the ISP, but one that I found was fairly common. Under this new understanding, the following code fails the ISP:

```
class Foo {
public:
    void a();
    void b();
    void c();
};

void piece_of_code_1(Foo &f)
{
    f.a();
    f.b();
}

void piece_of_code_2(Foo &f)
{
    f.c();
}
```

The idea there is that `piece_of_code_1()` is only using methods `a()` and `b()`, `piece_of_code_2()` is only using `c()`, but both “depend on” all three methods.

It would seem the operating definition of “depend on” here, is that whenever a piece of code uses an object, it automatically “depends on” _all of its methods_. That’s unhinged. If for instance the author of `Foo` were to remove `c()`, or make breaking changes to it, then `piece_of_code_1()` would still compile and run without a hitch. Thus showing that `piece_of_code_1()` is indeed _independent from_ `c()`.

One might need to recompile some code, though. Martin cites this as a significant problem:

> But recompiles can be very expensive for a number of reasons. First of all, they take time. When recompiles take too much time, developers begin to take shortcuts. They may hack a change in the “wrong” place, rather than engineer a change in the “right” place; because the “right” place will force a huge recompilation.

Realistically though, the problem isn’t the need to recompile. It’s the slow compile times to begin with. Not everyone is cursed with C++ Template Madness from Header-Only Hell — and no one should ever have to.

The second part of his paragraph however gets close to a real issue:

> Secondly, a recompilation means a new object module. In this day and age of dynamically linked libraries and incremental loaders, generating more object modules than necessary can be a signiﬁcant disadvantage. The more DLLs that are affected by a change, the greater the problem of distributing and managing the change.

Indeed API and ABI stability are important. When you ship an update, the less it imposes on your users, the better. In quite a few settings recompilation is flat out impossible, and the only acceptable updates are the ABI compatible ones. Linus said it best:

> WE DO NOT BREAK USERSPACE!

Amen. But the ISP is not a very good way to get there. It is more effective instead to focus on module _depth_, which implies relatively small APIs: the smaller an API, the fewer reasons it will have to require breaking changes.

## Dependency inversion principle

> In object-oriented design, the dependency inversion principle is a specific methodology for loosely coupled software modules. When following this principle, the conventional dependency relationships established from high-level, policy-setting modules to low-level, dependency modules are reversed, thus rendering high-level modules independent of the low-level module implementation details. The principle states:
> 
> - High-level modules should not import anything from low-level modules. Both should depend on abstractions (e.g., interfaces).
> - Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.
> 
> By dictating that both high-level and low-level objects must depend on the same abstraction, this design principle inverts the way some people may think about object-oriented programming.

Martin is obsessed with requirements changing under his feet. He gives an example in his [C++ report article](https://web.archive.org/web/20110714224327/http://www.objectmentor.com/resources/articles/dip.pdf "The Dependency Inversion Principle, Robert Martin") of a program tasked to transfer characters from the keyboard to the printer, split into 3 modules: `Read_keyboard`, `Write_printer`, and `Copy`. It might look something like this:

```
class Read_keyboard {
public:
    int read_char();
};
class Write_printer {
public:
    void write_char();
};

class Copy {
public:
    void do_the_copy()
    {
        int c = _reader.read_char();
        while (c != -1) {
            _writer.write_char(c);
            c = _reader.read_char();
        }
    }
private:
    Read_keyboard _reader;
    Write_printer _writer;
};
```

Martin has no problem with the low-level modules, but he doesn’t like how `Copy` directly depends on them: what if we want to reuse it in another context? What if we also want to write to disk? Or read from a gazillion input devices?

His solution to the problem is to make sure we can swap one dependency for another, using abstract interfaces:

```
class Reader {
public:
    virtual int read_char() = 0;
}
class Writer {
public:
    virtual void write_char() = 0;
};

class Read_keyboard: public Reader {
public:
    int read_char();
};
class Write_printer: public Writer {
public:
    void write_char();
};

class Copy {
public:
    // Inject the dependencies there
    Copy(Reader &reader, Writer &writer)
    : _reader(reader)
    , _writer(writer)
    {}

    // Use abstract interfaces in the business logic
    void do_the_copy()
    {
        int c = _reader.read_char();
        while (c != -1) {
            _writer.write_char(c);
            c = _reader.read_char();
        }
    }
private:
    // Abstract references
    Reader &_reader;
    Writer &_writer;
};
```

Now the `Copy` class can be used with any reader and writer, isn’t that lovely?

Well… the added bloat is significant, and in my experience writing desktop, command line, and embedded applications for various industrial purposes, rarely needed:

- In the vast majority of cases, the `Copy` module would never use different dependencies, and planning for this is just a complete waste of time.
- When we do need to swap out the dependencies for portability or testing reasons, most of the time we can just recompile the `Copy` module with different dependencies, no need to complicate the source code itself.
- Quite often, as is the case in this toy example, the polymorphism is only needed for one method. We could just pass in a function instead — even an old fashion function pointer from C is less burdensome than an abstract interface.

But no, Martin had to elevate this circumstantial technique to the rank of _principle_, thus advocating for its near systematic use. And then some of his followers doubled down and insisted we use this to mock everything in the tests. So not only do we bloat the code beyond belief, we miss a ton of bugs (because guess what, mocks aren’t the real thing). This is nuts.

My advice: don’t. Let your higher-level modules depend on the lower level ones, it is _okay_ to depend on concrete implementations by default. Again, don’t plan for a change you cannot anticipate, keep your program short and simple instead. And when change does come (it always does), you’ll have a simpler program to modify.

## My advice in a nutshell

- **Liskov substitution principle:** Good. Follow unquestionably.
- **Open-closed principle:** Obsolete. Just try not to break users.
- **Single-responsibility principle:** Ignore. Focus on module depth instead.
- **Interface segregation principle:** Ignore. Focus on module depth instead.
- **Dependency inversion principle:** _Avoid._ Only inject dependencies when necessary.