---
link: https://lalitm.com/responsibility-is-taken-before-it-is-given/
byline: Lalit Maganti
site: Lalit Maganti
date: 2026-08-15T21:31
excerpt: To get promoted as a software engineer in Big Tech, you generally need
  to show “ownership” of a product or area. This means being responsible for it
  and deciding what should happen over time. I often …
slurped: 2026-08-24T08:22
title: Responsibility Is Taken Before It Is Given
---

To get promoted as a software engineer in Big Tech, you generally need to show “ownership” of a product or area. This means being responsible for it and deciding what should happen over time.

I often hear more junior engineers ask: “How can I demonstrate ownership when my manager hasn’t given me anything to own?” This way of thinking is a trap: an easy one to fall into, but a trap nonetheless.

The problem is that they are treating responsibility as a complete, self-contained opportunity that someone must give them first. In my experience, they’ve got it the wrong way round: **responsibility is demonstrated before it is formally given.**

Put yourself in the shoes of your technical lead (TL): by “giving” you responsibility, they are implicitly accepting the consequences of your decisions. They need to trust you not only to get work done but to exercise judgement without constant supervision.

The most important question they ask themselves is: “Does this person deal with problems the same way I would, **or better**?” If the answer is yes, they can trust you with more responsibility.

## How I Became an Owner[#](#how-i-became-an-owner)

When I started working on [Perfetto](https://perfetto.dev/), I built its trace-analysis tooling mostly under my TL’s direction. As I began making implementation decisions myself, he would often spot major issues I had missed.

For example, I built separate classes for three or four data structures I thought were unrelated. My TL insisted they were all tables, even though I couldn’t see how to unify them. Today, Perfetto’s trace processor has more than 100 tables, all declaratively defined and generated from that common abstraction.

I always tried to understand his thought process: how had he even thought of something I had missed? What approach had he used to get there?

Over time, I adopted his approaches myself, using them to pressure-test API changes or performance improvements before proposing them. More of my ideas started receiving a simple “yes, go ahead.” By then, I was setting the agenda myself: talking to users, understanding their problems, translating those into code changes and deciding what to improve next.

Sometimes the dynamic even reversed: he would suggest something, and I would explain why it would not work.

Eventually, my TL started calling me “the owner of the trace-analysis tools” instead of “the engineer who works on them.”

## Take Responsibility Without Overstepping[#](#take-responsibility-without-overstepping)

To be clear, this does _not_ mean grabbing projects or stepping on other people to demonstrate ownership. If you exercise sound judgement within your existing scope, a reasonable TL should gradually trust you with more responsibility.

Of course, this does not work with a bad manager or in a bad environment, where earning responsibility may be more about politics than judgement.

Formal responsibility is a trailing indicator, not a leading one. You’ll find that people begin trusting and deferring to your judgement before the responsibility becomes explicit. **Responsibility is taken in small increments before it is given in full.**