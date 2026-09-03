---
link: https://lalitm.com/post/github-alternatives/
byline: Lalit Maganti
site: Lalit Maganti
date: 2026-08-01T17:00
excerpt: Why forges like Codeberg, GitLab, and Forgejo can replace GitHub's code
  hosting—but not its shared social layer.
slurped: 2026-08-24T08:45
title: GitHub has alternatives, but no replacement
---

[Codeberg](https://codeberg.org/), a Git code hosting platform, recently took a [decision to prohibit projects that mostly consist of generative-AI-written code](https://blog.codeberg.org/protecting-our-floss-commons-from-llms.html) which has prompted [concern](https://xn--gckvb8fzb.com/i-regret-migrating-to-codeberg/) and extensive [discussion](https://news.ycombinator.com/item?id=49021856) [elsewhere](https://lobste.rs/s/ax914v/protecting_our_floss_commons_from_llms).

The decision does not surprise me, and I don’t mean that as a criticism. Codeberg has always presented itself as a mission-driven alternative to GitHub, not neutral infrastructure.[1](#fn:1)

What interests me is the disappointment in the response. Many people reacted as though one of the few plausible GitHub replacements had ruled them or their projects out. They wanted Codeberg to be a universal alternative, a better GitHub and the obvious place to go when leaving it.

To me, that exposes a big gap in the open-source space. There are plenty of places to host a Git repository, but remarkably few places to host an open-source community. GitHub gives projects a shared pool of identities, habits and paths to discovery. None of the alternatives has reproduced that at a similar scale.

I don’t think the answer has to be another centralized platform, or that every project should live in one place. But decentralization is not enough on its own. Whatever replaces GitHub still needs a shared social layer: identities contributors already have, conventions they understand and ways to discover projects across the network.

## Why not self-hosting?[#](#why-not-self-hosting)

Whenever dissatisfaction with GitHub comes up, someone inevitably says: “Git is decentralized. Just self-host a forge.”

I’ve self-hosted Gitea for years, so this is an argument I’m very familiar with. Self-hosting works well for personal projects, but I wouldn’t use it for something I wanted strangers to contribute to. On GitHub, most people already have an account and understand how issues and pull requests work. On my forge, even reporting a small bug means creating another account, learning how my forge works and what conventions I want you to follow. Unless someone really cared, they probably wouldn’t bother. I know I wouldn’t.

And contribution is only half of it. GitHub used to be genuinely good at discovery. I regularly found projects because someone I followed starred them, often in areas I would never have searched for myself. It felt like a social network built around people making things.

GitHub has since redesigned that feed, and I almost never visit it anymore. Defaults are powerful: once discovery stopped being part of the experience GitHub put in front of me, it largely disappeared from my workflow.

## Why not GitHub?[#](#why-not-github)

The basic experience of GitHub has been getting steadily worse. It is slow, things regularly fail to load and notifications are unreliable. GitHub itself recently described two major incidents as [“not acceptable”](https://github.blog/news-insights/company-news/an-update-on-github-availability/).

Its pull request experience has been awful too. Large PRs are painfully slow to navigate and review. Stacked PRs[2](#fn:2) have been common inside large software companies for well over a decade, but only _just_ [became a thing](https://github.blog/changelog/2026-07-30-stacked-pull-requests-are-now-in-public-preview/) with GitHub and, even then, seems to be [quite](https://news.ycombinator.com/item?id=49113452) [buggy](https://news.ycombinator.com/item?id=49122109).

What frustrates me about GitHub’s push towards AI is that the core forge feels neglected while Copilot appears everywhere. An agent writing more code doesn’t help when the interface for reviewing it is already struggling.

Ghostty exemplifies this frustration. In late April, Mitchell Hashimoto announced that [Ghostty is leaving GitHub](https://mitchellh.com/writing/ghostty-leaving-github) because frequent outages were preventing its maintainers from working reliably:

> On the day I am writing this post, I’ve been unable to do any PR review for ~2 hours because there is a GitHub Actions outage. This is no longer a place for serious work if it just blocks you out for hours per day, every day.

Interestingly, he _also_ makes the point that GitHub is more than hosting:

> To the “Git is distributed!” crowd: the issue isn’t Git, it’s the infrastructure we rely on around it: issues, PRs, Actions, etc.

Hashimoto said Ghostty was in discussions with multiple commercial and FOSS providers and planned an incremental migration. The fact that such a prominent project had to shop around, rather than move to an obvious default, is exactly the gap I mean.

## Why not the alternatives?[#](#why-not-the-alternatives)

It’s worth going through the alternatives and the problems I see with each:

[GitLab](https://about.gitlab.com/) is capable, but it feels incredibly corporate, even more so than GitHub.[3](#fn:3) Nor have I found it as good as GitHub at helping people stumble across projects and developers.

[SourceHut](https://sr.ht/) is focused and transparent and, like Codeberg, openly values-driven.[4](#fn:4) Its email-oriented workflow, while battle-tested by projects like the Linux kernel, is unfamiliar to most GitHub users.

[Forgejo](https://forgejo.org/)’s federation project may eventually connect self-hosted instances into a shared network. It looks promising, but has been in development for quite some time, remains experimental[5](#fn:5) and is not yet a practical answer to the social fragmentation of self-hosting.

[Radicle](https://radicle.xyz/) is technologically interesting: repositories are replicated peer to peer, while issues and patches are stored alongside them. But it still feels too immature to replace GitHub for a public project. For example, its public web interface lets people browse repositories, but contributing requires them to install its CLI or desktop application. Someone encountering a bug should not have to install the forge’s software merely to report it.[6](#fn:6)

The project I’m most interested in is [Tangled](https://tangled.org/), based mostly on gut feel. I like its focus on the social experience around code. For example, its home page immediately shows me a bunch of cool projects, exactly like old GitHub used to do. But it is still in alpha, and it remains to be seen whether it can blossom into a true alternative.

## Could a boring company fill the gap?[#](#could-a-boring-company-fill-the-gap)

One thing that stands out is that, apart from GitLab, _every_ competitor is built around either decentralization or a social mission. That makes each of them fundamentally less “straightforward” than a for-profit company. It makes me wonder whether there is room for one.

Perhaps it could look more like [bunny.net](https://bunny.net/) than a venture-backed startup: a deliberately boring company with no ambition to become the operating system for software development or to reorganize programming around whatever technology investors currently find exciting. It might simply concentrate on making open-source collaboration pleasant and reliable, charge developers and smaller organizations directly, and grow at whatever pace that revenue supports.

The open question is whether this could be a viable business. The difficult part is exactly what I keep saying is missing: shared identity, shared conventions and discovery only become valuable once a platform has reached scale. Better repository hosting alone would not solve that cold-start problem.[7](#fn:7)

I don’t know who, if anyone, will solve it. But a real replacement will have to treat the social layer as the product, not as something that appears automatically once enough repositories are hosted.