---
link: https://medium.com/@tpierrain/bounded-contexts-user-interfaces-part-1-30f1be4fd864
byline: Thomas Pierrain. (υѕe caѕe drιven)
site: Medium
date: 2023-07-14T11:48
excerpt: Recently, we had some interesting debates at work about the boundaries between our different Bounded Contexts. When we discussed what happens in our final applications (for end users), we realized…
twitter: https://twitter.com/@tpierrain
slurped: 2024-08-26T18:18
title: Bounded Contexts & User Interfaces (part 1/2) - Thomas Pierrain. (υѕe caѕe drιven) - Medium
tags:
  - ddd
  - context
---

_Recently, we had some interesting debates at work about the boundaries between our different Bounded Contexts. When we discussed what happens in our final applications (for end users), we realized that we were not necessarily aligned. I also had the opportunity to discuss this with my friend_ [**Julien Topçu**](https://twitter.com/julientopcu) _to get his opinion on the matter (a very interesting discussion). Since these topics are not extensively documented, I thought it would be worth writing this series of 2 mini-articles._

## Which “contexts” are we talking about again?

Who hasn’t encountered difficulties in understanding the differences between Domain, Sub-Domain, Bounded Contexts, Modules, etc.? I don’t think anyone has ;-)

That’s why I prepared this little diagram years ago, to help colleagues better grasp all of this. The main difference lies in whether we focus on _the problem space_ or _the solution space_.

Some basics definitions from DDD

In the _problem space_, we talk about Domains (e.g., hospitality) and Sub-Domains (e.g., “booking”, “stay”, “loyalty”). A sub-domain is itself a Domain.

**A Domain encompasses knowledge as well as a set of problems we want to solve.**

When we then delve into the _solution space_ to solve these problems with software, people who, like us, practice [**Domain-Driven Design (DDD)**](https://medium.com/@tpierrain/domain-driven-design-explained-in-3-minutes-d45ed92fcc1c) very often map each Sub-Domain to a Bounded Context.

**A Bounded Context (BC) is an area we define** to apply a clear domain model and language to solve specific problems (for a sub-domain), along with all the necessary software components (code, database, messaging, etc.) that are consistent and valid for that purpose.

**A Business Context can be diffuse**, spread across various parts of software (often described as an “old monolith” or “big ball of mud”), **but a Bounded Context is not supposed to.**

The contribution of Domain-Driven Design is to emphasize the importance of delineating these Business Contexts into coherent and congruent areas in our software (often depicted as potatoes but later isolated into modules, services, etc.). **And to use language of domain people as a guidance to identify these boundaries.**

Then, we need to consider how we connect them all together in case we need to involve other BCs to solve our users’ problems (and Strategic DDD provides a set of patterns to achieve that). A map of Bounded Contexts (and their relationships) is called a Context Map, and here’s an example.

Example of a Context Map

## What is an application?

On the side of end-user applications, you can have mini-applications that only address a small sub-domain (although it’s rare), or more likely, applications that cover multiple subjects, multiple sub-domains (and even multiple Domains 😳 in some case).

In the multiple subdomains case, we will have to compose and assemble multiple Bounded Contexts in the same end-user-facing showcase, namely the Application.

When it comes to web applications, this assembly/mash-up code will be divided between front-end code (executed in a web browser) and back-end code (executed on the server).

Regarding the back-end code, it can either:

- Be a “modular monolith,” meaning a collection of several modules (understand several BCs) hosted in the same executable, which communicate with each other through method calls (in-proc)
- Be a module which communicates with other external (micro/macro) services through network calls (out-proc)

## Does an “application” BC make sense?!

The objective of this series is to clarify and consider the different options we have for grouping multiple BCs in our « Application » code (whether it’s back-end or front-end).

**In other words, should we consider this mash-up of BCs in our application code as a separate “application” BC** (and if so, to which business area(s) should we associate it, what name should we give it?), **or should we consider it merely as an aggregation of several BCs?**

I’ll let you ponder over the question (and make your suggestions if you wish in the comments) before discussing our proposals [**in the second and last episode of this series**](https://medium.com/@tpierrain/bounded-contexts-user-interfaces-part-2-2-a09d51cfc08f).

See you soon