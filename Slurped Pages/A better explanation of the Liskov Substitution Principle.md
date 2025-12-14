---
link: https://hillelwayne.com/post/lsp/
site: Hillel Wayne
date: 2023-11-07T01:00
excerpt: |-
  Short version: If X inherits from Y, then X should pass all of Y’s black box tests.
  I first encountered this idea at SPLASH 2021.
  The longer explanation A bit of background In A Behavioral Notion of Subtyping Liskov originally defined subtyping in inherited objects as follows:
   Subtype Requirement: Let P(x) be a property provable about objects x of type T. Then P(y) should be true for objects y of type S where S is a subtype of T.
slurped: 2025-12-07T11:04
title: A better explanation of the Liskov Substitution Principle
tags:
  - liskov
  - principles
---

Short version: _If X inherits from Y, then X should pass all of Y’s black box tests._

I first encountered this idea at [SPLASH 2021](https://2021.splashcon.org/details/splash-2021-SPLASH-E/7/Reframing-the-Liskov-Substitution-Principle-through-the-Lens-of-Testing).

## The longer explanation

### A bit of background

In [A Behavioral Notion of Subtyping](https://www.cs.cmu.edu/~wing/publications/LiskovWing94.pdf) Liskov originally defined subtyping in inherited objects as follows:

> Subtype Requirement: Let P(x) be a property provable about objects x of type T. Then P(y) should be true for objects y of type S where S is a subtype of T.

Later, Robert Martin [named](https://web.archive.org/web/20150905081111/http://www.objectmentor.com/resources/articles/lsp.pdf) the Liskov Substitution Principle:

> Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.

If you had `fly_to(Bird, location)`, then you should be able to call `fly_to` on any subtype of `Bird`. This means that `Penguin` and `Ostrich` couldn’t be subtypes of `Bird`, since you cannot call `fly_to` on a flightless bird!

Liskov would later “approve” of the rule in her book _Program Development in Java_.

### The Devil is in the Details

The LSP looks simple; the problem is applying it. How do you _check_ that a function works on all subclasses of a class? Functions can do a lot of things! So we have a set of enforceable rules, where if you follow all the rules, you (presumably) satisfy LSP. The most well known of these rules is that inherited methods of a class must not _strengthen preconditions_ or _weaken postconditions_.[1](https://hillelwayne.com/post/lsp/#fn:methods-vs-functions) That means it must accept a _superset_ of the values the parent method accepts and output a _subset_ of the parent’s possible outputs. Given

```
class Parent {
   run(x: int): out {
    require {x >= 4}
    # some code that computes out
    ensures {out % 2 == 0}
   }
}

class Child extends Parent {
# stuff
}
```

`Child.run` cannot be overloaded to take only _even_ integers, or to also output odd numbers. Preconditions and postconditions together are called contracts.

This is just the first rule and already we have three problems. First, it’s not super clear why this follows from the LSP. Second, it’s hard to remember whether it’s _weaken preconditions_ and _strengthen postconditions_ or the other way around. Third, most languages don’t have dedicated pre/postcondition syntax. For these languages the rule is taught as a restriction on the method’s _type signature_: the parameters must be contravariant and the return values covariant. This makes things even more arbitrary and confusing and isn’t as accurate as the original version.

Long tangent on types and contracts

The above is historically inaccurate: the original Liskov paper lists _both_ the contract rule and the type rule as separate requirements for subtyping. But IMHO the type rule is equivalent to the contract rule. Saying “parameters must be contravariant” is the same as saying “preconditions must not be strengthened.” Consider this case of contravariant parameters:

```
Parent.foo(x: Cat) {
    # code 
}

Child.foo(x: Animal) {
    # code
}
```

We can rewrite it purely as contracts:

```
Parent.foo(x) {
    requires typeof(x) == Cat;
    # code
}

Child.foo(x) {
    requires typeof(x) == Animal;
    # code
}
```

It’s less intuitive that we can rewrite contracts as types, but that’s also possible:

```
foo(x: Int): out {
    require {x >= 4}
    # code
    ensure {out % 2 == 0}
}

# becomes

foo(x: AtLeastFour): (out: EvenNumber) {
    # code
}
```

It’s true that most languages can’t statically typecheck this, but that’s a limitation on our current typecheckers, not an _essential_ problem. Some languages, like [Liquid Haskell](https://ucsd-progsys.github.io/liquidhaskell/), handle it just fine.

That said, even if the two rules are theoretically equivalent, in practice your programming language will favor checking some things as contracts and other things as types.

We can’t ensure LSP with function contracts alone. Consider![2](https://hillelwayne.com/post/lsp/#fn:pythonica)

```
@dataclass
class WrappedCounter:
    val: int # assume that 0 <= val < 1000

    def add(self, x):
        assert x >= 0 # Precondition
        self.val = (self.val + x) % 1000

class Counter(WrappedCounter):
    def add(self, x):
        assert x >= 0 # Precondition
        self.val += x
```

`Counter` is _obviously not_ a subtype of `WrappedCounter`, but it has the same method contracts! The difference is that `WrappedCounter` has an extra invariant, a property that holds true of an object at all times. `Counter` doesn’t have that property, so it can’t be a child class. That leads to rule two, subtypes must not weaken the _class invariants_.

The method rule and the invariant rule look somewhat related, but the next one comes out of left field.

```
@dataclass
class WrappedCounter:
    val: Nat
    limit: Nat

    def add(self, x):
        assert 0 <= x < self.limit # Precondition
        self.val = (self.val + x) % self.limit

class EvilCounter(WrappedCounter):
    def add(self, x):
        assert 0 <= x < self.limit # Precondition
        self.limit += x
        self.val = (self.val + x) % self.limit
```

Let’s look at our two rules:

1. Does it pass the method test? Yup, both have the exact same type signatures and preconditions.
2. Does it pass the invariant test? Yup, they both guarantee that `self.val ∈ [0, limit)`.

But `EvilCounter` is clearly _not_ a subtype of `WrappedCounter`. It doesn’t wrap!

We need a third rule, the history rule: the subtype cannot _change_ in a way prohibited by the supertype. In this case, this means that `limit` must remain unchanged. `WrappedCounter` follows this rule and `EvilCounter` violates it, so it’s not a subtyping relation.[3](https://hillelwayne.com/post/lsp/#fn:history-rule)

The history rule always felt like a tack-on. It makes me worry that we’re just patching up holes as we find them, and that somebody will come up with a counterexample that follows all three rules but still isn’t a subtype. Like what happens when we add in exceptions or generics? We might have to add another arbitrary rule. It all just makes me lose trust in the applicability of LSP.

## The Fix

In their college classes, Elisa Baniassad and Alexander J Summers [found a different approach](https://www.cs.ubc.ca/~alexsumm/papers/BaniassadSummers21.pdf):

> For a class to be substitutable for its supertype, it must pass all the supertype’s black box tests.[4](https://hillelwayne.com/post/lsp/#fn:black-box)

To check substitutability, we come up with a test:

```
def test_add_five():
    c = WrappedCounter(10)
    c.add(5)
    assert c.val == 15
```

This passes, so we’d expect the equivalent test to pass for `Counter`:

```
def test_add_five():
    c = Counter(10)
    c.add(5)
    assert c.val == 15
```

This also passes. If every test we write passes, then `Counter` is a subtype of `WrappedCounter`.

So let’s write a second test:

```
def test_add_million():
    c = WrappedCounter(10)
    c.add(1_000_000) 
    assert c.val == 10
```

And this passes for `WrappedCounter` but _fails_ for `Counter`. Therefore `Counter` is not a subtype of `WrappedCounter`.

Show limitation

Note that passing tests doesn’t _guarantee_ you’ve got a subtype, since there might be a failing test you just haven’t considered writing. It’s similar to how, even if you have a rule violation, you might never actually run into a subtype violation in production. This is more a _pedagogical_ technique to help people internalize LSP.

### This explains a lot

I like how the testing approach to LSP explains all the rules. If you give me a case where an LSP rule violation causes a problem, I can give you back a test case which fails for the same reason. Let’s Pythonize my earlier example of a precondition rule violation:

```
class Parent:
    def run(self, x: Int):
        assert x >= 4
        pass
        # return something

class Child(Parent):
    def run(self, x):
        assert x % 2 == 0
        super()
        pass
```

Here’s my test:

```
def test_parent():
    c = Parent()
    c.run(5)

def test_child():
    c = Child()
    c.run(5)
```

We don’t need to _do_ anything with the output of `run`, just call it. The first test (with `c = Parent()`) will do nothing while the second test (with `c = Child()`) will throw an error, failing the test. `Child` is not a subtype.

So that’s one way the testing approach makes things easier. It also shows why the history rule is necessary _and also_ why it originally seemed like a crude hack to me.

```
def test_history_rule():
    c = WrappedCounter(0, 10)
    c.add(6)
    c.add(6)
    assert c.val == 2
```

We originally needed the history rule because contracts and invariants only span the lifetime of _one_ method. But now we can just call a sequence of methods in one test!

## Does this work?

By “work”, I mean “does it help students internalize the LSP.” I found the paper a little confusing here: it says they “observed that students were able to better understand the responsibilities of the subtype in a general sense”, but also that “the same proportion of students” understood the rules, which kinda sounds like ‘no impact’? I emailed the researchers for a clarification and here’s their response:

> It was a big improvement. Students went from not demonstrating any intuition around the rule, to “getting it”. “Oh - subclass has to pass all superclass black box tests! I get it!”. And then you can delve into ways tests can break (narrowing preconditions, etc). But at the root of it all - it’s about the tests. They did way better on LSP questions, and I believe there was one question in particular that we asked before and after, and it was way better done (like from a failing grade to a good passing grade on average) with the testing approach.

They also told me about an indirect benefit: it also helped students understand the difference between black- and white-box tests! Once you get a sense for LSP in the original

> It helped clearly delineate whether tests were about “the packaging” (like what’s written on the box) or the “how” you’ve done it (which is specific to the implementation, and can change if you make different choices). You have to test both, and make sure both the packaging is okay, and check that the underlying implementation is not having unwanted side effects, or introducing errors — and it then became clear to students why we needed both kinds of tests, and why differentiating was useful.

Pretty cool!

_Thanks to [Liz Denhup](https://www.linkedin.com/in/elizabeth-denhup/) for feedback. If you liked this, come join my [newsletter](https://buttondown.email/hillelwayne/)! I write new essays there every week._