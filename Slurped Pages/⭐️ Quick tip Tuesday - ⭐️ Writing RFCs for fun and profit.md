---
link: https://dev.to/wesen/quick-tip-tuesday-writing-rfcs-for-fun-and-profit-3bo
byline: Manuel Odendahl
site: DEV Community
date: 2022-04-26T14:22
excerpt: I suck at writing documentation. I suck at writing emails. I suck at writing project specifications....
twitter: https://twitter.com/@mnmlmnl
tags:
  - architecture
slurped: 2026-10-12T09:40
title: "⭐️ Quick tip Tuesday: ⭐️ Writing RFCs for fun and profit"
---

[![Cover image for ⭐️ Quick tip Tuesday: ⭐️ Writing RFCs for fun and profit](https://media2.dev.to/dynamic/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F04fma54mbf4su79djmff.jpg)](https://media2.dev.to/dynamic/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F04fma54mbf4su79djmff.jpg)

[![Manuel Odendahl](https://media2.dev.to/dynamic/image/width=50,height=50,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.us-east-2.amazonaws.com%2Fuploads%2Fuser%2Fprofile_image%2F852108%2Fc4c6139f-b639-4642-bc3a-0e984d97c5ca.jpeg)](https://dev.to/wesen)

I suck at writing documentation. I suck at writing emails. I suck at writing project specifications. In fact I don’t think writing documentation or project specs in a legacy-ridden, quick-changing, of-dubious-quality codebase is all that useful.

Instead, I prefer to focus on writing RFCs.

RFC stands for “Request for Comment”, and I think of it as a short, formal document that describes a proposed change to the codebase. This can be proposing a new feature, describing a way to refactor a problematic function, a specific new pattern to be implemented, or introducing a new technology.

## [](https://dev.to/wesen/quick-tip-tuesday-writing-rfcs-for-fun-and-profit-3bo#rfcs-are-about-communication)RFCs are about communication


An RFC is not just a document, it is a living organism. It is meant as a **tool of communication**.

- **for the writer themselves**: writing is the best way to clarify your own ideas. If your thinking is not crystal clear, you will have a hard time putting it to paper in an elegant and persuasive way
- **for colleagues**: this allows team members to be involved and feel ownership. They can point out shortcomings, ask clarifying questions, propose alternatives. New features or patterns won’t come out of the blue.
- **for stakeholders**: a well-written RFC let’s stakeholders understand why technical changes are being made.

### [](https://dev.to/wesen/quick-tip-tuesday-writing-rfcs-for-fun-and-profit-3bo#rfcs-as-a-persuasive-tool)RFCs as a persuasive tool

Think of an RFC as a little propaganda machine. They are effective persuasion tools to make the argument for technical groundwork in an otherwise product-driven development cycle.

- writing **it convinces the author** that their thinking is thorough and the solution covers all possible cases
- reading **it convinces the team members** that the solution is sound and that their remarks have been taken into account
- reading **it convinces the stakeholders** that the work being proposed is meaningful, and that the team is taking ownership and accountability
- reading **it helps future readers of the codebase** understand why certain decisions where made

### [](https://dev.to/wesen/quick-tip-tuesday-writing-rfcs-for-fun-and-profit-3bo#rfcs-help-team-cohesion)RFCs help team cohesion

Because they expose the compromises that are inevitably made when doing technical work, they are also an important tool for team cohesion. If a team member disagrees with the solution chosen, they know that they have been heard and that a piece of writing that records their concerns. It allows them to point out that their point might actually have been more salient than expected when the implementation of the RFC starts to veer from the proposed steps, and corrective action can be taken without much discussion or finger pointing.

## [](https://dev.to/wesen/quick-tip-tuesday-writing-rfcs-for-fun-and-profit-3bo#keep-it-simple-and-punchy)Keep it simple and punchy

[![Boxe Francaise - Wikimedia](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fedoirjiob9muyc8eku8g.jpeg)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fedoirjiob9muyc8eku8g.jpeg)

I follow a few very simple guidelines for my RFCs.

- Use **Google Docs**. Collaborative editing and comments are key.
- **Never** more than 2 pages.
- **Edit, edit, edit** until you can’t edit anymore.
    - Think of writing an RFC as writing code: crystal clear, short and efficient. [Good With Words](https://www.fulcrum.org/concern/monographs/1v53jz538) and its accompanying [coursera course](https://www.coursera.org/specializations/good-with-words) are great resources.

I use the following structure:

- **Metadata** - date, author, status, title, tags.
- **Overview** - two or three sentences that explain the purpose and goal of the RFC.
- **Glossary** - this makes sure that everybody understands the domain problem and its terminology.
- **Proposal** - one or two paragraphs explaining the proposed change in detail. Use links to external documents, but keep the proposal self-contained.
- **Pros** - describe the benefits of the proposed work.
- **Cons** - describe the downsides. This is important. Not only does it help you think through the problem, but it also communicates that you are aware and have incorporated the feedback of other people.
- **Implementation steps** - describe concrete implementation steps. This helps you think through the problem and keeps everybody accountable. It also puts a concrete scope on the work.

I keep RFCs in 4 folders:

- **Drafts** - these are free form RFCs that are not ready to be shared yet
- **Under discussion** - the RFC has been edited well enough to be shared and start getting feedback.
- **Deprecated** - these are RFCs that were either not finished, or discarded after discussion. Keep them around, certain topics come up regularly, and you can point back to old discussions. Maybe the circumstances changed and an old RFC can be brought back to life.
- **Accepted** - the RFC has gotten buy-in and gets moved to the codebase itself as an Architecture Decision Record (a topic for a later post). I still keep the Google Doc around in case a new discussion springs up.

## [](https://dev.to/wesen/quick-tip-tuesday-writing-rfcs-for-fun-and-profit-3bo#write-many-share-a-few)Write many, share a few

Sharing an RFC puts a significant burden on the people involved. Think of it as a complicated PR: a team can only tackle so many at once.

I like to write many RFCs, but only share a few. Writing many helps me clarify my thinking, and I like to “keep a few on the fire”. I often come back to them and revise them as I learn more, circumstances change or my subconscious comes up with new insights.

## [](https://dev.to/wesen/quick-tip-tuesday-writing-rfcs-for-fun-and-profit-3bo#do-you-write-rfcs)Do you write RFCs?

What about you? Do you write RFCs? What structure and workflow do you use? Do you have advice about technical writing? Please share your comments below!