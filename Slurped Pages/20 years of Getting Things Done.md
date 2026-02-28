---
link: https://gagor.pro/2025/11/20-years-of-getting-things-done/
byline: Tom
site: Tom's Blog
date: 2025-11-27T01:00
excerpt: Reflections and practical tips from 20 years using Getting Things Done, plus a short workshop plan for leaders who want actionable routines.
tags:
slurped: 2026-02-24T10:37
title: 20 years of Getting Things Done
---

## My beginnings with GTD

> TL;DR If you’re not interested in the basics, you can jump straight to the [Flowchart](#flowchart-gtd-at-a-glance) and my learnings from the methodology.

I’ve been a long-time user of GTD (“Getting Things Done”). It started when I read [“Time Management: Strategies for System Administrators”](https://gagor.pro/book/2008/zarzadzanie-czasem/) while I was at university. A friend lent it to me - we were system and network administrators in the dorm at the time. The book introduced me to a few “radical” ideas about time management 😉.

Some of the advice was simple, for example:

- If you don’t want to forget something, send an email to yourself (this was before smartphones made it obvious).
- Don’t wait until someone tells you about system failures - use active monitoring like Nagios.

One example stuck with me. I was responsible for the company’s backups - a very important task. There was an LTO tape device with 12 cassettes (2-3 days of backups) and a piece of software that told me which tapes could be overwritten. I used to search a big box for the correct tapes. The whole task took 10-15 minutes and was a pain in the… you know 😉

The book suggested a simple idea: you don’t need daily manual checks. Backups are for disaster recovery and, for most situations, having backups from a day or two earlier is fine. If a backup is a week old it’s usually still acceptable - people remember recent work well enough. So I set up failure notifications on the LTO device, and only changed tapes when I got an alert. My boss was skeptical (I told him just in case), but after I started following the new system - nothing went wrong.

In the same company I introduced Nagios. They didn’t have monitoring before: there were around 10 servers, a mix of Windows and Linux machines. I added health checks gradually - web servers, mail servers, disk space, etc. We moved from being reactive (CTO: “What are you doing? Our shop is down!”) to proactive: we saw disks filling before failure and restarted services before anyone noticed. Adding [Monit](https://mmonit.com/monit/index.html) helped auto-heal some issues before they became visible.

Info

This early experience taught me the core GTD principle: **systems over memory**. Whether it’s backups, monitoring, or tasks - reliable external systems beat trying to remember and control everything.

This was a long introduction and I drifted a bit from GTD, but the point is the same: GTD changed how I organize work as a sysadmin and later as a DevOps engineer. I later received [“Getting Things Done”](https://gagor.pro/book/2012/getting-things-done/) as a Christmas present and learned even more. That all started almost 20 years ago!

## Basics of the methodology

Trying to simply describe GTD is hard - it’s a collection of rules and mantras. Start small: collect and categorize, then review and clean up regularly, finally plan daily and execute.

The initial target audience of the methodology were managers, people who already had systems for “what to do today”. GTD covers that and adds rules for what to do with tasks you can’t finish today.

I’ve worked in IT for most of my career, and Agile practices are close to my heart. There are similarities between GTD and [Scrum](https://en.wikipedia.org/wiki/Scrum_%28software_development%29) , or even more with [Kanban](https://en.wikipedia.org/wiki/Kanban) . If you know them, a few steps might look similar - just the names are different.

Let’s start with the basics.

**Core steps of GTD**

1. **Collect** - Capture everything.
2. **Clarify** - Decide if it’s actionable?
3. **Organize** - Where does it go?
4. **Reflect** - Review regularly.
5. **Engage** - Do the work!

### Collect

This is the first, and in my opinion the most important step - **capture everything** that has your attention: tasks, ideas, commitments, etc. When you capture everything, you will instantly stop saying: “I forgot about it”. It might not be important enough to be your next action, or actionable at all, but you will stop missing things.

Tip

**For me it was a game changer.** The place where you put your tasks and commitments is called: **Inbox**. If you see a connection to the Inbox directory of your email, you’re right. It works quite the same way.

A lot of stuff can land there, only some of it really requires your attention. At this stage, we do not filter anything, just collect.

Whether you use a paper notebook, email or “the best GTD app ever”, just put all your stuff there. You want your Inbox to be accessible, wherever you are and at any time. You might have a few Inboxes, but it’s good if you can easily move items to the main one.

Email will most probably be one of your Inboxes. One benefit of it is easy _delegation_ of collecting requests. If anyone catches you in the corridor and asks for something you can just respond:

> “I can’t note your request now and I don’t want to forget about you. Could you send me an email and quickly describe the issue?”

This puts the responsibility on the other side to “deliver” the request. If they won’t, it probably wasn’t important enough.

**App options:**

1. **Google**: [Tasks](https://tasks.google.com/) , [Gmail](https://gmail.com/) , [Calendar ](https://calendar.google.com/)
2. **Microsoft**: [Outlook](https://outlook.office365.com/mail/) , [To Do ](https://to-do.office.com/tasks/)
3. **Apple**: [iCloud Mail](https://www.icloud.com/mail) , [Reminders ](https://www.icloud.com/reminders)

For my private stuff, I use Google. At work, Exchange server with Outlook works too. Apple apps feel a bit too simple for me, although if you get used to them they’d be fine too.

What I like about Gmail and Outlook is that you can easily put emails on your task list. There’s also a ton of other apps that allow you to do all of that (and sometimes even better). I tried a few of them, but I returned to the basics. Many new GTD apps provide lots of custom labeling options, additional statuses and automation. For me those features were just noise. I don’t need them. Just find what works for you.

Info

About the goal of this stage - think about your mind like CPU cache: use it efficiently and don’t keep rubbish there. Let a reliable external system hold your obligations.

### Clarify

This is where the real work starts. At some point, daily or weekly, you have to review your Inbox. The main goal here is to decide **on the next actionable step** for each item.

You start with the question: _Is it actionable?_

**If the answer is “yes”**, then what’s the next action?

- **2 minute rule** - If it takes less than 2 minutes to complete, just do it now and get it out of your system.
- For the rest, check [**Organize**](#organize) .

Note

The classic GTD “two-minute rule” says if the next action takes less than ~2 minutes, do it now. I found 5 minutes to be a more practical threshold - adapt it to your workflow, but keep the habit of immediate execution for short tasks.

**If the answer is “no”**, you have a variety of options:

- **Trash** - irrelevant, outdated, no longer required things can go straight to the trash, and it’s done! 😄
- **Incubate** - maybe it’s not the best time to do it now, maybe you’re thinking about changing a car, but your current one still works. Let’s pause it for a while. For any item that you think “Someday” or “Maybe”, move it to incubation. You have it there if you need it, but let’s move it out of your mind for now.
- **Reference** - when you finish something, or just receive information that’s important but doesn’t require action, [archive it](#organize) , and move it out of your way.

Tip

Trashing items is easy and enjoyable, and it counts as work! I think it doesn’t require further explanation. It’s a bit different with the remaining two stages.

Incubation is a way to pause things. The collection process will produce lots of items. I have lists like: Someday, Ideas, Targets/Goals. They allow me to track items put “on hold” until they become relevant and worth taking back to my “next action” list.

I’ll describe Reference in the next step.

Warning

**Common pitfall**: People collect everything but never review. Result? A new form of anxiety. The review habit is more important than perfect categorization.

### Organize

For all actionable items, you have to decide which list it lands on. There’s a variety of options:

- **Next action** - obviously, your TODO list.
- **Projects** - if more than one step is required to complete it, it’s a “Project” that will require planning and splitting into smaller actions.
- **Waiting for** - you track here things that other people are doing for you.
- **Calendar** - for stuff that happens or repeats at a specific time.
- **Reference** - non-actionable, you just need to archive this.
- **Someday/Maybe** - plans for the future, things that might require collecting more details.

Note

By the book, “Organization” is a separate step from “Clarification”, but personally, you’re gonna do them one by one, so the border is really thin.

Next action is obvious, so I’m not gonna describe it further here.

_Projects_ require [cyclic reviews](#reflect) to ensure progress and if needed, new actionable items should be added to your “Next action” list so they can progress. Usually when you finish them, you move them to “Reference” to collect the outcomes, or archive documentation for later use.

_Waiting for_ allows me to track what others owe me: money, books I borrowed, or just checking if a return from a shop has reached my bank account. I don’t have to review these daily, but once a week/month to check if they’re under control.

_Calendar_ wasn’t that obvious for me initially. I never liked paper calendars (analog) that others used. I didn’t have good habits yet. It changed when I got a Nokia 72, where Calendar and notifications were part of the package, and it only improved when smartphones arrived. Checking my calendar in the morning (for things I should pay attention to today) and in the evening (do I have to do something special tomorrow?) became my new habit. Learning this took me a while, but right now even my wife knows that if it’s not in my calendar - it won’t happen 😁

Info

**Reference system**: I use a simple directory structure in my “Documents” directory. I create directories with the pattern: `YEAR-MONTH-DAY Short description`. This way they’re sorted chronologically despite any modifications. At the end of the year, or beginning of a new one, I move all the directories from the previous year to a new directory: `YEAR`. This way older stuff gets out of my way, but doesn’t disappear. File search is usually good enough to dig through it when I need. I’ve surprised many people with the docs I’ve collected 😉

I use my Someday or Maybe list for things like holiday plans or stuff that I’m not sure is worth doing at all. It sometimes waits until I decide that I don’t care anymore or when someone reminds me and asks again for it. I have another list called “Ideas” where I park some ideas for later use. I dig through it when I have some time for exploration. The main concept here is to: not lose anything (aka collect everything) and get it out of your head for now.

### Reflect

On one side we want to flush our short term memory of the things we have to do. On the other we need space to think about them and plan the execution. That’s where “Reflection” takes place.

To keep the system working, we need to review actions (tasks) frequently and at different intervals for different levels of reviewing. You don’t need to check your “Maybe” list daily, but if you want things important to you to move forward, you have to split them into smaller chunks and plan for daily execution.

Tip

For me it’s usually enough to do a bigger review once a month. Short term planning once a week - on Monday, as it allows me to plan the week. And I start each day by checking things: what I have to do today, and what I want to do today. That’s my routine.

We have to learn to trust the system and use it for our benefit. This doesn’t mean we follow it blindly all the time. There will be things that don’t fit our routines, or feel odd, inconvenient. Think about what would work better for you and try to apply it. Start small and allow the system to evolve over time.

### Engage

Now that we have everything organized, it’s time to pick an action and execute it. Then repeat, and again, and again…

Lists allow you to be in control of what you do next. You can choose based on your energy levels, time available, or priority. Make it your routine and deliver.

There are many applications and utilities that can support you. I remember in the beginning when I was using a cardboard folder to store documentation. I’m doing everything digitally now.

I also have to admit, I’m not perfect. It happens that I skip a daily review. I rarely check tasks during vacation. But every single time when I’m under pressure - I go back to my system.

## Flowchart: GTD at a Glance

I wrote a wall of text above, so let me share a picture with you, that should simplify things a bit.

flowchart TD

A[Collect] --> B[Clarify]
B --> |Is it actionable?| C{Actionable?}
C --> |No| D[Trash / Incubate / Reference]
C --> |Yes| E[Define Next Action]
E --> |Takes < 2 minutes?| F{< 2 minutes?}
F --> |Yes| G[Do it now]
F --> |No| H[Delegate or Defer]
H --> |Delegate| I[Waiting For List]
H --> |Defer| J[Add to Next Actions or Calendar]
J --> K[Organize]
I --> K
D --> K
K --> L[Reflect]
L --> M[Engage / Execute]
M --> A

## My thoughts on GTD

I’ve read the [“Getting Things Done”](https://gagor.pro/book/2012/getting-things-done/) book many times, discovering something new each time. I tried to follow the process strictly, but it never really worked for me. There were too many routines and some felt artificial. Let me describe what worked.

### Tasks vs Events

That was one of my first and very important discoveries - that there are “Tasks” and “Events”. Tasks are things you have to do - they go on your task list. Events happen at specific times (once or repeatedly) - they go in the calendar.

Tip

Just by recognizing them and putting them on the proper list made my life easier. I stopped forgetting promises and I was on time where I should be.

### Plan daily

I start each morning by checking what was left from yesterday and whether it still matters. I add things that matter to my “today” TODO list. Then I check my calendar to know the plan for the day and how much time I have for focus work. I can decide how many actions I can try to achieve and which ones have to happen today.

Initially, it was taking me at least 15 minutes, because I was reviewing too much. Today, I need a maximum of 5 minutes. This allows me to go through the day without pressure or stress. I see instantly if I have collisions in my obligations and need to reschedule some meetings or re-evaluate deadlines for some tasks.

### Keeping deadlines

There are tasks that you must deliver before a specific date. It might be a request from your boss for some kind of report. Or a questionnaire to fill out, or you need to buy flowers for your wife’s birthday.

Info

Most task managers support setting a “Due date” or some kind of reminders. I use them to remind me about actions that should be performed at a specific time. Let’s take the example of buying flowers for my wife. Planning it during my morning routine makes no sense, as I’ll be looking at it for the whole day and it will keep me distracted (of course it’s important, but not at this specific time). If I set a reminder (due date) for the task to pop up at 5PM - when I finish my work - I’ll remember it as the “next thing to do”.

### Cyclic cleanup

In Agile (and Scrum) teams review and groom work regularly. In GTD you do a weekly (or biweekly) review: clean up your lists, close what is irrelevant, and prioritize for the next days. Monday morning is often a good weekly-review slot.

Warning

**Don’t skip these - do it!** After collecting tasks for a few months, you’ll start noticing that not everything is important. Some actions you won’t recognize anymore. Just like your house needs a bigger cleaning a few times a year, your commitments need it too.

It’s surprisingly satisfying to mark irrelevant things as done (or delete them) - whatever works. They’re gone and out of your head now. It’s amazing how many you can clear and how fast 😄

### Trust the system

You will sometimes forget to do a daily review. You will have lazy days. Trust the system: tasks will be there when you come back. Keep collecting. Remember to review at least from time to time. If you’re under stress, overloaded or feeling like you’re sinking - get back to the process!

## Similarities between GTD and Scrum

Many ceremonies in Agile (especially Scrum) resemble GTD:

- Actions collection == Stories collection.
- Daily planning of tasks == Daily standups / dailies.
- Weekly cleanup and clarification of stories == Grooming / Refinement.

GTD adds routines that span different time horizons and individual work habits, but you can feel they’re not that far apart. I wouldn’t use Kanban for my personal life, but GTD fits well. Similar idea, but adapted for individual productivity rather than team workflows.

## Tips and tricks

### Teach others about your system

This is especially useful in relationships. Share calendars with your partner so expectations are clear. If your partner schedules something, treat it as “sacred” time.

Tip

I use three calendars:

- mine,
- my wife’s (read-only),
- kids’ (shared).

This keeps doctor visits, activities and family events visible. My wife doesn’t use GTD, but she understands the basic interface and trusts my system - that made me much more reliable. She knows that to rely on me, it has to be part of the system.

### Task collection

Many apps exist: Google Tasks, Microsoft To Do, Apple Reminders, Trello, Todoist, etc. I settled on:

- Google Tasks for private stuff,
- Microsoft To Do for work.

Both integrate with email and let you convert messages into tasks, which is very handy. I usually capture everything via my Phone. Paper lists never stuck.

If I can’t collect a task, I ask someone around to send me a message as a reminder. No one ever rejected 😁

### Corporate vs Private lists

GTD suggests one list. Practically, corporate policies, GDPR, and security concerns often force separation. I don’t store private personal data on corporate systems, but it happen to me to email myself from my private account to my corporate inbox, with a few trigger words so the item becomes a task in Microsoft To Do next day.

---

## From Personal Practice to Team Facilitation

After 20 years of practicing GTD, I’ve been asked to introduce this methodology to others - particularly leaders who already manage complex schedules but want more control and clarity. The challenge? Most time-management workshops are too theoretical. Leaders need **actionable routines they can apply immediately**.

Info

The workshop below evolved from my own experience adapting GTD to real-world constraints. It’s designed for a 1 hour session with hands-on exercises, not just slides. If you’re considering running something similar in your organization, feel free to adapt this structure.

## GTD for Leaders: No-Bullshit Workshop Plan

Why “No-Bullshit”? Most leaders already know something about task management. They might use some good practices, so I don’t want to bore them with obvious stuff. I want to perform a quick, deep dive that will leave them with actionable takeaways.

### Workshop Flow

1. **Introduction & Icebreaker (3 min)**
    
    - Brief intro, set expectations: “No theory, just actionable GTD.”
    - Icebreaker: Hand out blank cards/paper.
2. **Memory Dump Exercise (20 min)**
    
    - **Step 1**: “Write down every task currently on your mind (personal, work, etc).”
    - **Step 2**: “Add long-term tasks (few months to 1–3 years: buy a house, study, career goals).”
    - **Step 3**: “Add lifetime goals (travel, family, legacy).”
    - **Group reflection**: What do you focus on daily vs. what really matters?

Tip

This exercise reveals the gap between what people think about daily vs. what truly matters to them long-term.

3. **GTD Core Concepts (30 min)**
    
    - Collect, Clarify, Organize, Reflect, Engage.
    - Focus on practical tools and routines.
    - Use the flowchart above to visualize the process.
4. **System Setup (10 min)**
    
    - Choosing a tool (app, paper, hybrid).
    - Quick demo: set up a basic GTD system.
5. **Daily & Weekly Routines (25 min)**
    
    - How to plan daily and run weekly reviews.
    - Leadership tips: delegation, team GTD, shared calendars.
6. **Pitfalls & Real-World Tips (15 min)**
    
    - Common mistakes and fixes.
    - Q&A and experience sharing.
7. **Wrap-up (5 min)**
    
    - Action plan: what will you change tomorrow?
    - Further resources.

### Workshop Materials

And a [PDF version here](app://obsidian.md/slides/slides.pdf) .

Tip

**For facilitators**: The memory dump exercise is the most powerful part. Some participants realize they’ve been drowning in urgent tasks while ignoring important long-term goals. That moment of clarity is worth the entire session.

---

Enjoyed this post? [![Buy Me a Coffee at ko-fi.com](https://storage.ko-fi.com/cdn/kofi5.png?v=3)](https://ko-fi.com/tgagor)