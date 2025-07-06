---
link: https://cnr.sh/posts/2018-10-23-your-first-technical-presentation/
excerpt: Congratulations! You've been picked you to give a technical presentation. People are interested in what you have to say, and you're excited and nervous. What now?
slurped: 2025-06-28T09:17
title: Your First Technical Presentation | cnr.sh
tags:
  - presentation
---

## Your First Technical Presentation

Congratulations! You've been picked you to give a technical presentation. People are interested in what you have to say, and you're excited and nervous. What now?

When you're first starting out as an engineer, giving a presentation can be a bit daunting. Even for experienced developers (or anyone, for that matter), the idea of getting up in front of a room full of people can be pretty scary. In fact, [public speaking is the most common phobia in America](https://www.doitbest.com/pages/americas-most-common-fears). So, what can you do to make sure things go smoothly? Here are a few tips that I've picked up over the years.

## The story

You're obviously going to need to figure out what you want to say, and how you want to say it. All presentations involve telling a story. A good starting point to drive the story is to begin with the audience.

Pretend you're talking to an audience member one-on-one, casually. What might you say to them? You probably need to give them some back story and context around the area you're discussing. Then you get to the meat of it. You tell them the problem you're trying to solve. You tell them about the actual technical changes that occurred, how they work, why you made them, and so on. Finally, they're going to want to know whether it worked, and you're going to want to share with them what you learned along the way, what you'd have done differently, and what your plans are for the future.

The structure need not conform to the example above, but starting by thinking through a casual one-on-one conversation with an audience member is a really great way to tease out the basic structure of your presentation.

### A play in three acts

You might be familiar with a story structure called the [three-act structure](https://en.wikipedia.org/wiki/Three-act_structure).

> The three-act structure is a model used in narrative fiction that divides a story into three parts (acts), often called the Setup, the Confrontation and the Resolution.


![[Pasted image 20250628091903.png]]
The example I provided above loosely follows this structure. Start with the setup. Describe the surrounding ecosystem. Whatever it is that the audience needs to know. Then, hit them with the problem. Now, confront the problem. Describe your attempt(s) to fix it. Lastly, you give them the resolution. The outcome, learnings, future work, etc.

This style works for a lot more than just "I made this change" kind of presentations. It can be adapted for technical proposals that have yet to be implemented, as well. It can even be used for informational presentations about how a technology works. You'll find that most presentations can fit into this narrative form.

### Structure

The following is an example structure that I've used when giving a presentation about some technical improvement that's been made.

- Title slide
- Who am I?
- Table of contents
- Thing we wanted to do
- How we did it
- Problem with the way we did it
- How we do it now
- Results of change (improvements, performance, learnings, etc)
- Future work
- Questions

## The deck

Now that you have your story, I'm guessing you're going to want a slide deck to go with it. I'm going to skip the discussion about whether a deck is required or not. If you don't need or want one, move on to the next section. Here's some inspiration.

### Format

Decide on the format of the slide deck. Four common formats are:

- Bullet point
- Sentence slides ([example](https://www.slideshare.net/ZazzaNM/sxsw-2018-top-trends))
- Diagrams ([example](https://www.slideshare.net/dave.m/thingmonk-data-gravity-iot-and-time-series))
- Charts ([example](https://www.kleinerperkins.com/perspectives/internet-trends-report-2018))

All can be fine. Pick one based on content, taste, and comfort. They're not mutually exclusive, either. You can intermingle different formats in a single deck.

### Look & feel

You'll also need to decide on a template. Aside from your standard company template, these are some common styles:

- Photos ([example](https://www.slideshare.net/ZazzaNM/sxsw-2018-top-trends))
- Paper ([example](https://speakerdeck.com/ept/is-kafka-a-database))
- Minimalist ([example](https://www.slideshare.net/criccomini/samza))

### Other rules

- Don't put too much on one slide
- Don't read off your slides
- If you put code on a slide, make it big and bold, and keep it short
- Avoid live demos
- If a question is taking too long to answer, say you want to take it offline

## Tips and tricks

### Modularity

If you're going to be re-using your slides, consider using modular sections. With this approach, you break your presentation down into more distinct sub-sections. You might have 3–6 of these subsections, which you can then mix, match, and tweak based on the audience. Here's a [QCon talk I gave](https://www.slideshare.net/criccomini/samza-qcon) in 2014 that has multiple modular sub-sections (stream processing, motivation, Samza architecture, YARN, stateful processing). I re-used this talk a number of times, but trimmed or completely dropped some sections depending on the audience.

### Additive slides

Another trick you can use if you're trying to explain something complex is to take an additive approach to the slides. You start with a blank or nearly empty slide, then as you narrate, you add more and more to the slide. You can see an example of this at the 25 minute mark, [here](https://www.infoq.com/presentations/samza-linkedin).

### Buddy up

If you're feeling particularly nervous, sometimes it's nice to have someone to share the stage with. Find a friend or co-worker that's willing to help, and enlist them to split the presentation with you.

## Preparing

Now that you've got your story and your deck, you need to practice. The best thing you can do is set up a dry run with your team.

### Dry runs

Do at least one dry run before your presentation. Treat it like a real presentation. Do the full presentation as closely as possible to how you plan to deliver it when you are presenting. Follow these steps:

- Get a room
- Invite your team
- Make your team take notes for feedback (ideally, with slide numbers)
- Time the presentation
- After the presentation, the team provides feedback

This will feel awkward at first. Power through the first few minutes, and trust me, it'll be worth it. After the dry run, you get feedback, and update the presentation accordingly. If there were large changes, you'll probably want to do a second dry run. Keep iterating until you're happy with your presentation.

## That's all folks

Now you've figured out the story you want to tell, built a slide deck to tell it, and done a few dry runs. The next step is do give the actual presentation! I'm not going to go in-depth on that. There are a lot of resources online, and if you want to brush up on your presenting skills, a popular resource is [Toastmasters](https://www.toastmasters.org/).

---

#### Changelog

- **Fix redirects** on February 11, 2025
- **Migrate to markupdown (#1)** on February 11, 2025

---