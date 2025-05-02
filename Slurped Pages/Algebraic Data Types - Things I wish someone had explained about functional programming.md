---
link: https://jrsinclair.com/articles/2019/algebraic-data-types-what-i-wish-someone-had-explained-about-functional-programming/
byline: Written by James Sinclair  19th November 2019
excerpt: Algebraic data types and algebraic data structures sound similar. It’s like they ought to be the same thing. But they’re not. They both have ‘algebraic’ in the name, so it’s confusing. I got them mixed up at times. Others have too. But, they’re different concepts. Understanding the difference will help if you’re trying to learn functional programming.
slurped: 2024-12-14T09:06
title: "Algebraic Data Types: Things I wish someone had explained about functional programming"
tags:
  - functionnal
  - data_types
  - functionnal-programming
auteur: James Sinclair
---

This is part four of a four-part series: _Things I wish someone had explained to me about functional programming._

- [Part 1: Faulty Assumptions](https://jrsinclair.com/articles/2019/what-i-wish-someone-had-explained-about-functional-programming/)
- [Part 2: Algebraic Structures](https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/)
- [Part 3: Type classes](https://jrsinclair.com/articles/2019/type-classes-what-i-wish-someone-had-explained-about-functional-programming/)
- **Part 4: Algebraic Data Types**

Algebraic data types and algebraic data structures sound similar. It’s like they ought to be the same thing. But they’re not. They both have ‘algebraic’ in the name, so it’s confusing. I got them mixed up at times. Others have too. But, they’re different concepts. Understanding the difference will help if you’re trying to learn functional programming.

We’ve already talked about [algebraic structures](https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/). What are algebraic data types then?

## What’s an algebraic data type?

An algebraic data type is a structured type that’s formed by composing other types. Or, even shorter, it’s a type made of other types. That’s it. Not super complicated at all. If you’re a working programmer, you use them already every day.

And after reading this, you might think: ‘What’s so special about that? In JavaScript, that’s arrays and objects, right? Why does that even need a special name?’ And you’d be right. Arrays, Objects, Maps, WeakMaps, and Sets are all algebraic data types. You can put values of many types ‘inside’ them (so to speak). No big deal. But, you might also notice a functional programmer over there. The one with a smug look and condescending smile on their face. And after bracing yourself, you ask them “What’s so amusing?”

“Oh, you’re so cute,” they say, “Those are all product types. In functional programming we use _sum_ types to model the business domain. That’s where algebraic data types really shine.”

Now, let’s assume you manage to refrain from physical violence. After taking a deep breath, you might wonder: What are sum and product types? And why are they called that?

We’ll start with product types, because they’re the most familiar.

## Product types

Product types allow you to have more than one value in a single structure, _at the same time_. We already discussed how this includes things like arrays, objects, maps, and sets. JavaScript doesn’t restrict what you can shove in an array or an object. You can shove an object into an array, and more arrays into that object. There’s endless possibilities.

I mean that last sentence quite literally. Emphasis on _possibilities_. Why? Well, you might be wondering: Why do we call them _product_ types? It’s all about the _possibilities_.

To understand why, let’s write some code. We’re going to create a special type that can only contain two boolean values. It might look something like this:

```
class BoolTuple2 {
    constructor(a, b) {
        if ((typeof a !== 'boolean' ) || (typeof b !== 'boolean')) {
            throw new Error('Bool2Tuple must have exactly two boolean values');
        }
        this.a = a;
        this.b = b;
        Object.freeze(this);
    }
}
```

We’ve created a new product type. Now, how many ways do we have to create a valid `BoolTuple2` object?

The answer is four. We can write them out:

```
const tt = new BoolTuple2(true, true);
const tf = new BoolTuple2(true, false);
const ft = new BoolTuple2(false, true);
const ff = new BoolTuple2(false, false);
```

There aren’t any more values we can stick into `BoolTuple2`. It contains two booleans: No more, no less. And each boolean can have two values: `true` or `false`. So, we get 2 ⨉ 2 = 4 total values.

Now, let’s create another type that can only contain digits.

```
class Digit {
    constructor (c) {
        const validDigits = '1234567890';
        if ((c.toString().length > 1) || (!validDigits.includes(c.toString()))) {
            throw new Error('Digit can only contain single digits');
        }
        this.value = parseInt(c, 10);
        Object.freeze(this);
    }
}
```

There’s precisely ten valid integers we can store in `Digit`. We have one value slot, and 10 valid characters. 1 ⨉ 10 = 10.

Perhaps you can see where this is going already. But one more example to drive it home. Let’s create a composite type stores two booleans and a digit.

```
class Composite {
    constructor(boolTuple, digit) {
        if (!(boolTuple instanceof BoolTuple2) || !(digit instanceof Digit)) {
            throw new Error('Composite must be created with a BoolTuple2 instance and a Digit instance');
        }
        this.a = boolTuple.a;
        this.b = boolTuple.b;
        this.c = digit.value;
        Object.freeze(this);
    }
}
```

Now, how many possible valid values can our `Composite` type store? We could write it out, but there’s no need. We can calculate it since we know how many values each of the slots can have. We have three value slots: `a`, `b`, and `c`. The first two, `a` and `b`, can only have two values each. And the third value, `c`, can only have 10 possible values. So we can calculate: 2 ⨉ 2 ⨉ 10 = 40. Our `Composite` type can have 40 possible values.

And that, in short, is why we call them product types. Because we can calculate the number of possible values using multiplication. In practice, it’s not terribly useful to know. That is, ironically, unless you’re using a lot of sum types. As we’ll see, sum types generally have a limited range of values. In that case, it can be incredibly useful to know the precise number of combinations. It’s a way of making assertions about your code with mathematical certainty.

So, let’s talk about sum types.

## Sum types

Sum types are types where your value must be one of a fixed set of options. You may have seen `enum` types in languages like C# or Java. Sum types are similar, but more flexible. And if you’ve been following along, we’ve already seen a classic sum type: [Either](https://jrsinclair.com/articles/2019/elegant-error-handling-with-the-js-either-monad/).

The Either type can hold a Left or Right value, but never both at the same time. And it has functions that behave differently depending on which side the data lies. If we map over a Left, it does nothing. If we map over a Right, then we get a new Right with a (potentially) different value.

What’s interesting here is that we can put any data we want in Either. We can put an array or an object (or whatever) in the Left or Right slots. So you might be wondering how this is any different from product types. And the difference is that with Either, the data must be inside _either_ Left or Right. So, the data is always ‘tagged’ with a piece of information, encoded into the type. What the tag means is entirely up to us. For example, with Either, we might use the Left side to represent errors. If the data is inside Left, something has failed. If we’ve got a Right, then we’re still on the happy path. But they’re called Left and Right for a reason. We don’t _have_ to make Left represent errors. It might represent something else. For example, we might decide that Right represents ‘logged in’. And the Left represents ‘unauthenticated.’ It’s entirely up to us what we use Either for.

So, Either has two possible tags: Left or Right. But there’s no reason we can’t create a type that has more tags. For example, we could have a type that models directions for a game. We might create tags for North, South, East, and West. Or, in a front-end application we might define states for a UI widget loaded via network request. It might have states like `Loading`, `Display`, and `LoadingError`.

That’s all fine. But why are they called sum types? Well, if the value can only be one of a fixed set of options, then we count the possible values using _addition_. For example, booleans are a sum type. They can be either true or false. Not both at the same time. And we count the values by addition 1 + 1 = 2.

To give another example, let’s create a type that can hold either a boolean _or_ a digit:

```
class BoolOrDigit {
    constructor(val) {
        if ((typeof val !== 'boolean') && !(val instanceof Digit)) {
            throw new Error('BoolOrDigit can only hold either a boolean or digit');
        }
        if (typeof val === 'boolean') {
            this.value = val;
        }
        if (val instanceof Digit) {
            this.value = val.value;
        }
        Object.freeze(this);
    }
}
```

How many possible values can our `BoolOrDigit` type hold? Well, we know that it can be either a boolean (two values) or a Digit (ten values): 2 + 10 = 12. We count the possible values using addition. Hence the name ‘sum types’.

## Sum types in JavaScript

Now, you may be thinking, so what? Who cares about counting the number of possible values? As soon as you’ve got a string or number in there, it’s all infinity anyway. What’s the point? Are sum types of any practical use in JavaScript? And if so, how do we make them?

As you may have guessed, JavaScript’s languge features focus mainly on product types. It doesn’t have a lot of built-in features for sum types. But that doesn’t make sum types useless. It just means they’re not as convenient in JavaScript as they might be in some other languages. But we can still do it.

So, how do we make sum types? The good news is, JavaScript’s flexibility means there’s lots of ways to do it. The bad news is, JavaScript’s flexibility means there’s lots of ways to do it. One way is to use class inheritance, like we do with Either. Let’s imagine we’re creating some kind of UI widget. It’s loaded in with an asynchronous network call. So there’s three possible states it might be in: Loading, Error, or Display. Modelling the states might look something like this:

```
export class CardState {
    constructor() {
        if (this.constructor.name === 'CardState') {
            throw new Error(
                'CardState is an abstract class. You can’t have an instance of it',
            );
        }
    }
}

export class Loading extends CardState {}

export class LoadingError extends CardState {
    constructor(err) {
        super();
        this.err = err;
    }
}

export class Display extends CardState {
    // Example data:
    // {
    //     fullName: 'James Sinclair',
    //     companyName: 'jrsinclair.com',
    //     avatarUrl: 'http://example.com/james-avatar.jpg',
    //     email: 'james@example.com',
    // }
    constructor(data) {
        super();
        this.data = data;
    }
}

```

That defines our types, but how might we use them? Well, if we’re using React, we could do something like this:

```
import React, {useState, useEffect} from 'react';
import Spinner from '@atlaskit/spinner';
import {ProfileCard} from '@atlaskit/profilecard';

import { CardState, Loading, LoadingError, Display } from "./states";

// Simple function to build a URL.
function apiURL(email) {
  return `/data.json?email=${encodeURIComponent(email)}`;
}

// Load the data.
function loadData(email, setState) {
  fetch(apiURL(email))
    .then(response => response.json())
    .then(data => setState(new Display(data)))
    .catch(err => setState(new LoadingError(err)));
}

// A component to display if we have an error.
const ErrorCard = () => (
  <ProfileCard errorType={{ reason: "default" }} hasError />
);

// Our display logic. We expect the state argument to be
// a CardState instance
function renderComponent(state) {
  // Here we check the type of state. If it's not an instance
  // of CardState, something is wrong.
  if (!(state instanceof CardState)) {
    throw new Error('State must be an instance of CardState');
  };

  // Create a mapping between  states and React elements to display.
  const stateMap = {
    Loading: () => <Spinner />,
    LoadError: () => <ErrorCard />,
    Display: s => <ProfileCard {...s.data} />,
  };

  // Select which element to render, based on current state
  // or fall back to an error card.
  const component = stateMap[state.constructor.name] || (() => <ErrorCard />);
  return component(state);

}

// Our component with everything tied together.
export function UserCard({ email }) {

  // Initialise our hooks.
  const [state, setState] = useState(new Loading());
  useEffect(() => loadData(email, setState), [email]);

  return renderComponent(state);
}
```

[See a working version in codesandbox.io](https://codesandbox.io/s/sum-types-for-react-state-gfjvh)

Now, you might be thinking “You made a React component with a loading state. Big deal. I can do the same thing with a couple of if-statements.” And that’s fair. This isn’t a particularly amazing piece of code. What’s important is that we’ve engineered it so that our component can _only_ ever be in one of three states. There’s no possible state where it’s both loading and there’s data. There’s no possible state where there’s an error message but it’s still loading. The `state` variable must be a `CardState` instance. And there’s only three possible instance types. We’ve encoded these impossibilities into our display logic. That’s why functional programmers get so excited about sum types.

That said, you may have noticed that my `renderComponent()` function is a little clumsy. I’m doing a manual type check and using an object as a map between state and components. Maybe a `switch...case` statement would make it slightly better. But only slightly.

You may also notice that functional programmer from earlier still looking smug. That’s because languages like Haskell they have a feature called pattern matching. That type checking and the fallback case I’ve included are really just noise. (e.g. the bit with `|| (() => <ErrorCard />`) .With pattern matching and type checking you don’t have to add that fallback. The compiler can look at the types and tell you if you’ve missed a case. And if you have covered all the cases, the compiler can guarantee that you’ll never get an unknown type. These compiler features make sum types convenient to work with.

Pattern matching, in particular, offers features a bit like destructuring. This lets you pull bits of data out from the left-hand-side of the pattern match. Again, making sum types really convenient to work with. So, sadly, the functional programmer is somewhat justified in looking smug here. [TypeScript](http://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-0.html#tagged-union-types) and [Flow](https://flow.org/en/docs/types/unions/#toc-disjoint-unions) have some support for algebraic data types. But they’re both fairly limited.

So, algebraic data types are not the same thing as algebraic structures. But they’re both useful. And it’s good to know the difference. It seems the smug functional programmer was right. Algebraic data types are handy for encoding business logic. Sometimes what we make _impossible_ is just as important as what we make possible.

## Review

We’ve covered a lot of ground in this series. Let’s try and sum it all up.

- Learning functional programming is hard. Some of the difficulty is inherent to the subject. A lot of it isn’t. Functional programming is different to the way most people are taught to code. And switching to a new way of thinking can be difficult. But I think we can do a better job of explaining how things fit together.
- Algebraic Structures are a bit like code design patterns. Unlike design patterns though, they have a mathematical basis and are inter-related. Hence why they’re called ‘algebraic’. Also, unlike design patterns, they have confusing names made up by mathematicians.
- Type classes are a way of implementing polymorphism. They’re used in statically typed languages like Haskell or PureScript. They let you define functions with the same name but different type signatures. They’re often used to implement algebraic structures. So sometimes people use the terms interchangeably.
- Algebraic data types are not the same as algebraic structures. What are they? They’re composite data types made out of other types. There’s two main kinds of algebraic data types: Sum types and Product types. Together, they’re like a dynamic duo for encoding business logic. They help us make good things possible, and bad things impossible.

## What next?

I’ve tried to describe the ideas in functional programming that confused me most. I’ve written this hoping that it might save others some struggles I went through. But we’ve barely even glimpsed the white rabbit here. The rabbit hole goes much deeper—all the way to wonderland.

How do we go on then? Where do we start if we want to learn more?

Here are my recommendations:

1. Learn specific instances of algebraic structures before trying to learn the structures themselves. It’s much easier to see a pattern when you have some examples to look at. And it’s much easier to learn a tool when you can see what you might want to use it for. Read up on some instances of algebraic structures. I’ve written before about [Maybe](https://jrsinclair.com/articles/2016/marvellously-mysterious-javascript-maybe-monad/), [Either](https://jrsinclair.com/articles/2019/elegant-error-handling-with-the-js-either-monad/), and [Effect](https://jrsinclair.com/articles/2018/how-to-deal-with-dirty-side-effects-in-your-pure-functional-javascript/). If you know those then perhaps look into Reader, Future (also known as Task), and State. A good way to learn them is to try writing your own version.
2. Read through [Tom Harding’s series on Fantasy Land](http://www.tomharding.me/fantasy-land/). It goes through the different algebraic structures in Fantasy Land one-by-one. It breaks them down and shows how specific instances of where they can be useful.
3. You could take a Haskell course. Haskell is interesting because it bakes certain restrictions into the language. You can’t use loops or mutable variables, for example. Everything is an expression. When you take a Haskell course, you don’t have the old imperative tools available. So you’re forced to figure out how to get things done without them. It helps to have an instructor explain some of these things, even if you don’t intend to use it in your day job. And it’s helpful to try and take some ideas and figure out how they might apply in another language.

I have one final piece of advice: Keep persevering. Don’t give up. If you find that a bunch of blog posts don’t explain things in a way that makes sense, skip them. Keep looking until you find some well-written ones. The same goes for courses and tutorials. Everyone comes from different backgrounds. What makes sense for someone else might not work for you. And that’s OK.

---

Enormous thanks to [Jethro Larson](https://twitter.com/jethrolarson), [Joel McCracken](https://twitter.com/joelmccracken) and [Kurt Milam](https://twitter.com/kurtmilam) for reviewing an earlier draft of this entire series. I really appreciate the feedback and suggestions.