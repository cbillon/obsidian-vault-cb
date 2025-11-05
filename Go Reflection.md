---
tags:
  - go
  - reflection
---
## Laws of Reflection

If you are going to use `reflect`, you have to read [Rob Pike’s Laws of Reflection](https://go.dev/blog/laws-of-reflection). I confess it took me longer to find it than it should have, but maybe the lesson there is that your Wizbang IDE with all that fancy AI crap is no substitute for reading the docs.

If you refuse to read Pike’s Laws, then I have no choice but to force them in front of your eyeballs:

1. [Reflection goes from interface value to reflection object](https://go.dev/blog/laws-of-reflection#1-reflection-goes-from-interface-value-to-reflection-object)
2. [Reflection goes from reflection object to interface value](https://go.dev/blog/laws-of-reflection#2-reflection-goes-from-reflection-object-to-interface-value)
3. [To modify a reflection object, the value must be settable](https://go.dev/blog/laws-of-reflection#3-to-modify-a-reflection-object-the-value-must-be-settable)

The first two laws don’t seem very controversial. The `ValueOf()` and `Interface()` methods basically convert between values and reflection objects. These two laws are even double summed up by Rob:

> In short, the Interface method is the inverse of the ValueOf function, except that its result is always of static type interface{}.
> 
> Reiterating: Reflection goes from interface values to reflection objects and back again.

The 3rd law was harder for me to accept. I wanted to get code in the `main` package to call an unexported method in another package, but it turns out that’s just not possible. For the particular problem I was working on this ended up being a non-starter and I had to go back to the drawing board. That said, it was enough exposure to the `reflect` package that I have opinions about it.
### The Missing Law

When people started talking about thermodynamics, there was a general assumption that no one mentioned until later, thus the [Zeroth Law](https://en.wikipedia.org/wiki/Zeroth_law_of_thermodynamics) came into existence.

The authors of Go have kindly given us three laws of `reflect`, but I believe they have neglected to mention the 0th law, which should be:

0. Use `reflect` at your own peril. Misuse it, and it will `panic` with no regrets.

I believe this design decision, to panic when anything is the slightest bit wrong, is a very clear and loud statement that the maintainers of Go don’t want you to use the `reflect` package. It’s kind of a genius move to make it feel toxic right out of the gate.

It goes beyond just panics though. [`unsafe`](https://pkg.go.dev/unsafe) package is mentioned a bunch of times in the docs, and `unsafe.Pointers` are used in several interfaces. I think the only way to further discourage the use of the `reflect` package would be to make every method deprecated.

I think it’s safe to say that any production use of `reflect` should be highly scrutinized and done with extreme care. It’s not impossible. Anyone using the [`encode/json`](https://pkg.go.dev/encoding/json) is hooked on this stuff.

### Abolish the 3rd Law

My first use of reflection was with Java where `java.lang.reflect` is able to update pretty much any value you want. When put in combination with a custom ClassLoader and the ability to hot install [ByteCode](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html) you have a truly powerful (dangerous) tool at your disposal. `private` fields in Java don’t really have any meaning when up against `java.lang.reflect`.

In Go, that is not the case. You can read any field but you can only update fields that are exported (start with a capital letter). For methods on a struct you can’t even learn of their existence if they aren’t exported. And yet, you can call an exported method which presumably performs writes to data you wouldn’t otherwise be able to modify.

There are probably deeper reasons why unexported fields can’t be updated, like the compiler strips some metadata which makes it impossible. Putting that aside though, given that the `reflect` package is not fit for production use (refer to 0th law), we should be able to update any field and call any method even if it’s not exported. The safety of not being able to get my hands on an internal field seems unnecessary when we have the threat of panicking lurking in every corner. I’ve been given a shotgun to blow off my foot, but I have no shells.

With full ability to go bonkers with writes in `reflect`, it would be possible to build a wider set of test and debug tools. In Dolt’s particular case, we were attempting to break the storage format as part of our regular testing, and that’s code we didn’t want to reside in the Dolt code itself - hence going down this path. More generally though, it could enable a wider set of tools than we can build with the existing behavior.

At a very minimum, we could encode unexported fields in JSON structs. Wouldn’t that be nice?
## Closing

`reflect` is not for the faint of heart. I think that was exactly the point the maintainers wanted to get across, and they have succeeded. It’s hard to resist the urge to put some nice cushy wrapper code around `reflect`, but at the end of the day it would be impossible to read and maintain for the poor soul cleaning up my mess in 5 years. So I’ll just avoid using it, as The Go Gods desire.

In these divisive times, can we all agree to the new zero based laws? Still three laws, just starting appropriately at 0.