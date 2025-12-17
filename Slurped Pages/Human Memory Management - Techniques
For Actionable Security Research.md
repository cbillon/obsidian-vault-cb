---
link: https://grahamhelton.com/blog/human-memory-management
site: Graham Helton
excerpt: |-
  How to utilize an atomic note
  taking system to effectively research new topics and advance your
  career.
twitter: https://twitter.com/@grahamhelton3
slurped: 2025-12-17T09:11
title: |-
  Human Memory Management: Techniques
  For Actionable Security Research
tags:
  - obsidian
  - notes
---

This post originally started as my outline for my Bsides ATL talk by the same title. While it started out as an outline for my talk, I ended up expanding it out into full post because there were many little nuances that were easier to explain in a text format. With that being said, I spent far longer formalizing my thoughts on this topic than I thought I would. I hope you can glean some useful tips from it.

I should point out that this is what works for me, while this method has worked wonders for me, it will not work for you if you simply copy/paste what I do. Like any information, make sure you adapt it to what works best for you.

If you don’t care about the backstory for why I needed a system like the one I created, skip to the `Hello Markdown` section of this post.

## Background: What _Didn’t_ work for me

When I was in highschool and early college I had little interest in learning. It wasn’t that I wasn’t interested in learning _at all_, it was just that I wasn’t interested in learning what I was being taught.

It wasn’t until I started taking my cybersecurity classes that I was actually interested in learning. I figured if I was going to try to get a job doing something in security, I should probably try to retain some of the information I was being taught.

This was the first time where I was trying to really retain information (not just regurgitate it for a test) but I had never actually done this before and was _terrible_ at it. As it turns out, formal education doesn’t optimize learning for applying the knowledge, it optimizes it for multiple choice exams.

Here is an actual example of my notes I took in a google doc of an intro to unix class on the Unix `awk`:

