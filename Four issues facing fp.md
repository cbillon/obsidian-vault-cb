---
link: https://chadnauseam.com/coding/pltd/four-issues-facing-fp
---
#   four-issues-facing-fp 

A selling point of functional programming is that it's supposed to be simpler. But is it really? Of all the mainstream functional programming languages, Haskell is the functionalest, and it ended up being (famously) difficult to learn.

It turns out that Haskell is both simpler than other languages _and_ harder to learn. The issue is that it's so simple that you have to build a whole tower of abstractions to get anything done. Monads are the classic example of this - there's no direct way to represent effectful code in Haskell because Haskell is lazy, but Monads allow you to _describe_ an effectful program. Then you give that description to the magic name `main` and the runtime executes it. It's a beautiful abstraction but it adds a layer of indirection and complexity when compared to an imperative program with the same behavior.

You can imagine a sliding scale between _core-language-simplicity_ and _practical-program-simplicity_. Functional languages prefer core-language-simplicity, while imperative languages prefer practical-program-simplicity. When an imperative programmer wants to do something (like iterate over elements in a collection) they usually have a specific language feature for that purpose (like loops). This complicates the core language, but then actual programs written in that language are simple. Functional languages, on the other hand, attempt to provide a simple base and build the same concepts in user space.

So where imperative languages have loops, functional languages have recursion, and expect you to use that to recreate loops yourself. (I'm being a little mean, for lists you can use `map` and `fold` which are always in the standard library, and you in Haksell can use `deriving` to automatically write equivalent functions for any custom type you make. The point is in a functional language the complexity is in userspace so you could have written them yourself, but in an imperative language you could never write `while` yourself.)

Having a simple core language might sound like something only ivory tower academics really care about. But it does come in handy! In Haskell for example, the _only_ thing a function can do is return a value computed from its inputs. Same inputs, same value, no side effects. And this simplicity allows you to write abstractions that would comparatively be a nightmare to write any other way. Undo/redo is the obvious example, because without mutation it's easy to keep a reference to an old version (and persistent data structures make that not as ineffecient as it sounds). Purity also enables [Software Transactional Memory](https://hackage.haskell.org/package/stm) and Tweag's amazing [Funflow library](https://github.com/tweag/funflow). With purity you can create expressive type systems which enable formal proofs about your code, like how Haskell and Rust code never encounters null reference exceptions (because `Maybe`/`Option` force you to prove you've checked and handled that case explicitly), or like how Haskell's [`Data.Justified`](https://hackage.haskell.org/package/justified-containers-0.3.0.0/docs/Data-Map-Justified.html) allows you to prove a key exists in a dictionary so accessing it is safe.

Another example of that additional power is [optics](https://hackage.haskell.org/package/optics-0.4/docs/Optics.html) - they massively simplify operations like "I have a list of people, and each person has a hair color field, and I want to set each of their hair colors to `Brown`" - it's not a complicated idea, but its implementation in an imperative language would tiresome mess of loops, but with optics it becomes as simple as just writing that sentence in the optics eDSL.[[1]](https://publish.obsidian.md/#fn-1-4e1dd6eebd82352d)

Optics are well known, but they're still not used ubiquitously like I think they should be. And other forms of expressive power are used even less, or haven't been explored at all. And without that exploration, FP ends up being more limiting and less convenient than imperative programming. There are four under-resarched avenues I think are really promising.

## Making Debugging & Code Editing Easy 

These are a bit simpler and more surface-level.

1. ### Uniform Function Call Syntax (UFCS).
    
    This adds an alternative syntax for invoking functions.
    
    Without UFCS: `foo(x, y z)` or `bar(x)`.
    
    With UFCS: `x.foo(y, z)` or `x.bar`.
    
    This enables _drastically_ better autocompletion in languages with good type systems. You just write `x.` and your IDE shows you every function whose first parameter could be `x`.[[2]](https://publish.obsidian.md/#fn-2-4e1dd6eebd82352d)
    
    It also makes manipulating values in a functional style much easier, because it removes the nesting that comes when using functions to iteratively tranform a piece of data. Compare `average(drop(sort(take(mylist, 10), 5)))` vs `mylist.take(10).sort.drop(5).average`.
    
    One disadvantage of this is that it makes namespacing trickier. I really like the trick Ocaml uses. If you have a namespaced function like `serialization::tojson`, you could either write `serialization::tojson(foo)` or `serialization::(foo.tojson)`. Put another way, if you write `namespace::(...)`, that makes everything in `namespace` available inside `...`.
    
2. ### Don't rerun tests if the code doesn't change.
    
    [Unison](https://www.unisonweb.org/) does this right. If the code hasn't changed, and the test hasn't changed, don't rerun the test! This kind of analysis is much easier in a functional language, because the only things that can affect a function's execution are its code and its inputs.
    
3. ### Functional debugging
    
    This is something that's always annoyed me - debugging in functional language is usually _so bad_, but in principle it could actually be much better than in any imperative language. Take this pseudocode:
    
    ```rust
    let handle_request = (request, path, old_state) => {
         let f = path.get_handler;
         let request = request.of_json;
         match request.f(old_state) {
           Ok((new_state, response)) =>
             (new_state, Response200(response.to_json.to_string)),
           Error(err) =>
             (old_state, Response500("Internal server error!"))
         }
       }
    ```
    
    What I want is to be able to write a test that calls this function. Then, if the test fails, I want to be able to click on it and enter a "debugging view". Say my test calls this function with `"{username: 'chad'}", "/api/getuser", 0`. I want to see:
    
    ```rust
    let handle_request = ("{username: 'chad'}", "/api/getuser", 0) => {
         getProductHandler = "/api/getuser".get_handler;
         { username: "chad" } = "{username: 'chad'}".of_json;
         match { username: "chad" }.getProductHandler(0) {
    //     Ok(new_state, response) =>
    //       (new_state, Response200(response.to_json.to_string)),
           Error(ProductNotFound("chad")) =>
             (0, Response500("Internal server error!"))
         }
       }  == (0, Response500("Internal server error!")),
    expected (1, Response200("{'firstname': 'Chad', lastname: 'Nauseam'}"))
    ```
    
    What I've done is replace every variable with its value. Because a functional language has no mutation, there's no need to have a step-by-step debugger, we can just view the whole execution at once. This lets us easily find the bug - `"/api/getuser".get_handler` is returning a function named `getProductHandler`, which is probably wrong.
    
    The trickiest part here is how to visualize `let f = path.get_handler;`. Lets say `path.get_handler` returns a lambda like `(json, state) => Ok((state+1, whatever))` - my suggestion is that we look through the context for a binding of that lambda to a name, and if we find one we just show the name. Otherwise we'd probably just have to keep viewing it as `f` and add something like this:
    
    ```rust
    let handle_request = ("{username: 'chad'}", "/api/getuser", 0) => {
    
          ╭──────────────────────────────────────────────────────── function
          │                                                         goes here
          f = "/api/getuser".get_handler;
          { username: "chad" } = "{username: 'chad'}".of_json;
          match { username: "chad" }.f(0) {
    //     Ok(new_state, response) =>
    //       (new_state, Response200(response.to_json.to_string)),
            Error(ProductNotFound("chad")) =>
              (0, Response500("Internal server error!"))
          }
        }  == (0, Response500("Internal server error!")),
     expected (1, Response200("{'firstname': 'Chad', lastname: 'Nauseam'}"))
    ```
    
    Then, if user wants to find the bug in `"/api/getuser".get_handler`, they can simply click it and see the same visualization shown here.
    
    Another way we can easily beat imperative debuggers is with dedicated visualizations for higher order functions. For example `[1,2,3,4].map(myfunc)` could be visualized as
    
    ```haskell-nolines
           [1, 2, 3,  4]
    myfunc  ↓  ↓  ↓   ↓
           [4, 8, 12, 16]
    ```
    
    And then the user can click one of those arrows to see the execution visualization for `myfunc(value)`. Because functions like `map`, `filter`, etc. express your intentions much more specifically than an equivalent `for` loop would, these functions would be trivial to visualize in a way that allows the programmer to see the transformation that's happening all at once (and identify what's going wrong).
    
    To handle `do` notation, I would give `>>=` the ability to output a value representing its "hidden state", and show that next to each line. That might be tricky for some lesser-used effects like the list monad, but I think it should get you 80% of the way there.
    
    Combine this with some clever semantic highlighting to highlight the each variable in a unique color, and use an analysis of the execution trace to comment out (or otherwise make grey) any code that doesn't effect the output of the function, and I think you would end up with a debugging experience more ergonomic than anything an imperative programmer could ever dream of.
    

## Making Monads Easy With Effect systems 

Let's look at the type of Haskell's [`map`](https://hackage.haskell.org/package/base-4.16.0.0/docs/Prelude.html#v:map):

```haskell-nolines
map :: (a -> b) -> [a] -> [b]
```

And now let's look at [`mapM`](https://hackage.haskell.org/package/base-4.16.0.0/docs/Prelude.html#v:mapM). I've specialized it to `List` to make the analogy to `map` clearer:

```haskell-nolines
mapM :: (Monad m) => (a -> m b) -> [a] -> m [b]
```

They are very closely related. The difference is that `map` applies a "pure function", and `mapM` applies a "monadic function". (The monadic function is pure too, but the monad describes an effect.) For example, to increment every `Int` in a list you would use `map (+1)` (`+1` has the type `Int -> Int`), but to schedule an alarm for every `Int` in a list you would use `mapM scheduleAlarm` ([`scheduleAlarm`](https://hackage.haskell.org/package/unix-2.7.2.2/docs/System-Posix-Signals.html#v:scheduleAlarm) has the type `Int -> IO Int`). `scheduleAlarm` can be thought of as a function returning an `Int`, just like `+1`, except with a monadic effect stuck onto it.

Monads are very nice but I think this abstraction is a bit too low-level for a high-level functional language. What we'd really like to be able to do is make a type that says:

> If you give me a function with some effects `e` and a list, I'll apply that function to every value in the list and doing that will also have the effects `e`.

To do that we need an _effect system_. And the best effect system around is [Koka](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/paper-20.pdf)'s.[[3]](https://publish.obsidian.md/#fn-3-4e1dd6eebd82352d)

1. With an effect system, the type of a function that does IO stuff would be written like `a ->{IO} b`.
    
2. The type of a function that does IO stuff and also interacts with some state would be `a ->{IO, State} b`.
    
3. A function with no side effects would be `a ->{} b`, although any sensible language would add some sugar to allow you to just write `a -> b` and have the `{}` be inferred.
    

Now we can represent what we wanted above! The _true_ type of map is `(a ->{e} b) -> [a] ->{e} [b]`. There's something a little subtle here: `e` is a variable representing a _list of effects_. More on this later.

You don't lose any flexibility this way - it's basically just syntax sugar around monads, so you can add a language feature where users define a "custom effect" by giving the definition of a monad.[[4]](https://publish.obsidian.md/#fn-4-4e1dd6eebd82352d)

Let's look back to our `map` example:

```haskell-nolines
(a ->{e} b) -> [a] ->{e} [b]
```

Now let's be good functional programmers and rewrite that to make the variables explicit:

```haskell-nolines
forall a b e. (a ->{e} b) -> [a] ->{e} [b]
```

`a` and `b` are type variables of the kind that any Haskell or Ocaml dev would be familiar with, but what's `e`? Well, it's a variable, but of a different kind than type. To really make this work, you need some special sauce that I'll get into in the the section on _row polymorphism_.

Here are some common Koka functions in case you're curious about their types. (Koka uses `∀` in place of `forall`)

```haskell-nolines
random :: () ->{ndet} double
print  :: string ->{io} ()
error  :: forall α. string ->{exn} a
id     :: forall α µ. α ->{µ} α
```

With this approach you can even represent effects like the ST monad, mutation, try/catch (which actually removes an effect from a function), and more.

(By the way, I feel compelled to mention that the Unison language gets this one right too. They're doing really good work over there.)

### Effect handlers 

Now, isn't this just side-effectful imperative programming with extra steps? It may seem that way but it's actually much more powerful, because of _effect handlers_. Let's say you have a function whose type is:

```
getUserInfo : Username ->{network} Userinfo
```

Obviously this function is intended to query the network, but how do you actually run it? Well, usually you have a special binding `main :: () ->{io} ()` which is run when your program starts, so we just need a way to convert a `a ->{network} b` to `a ->{io} b`.

A function that does that is called an effect handler. So you might write

```
let network_code : () ->{network} () = ...
let main : () ->{network} () = handle_network network_code
```

The beauty is that you can have multiple effect handlers for different use cases. For example, you could write a file system handler that simulates a filesystem in memory, so that you can write tests for file-system-affecting code without it actually editing files on your machine. And handlers are easy to write, they're probably just interpreters for freer monads or something. [[5]](https://publish.obsidian.md/#fn-5-4e1dd6eebd82352d)

### Further research 

I think a really interesting research direction would be to convert each function into a dataflow graph, and use that to model effects with arrows rather than monads.

Effects are on `->`, which means that (unlike with monads) we know the input as well as the output, so we should take advantage of that.

Monads were actually introduced to FP because Haskell needed a way to "flatten" the dataflow graph into a single line so there would be a sensible order to execute effects in - but that was only needed because of laziness, and with a strict language I think the greater expressivity of arrows makes more sense. And you could probably annotate each edge in the graph with a number or something that would allow you to reconstruct a monad's order of execution anyway.

Or maybe this is a bad idea, I'm not sure, but I really do want a more ergonomic way to write arrows.

An example of when this would be useful would be writing a concept like "A new user's info is valid if their email must be a valid email, their password is a valid password, and their password isn't the same as the part before the @ of their email."

This operation is kind of like a graph: You can't even check the last constraint unless the first two constraints are satisfied. (If the user's password is invalid there's no point in telling them it can't be the first part of their email, same if their email is invalid.) So even though it could easily be implemented with a couple if/elses, it's really a graph problem, and there's literally no good way to encode this operation as a graph in any mainstream language.[[6]](https://publish.obsidian.md/#fn-6-4e1dd6eebd82352d)

## Recursion schemes 

Functional languages don't have loops. What we _do_ have is recursion. Is that a good enough replacement?

![why_is_recursion_so.jpg](https://publish-01.obsidian.md/access/8a529a39ad1753175d73ccc6abc0547e/images/coding/screenshots/why_is_recursion_so.jpg)

Let's just say there's room for improvement. Luckily, that improvement is here: it's called _recursion schemes_.

The idea is that most recursive functions fit into a couple simple categories. The simplest and most common is called a _traversal_.[[7]](https://publish.obsidian.md/#fn-7-4e1dd6eebd82352d) Instead of writing the actual recursive function yourself, you just have to specify what _scheme_ it belongs to, then declaratively fill in the blanks. But before I explain recursion schemes, first I want to point out something about recursive data structures.

Let's say you have a recursively-defined tree like this:

```haskell
data Tree a = Branch (Tree a) (Tree a) | Node a
```

What I want to point out is that there are two conceptually different types of data here, _recursive_ and _not recursive_.

```haskell-nolines
data Tree a = Branch ( Tree a )   ( Tree a ) | Node  a
                       ╰──┬──╯      ╰──┬──╯         ╰┬╯
                      recursive    recursive    not recursive
```

Traversals are recursive functions where the only thing you ever do on recursive data is call the recursive function. Take `depth` as an example.

```haskell
depth :: Tree a -> Int
depth = \case (Node _)     -> 1
              (Branch a b) -> 1 + max (depth a) (depth b)
```

Our recursive function is `depth`. The recursive data is the `a` and `b` on line 2. The only thing we do with `a` and `b` is call `depth` on them, so `depth` is a traversal.

To actually use recursion schemes, you need to mix up the data type a bit. Specifically, you need to add a new type parameter, and then replace the recursive data with that parameter.

```haskell
data Tree  a   = Branch  (Tree a) (Tree a) | Node  a
data TreeF a f = BranchF f        f        | NodeF a
```

(It's tradition to use `f` for the extra parameter and add `F` everywhere to disambiguate the types and the constructors.)

By the way, in theory you could actually reconstruct `Tree` from `TreeF` by doing some type magic like this:

```haskell
data Tree a = TreeF a Tree
```

But that's not necessary to use or understand recursion schemes.

To write a function using recursion schemes, you'll need to use a standard helper function. We'll call it `fold`, but it's different from the `fold` in Haskell's prelude. I'll show you the type, but it's easier to just work through an example:

```haskell
fold :: (TreeF a f -> f) -> Tree a -> f
```

The only thing we needed to know to compute `depth` was the depth of the subtrees. `fold` allows us to just say what we would do if we _already knew_ the depth of the subtrees, then it takes care of doing the actual recursion for us.

Here's how we'd rewrite `depth`. The rewritten version has the same type as before (`Tree a -> Int`), but internally it uses `fold` and a function `TreeF a Int -> Int`:

```python
# Before:
depth =          \case (Node _)     -> 1
                       (Branch a b) -> 1 + max (depth a) (depth b)
# After:       ╭────────────────────────────────────╮
depth = fold $   \case (NodeF _)    -> 1           #│
                       (Branch a b) -> 1 + max a b #│ `a` and `b` are `Int`s
#              ╰─────────────────┬──────────────────╯
#              this function has type `TreeF a Int -> Int`
```

Notice here that there's _no more explicit recursion_ - instead we write a function of type `TreeF a Int -> Int` and pass it to `fold`, which turns it into a function of type `Tree a -> Int`.

Now I'll forgive you if you feel like this has added a useless layer of indirection. They don't really make sense for a function as simple as `depth`, but for more complicated functions they're a lifesaver. `fold` here subsumes the traditional `foldr` you're used to using over lists, and there are even more powerful schemes that use the same framework.[[8]](https://publish.obsidian.md/#fn-8-4e1dd6eebd82352d)

For example, while `fold` only gives you the result of recursing over the subtrees, `para` also gives you the original subtrees. And you're not limited to functions that take a recursive data structure and return a simple value - the reverse, taking a value and returning a data structure, is just as easy with functions like `unfold`.[[9]](https://publish.obsidian.md/#fn-9-4e1dd6eebd82352d)

To learn more about recursion schemes and their virtues, I highly recommend checking out [Sum Type Of Way](https://blog.sumtypeofway.com/posts/introduction-to-recursion-schemes.html)'s guide, and the docs for the Haskell [recursion-schemes](https://hackage.haskell.org/package/recursion-schemes-5.2.2.2/docs/Data-Functor-Foldable.html#v:cata) library. But I'm not just interested in their convenience. Do you remember what I said earlier, about custom visualizations for higher order functions?

> Another way we can easily beat imperative debuggers is with dedicated visualizations for higher order functions. For example `[1,2,3,4].map(myfunc)` could be visualized as
> 
> ```haskell-nolines
>        [1, 2, 3,  4]
> myfunc  ↓  ↓  ↓   ↓
>        [4, 8, 12, 16]
> ```
> 
> And then the user can click one of those arrows to see the execution visualization for `myfunc(value)`. Because functions like `map`, `filter`, etc. express your intentions much more specifically than an equivalent `for` loop would, these functions would be trivial to visualize in a way that allows the programmer to see the transformation that's happening all at once (and identify what's going wrong).

Because recursion schemes are so much more declarative than plain recursion, you can easily imagine doing the same visualizations for `fold`, `unfold`, `para`, etc., and generalizing those visualization to trees and other data structures.

I think it's kind of nice how it all fits together - functional programming makes your control flow more explicit, which makes it easier to visualize a program's execution, and recursion schemes make control flow even more explicit, which makes it even easier to visualize.

## Reducing nesting with Row Polymorphism + Refinement Types 

Row polymorphism might not seem like they belong in the same section, but I'm combining them because for me they serve the same goal: reducing nesting levels.

### Row Polymorphism 

You ever notice a similarity between product types and sum types? Take

```
data MyProduct = {a: Int, b: Bool}
```

and

```
data MySum = A Int | B Bool
```

I mean, obviously they're different, but the similarity I want to point out is that they could be described by a list of labels and types. That's the similarity that _rows_ exploit.

In statically typed languages, values often have different types. In Haskell, it's possible to talk about things on the type level that aren't types - for example, `Maybe` isn't a type. `Maybe Int` is a type, but `Maybe` is a different _kind_. `Maybe`'s like a type-level function that takes a type and returns a new type. If you say that `Int` has kind `Type`, then `Maybe` has kind `Type -> Type`. (And `Maybe Int` has kind `Type` again.)

The suggestion here is to add a new kind called _rows_. A row is just a list of labeled types. For instance, a labeled type might look like `name: String`, and row might look like `name: String, location: Coordinate`.

Rows and labeled types aren't types themselves, so it makes no sense to talk about "a value with type row" or anything like that. But what you _can_ do is use a special type constructor to convert a row to a sum or a variant:

1. **Product**: `{* name: String, location: Coordinate *}`. Here `{* *}` is a type constructor that takes a row and returns a product type.
    
2. **Sum**: `{+ square: Float, rectangle: (float, float) +}`. Here `{+ +}` is a type constructor that takes a row and returns a sum type.
    

Here's a few different ways of writing the same function:

```rust
let manhattan =
  (point : {* x: Float, y: Float *}) : Float =>
    point::x + point::y // or `point::(x + y)`

let manhattan = // let's try pattern matching
  ({* x, y *} : {* x: Float, y: Float *}) : Float =>
    x + y

let manhattan =
  ({* x, y *}}) : Float // type of `x` and `y` is inferred
    => x + y

// usage:

manhattan({* x=3, y=4 *}) // 7
```

I already think this is cool. When you use rows to create product types, you get something like tuples except the values inside have names. But the real power comes when you introduce `row polymorphism`. The idea is that, like how you can introduce type variables, you can also introduce _row variables_. Let's modify `manhattan` to make it so you can pass _any product_ that has `x: Float, y: Float` in it:

```rust
// `r, ...s` is the row with everything in `r` followed by `s`
let manhattan =
  forall r.
  (point : {* x: Float, y: Float, ...r *}) : Float =>
    point::x + point::y

manhattan({* x=3, y=4, whatever=None *}) // 7
```

Or let's change it so it _modifies_ the product:

```rust
let manhattan =
  forall r.
  ({* x, y, ...r *}) : {* manhattan: Float ...r *} =>
    {* manhattan = x + y, ...r *}

manhattan({* x=3, y=4, whatever=None *}) // {* manhattan=7, whatever=None *}
```

This solves the problem of "well, this type has almost everything I want, but I wish it had just one additional piece of data". Let's say the code looks like this:

```rust
alias person = name: String, home: Location

let foo = (p: {* person, ...r *}) => ...
```

And you decide that in your code you want to add a `hair: Color` field too. You could just write:

```rust
alias person = name: String, home: Location
alias personWithHair = hair: Color, ...person

let myPerson = {* hair=Brown, name="Martin", location=Texas *}
// and `foo(myPerson)` still works!
```

Obviously you could also just make a new wrapper type around `person`, but that adds a hierarchy that's conceptually unnecessary - you should be able to express exactly what you want, which is "a person with an additional `hair: Color` field". Speaking from experience, having a bunch of wrapper types around products and (especially) sums gets old fast.

And of course, everything I've described works with sum types too. It's actually even more useful with sum types because it allows you to _finally make error handling good_. The proof is in Purescript's [checked-expressions](https://github.com/natefaubion/purescript-checked-exceptions) library. I think its docs explain it does better than I could, so just go read those if you're curious.

Ok, but here's the final piece: _rows are the natural way of representing effect systems_. Remember this, from the second section?

> Now we can represent what we wanted above! The _true_ type of map is `(a ->{e} b) -> [a] ->{e} [b]`. There's something a little subtle here: `e` is a variable representing a _list of effects_. More on this later.

An effect can be intuitively represented by a labelled type, so a function's effect is a list of labeled types:

1. `readFile : Filename ->{io: (), error: FsError} String`
    
2. `get : Url ->{network: (), error: HttpError} Response`
    
3. `both : () ->{io: (), error: FsError, network: (), error: HttpError} ()`
    
    (Yes, you can use the same label twice in the same row, it's not a problem and actually it's very convenient.)
    

You can use aliases to allow the user to replace `io: ()` with `io` for simplicity, but for some effects like `error` you really want to be able to specify the type - it's just like the the error type from `Result`!

#### Further research 

1. I actually have bigger and badder ideas for rows. What I really want is a library that's like a mix of Pandas and [differential-dataflow](https://github.com/TimelyDataflow/differential-dataflow) for storing tables of product types. Its type would be something like `Database(r)` where `r` is a row.
    
    The goal is to make it easier to do database-like operations. Imagine doing an inner join on `Db (id: Id, name: String)` and `Db (id: Id, location: Location)]` - it's easy in a database, why isn't it easy everywhere?
    
    Further reading: [Types for Tables: A Language Design Benchmark](https://news.ycombinator.com/item?id=29509439)
    
2. Another area that needs more eyes is making dependent records and rows work together. If you have any ideas for this let me know.
    
3. How should product access work? `x::y` to access `y` field from product `x` makes sense... but it would be nice if `y` was an optic so you could do cool optic stuff.
    

### Refinement Types 

Types are awesome. They're like documentation that the compiler guarantees are always up to date, make your code faster, make your IDE more helpful, and allow your compiler to warn you about your mistakes without even running your code. So if we can think of a way to make types more powerful, it's probably a good idea.

Dependent types are one promising avenue, and I think they're a good idea, but they're starting to become well explored. I want to bring attention to _refinement types_, which I think are even more exciting.[[10]](https://publish.obsidian.md/#fn-10-4e1dd6eebd82352d)

In my experience, dependent types feel like explaining something to a genius smart 4 year old, and refinement types feel like explaining something to an intelligent college - dependent types are probably technically more powerful, but refinement types give you 90% of the power for 10% of the effort. The reason refinement types have such a good power:effort ratio is because they're implemented under-the-hood with a powerful SAT solver.

Refinement types allow you to "refine" a type. If `Int` is the type of integers, `Int thats > 0` is the type of integers that are greater than zero. A value of that type can be used anywhere `Int` can - the only restriction is that to create a value of that type, the compiler must be able to prove that the value is greater than zero.

Sometimes someone sees that there's values inside the types and assumes that means these are scary dependent type, but it's actually a totally different (and conceptually much simpler) thing.

I've been a big vague - let's break down `Int thats > 0`.

1. The base type is `Int`.
    
2. `thats` is a keyword that indicates that on the left is a base type, and the right is a refinement.
    
3. `> 0` is a function that you're proving will return true when provided a value of this type - this function is also known as a predicate.
    

So a refinement type is a type + a logical predicate.

You can also imagine a different syntax where we give the value a name and pass it to the function explicitly, like `x: Int where x > 0`. These are equivalent and I might use them interchangeably - I prefer the nameless version when there's no obvious name, and the nameful version when there is (i.e. it's a function parameter).

So if a refinement type is `<type> thats <predicate>` - what predicates are allowed? Unfortunately you can't just write any old function - for starters, the function should terminate, that way you can compile in a finite amount of time. But you also need some additional restrictions to make the predicate easy for the compiler to understand, that way it can try to prove as much as possible for you.

In practice, your predicate can't use the full power of your programming language. You're restricted to just these features:

1. **Simple constants**, like `1`, `2`, `True`, `[]` etc.
    
2. **Variables**, if you use the `x: Int where x > 0` syntax.
    
3. **Simple operators**, like addition and subtraction, or multiplication by a constant, and `if`/`then`/`else` expressions.
    
4. **Relations** between simple types like numbers and booleans, like `>`, `==`, `and`, `not`, etc.
    
5. **Any pure function you want**, with the catch that the refinement type system won't know anything about your function except that it's pure. (So it won't evaluate your function, but it will assume that if you call it twice with arguments that are the same, the output will be the same.)
    
6. **Measures**, which are a subset of pure functions that can match, use these operators, and do structural recursion (so they're guaranteed to terminate).
    

You can actually write a lot of properties this way. For example, you could say a nonempty list is `List(a) thats /= []`. You could imagine defining a "predicate alias" for this, like:

```rust
let not_empty = a: List =>{predicate} a /= []
// I've made up a "predicate" effect that guarantees the function obeys our restrictions.
```

Then you could write `List(a) thats not_empty`.

Ok, enough talk, let's see them in action:

```rust
let head = (l: List(a) thats not_empty): a =>
  match l with {
    (a :: _) => a
  } // this match is exhaustive!
```

`List(a) thats not_empty` means that this function can only possibly be passed an empty list. So when matching, we don't need to match that case - the compiler can see that it would contradict the predicate.

We can also add postconditions - this is how things get "proven".

```rust
abs = (n: Int): (Int thats >= 0) =>
  if 0 < n
    then n
    else 0 - n
```

The job of proving that the output really is always `> 0` is done _automatically by the compiler! Then, a function that has an argument that should not be negative, like `log`, can accept the result of `abs`. The compiler knows that if `0 < n`, then `0 - n` is always positive, so it doesn't require any work from you to add this proof. In fact, you could write:

```
abs = (n: Int): (Int thats _) =>
  if 0 < n
    then n
    else 0 - n
```

And the compiler could _automatically_ infer that `_` is a stand-in for `>= 0`.

### More structural, more flat 

In addition to both starting with "r", something rows and refinements have in common is that they solve related problems. Rows solve the problem of extending a type, and refinements solve the problem of restricting a type. Specifically, they let you do this in a way that's _more structural_ and _more flat_.

1. **More structural**:
    
    Compare Haskell's `NonemptyList a` type with a hypothetical `List a thats not_empty`. The former is obviously nominal - you and I could both come up with our own `NonemptyList` types, which wouldn't be compatible with each other. `List a thats not_empty` doesn't have this problem - even if we define it independently or in different ways, SMT solvers are smart enough to understand that they're the same thing.
    
    Similarly, rows let you write functions that are compatible with any product that has certain fields, or any sum that has certain variants, no matter who defined them. The variant/field names do have to be the same, so it's still kind of nominal, but the goal isn't to totally remove nominality, just reduce it.
    
2. **More flat**: `List a thats not_empty` is known to be nonempty, but it still can be used anywhere that a `List a` can be used. `NonemptyList`s have to be converted to and fro all the time - not the end of the world, but it's ineffecient and boring. You should be able to say just what you mean - it's a list, and it's also not empty. Worse, things like `NonemptyList` don't compose - if you have `NondescendingList` and `NonemptyList`, how do you make `NonemptyNondescendingList`? It just doesn't work that well.
    
    Some of these nice properties are given to us when we extend types with rows. `{* a: B, ...r *}` can probably be used in most places `{* r *}` can be used - no need to wrap and unwrap, or introduce some hierarchy that doesn't exist conceptually. And `{* a: B, ...r *}` and `{* c: D, ...r *}` can be easily merged to create `{* a: B, c: D ...r *}`, which is not always easy to do with wrapper types.
    

---

1. Not that the optics eDSL is terribly easy to learn.[↩︎](https://publish.obsidian.md/#fnref-1-4e1dd6eebd82352d)
    
2. That's much more useful than the other way around, where you type `foo` and then your IDE tells you every value could be the first parameter).[↩︎](https://publish.obsidian.md/#fnref-2-4e1dd6eebd82352d)
3. Also check out [Type Directed Compilation of Row-Typed Algebraic Effects](https://sci-hubtw.hkvisa.net/10.1145/3009837.3009872) by the same author.[↩︎](https://publish.obsidian.md/#fnref-3-4e1dd6eebd82352d)
4. The only difference is that effect systems typically require that the only way to actually apply an effect is by calling a function, so you'd have to replace `getLine :: IO String` with `getLine :: () ->{IO} String`, which I feel is slightly inelegant, but a price worth paying.[↩︎](https://publish.obsidian.md/#fnref-4-4e1dd6eebd82352d)
    
5. I understand that to many this sounds like "just monoids in the category of endofunctors", but really they're not so bad.[↩︎](https://publish.obsidian.md/#fnref-5-4e1dd6eebd82352d)
6. The closest is Haskell arrows, but that doesn't quite qualify as "good" in my mind.[↩︎](https://publish.obsidian.md/#fnref-6-4e1dd6eebd82352d)
7. Also known as a catamorphism.[↩︎](https://publish.obsidian.md/#fnref-7-4e1dd6eebd82352d)
8. Unfortunately, they all have kind of silly names, but you get used to them. Although they're really not much more silly than the `map`, `fold`, `bind`, etc. that functional programmers are already used to.[↩︎](https://publish.obsidian.md/#fnref-8-4e1dd6eebd82352d)
9. An example of when you'd want to do that is when writing functions like `repeat :: a -> Int -> [a]`.)[↩︎](https://publish.obsidian.md/#fnref-9-4e1dd6eebd82352d)
10. Refinement types and dependent types are orthogonal, so you can have one or the other or both or neither if you want.[↩︎](https://publish.obsidian.md/#fnref-10-4e1dd6eebd82352d)