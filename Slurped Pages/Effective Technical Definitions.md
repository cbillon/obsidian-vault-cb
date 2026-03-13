---
link: https://cacm.acm.org/blogcacm/effective-technical-definitions/
byline: Lawrence Fisher
site: Communications of the ACM
date: 2025-09-25T21:35
excerpt: "Work in engineering, science, or technology can only be effective if
  it relies on precisely defined concepts. For the fundamental notions taught at
  school, particularly in mathematics, physics, and chemistry, the definitions
  honed over centuries, have become impeccable: integral, matrix, speed, mass,
  molecule. . . In engineering, definitions are often less satisfactory,
  sometimes giving the impression of following from a committee process where
  every member argued for the inclusion of his own pet idea, at the expense of
  the consistency of the result."
slurped: 2026-03-08T09:58
title: Effective Technical Definitions
---

Work in engineering, science, or technology can only be effective if it relies on precisely defined concepts. For the fundamental notions taught at school, particularly in mathematics, physics, and chemistry, the definitions honed over centuries, have become impeccable: integral, matrix, speed, mass, molecule. . . In engineering, definitions are often less satisfactory, sometimes giving the impression of following from a committee process where every member argued for the inclusion of his own pet idea, at the expense of the consistency of the result.

If you ask around, you will receive general advice stating that a definition should be clear, precise, unambiguous, complete. All good, but too general.

A definition could satisfy several of these properties and still be flawed. In this post, I will take an example–more precisely, a counter-example–from a prestigious source, analyze how it fails to achieve its goals, and draw some lessons for writing _good_ definitions.

**Defining Requirements**

