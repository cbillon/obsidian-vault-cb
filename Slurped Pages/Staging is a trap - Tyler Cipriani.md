---
link: https://tylercipriani.com/blog/2022/09/15/staging-is-a-trap/
byline: Tyler Cipriani
excerpt: Software staging clusters only grow.
slurped: 2026-06-01T08:38
title: Staging is a trap - Tyler Cipriani
---

![@mipsytipsy](https://photos.tylercipriani.com/2022-09-15_staging-tweet.png)

Software staging clusters only grow.

As production accrues more services, staging’s costs ramp up.

And maintaining a single, massive, production-like staging may no longer be the right answer.

But several, small staging clusters—each fit for their purpose—offers a more maintainable, cheaper alternative.

## 🍝 There is no perfect staging; there are only perfect stagings

Each reason for having a staging cluster requires a different level of “production-likeness” to fit its use.

You could put Apache and PHP on raspberry pi and call it Wikipedia’s “staging.”

And that’d be a fine place to demo a MediaWiki patch. But to be confident deploying that patch into Wikipedia’s production: (to paraphrase “Jaws” 🦈): you’re gonna need a bigger staging .

In the 1970s, the pasta sauce brand Ragu tasked the psychophysicist Howard Moskowitz with finding the perfect pasta sauce.[1](https://tylercipriani.com/blog/2022/09/15/staging-is-a-trap/#fn1)

Moskowitz concluded the perfect pasta sauce doesn’t exist—it depends on each individual’s wants and needs. **There is no perfect pasta sauce; there are only perfect pasta sauces.**

Howard’s work is why you’ll find [Ragu Old World Style®](https://www.ragu.com/our-sauces/old-world-style-sauces/old-world-style-marinara-sauce/) next to [Ragu Chunky Garden Vegetable](https://www.ragu.com/our-sauces/ragu-simply/chunky-garden-vegetable) next to 10s of other Ragus.

And much like pasta sauce[2](https://tylercipriani.com/blog/2022/09/15/staging-is-a-trap/#fn2): **there can be no perfect staging; only perfect stagings**.

## 💱 Staging trades cost vs. production-likeness

The requirements for a staging cluster depend on its use.

- **Demos** – Demoing new code requires the same software, maybe a few microservices, and (possibly) a subset of production data.
    - 🟢 Low cost
    - 🟢 Low production-likeness
- **Exploratory testing** – QA requires the same software, services, and a subset of production data. It’d be nice to run it on the same type of infrastructure, too.
    - 🟡 Medium cost
    - 🟡 Medium production-likeness
- **Deployment confidence** – 100% confidence requires a parallel universe you destroy whenever a deployment goes wrong.
    - 🔴 High cost
    - 🔴 High production-likeness

![Staging trades costs for nearness to production](https://photos.tylercipriani.com/2022-09-15_staging.png)

Staging is a trade-off: resources (money, people, time) against asymptotically approaching **actual** production.

The closer you get to production, the higher the costs and complexity.

## 🔁 Production should be reproducable

> **Setting up a staging server should be easy.** If it is not easy, you already have a problem in your infrastructure, you just don’t know it yet
> 
> – Patrick McKenzie 🐉, [Staging Servers, Source Control & Deploy Workflows, And Other Stuff Nobody Teaches You](https://www.kalzumeus.com/2010/12/12/staging-servers-source-control-deploy-workflows-and-other-stuff-nobody-teaches-you/)

Configuration management should make it easy to rebuild production from scratch. Otherwise, you’ve got a disaster in the offing.

This creates an environment suitable for demos and end-to-end test automation.

But organizations are evolving away from using pre-production staging to build deployment confidence.

They’ve replaced their high-cost staging with a mix of [canary deployments](https://martinfowler.com/bliki/CanaryRelease.html) and advanced [feature flagging](https://martinfowler.com/bliki/FeatureToggle.html).

Small environments with narrow scope—like testing or demoing—seem like a reasonable trade-off of cost vs. benefit.

But using pre-production staging as insurance for your deployments—requiring snapshots of production data and maybe even replayed traffic—seems too. darn. expensive.

## 📚 Further reading

- [Why we don’t use a staging environment](https://squeaky.ai/blog/development/why-we-dont-use-a-staging-environment)
- [Dear staging, we’re done](https://devops.com/dear-staging-were-done/)
- [Delete your staging environment](https://flagsmith.com/blog/delete-your-staging-environment/)

---

1. This is an anecdote related by Malcolm Gladwell in a 2007 Ted Talk, “[Choice, Happiness, and Spaghetti Sauce](https://www.ted.com/talks/malcolm_gladwell_choice_happiness_and_spaghetti_sauce)[↩︎](https://tylercipriani.com/blog/2022/09/15/staging-is-a-trap/#fnref1)
    
2. There’s a joke here about “spaghetti code” that I’m too lazy to find.[↩︎](https://tylercipriani.com/blog/2022/09/15/staging-is-a-trap/#fnref2)