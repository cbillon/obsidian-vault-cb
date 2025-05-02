---
link: https://medium.com/@tpierrain/bounded-contexts-user-interfaces-part-2-2-a09d51cfc08f
byline: Thomas Pierrain. (υѕe caѕe drιven)
site: Medium
date: 2023-07-15T16:39
excerpt: Back after setting some definitions and the main question in the first article here. Now, we’ll try to review several options and draw a conclusion. First, let’s remind ourselves of the main…
twitter: https://twitter.com/@tpierrain
slurped: 2024-08-26T18:22
title: Bounded Contexts & User Interfaces (part 2/2) - Thomas Pierrain. (υѕe caѕe drιven) - Medium
tags:
  - ddd
  - context
---
_Back after setting some definitions and the main question_ [**_in the first article here_**](https://medium.com/@tpierrain/bounded-contexts-user-interfaces-part-1-30f1be4fd864)_. Now, we’ll try to review several options and draw a conclusion._

First, let’s remind ourselves of the main question: **Should we consider the “application” code that combines various Bounded Contexts (BCs) as a separate “application” BC? (and if so, what should we call it?)** Or should we consider our application (front or back) as a mere aggregation of multiple BCs? (considering the additional application code as primarily technical)

_Small disclaimer: As a practitioner of hexagonal architecture (and a fan of DDD), I will only focus on the business code here, and not the technical or infrastructure code._

It seems to me that there are several cases (note: I won’t attempt to be exhaustive)

## Case #0: Big ball of mud (whether on the front-end or back-end)

In this unfortunately widespread configuration, there are no Bounded Contexts or any other form of isolation. All the contexts are mixed in the same code, often with a single modeling approach for various uses, perspectives, and different business contexts. In short, it’s a legacy mess. If you insist, you could refer to it as a “Legacy” BC, but we’re mostly dealing with an absence of boundaries between different contexts. No Bounded Context, no potato to sketch!

Classical big ball of mud application (aka. the ol’ monolith)

## Case #1: Well-isolated BCs in the application (whether on the front-end or back-end)

This can happen **when you use micro front-ends** **(MFE, for good front-end isolation)** and your back-end application is a modular monolith or an orchestrator of (micro/macro) Services behind (where different BCs are also well isolated). **Except for the shell code, each MFE represents a business BC.**

In this case, it seems to me that the different BCs play their roles, and there is no need for a separate “application” BC. **Such application is merely the host of multiple activities or behaviors, and each piece of code should be attached and categorized within a business BC** (except for the infrastructure code).

Possible Bounded Contexts of an Amazon web site implemented with MFEs

Front side : one MFE per BC, with a Modular Monolith back-end (and 1 module per BC)

Version where front-end BC is implemented as a set of Micro Frontends (MFE), while the back-end relies on micro/macro services (with one service per BC).

## Case #2: Well-isolated BCs in the application, but also one or more BCs that exists only on the front-end side

My friend [**Julien Topçu**](https://twitter.com/julientopcu) mentioned an example of this scenario from his past experience while he was working for one of the leaders in the European tourism industry. Julien mentioned the existence of a “cross-sell” BC that sells a set of partner products and services (e.g., car rental) after a flight or hotel search.

For performance and decoupling reasons, the code of the “_cross-sell_” BC directly made API calls from the front-end code of their travel website: “_It was a front-end module that captured train search criteria, for example, and sent it from our front-end directly to some partners APIs (part of the ‘Car’ domain)_”

**In this case as well, we are not talking about an “_application_” or “f_ront-end_” BC that is agnostic to the business for this cross-sell code**.

Instead, it is a business BC called “_Search_” (cross-sell is implicitly part of the Search domain for business professionals).

Our front-end here is the only location for the “Cross-Sell” BC wich -in that case- is part of the Search BC (and directly calling partners’ APIs)

## Case #3: The front-end is a single-page application (SPA) that merges all BCs into one (without MFE), or a mobile application (i.e. a complicated subsystem)

**In this case**, regardless of how the back-end is structured, **the front-end application is its own unique BC doing the mash-up of all BCs** (combining all the involved business concepts).

As Julien pointed out to me, this is often the case for “_complicated subsystems_” (see. [**Team Topologies**](https://teamtopologies.com/)) that cover a broad functional scope but require specific expertise that is not distributed across teams (e.g., mobile app development). In this specific case, and only in this case from my perspective, we can talk about a “_mobile_” BC or “_web app_” BC.

I don’t find this name ideal as it’s not business-related. However, **while I understand its necessity for mobile applications, I believe that for the web, merging all BCs into a single SPA in the era of MFE is not ideal for moving towards Continuous Delivery or autonomous teams**.

Here, the front-end app has only one “mobile” BC doing all the mashup

## Conclusion

**To me, an “application” Bounded Context (instead of a business-related term BC) often indicates a smell**.

An application is nothing more than a means serving ends that correspond to business activities or usages. **Hence we should be able to find business or domain-oriented names for our non infra code.**

It’s worth noting that 2 web sites with the same look and feel can have completely different implementations:

- One could be made of multiple BCs (if MFEs are in place)
- The other could be made of a single BC (as in the case of SPAs or mobile apps) to represent the lack of boundary in our front-end code (handling all subdomains in one module)

Since **Bounded Contexts are an isolation mechanism**, it’s logical that their presence or absence goes hand in hand with the level of isolation possible in our web or mobile applications.

**Also, my point is that an “application” BC is not a sufficiently business-oriented description to have a place when talking about BCs**. However, this term is very useful when discussing technical architecture.

If you’re unsure whether to separate your application code into different BCs (applies to both front-end and back-end code), I recommend asking yourself:

_“Are we using EXACTLY and PRECISELY the same business language in these two locations of the codebase?”_

If the answer is no, it’s probably an indication that you should organize your code into 2 or more distinct BCs (using modules or MFEs, for example)

In any case, I believe it’s preferable to give Bounded Contexts business or domain-specific names.