The counter-example is one I used in the introductory chapter of my book about requirements engineering.[1](https://cacm.acm.org/blogcacm/effective-technical-definitions/#one) It comes from an IEEE standard on systems engineering[2](https://cacm.acm.org/blogcacm/effective-technical-definitions/#two) and reads:

**_Requirement_**_: A statement that identifies a product or process operational, functional, or design characteristic or constraint, which is unambiguous, testable or measurable, and necessary for product or process acceptability (by consumers or internal quality assurance guidelines)._

I am not picking on some obscure document; this particular definition is religiously cited by dozens of software engineering books and articles, even including the “Systems Engineering Body of Knowledge.”[3](https://cacm.acm.org/blogcacm/effective-technical-definitions/#three)

The mangled syntax is not a good sign; let us be charitable and assume that “_a product or process operational, functional, or design characteristic or constraint_” actually means something. Translated into English, that something could be “a characteristic or constraint affecting the operation, function, or design of a product or a process.” At least, that is the only way in which I can parse the original so that it makes some sense. (It would have helped if the authors had at least known about the possessive in English and written “a _product’s or process’s_ constraint, although “a constraint _of_ a product etc.” is less contorted.)

Since the purpose of this post is not to poke fun at the venerable IEEE, but to understand how to do better for our own definitions, let us turn positive and state a first rule, expressed here informally: to be crystal-clear, a definition should use the simplest language possible. The _concepts_ may be complicated (we are talking about definitions in technology and science); no need to make comprehension harder by using contorted _phrasing_. One might even suggest that the more sophisticated the concepts are, the simpler the language should be.

Continuing to work from the ground up (from more superficial issues to the most important flaws in this counter-example), the end of the definition is not acceptable in a professional standard. First, again, a matter of form: there is no need to talk about “_product or process_” acceptability; the first part of the definition indicates that the requirement has to do with a feature or constraint of “a product or process.” That is what we are dealing with, so “acceptability” suffices.

The rule here is that a definition should be minimal and avoid repetition, which leads to confusion. Repetition is good in _explanations_ of the definition, but the definition itself should only include the elements that are critical to identifying the concept, and include each once. Anything else causes bloat and can lead to misunderstanding. (The advice of avoiding duplication also applies to requirements; Meyer[1](https://cacm.acm.org/blogcacm/effective-technical-definitions/#one) contains an analysis of why repetition is bad. More generally, there is some commonality between software requirements and technical definitions.)

**Overstepping the Mandate**

The end of the definition is flawed in a deeper way. We are told that a requirement should be “_necessary for acceptability_.” Fine, but how does one determine acceptability? That is a serious issue of systems engineering, but it is not part of the definition of requirement. The parenthetical remark “_(by consumers or internal quality assurance guidelines)_” is amateurish; acceptability should be determined by a well-defined process, not just summarily dismissed as something that “consumers or internal quality assurance guidelines” determine. “Consumers” is not an engineering term and is too restrictive; there are no “consumers” of a B2B product, for example (maybe they meant “customers”?). Who is the “consumer” in a compiler project? All serious companies have precise rules as to who determines the acceptability of a product or process (typically, QA engineers, not directly consumers) and how.

The authors of this definition are in over their heads here; it is not their job to determine (in this definition) who is in charge of determining acceptability. The only role of this concept in the definition of “requirement” is to state that one of the roles of requirements is to specify the conditions under which the final product will be acceptable. The occurrence of the term “acceptability” in this definition should be a reference (a hyperlink) to an “acceptability” entry in the standard. That entry can carefully and professionally specify what it takes to assess whether a product or process is acceptable, based on concepts of quality assurance.

The definition of “requirement” is trying to do too much. It encroaches into a task (defining acceptability) beyond its job (defining requirements).

The rule here is another form of minimalism: a definition should define one concept. (In rare cases–although not this one–we may need to define two or more mutually-dependent concepts together, which we can in fact consider as a composite concept so as to fit within the rule.)

**From the Descriptive to the Prescriptive**

The example goes beyond its job in an even more serious way. It does not just try to say what a requirement is; it also decrees what a requirement should be: “_unambiguous, testable, or measurable_.” By the way, why “_or_”? Do we get a choice between testability and measurability? Probably not; the “_or_” was likely intended as an “_and_,” but that is the least of the problems.

A definition should be **descriptive**. With the addition of the three conditions, the definition of requirements becomes **prescriptive**, overstepping its role. A definition of speed tells you what speed is (ratio or derivative of distance against time); it should not tell you how fast you may drive. The confusion makes the definition of “requirement” both incomplete and wrong:

- It is incomplete because the three qualities cited (unambiguous, testable, measurable) are only a fraction of the desirable properties of requirements. For example, good requirements should also be feasible, readable, prioritized. The requirements literature has in-depth discussions of the desirable quality factors; Meyer[1](https://cacm.acm.org/blogcacm/effective-technical-definitions/#one) has a chapter on the topic with 14 separate factors. Why start listing some of them and stop at three? Is a requirement that satisfies one of them but not the others still a requirement?
- In fact, what about a requirement attempt that does not satisfy the quality factors, even just one of those actually included? According to the definition, if it is not “unambiguous,” it is not a requirement. Nonsense. It is almost always the case that a requirements document (at least one expressed in some combination of natural language, UML, and use cases, the dominant practice) will contain _some_ ambiguity. It is still a requirement! A flawed requirement perhaps, but a requirement all the same. The speed of a car that is going over the speed limit is still a speed.

The rule here is simple: a definition should be purely descriptive. It is not a code of ethics. Any prescription on the use of the objects of the definition should come separately.

This analysis gets us started on the question of what makes a technical definition good or bad. I will examine further criteria in a forthcoming post. In the meantime, I hope this counter-example will encourage you to take a harder look at the effectiveness of some of the definitions you have encountered in your own work.

**References**

1. Meyer, B. _Handbook of Requirements and Business Analysis_, Springer, 2022.

2. IEEE Standard for Application and Management of the Systems Engineering Process, 1220-1998, [https://ieeexplore.ieee.org/document/741941](https://ieeexplore.ieee.org/document/741941). The cited definition is 3.1.33, p. 8 (p. 15 of the PDF).

3. Guide to the Systems Engineering Body of Knowledge (SEBoK), [https://sebokwiki.org/wiki/Guide_to_the_Systems_Engineering_Body_of_Knowledge_(SEBoK)](https://sebokwiki.org/wiki/Guide_to_the_Systems_Engineering_Body_of_Knowledge_\(SEBoK\)). 

[![Bertrand Meyer](https://cacm.acm.org/wp-content/uploads/2025/04/042425.BLOG_.Bertrand-Meyer-350.jpg)](https://cacm.acm.org/wp-content/uploads/2025/04/042425.BLOG_.Bertrand-Meyer-350.jpg)

_**Bertrand Meyer** is a professor and Provost at the Constructor Institute (Schaffhausen, Switzerland) and chief technology officer of Eiffel Software (Goleta, CA)._

Submit an Article to CACM

CACM welcomes unsolicited [submissions](https://cacm.acm.org/author-guidelines/#CACMsubmission) on topics of relevance and value to the computing community.

You Just Read

#### Effective Technical Definitions

© 2025 Copyright held by the owner/author(s).