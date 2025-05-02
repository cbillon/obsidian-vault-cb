---
link: https://jrsinclair.com/articles/2019/type-classes-what-i-wish-someone-had-explained-about-functional-programming/
byline: Written by James Sinclair  12th November 2019
excerpt: Type classes are not the same thing as algebraic structures. But you’ll
  find many people use the terms interchangably. And that can be confusing. It
  confused me for a long time. In this article we look at what type classes
  actually are. And we’ll also look at why programmers from other languages are
  so enthusiastic about them.
slurped: 2024-12-14T09:06
title: "Type Classes: Things I wish someone had explained about functional
  programming"
---

This is part three of a four-part series: _Things I wish someone had explained to me about functional programming._

- [Part 1: Faulty Assumptions](https://jrsinclair.com/articles/2019/what-i-wish-someone-had-explained-about-functional-programming/)
- [Part 2: Algebraic Structures](https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/)
- **Part 3: Type classes**
- [Part 4: Algebraic Data Types](https://jrsinclair.com/articles/2019/algebraic-data-types-what-i-wish-someone-had-explained-about-functional-programming/)

In the last article we discussed algebraic structures. They’re super abstract, which can make them difficult to get into. But they’re also powerful. So powerful, it’s surprising more people aren’t writing about algebraic structures everywhere. And there’s reasons for that. Sometimes people write about one algebraic structure as if they represented all structures. Monads, for example. Sometimes it’s because people don’t know what they’re called. But most often, it’s because people write about type classes instead. So, let’s talk about type classes.

## Typeclasses vs. algebraic structures

Type classes are not the same thing as algebraic structures. But you’ll find many people use the terms interchangeably. And that can be confusing. It confused me for a long time. For example, the Haskell community has a popular reference on algebraic structures. It’s called ‘Typeclassopedia.’ Why do people talk about type classes when they mean algebraic structures? The reason is, type classes are used to _implement_ algebraic structures. They’re a language feature, rather than a mathematical concept. In a languages with type classes, you’ll find they’re not used for much else. So you can understand why people might be a little loose with terminology.

It’s even more confusing if you come from a JavaScript background. JavaScript doesn’t have built-in language support for type-classes. That makes it clunky to use them (though not impossible). In the JavaScript world, we tend to talk about algebraic structures instead. And that’s OK. But let’s assume you’re serious about learning functional programming. At some point you’ll run out of good JavaScript tutorials. Eventually, you’ll need to learn from people writing about other languages. When you get there, it will help a lot to understand type classes.

## What is a type class then?

What’s a type class? In short, type classes are a way of doing _polymorphism_. And they happen to be most convenient for building algebraic structures. But to get a good feel for why they exist, let’s do a thought experiment. It’s a little round-about, but we’ll get there. Bear with me.

To start, think back to our trusty [functor structure](https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/#thefunctoralgebraicstructure). What if (in an alternate universe) we didn’t have the built-in `.map()` method for arrays? Good old `Array.prototype.map` ceased to exist. It would be inconvenient. But not for long. It wouldn’t be hard to get our `.map()` method back. We could write our own:

```
Array.prototype.map = function map(f) {
    const out = [];
    for (let x of this) {
        out.push(f(x));
    }
    return out;
};
```

Not too hard, was it? And now, let’s look at another functor. Here’s a `.map()` method for [Maybe](https://jrsinclair.com/articles/2016/marvellously-mysterious-javascript-maybe-monad/):

```
Maybe.prototype.map = function(f) {
    if (this.isNothing()) {
        return Maybe.of(null);
    }
    return Maybe.of(f(this.__value));
};
```

So far, nothing radical going on here. But let’s take this thought experiment a little further. Imagine we wanted to use functions instead of methods to make functors. As in, we’d like to create functors like Maybe and Array, but not use methods at all. Plain functions. No `this`. (This is not an unreasonable idea at all, by the way).

Could we do it? Well, yes. Of course we could. All we do is take `this` or `this.__value` and make it a parameter. And so our two map functions might look like so:

```
// Map for arrays.
function map(f, xs) {
    const out = [];
    for (let x of xs) {
        out.push(f(x));
    }
    return out;
};

// Map for Maybe.
function map(f, x) {
    if (x.isNothing()) {
        return x;
    }
    return Maybe.of(f(x.__value));
};
```

Except, now we have a problem. This code above won’t work. JavaScript won’t let us have two functions called `map` in the same scope. One will overwrite the other. Instead, we either use methods, _or_ rename our functions. For example:

```
// Map for arrays.
function arrayMap(f, xs) {
    const out = [];
    for (let x of xs) {
        out.push(f(x));
    }
    return out;
};

// Map for Maybe.
function maybeMap(f, x) {
    if (x.isNothing()) {
        return x;
    }
    return Maybe.of(f(x.__value));
};
```

If you’re used to JavaScript, this makes sense. You can’t have two functions with the same name in the same scope. But in a language like Haskell, it’s different.

Why? Because of types. Haskell has a ‘static’ type system. JavaScript has a ‘dynamic’ type system. In JavaScript, there’s no way for the computer to tell that `map` for array is different from `map` for Maybe. But in Haskell, the type signatures for those two functions are different. They might look something like this:

```
-- Type signature of map for arrays/lists.
map :: (a -> b) -> [a] -> [b]

-- Type signature of map for Maybe
map :: (a -> b) -> Maybe a -> Maybe b

```

Two different type signatures. Because the types are different, Haskell’s compiler can figure out which `map` to call. It can look at the arguments, figure out their types, and call the correct version. And so the two versions of `map` can exist side-by-side. (Unlike in JavaScript).

Languages with this feature use it to create algebraic structures. We can say, for example, “I’m going to create a new instance of Functor. Here’s its `map` function.” In code, it might look like this: [1](https://jrsinclair.com/articles/2019/type-classes-what-i-wish-someone-had-explained-about-functional-programming/#notes-fn-2)

```
instance Functor List where
    map :: (a -> b) -> [a] -> [b]
    map f xs = foldl (\x arr -> arr ++ [f x]) [] xs
```

And we could declare Maybe a functor too:

```
instance Functor Maybe where
    map :: (a -> b) -> Maybe a -> Maybe b
    map f (Just a) = Just f a
    map _ Nothing  = Nothing
```

Don’t worry if all that Haskell is gobbledygook. All it means is we can define different versions of `map` for different types. This language feature is built into Haskell. And it lets us declare a name for these _things-that-can-be-mapped_. In this case, Functor.

Languages providing this feature call this thing-you-can-create-an-instance-of, a type class. And type classes are often used to create algebraic structures. But that’s not the only thing you can do with them. What type classes do is enable a specific kind of polymorphism. That is, they let us use the same ‘function’ with different types. _Even if we don’t know up front what those types might be_. And that happens to be a convenient way to define algebraic structures.

Now, if you’re paying careful attention, you may have noticed that keyword `instance`. It’s in both the Haskell code blocks above. And you may well wonder: An instance of what? How do we declare a new type class? In Haskell, the definition for functor looks something like this:[2](https://jrsinclair.com/articles/2019/type-classes-what-i-wish-someone-had-explained-about-functional-programming/#notes-fn-3)

```
class Functor f where
    map :: (a -> b) -> f a -> f b
```

This code says we’re creating a new _type_ class called ‘Functor’. And we use the shortcut `f` to refer to it in type definitions. For something to qualify as a functor, it must have a `map` function. And that `map` function must follow the given type signature. That is, `map` takes two parameters. The first is a function that takes something of type `a` and returns something of type `b`. The second is a functor of type `f` with something of type `a` ‘inside’ it.[3](https://jrsinclair.com/articles/2019/type-classes-what-i-wish-someone-had-explained-about-functional-programming/#notes-fn-4) Given these, `map` must return another functor of the same type `f` with something of type `b` ‘inside’ it.

_Whew_. The code is much easier to read than the explanation. Here’s a shorter way to say it: This is a type class called functor. It has a `map` function. It does what you’d expect `map` to do.

Again, don’t worry if all that Haskell code doesn’t make sense. The important thing to understand is that it’s about polymorphism. This particular kind is called _parametric polymorphism_. Type classes let us have many functions with the same name. That is, so long as those functions handle different types. In practice, it allows us to think of all those map functions as if it were one single function. And the `Functor` definition makes sure that they all do logically similar tasks.

## Type classes and JavaScript

JavaScript doesn’t have type classes. At least, it has no built-in language support for them. It _is_ possible to create type classes in JavaScript. You can see an example in this [type class implementation based on Sanctuary](https://github.com/sanctuary-js/sanctuary-type-classes). If you look closely, you’ll notice that we have to do a bunch of the work to declare them. This is work that the compiler would do for us in a language like Haskell. For example, we’re required to write a predicate function for each type class instance. That predicate determines whether a value can work with the type class we define. In other languages, the compiler would take care of that. Most of the time though, a library author does that work, not the end user. So it’s not as tedious as it might seem.

In practice, almost nobody uses type classes in JavaScript. Which makes me sad. I do wish they were more popular. But for now, the reality is that type classes aren’t practical for most code bases. But all is not lost. We still have polymorphism, even if it’s not _parametric_ polymorphism. Instead of type classes, we use prototypical inheritance. This lets us pass around a bunch of methods along with a value. As a result, we can write a map function (as opposed to a method) that works like this:

```
const map = (f, x) => x.map(f);
```

As long as `x` has a `.map()` method that obeys the functor laws, this will work just fine. And we achieve much the same thing as type classes. This is what makes libraries like [Ramda](https://ramdajs.com/), [Sanctuary](https://sanctuary.js.org/) and [Crocks](https://crocks.dev/) so powerful. It’s also another reason why that [Fantasy Land](https://github.com/fantasyland/fantasy-land/blob/master/README.md) specification is so important. It gives us all that wonderful polymorphic goodness.

That said, type classes have their advantages. For example, Haskell can refuse to compile if it knows we haven’t defined `map` somewhere. JavaScript though, doesn’t know until it runs the code (often in production).

## Is this article a waste of time?

Well, it _is_ a waste of time if you’re looking for quick tips to write better JavaScript code. This article won’t help you with that. But this series isn’t about quick practical tips. It’s about helping you help yourself. My aim is to help people avoid the traps I fell into. One of those traps was not understanding type classes. And not understanding how they’re different from algebraic structures. It’s my hope that this will help you understand what others are talking and writing about as you explore.

So, we’ve got a handle on algebraic structures and type classes. But the confusing terminology doesn’t stop there. You might think that _algebraic data types_ is another name for algebraic structures. I did. But no. They’re something different again. [Algebraic data types](https://jrsinclair.com/articles/2019/algebraic-data-types-what-i-wish-someone-had-explained-about-functional-programming/) will be the topic of the next article.

---

Enormous thanks to [Jethro Larson](https://twitter.com/jethrolarson), [Joel McCracken](https://twitter.com/joelmccracken) and [Kurt Milam](https://twitter.com/kurtmilam) for reviewing an earlier draft of this entire series. I really appreciate the feedback and suggestions.