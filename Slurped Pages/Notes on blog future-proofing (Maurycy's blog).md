---
link: https://maurycyz.com/misc/futureproofing/
excerpt: >-
  One of the great things about web pages is that they are long-lived and
  mutable.

  There's no need to aim for perfection on the first draft:

  A page can continue to be improved for years after its original publication.
slurped: 2026-08-31T09:38
title: Notes on blog future-proofing (Maurycy's blog)
---

**2026-01-23** — **2026-03-06**

_One of the great things about [web pages](app://obsidian.md/misc/starting_a_blog/) is that they are long-lived and mutable_. There's no need to aim for perfection on the first draft: A page can continue to be improved for years after its original publication.

However, this mutability comes at a cost:

[![](app://obsidian.md/misc/futureproofing/first_server.jpg)](https://commons.wikimedia.org/wiki/File:First_Web_Server.jpg)

DO NOT POWER [IT] DOWN!! — The first web server.

Servers are just computers: If they ever break or are turned off, the web site vanishes off the internet.

_If you've ever been reading something more than a few years old_, you've probably noticed that [none of the links work](https://en.wikipedia.org/wiki/Link_rot). Even if the destination site still exists, It's common for them to have [changed the URL format](https://www.w3.org/Provider/Style/URI) so that old links don't work.

To be clear, links are a good thing: They allow readers to look deeper into a topic, and [external links](app://obsidian.md/real_pages/) are how we find new places on the internet.

## Preserving external links:

_3rd party are services like [archive.org](https://archive.org/) are hit-and-miss_: By most accounts, only [around 50%](https://arxiv.org/abs/1212.6177) of pages ever make it to the archive, and even if they have a copy, it's still just a web site: Many other archiving services have vanished or lost data. These services are good for archiving one's own site, but aren't great at defending against link rot.

_If I want to be sure links will always work, they have to be archived locally._

I don't want to run a [crawler:](https://en.wikipedia.org/wiki/Web_crawler)

Unless carefully watched, these can place a lot of [load on the target](https://lwn.net/Articles/1008897/) server or/and fill up my disk with infinite dynamic pages: These could be intentional [honeypots](app://obsidian.md/babble/entry-point) or something as harmless as a web based calendar.

I'd spend more time putting out fires than actually writing.

With that in mind, I decided to use Chromium's "save" feature to archive single pages. This has one huge benefit over something like recursive wget:

It saves the final DOM, not what was served over HTTP.

_A lot of sites use Javascript to render content_: For example, Substack uses it render math, and despite popular belief, there's more then just Nazis on there: It's also home to [Lcamtuf's](https://lcamtuf.substack.com/) excellent blog. Other sites go further by delivering all content as JSON and rendering it client side. You might think that only large corporate sites do this... [but that's just not the case](https://kristoff.it/blog/static-site-paradox/).

These types of pages could be preserved with a caching proxy, but the odds that fifty megabytes of Javascript work in ten years are not good:

It's better to run the Javascript now and save the results for later.

Format choice

Chrome supports saving in two formats: MHTML and standard HTML with a directory to store the resources.

On paper, [MHTML](https://datatracker.ietf.org/doc/html/rfc2557) very nice — it's a standardized, single-file web archive with browser support — unfortunately it's only really supported by Chrome: depending on a single application is not great for long-term preservation.

Right now, I have enough space to store both formats: When a link breaks, I'll either serve MHTML (faster, more faithful) or the multi-file archives (more compatible) depending on the current state of support.

## This site itself:

This blog uses an [(almost) zero-dependency site generator](app://obsidian.md/misc/new_ssg/): The only thing it needs is a C compiler.

_When it does break, all the previously generated HTML can be served as-is_: It's only used to update the site.

All the blog posts have URLs beginning with `/projects`, `/misc`, `/tutorials` or `/astro`: If I reorganize things, it won't take up a lot of namespace to keep the old URLs working.