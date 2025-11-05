---
link: https://www.dolthub.com/blog/2024-10-04-reflecting-on-reflect/
byline: Neil Macneale
site: DoltHub
date: 2024-10-04T00:00
excerpt: Golang reflection is wonderfully horrible. Let's reflect on it.
twitter: https://twitter.com/DoltHub
slurped: 2025-11-05T09:15
title: Reflecting on Go Reflection
---

![Reflecting](https://www.dolthub.com/blog/_astro/tree_reflection_1.pCt3bbYl_1wF8Mq.webp)

_[Dolt](https://github.com/dolthub/dolt) is a database with branching and merging written in Go, and this post is one our many [Go related blog posts](https://www.dolthub.com/blog/?q=golang)_

Despite my 20 plus years of writing code for a living, I’ve never been a programming language enthusiast. I don’t seek out new languages just for fun. Having worked at Amazon, Google, and [some other places with smart people](https://www.linkedin.com/in/macneale/), I’ve heard the full spectrum of opinions from developers about their particular language of choice. Faster! Safer! Dynamic-er! These are usually very firmly held opinions which result in a lot of debate and agitation. For my part, I usually try and find a way to remove myself from such conversations.

I’m a practical programmer. I generally use the most mundane features of any language. If I’m successful at any given programming task, then some poor soul is going to be maintaining that code in 5 years. So I tend to avoid getting into the weeds with the esoteric stuff. Unless I break my own rule…

There occasionally is the need to do something gross and difficult to maintain. Like the time I needed `void**` in a C program to pass around anonymous functions, only to cast them into whatever I thought was necessary. That was in a “security” product. Yikes. Or that time I used JNI to access an OS specific system call to get that performance boost. That sure put a nail in the coffin of portability for that program.

Which brings us to Golang’s [reflect](https://pkg.go.dev/reflect) package. As part of an experiment a few weeks ago, I needed to maliciously alter a piece of data in a Dolt database. As I had never used the Go `reflect` package, I thought I’d give it a try. Not knowing any better, I fell into the reflection pool and almost got lost in a parallel universe forever. The dream world depiction of this experience by AI is quite accurate. I felt like there was no longer any up or down and that inside was outside.

![Falling In](https://www.dolthub.com/blog/_astro/falling_in.BHmI0N-V_1o115f.webp)

## Dog is my Copilot[#](https://www.dolthub.com/blog/2024-10-04-reflecting-on-reflect/#dog-is-my-copilot)

As I usually do, I started writing some code to figure out how to use the `reflect` package. Living in the age of [Copilot](https://copilot.microsoft.com/) and [other generative AI](https://supermaven.com/) tooling, some pretty credible looking code pops into my text editor and I’m on my way.

```
		bsVal := reflect.ValueOf(blockStore).Elem()

		tables := bsVal.FieldByName("tables")

		typ := tables.Type()
		fmt.Printf("tables.Type: %v\n", typ)
		for i := 0; i < typ.NumField(); i++ {
			fmt.Printf("tables %d: %s\n", i, typ.Field(i).Name)
		}
		for i := 0; i < typ.NumMethod(); i++ {
			fmt.Printf("method %d: %s\n", i, typ.Method(i).Name)
		}
```

What this does isn’t terribly interesting, and isn’t actually the point. The point is that it looks reasonable, especially to someone who doesn’t know the design patterns of `reflect`. It looks to me like it’s going to take `blockStore` and start interrogating it by printing out its fields and methods.

Cool story, but no, it actually panics right away. The documentation, readily available at my fingertips by hovering over the `FieldByName` method, is quite clear that it will panic if given bad input. That much was immediately clear, but nevertheless I still struggled to make use of the `reflect` library until I understood its governing principles.

![Ripples](https://www.dolthub.com/blog/_astro/ripples.Cm0cAlhA_SoW5I.webp)

## Laws of Reflection[#](https://www.dolthub.com/blog/2024-10-04-reflecting-on-reflect/#laws-of-reflection)

If you are going to use `reflect`, you have to read [Rob Pike’s Laws of Reflection](https://go.dev/blog/laws-of-reflection). I confess it took me longer to find it than it should have, but maybe the lesson there is that your Wizbang IDE with all that fancy AI crap is no substitute for reading the docs.

If you refuse to read Pike’s Laws, then I have no choice but to force them in front of your eyeballs:

1. [Reflection goes from interface value to reflection object](https://go.dev/blog/laws-of-reflection#1-reflection-goes-from-interface-value-to-reflection-object)
2. [Reflection goes from reflection object to interface value](https://go.dev/blog/laws-of-reflection#2-reflection-goes-from-reflection-object-to-interface-value)
3. [To modify a reflection object, the value must be settable](https://go.dev/blog/laws-of-reflection#3-to-modify-a-reflection-object-the-value-must-be-settable)

The first two laws don’t seem very controversial. The `ValueOf()` and `Interface()` methods basically convert between values and reflection objects. These two laws are even double summed up by Rob:

> In short, the Interface method is the inverse of the ValueOf function, except that its result is always of static type interface{}.
> 
> Reiterating: Reflection goes from interface values to reflection objects and back again.

The 3rd law was harder for me to accept. I wanted to get code in the `main` package to call an unexported method in another package, but it turns out that’s just not possible. For the particular problem I was working on this ended up being a non-starter and I had to go back to the drawing board. That said, it was enough exposure to the `reflect` package that I have opinions about it.

![Laws](https://www.dolthub.com/blog/_astro/laws.UfM5ltes_QL2zp.webp)

### The Missing Law[#](https://www.dolthub.com/blog/2024-10-04-reflecting-on-reflect/#the-missing-law)

When people started talking about thermodynamics, there was a general assumption that no one mentioned until later, thus the [Zeroth Law](https://en.wikipedia.org/wiki/Zeroth_law_of_thermodynamics) came into existence.

The authors of Go have kindly given us three laws of `reflect`, but I believe they have neglected to mention the 0th law, which should be:

0. Use `reflect` at your own peril. Misuse it, and it will `panic` with no regrets.

I believe this design decision, to panic when anything is the slightest bit wrong, is a very clear and loud statement that the maintainers of Go don’t want you to use the `reflect` package. It’s kind of a genius move to make it feel toxic right out of the gate.

It goes beyond just panics though. [`unsafe`](https://pkg.go.dev/unsafe) package is mentioned a bunch of times in the docs, and `unsafe.Pointers` are used in several interfaces. I think the only way to further discourage the use of the `reflect` package would be to make every method deprecated.

I think it’s safe to say that any production use of `reflect` should be highly scrutinized and done with extreme care. It’s not impossible. Anyone using the [`encode/json`](https://pkg.go.dev/encoding/json) is hooked on this stuff.

![Missing](https://www.dolthub.com/blog/_astro/4laws.6FkW1AOz_Z2rQyb5.webp)

### Abolish the 3rd Law[#](https://www.dolthub.com/blog/2024-10-04-reflecting-on-reflect/#abolish-the-3rd-law)

My first use of reflection was with Java where `java.lang.reflect` is able to update pretty much any value you want. When put in combination with a custom ClassLoader and the ability to hot install [ByteCode](https://docs.oracle.com/javase/8/docs/api/java/lang/ClassLoader.html) you have a truly powerful (dangerous) tool at your disposal. `private` fields in Java don’t really have any meaning when up against `java.lang.reflect`.

In Go, that is not the case. You can read any field but you can only update fields that are exported (start with a capital letter). For methods on a struct you can’t even learn of their existence if they aren’t exported. And yet, you can call an exported method which presumably performs writes to data you wouldn’t otherwise be able to modify.

There are probably deeper reasons why unexported fields can’t be updated, like the compiler strips some metadata which makes it impossible. Putting that aside though, given that the `reflect` package is not fit for production use (refer to 0th law), we should be able to update any field and call any method even if it’s not exported. The safety of not being able to get my hands on an internal field seems unnecessary when we have the threat of panicking lurking in every corner. I’ve been given a shotgun to blow off my foot, but I have no shells.

With full ability to go bonkers with writes in `reflect`, it would be possible to build a wider set of test and debug tools. In Dolt’s particular case, we were attempting to break the storage format as part of our regular testing, and that’s code we didn’t want to reside in the Dolt code itself - hence going down this path. More generally though, it could enable a wider set of tools than we can build with the existing behavior.

At a very minimum, we could encode unexported fields in JSON structs. Wouldn’t that be nice?

![Fireworks](https://www.dolthub.com/blog/_astro/fireworks.C7t7-fWs_2vzmEd.webp)

## Closing[#](https://www.dolthub.com/blog/2024-10-04-reflecting-on-reflect/#closing)

`reflect` is not for the faint of heart. I think that was exactly the point the maintainers wanted to get across, and they have succeeded. It’s hard to resist the urge to put some nice cushy wrapper code around `reflect`, but at the end of the day it would be impossible to read and maintain for the poor soul cleaning up my mess in 5 years. So I’ll just avoid using it, as The Go Gods desire.

In these divisive times, can we all agree to the new zero based laws? Still three laws, just starting appropriately at 0. Come tell us we’re wrong on the [Dolt Discord server](https://discord.gg/gqr7K4VNKe).