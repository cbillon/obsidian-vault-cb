---
link: https://blog.tjll.net/you-dont-love-systemd-timers-enough/
byline: Tyler Langlois
site: Tyblog
excerpt: A cron job for every man, woman, and child.
slurped: 2026-06-03T09:43
title: You Don't Love systemd Timers Enough
tags:
  - linux
---

### [«](https://blog.tjll.net/this-is-your-sign-to-self-host/) You Don't Love systemd Timers Enough

- 5 May, 2026
- 2,139 words
- 9 minute read time

My favorite metonymic technology term is "cron job": even though `cron` may not literally be the daemon that executes actions on a schedule, we apply the term to anything that walks like a `cron` and quacks like a `cron`. As [Patrick McKenzie](https://x.com/patio11/status/1990437415383683416?s=20) likes to point out, cron jobs are one of the most eminently useful computing primitives. They offer utility that's almost immediately obvious for plenty of use cases that almost everybody has: do _this_ every day; do _that_ once a month.

And yet. You probably _shouldn't_ use literal `cron` (or its more modern cousins) for scheduled tasks! In 2026 there are more modern options available, and my favorite is the humble systemd timer. I love systemd timers. If you don't love them yet, maybe I can show you the reasons why you should love them, too.

#### _My_ `cron`? Cooked?

A systemd _timer_ is a type of unit that schedules other units (usually a service) on a particular schedule. (How a systemd _service_ unit works is another article, but you can logically consider the `.service` target of a systemd timer to be a script.) Timers are effectively a functional replacement for a traditional `cron` daemon (though you could conceivably run both), and timer calendar settings offer some similarities to help bridge the gap from traditional cron-like expressions.

At this point the systemd haters peer out of the woodwork in anticipation of torpedoing timers because they are part of the systemd project and because they replace mature (if clunky) technology. I'd rather not spend our time arguing about `cron`, so briefly consider why newer solutions like systemd timers that benefit from years of hindsight are better:

- Ambiguous `$PATH` settings make `cron` script execution difficult to predict.
- `stdout` and `stderr` output often ends up in a black hole (and, often, sent to the host's mail system, which is usually _not_ what you want to happen.)
- Execution history is difficult to follow and interrogate.
- You might feel cool knowing the scheduling grammar by heart, but `01,31 04,05 1-15 1,6 *` isn't easy or intuitive for humans to read.

Incidentally, timers solve all these problems (and more.)

#### Prime Time for a Timer Primer

We can cover the basics without a lot of ceremony. First you need a target for a timer to execute. On a Linux host with systemd operational, placing the following unit contents at `/etc/systemd/system/roulette.service` installs a service with a 1 in 10 chance to be free (i.e., shut down your computer):

Systemd

- Font used to highlight strings.
- Font used to highlight keywords.
- Font used to highlight type and class names.

```
[Unit]
Description=1 in 10 chance to break your chains

[Service]
ExecStart=/usr/bin/env bash -c '[[ $(($RANDOM % 10)) == 0 ]] && systemctl poweroff || echo LIVE ANOTHER DAY'
```

Update: [2026-05-05 Tue]

Twitter mutual [HSVSphere](https://x.com/HSVSphere) [points out](https://x.com/HSVSphere/status/2051695985147973828?s=20) that the service option `ExecCondition=` offers a native way to handle _conditional_ execution. This is a more tightly-integrated way to express "should I continue to execute?" and I agree that it offers a clearer way to express intent at the unit level (I'm using absolute paths here for a NixOS system):

Systemd

- Font used to highlight strings.
- Font used to highlight keywords.
- Font used to highlight type and class names.

```
[Unit]
Description=1 in 10 chance to break your chains

[Service]
ExecCondition=/run/current-system/sw/bin/bash -c '[[ $(($RANDOM % 10)) == 0 ]]'
ExecStart=/run/current-system/sw/bin/systemctl poweroff
```

This has the same effect as the prior `bash` conditional, and you end up with different wording in the journal that (in my opinion) expresses the situation more clearly for you when the condition is met:

May 05 11:05:32 diesel systemd[3117]: Condition check resulted in 1 in 10 chance to break your chains being skipped.

In general, leaning into the options that systemd presents is a better experience than scripting your own. (Another example would be to use `OnFailure=` to react when your service scripts fail or `Restart=` to attempt recovery in the case of ephemeral failures.)

Associate that _service_ with a _timer_ by placing a file with the same file stem (`roulette`) at `/etc/systemd/system/roulette.timer`:

Systemd

- Font used to highlight keywords.
- Font used to highlight type and class names.

```
[Unit]
Description=impending destruction

[Timer]
OnCalendar=10:00

[Install]
WantedBy=timers.target
```

What I mean by _associate_ is that, by default, a timer's `Unit=` setting will choose a service unit with a matching stem suffixed by `.service`. In this case, `roulette.service`. You can always change this if you want to execute a service with a different unit name.

I want to call out a few things right away:

- Per normal service unit semantics, the `ExecStart=` target _does not run as a shell command by default_. You should treat the absolute path target like a script or, in our case, an interpreter that expects a script as a string argument. For example, `ExecStart=/usr/bin/echo Hello | /usr/bin/awk` straight-up won't work; the pipe makes no sense in context here.
- The `ExecStart=` argument does _not_ inherit any environment variables by default (outside of some system manager defaults), so we begin with a pretty bare `$PATH` by default. Executing `/usr/bin/env` is a shortcut to ensure things like `systemctl` are available, but out of the box, you get a clean state to begin with. If we had used a bare `ExecStart=/usr/bin/bash`, we'd have the basics in `$PATH`, but using `env` here is an extra safeguard.

You can roll the dice without the aid of the timer at all:

shell

Although note that _you cannot `enable` this service_ without any usable `[Install]` section: our timer is the canonical way to make the service run in a consistent way. Also useful to highlight that `systemctl` operates on `roulette.service` by default without any explicit suffix.

When applied to a `.timer` unit, the `systemctl start` subcommand puts it on the clock, per se, but does not actually execute the `Unit=` target.

shell

```
systemctl start roulette.timer
```

The _timer_ is now active, but not the _service_.

Depending on the moment in time, `status` will tell you when to next expect the timer to decide your fate:

shell

```
systemctl status roulette.timer
```

You'll see plenty of information about the timer on its `status` page, including the next time it'll fire:

```
Trigger: Sat 2026-04-18 10:00:00 MDT; 35min left
```

That's the simplest timer onboarding: create a target, place the target service file alongside a timer with a schedule, and start the timer (not the target) to get the schedule started. Because the `.timer` defines an `WantedBy=` within `[Install]`, we can ensure the timer comes up at boot time too, not just when we `start` it:

shell

```
systemctl enable roulette.timer
```

Let's move on past the basics.

#### Time Lord

Arguably the most important bit of information about timers is how to express a schedule, whether a repeating period of time (which the manual usually refers to as a time span) versus a calendar event (or a timestamp). Fortunately, I think the `man` page for this under `systemd.time(7)` is actually very good with plenty of examples. You should use it as the _first_ resource when writing timers; it's good (or better) than, uh, casual blog posts by casual writers.

systemd also ships with a command-line tool called `systemd-analyze` which includes the ability to validate and explain time expressions from the command line directly in an imperative way to help understand them. You can even disambiguate the classic wildcard `cron` expression which `systemd-analyzer` can parse and then explain to you, complete with the expected execution times:

shell

```
systemd-analyze calendar '*-*-* *:*:*'
```

```
Normalized form: *-*-* *:*:*
```

This blog post is not the place to reproduce the entirety of `systemd.time(7)` verbatim, so I encourage you to Read The Helpful Manual (RTHM). Writ small, you can pretty simply define either a recurring wallclock period _or_, in contrast to plain old `cron`, a recurring period of time against some previous event.

The first category of time expressions is easy to envision. For example, in fully-qualified form, `daily` means:

```
*-*-* 00:00:00
```

You can use shorthand terms like `daily`, write out the complete form, or use any other supported value listed out in `systemd.time(7)` and subsequently validate your assumptions against `systemd-analyze`.

The _second_ category of time expressions apply to "run this relative to some other event." This distinction from "run at the same time very day" is very often what you _actually_ want. Consider a job that clears out a temporary directory, for example: if a `cron` expression lapsed right after boot, there probably isn't much to clean out of `/tmp` at all. But if you encode "execute an hour after my computer has started and then every hour after that", the schedule logic is meaningful for what the related service is actually doing.

This is easy to do in a timer:

Systemd

- Font used to highlight keywords.
- Font used to highlight type and class names.

```
[Timer]
OnBootSec=1h
OnUnitActiveSec=1h
```

That is: "run an hour after the machine starts" (which will execute once) and also "run one hour after my `Unit=` runs" (which implicitly makes the timer repeat indefinitely.)

Periodic time spans like this fit the "every once in a while" use case surprisingly more often than "run at this minute every hour" and similar expressions. Another good example is a timer I use every December to poll the [Advent of Code](https://adventofcode.com/) API for a Slack bot I wrote for some friends. The `*/15` cron expression honors the "every 15 minutes" policy that their API requests, but since that's the easiest way to express it in cron language, I'm sure it makes spiky traffic alongside everyone else polling the API! Starting my timer when I've made a code fix that runs _whenever_ 15 minutes has lapsed is all I care about, and probably creates less of a thundering herd problem.

Calendar versus time span units is probably the biggest conceptual leap from a traditional cron job, but timers offer more, too.

##### Bird's-Eye Countdown

My favorite high-level command to get a picture of a machine's timer situation is the `list-timers` subcommand. Here's my host's summary:

shell

```
NEXT                                 LEFT LAST                                  PASSED UNIT                         ACTIVATES
```

From one command you glean a total picture of anything executing on a timer schedule. Very useful.

`list-timers` is part of a family of systemd subcommands that I use fairly often. Others that are useful include `list-units` and `list-paths` (the latter is a more recent addition to `systemctl`.)

##### Suspended Reanimation

Waking a suspended system to run an important script even if you're not around to perform the physical action of, say, lifting a laptop lid _sounds_ like a daunting feat until you find `WakeSystem=`:

```
WakeSystem=
```

You can imagine the utility for something like this. On a distribution that supports downloading package updates before using them (like Arch or NixOS, for example), you can pre-fetch update packages late a night for morning updates when you're at a keyboard, and there are plenty of other ideas you could apply this too as well. The man page highlights that you'll need to manually re-suspend if you intend for that to happen after your `.service` is done.

##### Splay-away

I touched on the thundering herd problem a few paragraphs ago, which is the systems problem of, "what happens when a set of processes all wake up at the same time?" If every Debian system in the world were hard-coded to `apt update` at `00:00:00`, midnight would be a bad, spiky time for everyone.

Two timer options called `FixedRandomDelay=` and `RandomizedOffsetSec=` help:

```
FixedRandomDelay=
```

I've used this for real systems that check in to update software. Not only does it help with thundering herd problems, but spreading out execution along a uniform distribution ensures that the behavior is consistent and avoids disruptive activities like restarting daemons that may be coordinating distributed services.

In general, timing options are very configurable and expose a great deal of granularity (again, all of which are explained in the `man` page.)

##### Insistent Persistence

This option is particularly well-suited for scripts on a schedule that shouldn't ever be skipped due to a suspended laptop but may not warrant `WakeSystem=`:

```
Persistent=
```

If you're scheduling a system to checkin to configuration management but the host has undergone downtime, slapping `Persistent=` on the `.timer` can mean the difference between converging onto correct state immediately after coming online versus whenever the timer may normally fire (which could be a long time.) There are other good examples of service activations you don't want to wait for if the timer detects a missed activation: system updates, checking for batch jobs, that kind of thing.

#### A Quick Recap

If you start writing timers in earnest, bear the following in mind:

- Timers that run in the context of a _user_ manager (the kind you interact with via `systemctl --user`) are totally valid, but keep an eye on the **target** you use for `[Install]`. Sometimes the appropriate target to use for this is `default.target` depending on the distribution.
- The usual caveats about keeping a correct system clock apply like they always did with `cron`. Fellow systemd cult members can rely on `timedatectl timesync-status` to peek at their synchronization status.
- Many editors natively support the systemd unit file format which helps when unit files become large. I use the [emacs systemd](https://melpa.org/#/systemd) package.

---

---