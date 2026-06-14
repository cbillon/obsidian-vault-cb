---
tags:
  - moodle
  - plugins
  - install
---
https://moodle.org/mod/forum/discuss.php?d=471110
from Sergio Rabellineau
I'm a plugin developer and a sysadmin of many Moodle instances: actually, but things are likely to change in the future, the plugin management does not have any enforcement that could simplify their management further. Someone uses the "current" distribution via the zip, others update their [git](https://moodle.org/mod/glossary/showentry.php?eid=10110&displayformat=dictionary "Glossary of common terms : Git") repository but with tags not perfectly aligned with the Moodle naming (e.g. MOODLE_501_STABLE), so it's up to you to understand what each plugin developer is doing. Sometimes there is a new zip, but no updates on the corresponding git too...

To better standardise, my suggestion is to create an intermediate repository for each plugin you would like to use, connect the official git as upstream, and pull from your Dockerfile using your standard for tags/branches. So a sort of man-in-the-middle. This also allows putting in production temporary changes (bugfixing) without waiting for global changes.

![[Pasted image 20260604113709.png]]
I personally would not dare to use Windows for any kind of Moodle hosting or even testing, you will hit big problems sooner or later...

![[Pasted image 20260604114200.png]]
The longer term goal is to move _everything_ out of the web root and the only things _in_ the web root are the router, and a few other bits and pieces. Ideally _none_ of the plugins should be accessible directly. Everything should be loaded from a common location (the router).

![[Pasted image 20260604114427.png]]
> - Could it be that the Moodle organization assumed that everyone runs Moodle as the only application on a dedicated server?

No. I certainly don't. I would _strongly recommend_ that any website is hosted on it's own _subdomain_. That's a completely different thing to a different _dedicated server_, and is completely unrelated to this change in Moodle itself.

> - The decision to mess with pointing "Document root" away from your webroot seems very odd. It can obliterate your website.

It really shoudn't obliterate anything. This is a very safe change. Moodle _should_ have been implemented like this 20 years ago. [Mahara](https://moodle.org/mod/glossary/showentry.php?eid=7663&displayformat=dictionary "Glossary of common terms : Mahara") was designed like this from the start (19 years ago). Symfony has always done this; so has Laravel, and plenty of other apps.

> - The addition of Composer, along with its warnings, for something not needed, also seems a bit strange.

The change for Composer is a separate thing. Let's not conflate the different issues.

To be clear, we will be using Composer to manage third-party libraries in the future. The addition of the _warning_ is to _prepare_ people for this fact. No packages are currently installed. We want to ensure that people get used to, and have the piecs in place for the new workflow before we do use it.

> - It appears these changes will present significant challenges to "Installatron's," (even though we don't like these Installatrons). I think the days of Installatrons are over because I can't see how they can manipulate Moodle's Document Root requirement. If they do, I would love to see how they manage it. And if they do so with some strange configurations, who will be able to help solve problems?

We do not recommend the use of "Installatrons". It is really up to the people supporting those systems. I imagine that they already have systems in place because many other products already do this.

> - That you cannot easily run different versions of Moodle on a server seems to be a very restrictive way of thinking.

But you can... I run about 30 different Moodle sites with 7 different versions on my local machine. Whilst this is my local development machine, the only thing that makes it 'special' is that it's my laptop and I frequently make changes to it to test the many weird, and wonderful, things that people do. The configuration is very similar to any production system.

There is _nothing_ stopping people from doing this in production (though I would not recommend having any unsupported versions of Moodle).

> - That you should not use your web server to serve other non-Moodle web pages is bewildering.

I have to disagree. You wouldn't take a word document and dump a random image into it. Why is this any different?

I'd really recommend spending the time to learn about subdomains. You get them, for free, as part of your [CPanel](https://moodle.org/mod/glossary/showentry.php?eid=7547&displayformat=dictionary "Glossary of common terms : CPanel").

> - That having to make symbolic links to get around these problems is a complete mess.

Why? This is not a bodge or hack. I have been using this as a solution to host things for decades. There is nothing wrong with this.

![[Pasted image 20260604115909.png]]

Hi Moodlers,

Yesterday I started experimenting with a new way to quickly set up a local Moodle demo site on macOS and Linux.

It is based on [MDC](https://github.com/mutms/mdc), my fork of the Moodle Docker Container project. The main difference is that there are very few configuration options, which makes it much easier to use for non-technical people.

The code is at [https://github.com/mutms/demo](https://github.com/mutms/demo)

To use it:

1. Install [Docker Engine](https://docs.docker.com/engine/install/) or [OrbStack](https://orbstack.dev/) — Docker Desktop is not required.
2. [Git](https://moodle.org/mod/glossary/showentry.php?eid=10110&displayformat=dictionary "Glossary of common terms : Git") is pre-installed on macOS and most Linux distributions, but install it if needed.
3. Run the following commands from a terminal:

```
git clone https://github.com/mutms/demo
cd demo
bin/init-moodle
```

Your demo site will be available at [http://127.0.0.1:9501/](http://127.0.0.1:9501/)

There are also scripts to back up and restore your demo site with all sample data and Moodle source code — you can even pass a backup to somebody else to try it out.

I have also started working on a set of scripts to set up WSL2 on Windows and use the same demo scripts. It appears to work well and performance is decent. I will create a separate GitHub repo for that later this week.

Ciao, Petr