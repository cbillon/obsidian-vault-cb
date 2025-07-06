---
link: https://cnr.sh/posts/2017-09-11-trust-but-automate/
excerpt: If you want to enforce a technical guideline or rule, you must automate the enforcement mechanism. Relying on human beings to do the enforcement is too error prone, yet this is exactly the pattern ...
slurped: 2025-06-28T09:21
title: Trust, but automate | cnr.sh
tags:
  - rules
  - automation
---

## Trust, but automate

If you want to enforce a technical guideline or rule, you must automate the enforcement mechanism. Relying on human beings to do the enforcement is too error prone, yet this is exactly the pattern that I see pop up again and again.

Suppose a team wants to define database schema rules. Things like banning the use of BINARY in MySQL in favor of VARBINARY, or the requirement for create and modify time columns in every table. It’s surprising how a group of otherwise intelligent engineers will sit in a room and agree that these are the required rules, and then go and publish them on a page somewhere. "There, done," they say. Wrong.

## Humans aren’t good at this

People aren’t perfect. Rules are circumvented, mistakenly or otherwise. You need something to enforce the rules. Here again, humans are poor performers. They get bored. They are biased. They can be persuaded. They forget things. Computers don’t do this. It should go without saying, but if the way you plan to enforce your rules is via a human, it’s probably going to fall apart. Even if you documented the rules. Even if the document has animated gifs, bold letters, and threats. It just doesn’t work well.

The solution is to seek out what can be automated, and spend the time to automate it. Ultimately, it’s the responsibility of the person (or group) that defined the rules to make sure that the rules are enforced. This means it’s their job to automate the enforcement, whether that’s writing scripts, turning on commit hooks, or enabling lint checks.

## What about exceptions?

The problem with rules is that there are always exceptions. When the rubber meets the road, when it’s 3 a.m. and the site is down and you just need to get something fixed. You need to _consciously_ and _intentionally_ circumvent the rules. But you’ve just gone and automated everything! You need to bake the exception flow into the automation process.

At LinkedIn, we had the concept of an "[ACL](https://en.wikipedia.org/wiki/Access_control_list) override". You could execute an ACL override when committing code to a repository that you didn’t normally have access to. After approval from another engineer, the override was logged, and an email was sent to the entire engineering organization to alert everyone that an override had occurred. The tool was seldom used, but it was there in case it was needed.

Make sure that automation allows for its circumvention, but does so in a way that makes it clear that this is not normal. Causing the user a little pain is fine, and arguably a good thing. Log and alert when a circumvention occurs, and make it clear to the user what they’re trading off by breaking the rules.

## Limitations

Sadly, not all rule enforcement can be automated all the time. Whenever a human being is required to make a qualitative judgement, automation bumps up against its limits.

Let’s take [Semantic Versioning](http://semver.org/) as an example. Take a moment to review the website if you’re unfamiliar with it. One of the rules from the website reads:

> Patch version Z (x.y.Z | x > 0) MUST be incremented if only backwards compatible bug fixes are introduced. A bug fix is defined as an internal change that fixes incorrect behavior.

Backwards compatibility is something that can be ([kind of](http://hisham.hm/2016/03/24/you-cant-automate-semver-or-there-is-no-way-around-rices-theorem/)) automated and tested via a script (assuming there’s a rigorous way to define a public API), so that’s good news. But what about this claim about a bug fix being something that fixes incorrect behavior. Who determines what a bug fix is versus what a feature is? One could try to define a "feature" and "bug fix" issue type in an issue tracker, and link the issues to the commits. Transitively, this would connect the issue type to the version bump, and automated checks could then verify that "feature" issues are only resolved when a minor version bump occurs, not a patch version. Still, who defines whether an issue is a feature or a bug? Things get a bit wishy-washy.

Ultimately, some processes are just going to boil down to humans making judgement calls. In these cases, automate what you can, but be aware that you no longer have a rule — you have a guideline. You can still derive useful information, but it’s dependent on the quality of the human judgements being made. This is where culture becomes important. If people care about the output of the automation, they’ll be more likely to think about their decisions, and input things that make sense.

## In summary

Keep your eye out when engaged in conversations surrounding guidelines in rules. If you’re in a meeting where people define the end-point as documenting the rules, push back. Tell them to trust users to read the rules, but automate the enforcement.

---

#### Changelog

- **Fix redirects** on February 11, 2025
- **Migrate to markupdown (#1)** on February 11, 2025

---