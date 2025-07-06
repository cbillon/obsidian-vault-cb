---
link: https://cnr.sh/posts/2018-11-12-dont-tell-people-what-to-do-ask-them-how-to-do-it/
excerpt: When I first joined LinkedIn, there was very little automation in production deployments. Believe it or not, each deployment day had a wiki page with a run book of all the services that needed to ...
slurped: 2025-06-28T09:09
title: Don't tell people what to do, ask them how to do it | cnr.sh
tags:
  - method
  - why
  - what
  - how
---

## Don't tell people what to do, ask them how to do it

When I first joined LinkedIn, there was very little automation in production deployments. Believe it or not, each deployment day had a wiki page with a run book of all the services that needed to be deployed, complete with configuration changes that needed to be executed. Roll outs happened on Thursday evenings, and the entire site was brought offline during the deployments. Metrics and monitoring weren't much better. We had very limited visibility into metrics and logs. Obviously, this caused a bunch of problems for the engineers. For one thing, it meant that we had to rely heavily on the operations staff anytime we needed to do just about anything.

Early on, and young in my career, I was tasked with shipping a new web service. This was my first real foray into production software development, and I was pretty clueless as to how to get my bits of code into production. Over the course of the project, I had to go frequently to the operations team (what we'd now call the SRE team) to ask them to do something for me. Every time I'd ask them to do something, they'd respond with a thinly veiled snipe that amounted to, "No, go away." I'd walk away thinking, "OK, now what? I need them to ship my software." Usually, it went something like this.

> **Me**: Hey, can you please install nltk on this machine for me?  
> **SRE**: Nope. We only run standard machine images. I'm not making a beautiful snowflake for us to maintain for all eternity.

I was pretty sympathetic with their plight. The way we were shipping software was a mess, and they were bearing the brunt of the pain. I decided to take a different strategy. Rather than telling them what they needed to do, I'd explain the problem I had and ask for their take on how to best solve it.

> **Me**: Hey, I am trying to train this NLP model. I've found this really killer way to do it, but it's using this stemming library that's only available in nltk. I'm not sure how to productionalize this.  
> **SRE**: Yea, we're not installing NLTK on our production machines. Is this something you can run on the Hadoop cluster instead?

This turned out pretty effective. Unsurprisingly, treating people like they have valuable thoughts is an effective way to get things done (rather than as mindless automatons to execute commands on your behalf).

As time passed, I started noticing another pattern with this strategy. If I spent some time beforehand thinking through what the operations team might say to the problem I was going to present, I discovered I could actually steer the conversation. When they responded to my questions in a predictable way, I had my response to their response ready. I tried it out a few times, but soon realized that I was responding in much the same way that I'd initially approached them - something along the lines of, "That won't work because..."

> **Me**: Hey, I am trying to train this NLP model. I've found this really killer way to do it, but it's using this stemming library that's only available in nltk. I'm not sure how to productionalize this.  
> **SRE**: Yea, we're not installing NLTK on our production machines. Is this something you can run on the Hadoop cluster instead?  
> **Me**: That's not going to work. The data we need isn't in Hadoop, and security is saying they won't let us ETL because it's got sensitive content.  
> **SRE**: Good luck to you, sir.

Back to the drawing board. After some more attempts, the socratic method proved be surprisingly effective. Instead of telling them why their idea wasn't going to work, asking them how they'd handle the problems that I saw with their approach led to really great conversations.

> **Me**: Hey, I am trying to train this NLP model. I've found this really killer way to do it, but it's using this stemming library that's only available in nltk. I'm not sure how to productionalize this.  
> **SRE**: Yea, we're not installing NLTK on our production machines. Is this something you can run on the Hadoop cluster instead?  
> **Me**: Yea, that does seem like a better fit. We'd have to get the training data in there, though. I think it might have sensitive data. Any idea how to handle that?  
> **SRE**: You know, I think Sally had to solve this problem six months ago. I think she has a data masking script that we might be able to use to get the data scrubbed before it leaves production.  
> **Me**: Woah, that sounds exactly what I need. Way better than having to pip install this stuff in some random production machines, too.

This is an effective way to lead a discussion. Even better, occasionally, they'd respond in a way I hadn't predicted, often with a much better idea than anything I'd thought of. The flow goes something like:

1. What problem are you trying to solve?
2. What are the potential solutions?
3. Which solution do you like best? Why?
4. Ask others for their thoughts.
5. If they have a different solution than yours, ask why they prefer it to yours.

The next time you find yourself needing something from somebody, don't just go to them and tell them what you need them to do. Instead, take a step back and think about the problem you're trying to solve. Spend a bit of time brainstorming possible solutions and the tradeoffs between them. Then, when you do go ask for help, don't just tell the other party what you want them to do. Present them with the problem you're trying to solve, and see what they say. I promise you it'll lead to a much better outcome.

---

#### Changelog

- **Fix redirects** on February 11, 2025
- **Migrate to markupdown (#1)** on February 11, 2025

---