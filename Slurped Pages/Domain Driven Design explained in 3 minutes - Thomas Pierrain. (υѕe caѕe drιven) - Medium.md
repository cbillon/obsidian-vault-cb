---
link: https://medium.com/@tpierrain/domain-driven-design-explained-in-3-minutes-d45ed92fcc1c
byline: Thomas Pierrain. (υѕe caѕe drιven)
site: Medium
date: 2022-10-02T10:10
excerpt: Domain Driven Design (DDD) generates lots of fantasies. For some it’s a set of concrete practices, for others it’s something that looks complicated & barely definable. Aggregate is probably the most…
twitter: https://twitter.com/@tpierrain
slurped: 2024-08-26T18:34
title: Domain Driven Design explained in 3 minutes - Thomas Pierrain. (υѕe caѕe drιven) - Medium
tags:
  - ddd
---

Domain Driven Design (DDD) generates lots of fantasies. For some it’s a set of concrete practices, for others it’s something that looks complicated & barely definable. Aggregate is probably the most commonly quoted term when people are talking about DDD. More and more often I also hear people saying they do DDD because they use CQRS…

If I need to explain what DDD is to a newcomer, I would say this:

> DDD is both an approach and a 2-levels toolbox to build software.

## An approach first

**To be curious & sturdy about the domain you need to solve problems for**. It should be like an obsession to us (devs) to align everything we do with your domain. During that journey, it’s key to understand what is core domain (i.e. your competitive advantage) and what is not. Indeed you won’t put the same efforts & make the same trade offs if you are working for your core domain or a supportive one. Eric Evans also told us 2 important things to understand his approach:

## 1. Domain Language really matters

**Identifying & using the proper language -the language of your domain- to model & to code is paramount**. Modelling helps to align people’s mental models & mindset around that domain. The more we will talk, think & model about our domain, the crisper our ideas will be to design & build our software (it’s an iterative process we call the Whirlpool model exploration). **Being FAITHFUL to our domain language** wherever in our code, database, DTOs, UX… is what is called Ubiquitous Language.

## 2. Context is key

Because there is no such thing as a unique domain language; there are many perspectives, vocabularies, meanings & versions depending on the domain people and usages you are talking about. For instance, an “Invoice” for our customers when they are buying our SaaS solution to monitor their Cash Flow is not the same thing as one of their Registered ‘Invoices’ we want to reconcile banking transactions for afterwards (in our software).

It’s key to understand that if you don’t want to end up with misunderstandings, bugs and issues. Eric suggests we put boundaries around the various Contexts we’ve identified so that some models/concepts/constraints/specificity of one context won’t leak into another one. Building specific and contextualized models to solve people’s problems, protecting and isolating your various models and thinking carefully how to connect them will bring you efficiency, low coupling & high cohesion in your architecture. To carefully identify and understand which Contexts are involved (hence which models) in our solution, we love to draw them as potatoes in Context Maps and to qualify their relationships & coupling. Moreover, we love to align our teams organization with our Bounded Contexts (it fosters autonomy & good time to market)

That’s it for the approach. And **if you just understand that and use it wisely: congratulations, you are already a DDD practitioner!**

But we said that DDD is both an approach and a 2-levels toolbox to build software. Let’s conclude with the toolbox.

## A 2-levels toolbox

The toolbox established by Eric in his blue book has 2 levels:

## **A tactical part**

Aiming to deal with code with things such as Aggregates, Entities, Value Objects, Repositories, etc. I’ll let you dig further into this toolbox on your own, but since some of them are depending on your tech environment, languages & programming paradigm (either OOP or FP) I think this is less important to master it than the approach or the strategic part.

## **A strategic part**

Aiming to deal with architecture and socio-tech concerns (i.e. what kind of team power relationships you must deal with to build software). You will discover Context Maps, and tons of patterns to survive whatever the situation. Things such as: Anti-corruption Layer (ACL), Customer-Supplier, Conformist, Open Host Service, Published Language, etc. This was the real new thing brought by Eric in his book and probably the most important section to understand after _the Approach_. Last but not least: Even if it has been invented after by [Alberto Brandolini](https://twitter.com/ziobrando), I also suggest you to discover [**Event Storming workshops**](https://www.eventstorming.com/) that will help you a lot to understand & clarify things.

a Context Map