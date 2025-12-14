---
link: https://hillelwayne.com/post/problems-with-the-4doc-model/
site: Hillel Wayne
date: 2023-07-05T02:00
excerpt: |-
  Two universal facts about user documentation:
   Documentation is really important. We are really bad at writing it.  We don’t have good theories on what makes for useful documentation. That is except for the four document model, or Diátaxis.1 I’m glad that people use it. I’m also a little frustrated that people use it even when its inappropriate. My problem is that it’s not a universal or comprehensive model: there’s a lot of documentation people need to write that doesn’t neatly fit the model.
slurped: 2025-12-07T11:06
title: My Problem With the Four-Document Model
tags:
  - documentation
  - diataxis
---

Two universal facts about user documentation:

1. Documentation is really important.
2. We are really bad at writing it.

We don’t have good theories on what makes for useful documentation. That is except for the [four document model](https://documentation.divio.com/), or [Diátaxis](https://diataxis.fr/).[1](https://hillelwayne.com/post/problems-with-the-4doc-model/#fn:diataxis) I’m glad that people use it. I’m also a little frustrated that people use it even when its inappropriate. My problem is that it’s not a universal or comprehensive model: there’s a lot of documentation people need to write that doesn’t neatly fit the model.

## What is the 4doc model?

![four document types on two axes](https://documentation.divio.com/_images/overview.png)

The 4doc model says that user-facing documentation should fall into four categories:

- **Tutorials** describe how to get up and running as fast as possible.
- **How-tos** describe how to do _specific_ things.
- **Explanations** teach a topic you already know in-depth.
- **References** describe exactly what things are and what they do.

Each category serves a different purpose, targets a different audience, and is written in a special style. Tutorials are friendly and simplified, while references are dry and comprehensive.

4doc was originally started by Daniele Procida at Divio to improve their documentation, and has since become popular in the broader tooling ecosystem.

## My problems

I think of the 4doc model like the five-paragraph essay. In the US, kids are taught to write essays in three parts:

1. A starting paragraph the context and the main thesis,
2. Three body paragraphs that each independently support the thesis
3. A concluding paragraph with summarizing the arguments and thesis.

It’s a good starting point to learn essay writing. The problem is that students think it’s the _only_ way to write essays, and then they cram everything they write into a format straitjacket.

4doc is _much_ better than 5para: I’ve never read a halfway-decent 5-paragraph essay but I’ve read plenty of good 4doc pages. Even so, it still leads to the same kind of “this is the only way” thinking. People use the model even when it’s not the right choice and they don’t vary up the format even when they should.

My specific issue is that the model is not **universal** and it’s not **comprehensive**.

### It’s not universal

From [the structure page](https://documentation.divio.com/structure/):

> Good examples of the scheme in substantial projects include:
> 
> - the [Divio Developer Handbook](https://docs.divio.com/)
>     
> - [Django’s documentation](https://docs.djangoproject.com/en/3.0/#how-the-documentation-is-organized)
>     
> - [django CMS’s documentation](http://docs.django-cms.org/)
>     

Divio and Django CMS are _tools_. Tools and libraries are different than frameworks and languages. Tools have simple [conceptual models](https://en.wikipedia.org/wiki/Conceptual_model), the set of interrelated ideas you need to internalize to use them properly. You just need to know _what_ they do and _how_ to use them. The 4doc model, developed at Divio, most reflects the documentation needs of tools.

Look at django. Frameworks are one step up from tools and are still mostly about “what and how”, but they also have a lot of interrelated concepts you need to understand. While it still ostensibly follows the 4doc model, the [“explanation” section](https://docs.djangoproject.com/en/4.0/topics/) is triple the size of the tutorial and how-to sections combined.[2](https://hillelwayne.com/post/problems-with-the-4doc-model/#fn:django-measurement) The explanation section also _teaches_ the user, which isn’t supposed to happen:

> It’s not the place of an explanation to instruct the user in how to do something. Nor should it provide technical description. These functions of documentation are already taken care of in other sections. — [divio](https://documentation.divio.com/explanation/)

It’s only possible to separate explanation and instruction for things if you don’t need a good mental model. Things like tools.

If we zoom out to programming languages (PLs), the 4doc model breaks down entirely. If you look at the [Python](https://docs.python.org/3/) docs, the tutorial section needs to also explain and reference, and the references are both explanation and how-to. Language concept maps are just too dense to separate.

I’ve seen some languages try to strictly follow the 4doc model and it doesn’t work. 4doc is oriented around random-access, languages are taught in a sequence-of-lessons. The two don’t mix well.

In short, the 4doc model is **not universal**. It was designed for documenting _tools_ and doesn’t fully address the needs of frameworks and programming languages.

### It’s not comprehensive

4doc would be comprehensive if all useful forms of documentation fell into one of the four types. This isn’t true for the same reason 4doc is nonuniversal: it only covers documentation forms that are necessary for tools.

If you’re documenting a complex topic like a programming language, there’s at least (at least) two more kinds of docs you’ll want:

#### 1. Conceptual Overviews

If I was presented with 4doc-following documentation and had to add just one thing to it, I’d write a conceptual overview.

Conceptual overviews cover the concepts of the tool/framework/language. It presents what the big picture of what this is all _for_ and previews the conceptual model users will learn. The denser the model, the more you need a conceptual overview.

The 4doc model describes documentation types in terms of cooking, so here’s one for conceptual overviews:

> All soups work in the same basic way: prepare ingredients like vegetables and meat and boil them in water. This softens the ingredients and flavors the water (“broth”). We can make higher-quality soups by first cooking some ingredients in oil (which develops more flavors), adding some ingredients later in the boil (so they don’t get _too_ soft), using a boiling different liquid (like chicken stock or beer), or using spices.

For a more in-depth example of an overview, you can see [one I wrote for TLA+](https://learntla.com/intro/conceptual-overview.html). It first explains the problem TLA+ is trying to solve (designing complex concurrent systems), introduces the concepts involved (specs, behaviors, properties, models), and only _then_ shows any code. And even then it doesn’t explain how to _run_ the code, just how various aspects of the code relate to the concepts.

(“Aren’t these just a kind of explanations?” Explanations are intended for people who already know, or are in the process of learning, your subject. It comes after the tutorial. Conceptual overviews come _before_ the tutorial. They are for people who are _getting ready_ to learn the subject, or for outsiders evaluating if they _want to learn it_.)

#### 2. Snippets and Examples

Examples are complete code intended to be studied. Snippets are samples of code intended to be used. As much as we make fun of Stackoverflow-based development, we copy and paste from websites because that works and makes us productive. If you provide them from the start, developers don’t have to find them from sketchy third parties.

Examples also show _how something is done_, which is subtly-but-fundamentally different from _how-to do something_. Imagine I present you an example of a code, and then an example of the same code, optimized. I’m not giving you a rote how-to on optimizing code, I’m demonstrating a way of _thinking_ about optimizing code. It all comes back to the mental model.

Or take Julia Evans’ [debugging mysteries](https://mysteries.wizardzines.com/), a set of [Choose Your Own Adventures](https://en.wikipedia.org/wiki/Choose_Your_Own_Adventure) where the reader debugs a problem. It looks like a how-to, but it’s a branching path, not a linear set of steps.

![An example of a debugging mystery, where the user can try a few different things and see why they do (or don't) work](https://hillelwayne.com/post/problems-with-the-4doc-model/img/wizardzines.png)

The cooking analogy would be a video of a chef making a dish and explaining what she’s thinking while she makes it. Or maybe doing a taste-test of undersalted dishes. It’s not a perfect analogy. Written examples don’t make a whole lot of sense in tradecraft.

## Good is the enemy of better

I want to be clear: I don’t think 4doc is _bad_. I think it’s quite good! But I think you have to think about _how_ you’re using it. Like any other tool, you have to evaluate what it does for you. The model says at much:

> Diátaxis is based on sound theoretical principles and has been proven in practice, but it’s not the final word in documentation. … the structure it proposes is not intended to be a plan, something you must complete in your documentation. It’s a guide, a map to help you check that you’re in the right place and going in the right directions. — [How to use Diátaxis](https://diataxis.fr/how-to-use-diataxis/)

I’m unhappy that the original Divio page doesn’t have this disclaimer and instead has big quote calling it the “The Grand Unified Theory of Documentation”. Procida doesn’t seem to be believe this and I wish divio would stop pitching 4doc as the one-and-only documentation system. It might be good for getting people on board but it’s bad for broader documentation innovation.

_Thanks to Lito Nicolai for feedback. If you liked this, come join my [newsletter](https://buttondown.email/hillelwayne/)! I write essays once a week._