![](https://grahamhelton.com/assets/images/Pasted-image-20230912213125.png)

This information is virtually useless. In order to somewhat make sense of it, you have to already understand `awk` and even if you do it’s still doesn’t really make sense. I literally wrote “This is probably not enough for the exam!!!!”.

## Handwritten Notes

After trying (and failing) to take notes in word documents, I must have realized that copy/pasting and blindly writing down what professors were saying was probably not the best way to learn things so after watching dozens of youtube videos on from self proclaimed _productivity gurus_, I landed on the idea of taking notes by hand. This way I could actually process what I was taking notes on! Right?

As it turns out, blindly writing technical content such as Cisco switch configurations by hand was ALSO not a great way to learning things… (Even though It’s great for regurgitating information for a multiple choice exam)

![](https://grahamhelton.com/assets/images/Pasted-image-20230912213700.png)

## No Notion?

Next up I moved to the venerable Notion.so. This was the first application where I was able to get a decent “note structure” going. I was quite happy with this system for learning simple things such as terms that appeared in the Linux+ Certification objectives.

![](https://grahamhelton.com/assets/images/Pasted-image-20230912215058.png)

While notion was fine for the time, I ended up looking at note taking as a tedious task to be employed when I needed to regurgitate information for an exam and I spent more time thinking about how to take notes than actually taking notes.

This may have worked in college where I had nothing but time, but getting the information out of my brain and into a notion page was far too time consuming to be useful and finding information I had written down was way too tedious if not impossible.

I still use notion for simple things like planning vacations, but ironically I cannot figure out how to take good, effective notes in it.

## Realizing I need a better system

Upon entering the _real world_, I realized that working in security is _VERY_ different than being a college student. I actually had to research information and apply it in complex ways where success wasn’t measured by a letter grade of a multiple choice exam.

This often required the gathering of broad knowledge of many different complex technologies that no one person could be reasonably expected to fully understand, extracting the useful or relevant information, and applying it to the real world. Typically in a short amount of time. A simple notion page with term/definitions notes was not going to cut it when success is measured in this way.

After many years of constant refinement of my process for collecting information. I’ve finally cobbled together a way of storing information in a way that was _actually_ useful when I needed it and _actually_ made me more productive (and didn’t just make me _feel_ more productive). My process has become streamlined in a way that I don’t think I could function without it. This process I’ve curated over many years is what the rest of this blog is about.

## Hello Markdown

I can’t delve into the the details without mentioning markdown. Markdown is a “lightweight markup language that you can use to add formatting elements to plaintext text documents”. This is the same _language_ that is used to write README.md files on places like github.

Markdown allows for quick formatting of text without having to mess around with menus like you would in other programs. This makes markdown fantastically efficient for getting your thoughts out of your brain and into your note taking system. [This is a great resource on how to write markdown](https://www.markdownguide.org/cheat-sheet/). Markdown is the cornerstone of my note taking system.

With that being said, I’m not going to discuss all the reasons you should learn at least the basics of markdown in this post but if you plan on doing any amount of note taking or plan on writing documentation, you will greatly benefit from learning it (it takes about 5 minutes).

## Hello Obsidian

[Obsidian](https://obsidian.md/) is a fantastic tool because I rarely have to think about using it. Like most great tools, the learning curve is relatively low for basic usage but the opportunities are endless if you invest the time to become proficient with it. I’ve found that this is generally true of many tools that are worth your time to learn. The process of getting thoughts onto paper (er… file?) is friction-less with obsidian once you understand a few core concepts.

Just like markdown, this post is not an in depth guide on how to use obsidian as it’s relatively straightforward to get started using and there are many other guides out there covering how to set up your vault’s appearance, hotkeys, etc. There are however, are a few unique aspects of obsidian that I’d like to touch on to make things a little more clear for the rest of this post.

While simple to get started with, Obsidian is fairly unique among note taking applications for a few reasons. The most important difference to understand about obsidian is that it is a rendering engine for markdown files.

This is a fundamentally different way of thinking about notes if you’re coming from something like onenote or notion. Obsidian is to markdown files as chrome/firefox is to html files. It takes text (markdown) and renders it into something more useful to a human (styling, links, formatting, etc).

When using obsidian, it’s important to remember that new notes you create are simply markdown files on your file system that can be viewed in a file explorer as seen below. File explorer on the left, obsidian on the right.

![](https://grahamhelton.com/assets/images/Pasted-image-20231002180757.png)

This may seem like a simple distinction, but there are large implications for automation and extensibility that we will get to in a later section.

## The Power of Linking

One of the key advantages of obsidian comes from it’s [linking feature](https://help.obsidian.md/Getting+started/Link+notes) that allows you to link notes together by referencing a note using the `[[double bracket]]` notation.

You may have seen pictures of massive “knowledge graphs” within obsidian like this one.

![](https://grahamhelton.com/assets/images/Pasted-image-20231002182138.png)

While this graph is often the most eye catching and fun to explore, in most of my use cases, it’s not very useful outside of looking pretty. The real power of obsidian’s linking feature is more often found in the local graph view. Here is the local graph view for my note on `NTLM`. Which can be accessed by pressing the triple dot icon in the top corner of the note.

![](https://grahamhelton.com/assets/images/Pasted-image-20231008111850.png)

This view shows links to or from the currently open note. This is quite powerful as it allows you to easily identify link information from various notes together.

In the following example you can see that my note on `NTLM` is referenced by many other notes (and it directly links to `hashcat` and `NetNTLMv2`). This means that in the notes for `mimikatz`, the `NTLM` note is linked either by obsidian automatically making it (via adding it by looking at the “backlinks” tab) or by a manual entry which is done by adding two square brackets around the note like so: `[[NTLM]]`.

![](https://grahamhelton.com/assets/images/Pasted-image-20231002230215.png)

Sure enough, if we take a look at the `mimikatz` note, you can see that `NTLM` is linked in two different places as well as some other notes. Once again, this is done in obsidian by surrounding the note you wish to link to in two square brackets. IE: `[[NTLM]]`

![](https://grahamhelton.com/assets/images/Pasted-image-20231005191853.png)

Why is this powerful? Imagine you’re working on a penetration test and you discover an NTLM hash. It’s been a few months since you’ve done anything with NTLM so you don’t remember exactly what this means.

Simply open up your obsidian vault to the `NTLM` note and you can very quickly identify that many attack tools are used to obtain NTLM hashes and there is a “new” version called `NetNTLMv2`. This gives you attacks you can execute to obtain NTLM hashes, how to detect those attacks, tools you can uses to execute the attacks, information in the NetNTLMv2, and more. Linking is all about providing you with context surrounding the current note.

Not convinced of how cool this is? Fair enough. Imagine you’re asked “Hey, I noticed that we have a bunch of scheduled tasks running in our environment that no one in our team remembers creating, is that bad?”

You don’t quite remember what scheduled tasks are so you pull up your notes and can immediately identify what they are and that they’re a common windows persistence technique used by attackers.

![](https://grahamhelton.com/assets/images/Pasted-image-20231011205637.png)

We can take this one step further by noticing that one of the nodes linked to in the local graph for `scheduled tasks` is a note for a tool called `sharpersist`. Upon investigating that link we can see exactly how an attacker could create a scheduled task using this tool and what it looks like.

![](https://grahamhelton.com/assets/images/Pasted-image-20231011205616.png)

All of this contextual information can be quickly gathered by simply following this note taking system that is described below. The more information you collect the more context and connections you’ll be able to make.

All of this information already exists within your notes but utilizing a powerful note taking system allows you to enrich the information in a different way that presents a new perspective. So how do we build a note taking system that allows for this automatic enrichment of information? First we have to discuss the two different types of notes.

## Types of notes

I typically use two different _types_ of notes that serve slightly different purposes. The two types of notes I create are:

- **Content notes**: Notes that store information on a single item.
- **Concept notes**: Notes primarily store links to content notes pertaining to a single concept.

## Content Notes

These notes are exactly what you would expect, they’re notes on whatever the title of the note is. For example, a note titled `Inveigh` will only contain notes on that single tool. Emphasis on 1 concept/tool/technology = 1 note.

A great content note for me should be the following:

- **Atomic**: Covering a single topic or concept. Any ancillary topics/information should be made into it’s own note and linked to using the `[[double bracket]]` notation.
- **Modular**: Notes should always be easy to update and curated upon new or updated information being available. Utilize markdown formatting for efficiency and speed.
- **Guiding**: Notes should guide you to related notes, articles, videos, or sources.

### Atomic Notes Explained

Taking atomic notes is one of the most important (and misunderstood) concepts I utilize in my note taking system. Taking _atomic_ notes means creating one note file per concept/tool/idea/technology. This means I have a note called `nmap`,`TLS`,`interviewing`,`atomic red team`, `Active Directory`, etc. I’m currently sitting at a little over 1.1k files in my vault.

Creating atomic notes has many advantages over how we generally think of taking notes. Atomic notes allow for:

1. **Centralized information**: Information on a topic scattered between 15 notes briefly touched on in 15 different trainings all containing different information about how to use a single tool is too cognitively expensive to manually hunt for all the relevant information for every time you need to retrieve information on that tool.
2. **Understanding context surrounding your notes**: This is where obsidian local graph comes into play. You can gather context surrounding your notes very quickly.
3. **Identifying related concepts**: Somewhat related to the last point, you can quickly identify relationships between notes and have that relation persist in your notes instead of just your brain.

When taking a training class that is split into 5 days, taking 1 monolithic note per day called something like `xyz_training_day_1` will make it harder to find the information it contains in the future. Instead, create a new _atomic_ note for each topic/concept/tool the course is teaching you.

For example, if your instructor starts discussing the tool `bloodhound`, create a new note simply called `bloodhound`. If you’re discussing `ECB` encryption, make a note called `ECB` or `Electronic Code Book`. This has the benefit of allowing you to link notes together in the future as well as being able to easily find your notes on something that is likely to be mentioned many times.

An issue I ran into is “What do I name my files?” While it may be simple to create notes for tools with unique names such as `bloodhound`, it may be less straightforward to create notes on items that are either acronyms (such as the aforementioned `Electronic Code Book` vs `ECB`) or concepts that can have many similar names such as , `EDR evasion`, `EDR bypass`, etc.

I’ve found that just using the most common phrasing and then linking to the less common phrasing works well. This way if I am looking for information on `EDR evasion`, and I land on the `EDR bypass` note, I can quickly identify “Oh, there is another note on the same topic with a different name” by looking at the local graph.

![](https://grahamhelton.com/assets/images/Pasted-image-20230915233555.png)

Lets see another example. Let’s say that we are working a pentest and we have utilized our `bloodhound` notes to get bloodhound up and running. Great, but that’s not all we can do. As you can see by looking at the local graph option in obsidian, bloodhound has also been mentioned by a note called `Detecting Recon`.

If we take a look at this note, we can see that at some other time (potentially years ago), I wrote down how to detect bloodhound using splunk queries. Because of our atomic note format, we are able to identify a link between the note `bloodhound` and a mention of `bloodhound` in the note `detecting recon`. ![](https://grahamhelton.com/assets/images/Pasted-image-20231002232237.png)

It cannot be overstated how immensely powerful this is, but there is one caveat. While obsidian does have some functionality that’ll let you link notes automatically using the “backlinks” tab, I find that this system works best if you make the connection while you’re taking the notes.

When creating the `detecting recon` note, I saw the section on detecting bloodhound and manually typed `Detecting [[bloodhound]]`. This double bracket notation around the word is what obsidian uses to generate the “links”. It’s one of the major mindset shifts I’ve had to develop but once you learn to always be thinking “What relationship does this concept I’m currently learning about have in common with the notes already in my vault”. When I’m taking notes, that is constantly in the back of my mind. Not only does it help you link your notes, but it also helps you retain the information better.

### Modular Notes Explained

The implementation details of how you choose to structure your notes is largely up to you, however, if you plan on being able to quickly skim through your notes to get up to speed on something you’ve forgotten since taking the note, I would encourage you to make your notes easy to skim by using markdown to create a modular structure to your notes.

For example, this note on the tool `Rubeus` is nice and easy to skim for relevant information on the attack I want to execute and adding new information is as simple as adding a new heading then filling in the information.

![](https://grahamhelton.com/assets/images/Pasted-image-20231011205814.png)

Contrasting the previous note on `Rubeus` with this poorly written note on `attacking kubernetes`, we can see that this note is much less modular as it does not allow for easy skimming of information and there is no easy way to quickly add or remove information. It’s a total mess.

![](https://grahamhelton.com/assets/images/Pasted-image-20230916002600.png)

If I wanted to add or remove information to this note I wouldn’t be able to just create new section and add information or select a whole section and remove it. It’s possible but it makes it more of a chore than it needs to be. Architect your system to allow you to be lazy and you will thank yourself every time.

### Guiding Notes Explained

The purpose of notes being _guiding_ is to provide connection & context to your notes as well as funnel your knowledge into a single spot for future reference.

#### Guiding Notes: Connections & Context

Guiding notes are notes that provide connection & context to other sources. They do so by linking to both external sources where information came from such as a blogpost and linking to internal notes on related topics using the `[[double bracket]]` notation. This is best explained using an example.

In the follow example, I was watching a [webcast discussing atomic red team](https://www.youtube.com/watch?v=VTkRkgBjAEA). While watching the webcast, I was briefly reminded of a topic in a book I had recently read about communicating at the executive level. While taking notes on the webcast, I was able to quickly jot down the note “This is amazing for communicating with executives”.

That simple note was enough to make a connection that “atomic red team” has some relationship to a note I already had titled “communicating with executives” which in turn has some relationship to [operating at staff](https://staffeng.com/guides/operating-at-staff/).

![](https://grahamhelton.com/assets/images/Pasted-image-20230916010023.png)

Now that this context has been provided, next time I’m running atomic red team (or communicating with executives) I can see that there is some other value in it other than purely technical security related value.

This linking feature can also be used to create new notes for later exploration. For example, in the following note I have links to different C2 frameworks such as `cobalt strike` and `mythic`. By simply creating a link to that note it appears in the graph view, I can both identify that `mythic` is a type of C2, and I can identify if any other notes are linking to `mythic`.

In this case there is a note called `APT36` that also mentions `mythic`. Because of these atomic notes, the `mythic` note now has a link from `C2` to `APT36`. Once again, providing new an interesting context by simply enriching the preexisting information.

How is this useful? Imagine I (as an internal red team specialist) am asked by the powers at be to “Emulate APT36”. Now all of a sudden I have instant context as to what that means without having to go research it. I simply pull up my notes and can identify the connection.

![](https://grahamhelton.com/assets/images/Pasted-image-20230928202351.png)

Still not convinced? Imagine you’re executing a pentest and for some reason or another you’re explictly told “YOU ARE NOT ALLOWED TO RUN RESPONDER”. Maybe it’s because last time the client had a pentest, the tester took down the network using responder, maybe you’re having python dependency errors (it’s always ~~DNS~~ python dependency errors), or you know that they’re running detections specific to responder itself.

Whatever the case may be, you still need to execute this attack. Using obsidian and taking a look at your `LLMNR Poisoning` note, you can easily gain more context via the local graph view to see what other options you have. Sure enough, we have a link to a tool called `inveigh`.

![](https://grahamhelton.com/assets/images/Pasted-image-20231005201410.png)

Upon opening our note for `inveigh`, we can see that this is a tool simliar to responder that should allow us to execute the same attack. Note that `Inveigh` is linked to `LLMNR Poisoning` because inside of the `Inveigh` note, there are double brackets around`[[LLMNR Poisoning]]`. This is what creates the link between the two.

![](https://grahamhelton.com/assets/images/Pasted-image-20231005201520.png)

Another benefit of notes being _guiding_ is it allows you to quickly visualize new concepts that you have yet to explore. This happens often when researching new areas of interest. I like to call this taking inventory of the known unknowns.

You may read a blog post on some new technology that you need to learn. Upon first researching a new topic, you may create a note that links out to 10 different terms commonly used with that technology. The problem is that you don’t yet understand what any of them mean but inside of your notes, you create links to them using double brackets such as `[[GKE]]`. As you learn more about the topic, you can identify gaps in your knowledge by looking at notes that are created but not filled with content.

This is fantastic for ensuring you don’t miss an important topic when researching a new concept. Below, you can see that when I was researching kubernetes, the concept of managed kubernetes services kept coming up via mentions of `GKE`,`OpenShift`, `EKS`, and `AKS`. Creating links to these allowed me to keep track of these technologies as something I don’t yet understand because the notes on each of those didn’t have any information stored in them. Note that the greyed out _nodes_ for each of these technologies means that there is a link to that note, but the actual file itself does not exist yet.

![](https://grahamhelton.com/assets/images/Pasted-image-20230928222128.png)

Additionally, this allows for the identification of concepts that are repeatably mentioned notes. In this example, TLS is mentioned by many different notes. This probably means that TLS plays a central role in many different areas of security and I should work to have a very solid understanding of how TLS works. This is a great way to identify what you should learn more about. If something has a lot of connections in the graph view, but you don’t understand it, it’s probably a gap in your knowledge that would help in many areas if filled.

![](https://grahamhelton.com/assets/images/Pasted-image-20230928222513.png)

#### Guiding notes: Funneling knowledge

Finally, taking guiding notes allows you to consolidate information. It is a waste of time and effort to retake notes on subjects you’ve already taken notes on. Not only does having a single note for each information point save you time, but it also has the added benefit of funneling all of your knowledge on a topic into a single location that you can navigate to when reviewing a topic.

Take this example of `Pivoting` in my `Cobalt Strike` note: One of the methods you can use is a SOCKS proxy. Since a SOCKS proxy is a common technology, I knew I already had a note on it so instead of describing everything over again in my cobalt strike document, I simply linked to the `SOCKS proxy` note.

![](https://grahamhelton.com/assets/images/Pasted-image-20230928204138.png)

Now, if I am ever reading about `Pivoting` in my `Cobalt Strike` notes and I forget what a SOCKS proxy is, I can simply open the note on the topic and familiarize myself with it. Additionally, if I’m doing something with SOCKS proxies that does not have anything to do with cobalt strike, I don’t have to remember that my notes on SOCKS proxy are actually stored in my `Cobalt Strike` notes.

The more notes you have the more useful this system becomes and you will quickly realize how tangentially related many topics are. In fact, many topics in security are just many foundational technologies utilized in a new way. (Looking at you Zero Trust…)

### Content Note Example

Now that we’ve discussed the three principals that make up a good content note, lets take a look at a final example of what I would consider a good content note. Remember that this is done by creating a single note per technology/tool/concept/subject.

![](https://grahamhelton.com/assets/images/Pasted-image-20230912225618.png)

While this note may look busy, it’s jam packed with information. Lets go through each criteria for a good concept note and see how it holds up.

- **Atomic**: This note is about a single tool, there are no random topics or information covered that would be better placed in a note about that topic. For example, you can see that I have links to the mitre attack framework, tools such as elastic, splunk, and sysmon, and the concept of adversary emulation. I even have a link to the concept of “communicating with executives”. All of these links have their own notes because further information on them would not fit under the `atomic redteam` note but they may be interesting to explore further.
- **Modular**: This note is easy to add and subtract information from if needed. There is no overly complicated reformatting that would need to be done and everything is easily organized under headings. Just inset a new heading or add to an existing one.
- **Guiding**: This note guides you to a both external and internal information by linking to other notes and websites/resources to get more information.

## Concept Note

Concept notes are different from _content_ notes in that their general purpose is to direct you to content notes and provide organizational structure. You can almost think of concept notes as a folder containing links to information and context for the more broad concept. (The obsidian community sometimes calls these a “map of contents” or an _MOC_, although that definition seems to be a bit overloaded). An example of a concept note would be my windows privilege escalation note pictured below.

![](https://grahamhelton.com/assets/images/Pasted-image-20230913210941.png)

There is very little “content” in this note because as my notes grew on the topic, it became too much for a single note. An important thing to understand is that most _concept_ notes start out as a _content_ note. The more information you gather on a topic, the faster notes will go from _content_ to _concept_.

### Why are concept notes useful?

When you first start learning about a topic you typically start with a very broad concept such as `wtf is a kubernete`. As you cultivate more information on the topic, you begin to gather information on different subtopics. As you identify new sub topics, you should consider making each sub topic it’s own atomic note instead of combining them all into one unwieldy monolithic note.

Your top level note `kubernetes` will eventually turn into a concept note that has _some_ information on kubenetes as a whole, but for more specific information on a topic such as `Helm Charts`, a separate note can be linked to from your top level concept note (`Kubernetes`) which essentially takes on the role of guiding you to sub topics of the top level concept.

This helps you avoid the [“draw the owl” problem](https://www.danielzarick.com/blog/draw-the-owl/). It allows you to take a complex topic such as `Kubernetes` or `Active Directory Post Compromise Attacks` and break it down into manageable notes without having to spend your entire day organizing your notes into folders.

The following is my Content note for `Active Directory Post Compromise Attacks`. Should I need to perform any of these, I can easily identify which attacks I can do and how to do them by navigating to that note.

![](https://grahamhelton.com/assets/images/Pasted-image-20231005203214.png)

For example, clicking on the `Kerberoasting` note I can easily see what kerberoasting is, how to execute the attack, which tools can be used, and how to defend against it. The big concept to point out here is that part of the kerberoasting attack the tool Rubeus can be used. However, Rubeus is it’s own tool that isn’t unique to _only_ kerberoasting. This means that a new note simply called `Rubeus` is created. Any information about conducting kerberoasting with Rubeus is stored under the `Rubeus` note.

![](https://grahamhelton.com/assets/images/Pasted-image-20231005203911.png)

## What makes a great system?

Just as important as taking good notes (whether concept or content notes) is a system to orchestrate all of it (I’ve been reading too much about kubernetes haven’t I…). I’ve identified a few elements that have become hard requirements for my note taking system:

- **Accessible**: Identifying if you have a note on a topic and accessing it must be effortless. This is key as writing atomic notes requires you to quickly create notes for new concepts as you come across them (see the section on taking inventory of known unknowns above). It is not uncommon to create 20+ new notes when down a deep research rabbithole.
- **Automatable**: On top of being accessible, notes need to allow for some level of automation. This typically means setting up templates to automatically set up the rough structure of your note.
- **Extensible**: No note taking application will have every feature you need. Being able to automate things (such as mass re-naming) with a bash script is essential for keeping things maintainable. See: [YOINK!](https://github.com/grahamhelton/yoink)
- **Local**: It should go without saying that having files stored locally is a massive relief. Not only does this protect your notes from all sorts of corporate calamities, but it also might be a requirement if you’re taking notes for your day job as storing your _how to hack `$client`_ on servers you don’t own might be irresponsible. (Note that obsidian sync uses AWS if you opt to set that up).

How you utilize your system is up to you as there is no _one size fits all_ solution, however, here are a few pointers to get you going in the right direction.

## Accessible notes

Fortunately, obsidian makes accessing notes very easy as long as your are taking atomic notes. By simply pressing `ctrl+o` you can search for (and open) any notes you have. This makes it very easy to identify if you already have notes on a topic while down a research rabbithole.

![](https://grahamhelton.com/assets/images/Pasted-image-20230915225959.png)

I also don’t use the “file browser” tab on the left side of obsidian anymore because all my my notes are atomic and linked I don’t need to spend time figuring out “Does Cobalt Strike need to go under the `redteam` folder or the `software` folder?”. Notes are more accessible by grouping them into content notes without needing to organize them into folders.

A quick note on attachments: I use a lot of screenshots in my notes because they help add context. Because obsidian uses markdown, the screenshot in your notes is really just a link to the file on your file system. You can set up rules for where screenshot files are put in your vault but I just configured my vault to just put every single screenshot in the same folder and use custom tooling to grab them if I need them.

Note that if you’re using obsidian for pentest report writing for multiple clients, I would recommend being VERY knowledgeable of where your files are getting stored to avoid spillage. Consider using totally different vaults until you’re totally sure you know how obsidian is handling your files. For personal notes, this isn’t much of an issue.

## Automatable notes

Automating your note taking process to make it as friction-less as possible is crucial to efficiently translating your thoughts into useful notes. The more time you spend fighting with your notes, the less time you have for learning and the harder it will be to actually get started on something.

There are many ways to facilitate this using hotkeys, plugins, and workflows but I find that every time my automation system gets too complicated I spend more time finagling with it than I do taking notes. Here are some of the automations I take advantage of.

### Templates

Templates are as simple as they are powerful. Using the built in obsidian template plugin you can define shortcodes in your template file that will be “rendered” upon the application of the template. Here is a generic template I use for my notes. This defines frontmatter (meta data about the document such as the date), creates a heading with the title of the note, and adds a references section. ![](https://grahamhelton.com/assets/images/Pasted-image-20230928213807.png) For example, on creation of the note titled `bsidesatl`, this is created (note that I added the “career” tag manually).

![](https://grahamhelton.com/assets/images/Pasted-image-20231011200559.png) Upon insertion of the template to the file, the shortcodes are rendered. That’s it. Super simple yet super powerful.

### Hotkeys

Hotkeys are a great efficiency boost in any application, but obsidian has a built in feature that is a game changer for me - vim keybinding support. If you don’t know what vim is or how mind mindbogglingly fast it you edit files once you understand it, I highly recommend you look into it. Unfortunately this post would be double the length it already is if I talked about all the vim shortcuts so you’ll have to learn on your own :)

Other (non-vim) hotkeys I use frequently are:

- `ctrl-p`: Opens the command palette, allowing you to use plugins, insert templates, and much more.
- `ctrl-n`: Creates a new blank note.
- `ctrl-o`: Opens recent files
- `ctrl-g`: Opens graph view

## Extensible Notes

This one is pretty simple thanks to obsidian’s markdown-centric principles. Anything you wish to do with a programming or scripting language can be done with your notes. This can mean writing a tool to: mass rename terms or fix formatting issues, reformat notes, etc.

I have written many tools that take advantage of this feature. One example is [YOINK!](https://github.com/grahamhelton/yoink) which is a tool that extracts all images from a note file.

Another is a custom tool I use moving files from my obsidian vault to my website folder and reformats the images to have `-` in the name instead of a space.

![](https://grahamhelton.com/assets/images/Pasted-image-20230928220530.png)

## Local Notes

Another often overlooked element of a good note taking system is where the data is stored. Depending on your use case, you may not be allowed to store the data you’re collecting in your vault on external servers. This is very common in security as we are often given an incredible amount of access to data. Keeping that data locally is a smart move (if not legally required).

This also means that you can manage your own notes with something like git version control. Because you have access to the local data, you can decide what you’d like done with it. It’s not forced upon you and it never goes down. Local files don’t have a [twitter status page](https://twitter.com/NotionStatus).

Obsidian allows you to create tags using the `#tag` syntax. I don’t use tags that often besides in the “frontmatter” of my posts (now called [properties](https://help.obsidian.md/Editing+and+formatting/Properties)). The only reason I tag posts is because the graph view of obsidian allows you to color nodes by tag. This allows me to very quickly see a pretty graph of my knowledge. Literally the only reason I use tags is because you can make pretty colors. I’ve experimented with using them in other ways but I never _needed_ them so I just don’t use them for anything else.

![](https://grahamhelton.com/assets/images/Pasted-image-20231011200829.png)

## Tips for putting it into practice

The first step you should take to start taking great notes is just start somewhere and stay consistent. It’s better to have notes on something stored in a centralized place that you can find with some effort (IE: You at least know what application you took the notes in) than to not have notes on a topic at all and have to spend exponentially longer re-researching the topic again. With that being said, here are some other tips that I wanted to go into more detail on but had to cut from this post since it’s already gotten to be over 7000 words.

- When taking notes on a subject, the amount of detail you should include can typically be answered by asking the following question: If I were to be reading this 1 year from now, what information would I need to capture now to understand this topic 90% as well as I do now?
- Always be thinking “How much does this topic relate to other topics I have notes on?” For example: if you’re learning about `kubernetes` and it reminds you of `go`, you could create a link in your notes between the two.
- Don’t let perfect be the enemy of good: You’ll never have a “perfect” system, but you can always adjust your system as you learn new strategies. The only “perfect” system is one that you use.
- Beware the linkedinfluencer who says you _must_ do XYZ to be a _1337 hax0r_. Be open minded and figure out what system works best for you.
- Personally I don’t care about spelling in my notes. Spellcheck takes too much time for personal notes where I’m the only target audience. I disable the spell checker in obsidian.
- Don’t just copy/paste what I do. Much of it won’t work for how **YOU** learn.
- A good system is like a car. It requires regular maintenance to continue running well. Whenever I stumble upon a note that isn’t clear or is outdated, I’ll update it.
- A good system makes it easy to input new data.
- I presented a very cherry picked view of my notes. MANY of my notes are pretty short. Don’t expect every note to be teeming with information. Here is an example of what probably 70% of my notes look like

![](https://grahamhelton.com/assets/images/Pasted-image-20231005204516.png)

## Downsides?

What is frustrating to me is that people who talk about obsidian never discuss the downsides. This can make it seem like you’re just “not getting it” which isn’t usually the case. So lets discuss some of the downsides of taking notes in this way.

Obsidian isn’t designed to be a tool utilized by many people at once. While you _can_ do this by utilizing a git version control system, this assumes everyone is going to follow the same note taking methodology as you. I personally only use obsidian for _notes_, not reports or anything that is going to be looked at by other people. If you’re interested in how to do this, [TrustedSec’s blog](https://trustedsec.com/blog/obsidian-taming-a-collective-consciousness) on their obsidian vault is interesting.

There is no great way to send your notes to someone. You can export a to a PDF or use something like [pandoc](https://pandoc.org/) to convert your markdown to a word document, but honestly that’s a really hacky way to do things and usually takes a lot tinkering and still doesn’t turn out perfect. Additionally, sending someone a note is not very useful because they can’t follow the links to gather context and they’ll probably ask you “Uh why are there double brackets around half the words in your notes?”. It’s doable in a pinch but not a great way to share research.

My setup intentionally decouples the notes from the source material. I can’t easily answer the question “What information did you learn in GXPN?” by looking at my notes because I don’t simply have a note called `GXPN_Day_1`. Instead, each topic covered is broken down into an atomic note. This could be remedied with obsidian’s tagging system though if this is important to you.

Finally, it takes more upfront time and effort to build a system that will be quick and effortless when you’re actually taking notes. It took me years to develop this system and it’s constantly evolving. Give it a few months.

## Fin?

This post was much longer than I intended it to be. Shockingly, I have a lot more I could talk about when it comes to obsidian, markdown, note taking, and research and I didn’t even get to mention canvas mode. Perhaps next time. I hope you’re able to take portions of this post and implement it into your own note taking to help you in your career. I know it’s greatly helped mine. Fun fact, this post is over 7000 words which makes it longer than any paper I have written for any formal education (bachelors and masters). I guess my system worked. Plus its as free as the content you choose to consume :)

If you have any questions, comments, typos, or anything else, feel free to reach out to me on [any of these platforms](https://grahamhelton.com/pages/links/).

## FAQ

I know I’ll get some questions on the following topics so I’ll get them out of the way.

- Can you share your vault files
    - Unfortunately, I can not. Not only because I have a lot of notes on a lot of paid training, but because my note are _mine_. They contain a ton of notes to myself, personal stuff like how resolved issues in my homelab, etc. Plus my vault won’t be very useful to you since it’s unique to me.
- What theme are you using?
    - I use [Deeper Work](https://github.com/lucas-fern/obsidian-deeper-work-theme) with some slight custom CSS tweaks (underlines under Heading 1)
- What plugins do you use?
    - I only use a few.
        - Vim mode: The default plugin the enables vim keybindings. Lifechanging.
        - Typewriter Scroll: Keeps the text centered on the screen
        - Image toolkit: This one is essential for me. Allows you to click on an image to zoom in or copy it to your clipboard.
        - A few other random ones that I try out every now and then but none that are super useful