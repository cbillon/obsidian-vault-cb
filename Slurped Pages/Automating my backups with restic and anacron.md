---
link: https://ntietz.com/blog/automating-my-laptop-backups/
excerpt: |-
  I've been running my backups by hand[1] every week on my laptop for as long as they've been set up.
  Automating them was something important but was on the back burner, because, well, it never felt very important.
  Then I lost a few days of work when my SSD died, and it felt more urgent.
slurped: 2026-03-16T15:31
title: Automating my backups with restic and anacron
tags:
  - backup
---

I've been running my backups by hand[[1]](https://ntietz.com/blog/automating-my-laptop-backups/#fn-by-hand) every week on my laptop for as long as they've been set up. Automating them was something important but was on the back burner, because, well, it never felt very important. Then I lost a few days of work when [my SSD died](https://ntietz.com/blog/setting-up-a-new-machine-2023/), and it felt more urgent.

Haha, no, I still kept doing it manually every week.

It was really a friend talking to me and reminding me that I should do it that kicked me into gear. And there was an episode of [Changelog](https://changelog.com/podcast) which talked about [ntfy](https://ntfy.sh/). Things kind of came together and I decided to finally automate the backups. Then I procrastinated for two months, and did it in January of 2024!

## What do I want?

For automated backups, we have a few requirements.

First, clearly, we need backups to run automatically. These should run _daily_. And I also want a snapshot pruning job to run _weekly_ to keep my storage costs down.

Then we also need alerting. I want to know if a backup has not been generated for three days. (One or two day gaps are expected, since I'll often have my laptop off for travel.) I want this to send a notification to my phone in some form: alert, email, text, I don't care. Locally notifying my on my laptop would also be nice, in case the laptop is on but backups broke.

And finally we need to have periodic tests of the backups. Backups aren't worth a lot if they don't work, so you should sometimes check that they do! And I definitely did this one, but no spoilers. Definitely did it.

## Running tasks daily

Running things on a schedule is the bread and butter of [cron](https://en.wikipedia.org/wiki/Cron). The only snag is that I do not expect my laptop to always be powered on, so the job may sometimes be scheduled to run when it's sleeping or off. The answer to this came from [Fedora's docs](https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/monitoring-and-automation/Automating_System_Tasks/): we should use [anacron](https://en.wikipedia.org/wiki/Anacron).

anacron lets you run jobs just like cron, but handles downtime. If the computer is off (or on battery power), jobs are not run. Then when the computer is back on AC power, it will run the jobs! For backups, this is great.

To set it up, I created two files. For the daily backups, I put this script in `/etc/cron.daily/0backups`:

```
#!/bin/bash
set -o errexit -o nounset -o xtrace

cronic /home/nicole/Code/management-scripts/nightly_backup.sh
```

And for the weekly pruning, this is in `/etc/cron.weekly/0prune`:

```
#!/bin/bash
set -o errexit -o nounset -o xtrace

cronic /home/nicole/Code/management-scripts/weekly_prune.sh
```

There's one other thing in there, `cronic`. When tasks fail with anacron, they send mail to you! Not like email, though that can be supported I think? But local system mail, which the `mail` command will show you.

With anacron, it will send mail for _any_ output, which is pretty annoying for automated daily tasks. I just want to know if it fails! What [cronic](http://habilis.net/cronic/) does is collect all the input and only emit it if there is a failure (a non-zero return code), so you only get mail for the failures.

To send and receive that mail locally, I installed postfix and configured it for local-only delivery, which is the default on Fedora. On my Debian machine, I had to install the `mailutils` package also to have the `mail` command. Having this mail was _critical_ for debugging my jobs, because otherwise I could not see what was happening when it ran!

## Alerting if it breaks

Okay, so now we have our backups running daily. Or so we hope. How will I know if it breaks?

The answer is, like the answer to so many things, to throw more software at it! Here we have two pieces involved.

First we have [ntfy](https://ntfy.sh/), which lets you send push notifications when something happens. (I know we want to know when something _doesn't_ happen, sssshhhhh, we'll get there). I have it send me a notification whenever the jobs run. You can configure its priority, so a successful backup is a quiet notification, but a failure gives me an actual notification that buzzes my phone.

This is an example of how I have it setup in my backup script, with keys omitted. Basically, if the `backup.sh` script succeeds, it will ping ntfy (and another service, sssshhhhhh), but if it fails, it'll ping ntfy even _harder_.

```
bash backup.sh \
    && curl -H prio:low -H "Title: Backup success" -d "$DEVICE backup succeeded!" -s $NTFY_URL \
    && curl -fsS -m 10 --retry 5 -o /dev/null $HC_URL \
    || curl -H prio:high -H "Tags: rotating_light" -H "Title: Backup failure" -d "$DEVICE backup failed!" -s $NTFY_URL
```

So that's the happy path, and it does give me a lot of peace of mind to see a notification every morning that my servers backed up successfully.

To get the alerts if the backups never run, we turn to [Healthchecks.io](https://healthchecks.io/). It does integrate with ntfy, but I'm not sure that it goes the direction I want and can _receive_ from ntfy. Anyway, I didn't figure that piece out. What I did do is integrate it in the same way, with a curl if the job passes and nothing if it fails.

For each machine I back up, I have them set up to expect a ping every day. If Healthchecks.io receives the ping on time, then we're all good. If not, it waits for the grace period (6 hours for my servers, 3 days for my laptop to accommodate travel[[2]](https://ntietz.com/blog/automating-my-laptop-backups/#fn-travel)), and then it alerts me by email.

## Testing the backups

So that just leaves us with testing that our backups work. I, uh. I'll get back to you on automating this bit.

For now, I have a periodic test where I will restore files. It's manual, and it works, and it gives me peace of mind to see the backups restoring successfully.

If anyone has better ideas for automation of backups, or if you have a way you like to test backups, I'd love to hear about it!

---

---

1. Here, "by hand" means running the script that does it, but having to run that script myself. [↩](https://ntietz.com/blog/automating-my-laptop-backups/#fr-by-hand-1)
    
2. I was away for four days last week, and it did indeed alert me! [↩](https://ntietz.com/blog/automating-my-laptop-backups/#fr-travel-1)
    

_If you're looking for help on a software project, please consider [working with me](https://cuteandfuzzylogic.com/)!_