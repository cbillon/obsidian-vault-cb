---
link: https://www.drupal.org/node/2845379
excerpt:
slurped: 2026-08-22T07:51
title: Client Challenge
---

# Provide optional composer integration but don't force users to understand how to use composer

One of the items mentioned during some of Dries' keynotes is that there's a danger of Drupal leaving beginners and site builder behind. A major step in the current march forward is Composer usage. While Composer is a great tool and should become the standard for building complex projects, it should not be _required_ to build a Drupal site unless a GUI is provided; additionally it should always be possible to download files from d.o, put them in the correct location (after extracting the archive) and have it work.

> it should always be possible to download files from d.o, put them in the correct location (after extracting the archive) and have it work.

We can wish for this all we want, but it's technologically impossible due to the way Composer is built.

Mixologic and I discussed this, and agreed that the right solution is to offer a rewritten Update Manager that embeds Composer (basically shipping with updater/ next to core/). That way people either use Composer, or they use the UI which runs Composer and then moves the changes into place. We also get core updates for free.

The problem here is that it still requires composer. Most shared hosting servers do not have it installed by default, so to add composer requires knowledge of how to install it - and we're back to the command line. So while having GUI access is definitely a step in the right direction, users who cannot use the command line are still cut off.
For my part, I'm not against having a UI to soften the steep Composer learning curve.
Add a GUI for Composer, do not require composer for building sites, do not leave non-experts behind
Agreed that we don't want to force anyone to have to use the shell to deploy/maintain Drupal.

There's a couple of constraints that we want to consider when designing a system that will allow us to abstract away the complexities of using composer:

Composer is designed to be a build time/development tool, and is not intended to exist or be available in production systems. What that means in practice is that the normal constraints of "code outside the firewall and open to the internet" are not baked into composer, and thus it could be considered a security risk to have it available on a production system that interfaces with threats. Thats not to say it cannot be on live sites, just that upstream, the developers are not likely to focus on making it hard to compromise. Even if we were to audit it now and find that its fine, that doesn't mean that the next version wont have some obvious remote code execution vulnerability baked into it.

The second aspect of composer is that it is designed to be executed in a command line php environment - which means it is expecting to have the ability to consume considerable amounts of memory, and take significantly longer execution time than most webserver php environments are geared for. Drupal 8's 64MB minimum requirement would probably need to bump to 256MB or higher, and we might need as much as 180 seconds worth of timeout for php. Or more.

I bring up these two points to underscore the difficulty we may encounter in attempting to bundle composer within the drupal application itself. There are likely ways we can cordon it off for security purposes, and ways we can ensure its only running on a cron or something similar. Another alternative is to construct a site management tool that is separate from your web app, much like wordpress's desktop administration application ([https://developer.wordpress.com/calypso/](https://developer.wordpress.com/calypso/)).

Composer isn't the only sitebuilding barrier, and we ought to address the "Constructing, Extending, Updating, and Maintaining" the codebase your website needs to run in a holistic manner - so that we can handle any other "Buildtime/Development tools"- i.e. we can add build tools like NPM or bower to solve the 'gather front end assets too' so as to provide a seamless, delightful experience for drupal users who do not fall into the classification of "working on enterprise sites for enterprise clients"

I also strongly believe that we should start by designing the ideal user experience we would like to see in a tool like this, and aim high enough to knock it out of the park.

it's potentially dangerous to keep Composer, the software that you use to resolve your dependencies, in your production environment in such a manner that your production environment can actually execute it.

In other words 'composer dependencies' themselves are not potentially dangerous, but having "composer/composer" itself, as a dependency, is potentially dangerous.

As far as the updating and codebase management aspect, this is a situation where we _want_ a Drupalism. Composer/NPM etc are the 'industry standard best practices'' that we would adopt, and not reinvent. The goal would be to add value to those tools in such a way that gives drupal a distinct competitive advantage, specifically for the market of site builders/non developers/folks without the same resources as "enterprise" etc.

Thats why Im advocating for something that is not just a gui version of composer, but something more specific to Drupal, the product, that hides all the work composer is doing from the end user. I.e. this tool shouldn't even have the word "composer" in its interface. We want to give the users a button that says "add this module to my site" and it would do everything necessary to ensure that they end up with all of the php, javascript, css, fonts, and any other assets required to start using the new feature they want.