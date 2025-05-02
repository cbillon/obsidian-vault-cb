---
link: https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/
byline: Written by James Sinclair  3rd November 2019
excerpt: "Algebraic Structures are something I wish I’d understood better,
  sooner. I had a hazy idea of what they were, but didn’t know the correct
  terminology. That was a massive barrier to finding out more. This article is
  my attempt to stop that happening to others. We’ll look at: What are algebraic
  structures? How do we use them in JavaScript? Why would we bother? What’s the
  big deal?"
slurped: 2024-12-14T09:05
title: "Algebraic Structures: Things I wish someone had explained about
  functional programming"
---

This is part 2 of a four-part series: _Things I wish someone had explained to me about functional programming_

- [Part 1: Faulty Assumptions](https://jrsinclair.com/articles/2019/what-i-wish-someone-had-explained-about-functional-programming/)
- **Part 2: Algebraic Structures**
- [Part 3: Type classes](https://jrsinclair.com/articles/2019/type-classes-what-i-wish-someone-had-explained-about-functional-programming/)
- [Part 4: Algebraic Data Types](https://jrsinclair.com/articles/2019/algebraic-data-types-what-i-wish-someone-had-explained-about-functional-programming/)

Algebraic Structures are something I wish I’d understood better, sooner. I had a hazy idea of what they were, but didn’t know the correct terminology. That was a massive barrier to finding out more.

## What is an algebraic structure?

What is an algebraic structure? Well, according to Wikipedia:

> In mathematics, and more specifically in abstract algebra, an **algebraic structure** on a set AA (called **carrier set** or **underlying set**) is a collection of finitary operations on AA; the set AA with this structure is also called an **algebra**.[1](https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/#notes-fn-1)

…and… that doesn’t help much. Sets? Finitary operations? What does that have to do with writing code? We’re trying to learn about functional _programming_ here. What do algebraic structures have to do with anything?

Well, let me ask you a question. Have you ever been around more experienced functional programmers? Ever heard them throwing around a bunch of inscrutable jargon? Words like ‘monoid,’ ‘applicative,’ ‘semiring,’ ‘lattice,’ ‘functor,’ or the dreaded ‘monad’? Ever wondered what all that was about? The collective term for these concepts is _algebraic structures_.

It took me a long time to figure this out. And even once I did, it didn’t help as much as I’d hoped. In IT, there’s always someone ready to criticise incorrect terminology. They’re like hyenas waiting to jump on an unguarded kill. And the functional programming community is no exception. Knowing the name ‘algebraic structure’ helps protect yourself from that. But not much else. If you do a web search for ‘algebraic structures’, you won’t get helpful results back. And qualifying it with ‘JavaScript algebraic structures’ doesn’t improve things much.

There’s a reason for the paltry search results. But we’ll come back to that in a later article. For now, let’s try and understand what algebraic structures are about.

If you’ve read this far, perhaps you’ve read some of my earlier articles. To be specific, the ones about [Maybe](https://jrsinclair.com/articles/2016/marvellously-mysterious-javascript-maybe-monad/), [Either](https://jrsinclair.com/articles/2019/elegant-error-handling-with-the-js-either-monad/), and [Effect](https://jrsinclair.com/articles/2018/how-to-deal-with-dirty-side-effects-in-your-pure-functional-javascript/) (also known as ‘IO’). We use Maybe, Either, and Effect for different purposes:

- _Maybe_ helps us deal with `null` or `undefined` values;
- We can use _Either_ to handle errors; and
- _Effect_ gives us control over side effects.

Each one serves a useful purpose.

You might also notice that we often create them using objects. And these objects have methods with names in common. For example, Maybe, Either, and Effect all have a `.map()` method. Each one also has `.ap()` and `.of()` methods. And all three have `.chain()` too. This is not a coincidence. They’re following a pattern—three patterns, to be precise. And these patterns are (you guessed it) _algebraic structures._

But, what _are_ they? You may have come across [design patterns](https://en.wikipedia.org/wiki/Software_design_pattern) before. They describe, well, _patterns_ that we see repeated in code. According to Wikipedia:

> [Design patterns are] not a finished design that can be transformed directly into source or machine code. It is a description or template for how to solve a problem that can be used in many different situations.

Software design patterns were popularised by a bunch of smart people. They observed common approaches to programming problems and then wrote books about it. Like design patterns, algebraic structures also represent templates for solving a problem. And they can be used in many different situations. _Unlike_ design patterns though, algebraic structures have their basis in mathematics. They’re not based on general observation alone. In practice, this means that they tend to be more formally defined and more general. They also have specific laws they must conform to.

Counter to intuition, the laws don’t make algebraic structures more restrictive. Instead, they tend to be the kind of thing you look at and think “well, duh”. But having them there means that we can make deductions and assumptions about how the code works. And that in turn allows us to make optimisations and refactor code safely. Even better, we can write tools that will make the computer do those for us. But we’ll come back to that.

## Algebraic Structures in JavaScript

Let’s take a look at algebraic structures in JavaScript. We have a specification for algebraic structures called [Fantasy Land](https://github.com/fantasyland/fantasy-land/blob/master/README.md). It assumes that we’re going to write our algebraic structures using classes and objects. (Though, that’s not the only way to do it).

For each structure, the specification lists methods an object must have to comply. The methods must also:

1. Conform to a specific type signature (even though it’s JavaScript); and
2. Obey some laws.

Now, I don’t recommend going and reading the specification. It’s written for library authors, not for regular programmers.[2](https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/#notes-fn-3) The explanations there don’t explain what the structures are _for_. That is, the specification doesn’t tell you what problems the structures solve. But it does tell us the laws for each structure, and gives us a consistent naming convention.

So, in Fantasy Land, an algebraic structure is an object. But the object has to have some specific methods. Those methods must match a given naming convention and specific type signatures. And each method also has to obey some laws.

Sounds super abstract, I know. The definition is kind of dry and boring. Bear with me. What we can _do_ with algebraic structures is much more interesting. Let’s look at an example.

## The functor algebraic structure

‘Functor’ is an algebraic structure—often the first one people learn. The functor structure must have a `.map()` method with the following type signature:

```
map :: Functor f => f a ~> (a -> b) -> f b
```

I’ve written the signature above in the [Hindley–Milner notation](https://jrsinclair.com/articles/2016/gentle-introduction-to-functional-javascript-style/#hindley-milnertypesignatures) Fantasy Land uses. In TypeScript, it might look like this:

```
interface Functor<A> {
    map<B>(f: (a: A) => B): Functor<B>;
}
```

Let’s break that down. Both type signatures say something like the following:

- The `.map()` method takes a function as an argument.
- That function must take something of type `a` and transforms it into something of type `b`. The types `a` and `b` can be anything—even the same type.
- And when you call `.map()` on a functor of `a`, you’ll get back a functor of `b`.

If you’ve read about Maybe, Either, and Effect, this will be familiar. This `.map()` method takes an ordinary function and makes it work with functors.

There’s more to the specification though. Functor has two laws, as follows:

1. If `u` is a functor, then calling `u.map(x => x)` must be equivalent to `u`. This is the ‘identity law.’
2. If `u` is a functor, and `f` and `g` are functions, then calling `u.map(x => f(g(x)))` is equivalent to calling `u.map(g).map(f)`. This is the ‘composition law.’

If our class (or object) satisfies all these criteria, then we can call it a functor.

You may have noticed that Array has a `.map()` method. And if you care to check, you’ll find that it obeys the functor laws. Hence it’s safe to declare Array a functor. But Array is not the only functor. As discussed, Maybe, Either, and Effect are functors too. They each have a `.map()` method and they obey the laws.

Functor isn’t the only algebraic structure either. There’s a bunch of them. You can read all the method names, type signatures and laws in the [Fantasy Land Specification](https://github.com/fantasyland/fantasy-land/blob/master/README.md). But, as mentioned, I don’t recommend starting there. Instead, check out Tom Harding’s excellent series: ‘[Fantas, Eel, and Specification](http://www.tomharding.me/fantasy-land/).’ It runs through specific examples of how to use algebraic structures in JavaScript.

## What’s the point of algebraic structures?

Now, if you made it through all that without nodding off, I commend you. And you may well be wondering, ‘What’s the point?’ Why do we care that someone wrote down a bunch of rules in a specification?

That’s a good question. Because, on their own, these algebraic structures don’t do much of anything. Sure, they might be interesting to mathematicians. But what good are they to working programmers?

Well, as we said, algebraic structures don’t do much of anything on their own. They’re just abstract descriptions. It’s not until we create _instances_ like Maybe, Either or Effect that we can do anything useful. And we don’t _need_ a specification to make these work. It would be no trouble to call `.map()` another name. For example, we could rename `.map()` to `.try()` for Either and Maybe. It might be easier to understand that way. Or change Array’s `.map()` method to `.select()`. There’s nothing special about the names. So what does a specification for algebraic structures give us? Why bother conforming?

Take a step back with me and consider something. Notice that we called Maybe, Either, and Effect _instances_ of algebraic structures. That’s a bit odd. Maybe, Either, and Effect are classes.[3](https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/#notes-fn-2) It’s unusual to talk about classes as instances. It’s more common to talk about _objects_ as instances of a _class_. Classes are normally the abstraction, and objects are the concrete _thing_ we use to get stuff done. But we’ve started talking about _classes_ as an instance of something.

Let’s think about that. Why do we use classes for anything? Because they abstract common behaviour. That behaviour is shared between a bunch of objects. Algebraic structures, in turn, abstract common patterns shared between a bunch of classes. At least, that’s one way to think about it.

How does this help us? In two ways:

1. Algebraic structures help us in the same way all other abstractions help us. They hide some detail so we can think clearly about the bigger picture. Once you learn a handful of instances like Array, Maybe, Effect, etc. then you begin to see the pattern. That makes it easier to learn other instances that share the same pattern. And it gives us a precise way to communicate with other programmers. Clarity of thought and precise communication. There’s legitimate value here, regardless of how hand-wavy it might sound.
2. There’s more concrete benefits too though. We said earlier that algebraic structures are based in mathematics. We can make that math do work for us. The specifications include laws—mathematical laws. We can take advantage of those laws to make the computer derive code for us. Compilers can use those laws to optimise our code. And do so with mathematical certainty that we’ll still get the same result.

This second point is worth exploring further. Let’s try it out with Functor. One of the functor laws is the composition law. It says that mapping twice is the same as mapping a function composed of two other functions. That is:

```
// Here, ≣ is an operator I’ve made up to signify ‘is equivalent to’
a.map(g).map(f) ≣ a.map(x => f(g(x)))
```

Now, imagine `a` is an array with millions of elements. Both sides of the equation above will produce a result. But the one on the left will be slower and use a lot more memory. That’s because most JS engines will create an intermediate array for `a.map(g)` before mapping `f`. On the right-hand side though, we do all the calculations in a single pass. Let’s assume we know for certain that `f` and `g` are pure functions. In that case, a compiler can swap the left side for the right with complete safety. We get performance improvements ‘for free’.

Similarly, we can have the computer derive functions for us. For example, imagine we’re working with modern JS. Our Array prototypes have `.flatMap()` defined. And `.flatMap()` looks eerily similar to [Fantasy Land’s `.chain()`](https://github.com/fantasyland/fantasy-land/blob/master/README.md#chain-method). Similar enough that we can treat them as equivalent. And because _math_, the algebraic structures let us derive another function, `ap()`, ‘for free’. One implementation might look like this:

```
function ap(m) {
    return m.flatMap(f => this.map(f));
}
```

Now, this implementation ([stolen from the Fantasy Land specification](https://github.com/fantasyland/fantasy-land/blob/master/README.md#derivations)) has a `this` in it. That means we’re supposed to attach it to the prototype of our class. For an array that would be something like this:

```
Array.prototype.ap = function ap(m) {
    return m.flatMap(f => this.map(f));
};
```

But this is a big no no. Modifying the prototypes of built-in objects is dangerous. It’s kind of like nuclear weapons. It’s fine, as long as nobody else uses them. But as soon there’s a chance of other people using them, then we’re all in danger. Who knows when someone might blow us all up? Hence we all agree not to muck around with that kind of thing. And that’s OK, because we can attach `.ap()` to any individual array we want. It won’t bother anybody else (as long as you’re not using IE6). Or, we can use `Function.prototype.call` to tell the computer what `this` should be. That might look like this:

```
const bases = ['ice cream', 'banana', 'strawberry'];
const toppings = ['nuts', 'chocolate sauce', 'sprinkles'];
const combine = a => b => `${a} with ${b}`;
const basesWith = bases.map(combine);
const combos = ap.call(toppings, basesWith);
console.log(combos);
// ["ice cream with nuts", "ice cream with chocolate sauce", "ice cream with sprinkles", "banana with nuts", "banana with chocolate sauce", "banana with sprinkles", "strawberry with nuts", "strawberry with chocolate sauce", "strawberry with sprinkles"]
```

Now, the Fantasy Land specification calls this `.flatMap()` method `.chain()`. As a result, we lose a little bit of interoperability there. But that’s OK too. It’s not hard to adjust the derivation so that it could work with either name.

```
function chainOrFlatMap(x) {
    return (typeof x.chain === 'function')   ? x.chain.bind(x)   :
           (typeof x.flatMap === 'function') ? x.flatMap.bind(x) :
           () => {throw new Error('We received an object that doesn’t have chain or flatMap defined')};
}

function ap(m) {
    return chainOrFlatMap(m)(f => this.map(f));
}
```

What’s the point of this though? We wrote that function ourselves. The computer didn’t write it for us. That’s true. But some other languages have better support for algebraic structures. And in those languages the compiler _will_ write that code for you. And yet, even though we wrote that code ourselves, it’s still useful. Notice that there’s nothing specific to arrays or Maybe or Either or anything else in that code. All it needs is `.map()` and `.flatMap()` (or `.chain()`). This code will work with anything that implements those methods and obeys the laws. _Anything_. It will work for arrays, Either, Maybe, Effect, Future, and so on. _With no change_.

It gets better though. Because we can then write our own functions that use `.map()`, `.ap()` and `.chain()`. If those methods are all we rely on, our new functions will work anywhere too.

Write once. Run in a bunch of different scenarios. That’s the promise of algebraic structures. Need a function to run even if we might have to deal with `null`? Stick it in a Maybe. Need a function that works with a value that we don’t have yet? Perhaps it will come back from an HTTP request some time in the future. No problem, stick it in a [Future](https://github.com/fluture-js/Fluture). Need precise control over when side-effects happen? The same code will work in an Effect too. Plus ‘free’ performance optimisations and other pre-written code. Algebraic structures make all this possible. Hence why they called the algebraic structure specification for JavaScript ‘Fantasy Land’. It sounds, well, like a fantasy.

## Where are all the blog posts?

If algebraic structures are so fantastic though, where are all the blog posts? Why doesn’t a search for ‘algebraic structures’ turn up hundreds of articles? Where are all the programmers talking about how wonderful algebraic structures are?

There _are_ lots of blog posts about how wonderful algebraic structures are. But there’s a couple of reasons why they don’t show up in search results.

1. A lot of people write about algebraic structures but don’t call them that. Instead, they’ll use one structure to stand in for all structures. For example, they might write a post or give a talk about why monads are great. Or how wonderful functors are. And that’s fine. But it means fewer articles about algebraic structures showing up in search engines.
2. The authors of these posts tend to come from languages like Haskell, PureScript, and Scala. These languages have an alternate way of creating algebraic structures. They don’t use classes and objects. Instead, they use something called ‘type classes’. And you’ll find plenty of tutorials on how great type classes are.

So, in the next article, we’ll talk about type classes. Stay tuned… In the meantime, please do go and read [Tom Harding’s series on Fantasy Land](http://www.tomharding.me/fantasy-land/), it’s really good.

---

Enormous thanks to [Jethro Larson](https://twitter.com/jethrolarson), [Joel McCracken](https://twitter.com/joelmccracken) and [Kurt Milam](https://twitter.com/kurtmilam) for reviewing an earlier draft of this entire series. I really appreciate the feedback and suggestions.