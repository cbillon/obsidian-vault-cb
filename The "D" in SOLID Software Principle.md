---
tags:
  - go
  - solid
---


[site](https://hashnode.com/@larryd)

Recently I reviewed a few good old articles around topics related to clean architecture. One of them is this interesting article [A little architect](https://blog.cleancoder.com/uncle-bob/2016/01/04/ALittleArchitecture.html), to explain Dependency Inversion ([DIP](https://en.wikipedia.org/wiki/Dependency_inversion_principle)) in [SOLID](http://en.wikipedia.org/wiki/SOLID_\(object-oriented_design\)) principle.

I've also been doing some tangible work in the real system, it's good to expand some thinking on what this DIP to do with the maintainable software.

The "official" words to define the rule of DI principle:

> _High-level modules should not depend on low-level modules._ Depend upon abstractions, [not] concretes."
> 
> _Abstractions should not depend on details. Details should depend on abstractions._

A bit dry definition, what do these words mean in practical and what real value we could gain from it?

## [](https://wpding.hashnode.dev/the-d-in-solid-software-principle#heading-what-are-dependencies "Permalink")What are dependencies?

This one is easy to understand as the software systems we build today heavily rely on external infrastructure. Think of database, file storage, cache or queue. They are mainly provided by cloud platforms or third party services (payment, email etc). Our software needs to communicate to those external pieces (usually via the form of client SDKs) to fulfil our business requirement. These all create dependencies.

## [](https://wpding.hashnode.dev/the-d-in-solid-software-principle#heading-what-exactly-are-inverted "Permalink")What exactly are inverted?

This is somehow not so obvious. The DIP basically guides us to manage these dependencies in a loosely coupled way. A loosely coupled system is that high level domain layer (the core biz logic which provides real values to end users) should not know any detail about DB, storage or message queue.

So when we say dependency inverted, essentially we mean these dependencies are abstracted away from biz domain based on [contract](https://en.wikipedia.org/wiki/Design_by_contract).

💡

What **contract** means? Simply speaking it's a set of method signatures with input and output types grouped around the domain, not including any specific type from external dependencies.

When doing this, what end up in the static code structure is that biz domain never directly _"interact"_ (import and invoke) with these details. some examples including concrete types representing DB, client libraries (http) or any specific data structure from 3rd SDKs. In fact biz layer don't even know whether we use DB or NoSQL storage, nor we use Stripe as payment provider etc. Even though from runtime perspective (when your software gets built and deployed in the production), it's the business domain layer calling into low level to trigger the real action.

The real value of DIP if we follow it correctly, is that it will give us a clear separation between business domain layer and external infrastructure, easy to test and verify core business logic in isolation without needing to know external details. Also it's easy to replace a certain piece of infra without updating other parts as long as they conform to the defined contract. Business domain layer remains untouched.

## [](https://wpding.hashnode.dev/the-d-in-solid-software-principle#heading-achieve-dip-in-practice "Permalink")Achieve DIP in practice

The key part is for Biz layer first to define these abstracted contracts needed to fulfil certain business requirements.

These contracts are then implemented in low level modules (details). It's low level that imports the abstracted contracts needed by high level biz domain, not the other way around. All the infra details will be handled here.

To illustrate the idea where the inversion happens, consider a common case where we need to build a API to expose some HTTP endpoints for the client app (say user checkout flow in an online shopping scenario). see digram below:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704082070708/54f8ab5a-c196-466f-baf3-054dd2c11611.png?auto=compress,format&format=webp)

Compared to red execution arrow, the blue arrow goes backwards if we look from code "import" perspective, hence "**inverted".**

Hope this post gives you a better idea on what the DIP is. To wrap up, I take one part of the whole conversation from Uncle Bob's article:

From this little ambitious architect:

> The high level policies (I presume you mean the business rules) call down to the low level policies (I presume you mean the database or any external infra). So the high level policies depend upon the low level policies in the same way that callers depend upon callees. Everybody knows this!

The reply from maybe a real architect:

> _At runtime this is true. But at compile time we want the dependencies inverted. The source code of the high level policies should not mention the source code of the lower level policies._

Happy Architecting!

## [](https://wpding.hashnode.dev/the-d-in-solid-software-principle#heading-further-read "Permalink")Further Read

[DIP in the wild](https://martinfowler.com/articles/dipInTheWild.html)

[SOLID relevance](https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html)

[Dependency Injection (DI) and Inversion of Control (IoC)](https://martinfowler.com/articles/injection.html)

[https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)


# [Go Favor Composition: Struct Embedding](https://wpding.hashnode.dev/go-favor-composition-struct-embedding?source=more_articles_bottom_blogs)

[Let’s explore another Go unique design: Embedding. This post is for first type of embedding: struct …](https://wpding.hashnode.dev/go-favor-composition-struct-embedding?source=more_articles_bottom_blogs)