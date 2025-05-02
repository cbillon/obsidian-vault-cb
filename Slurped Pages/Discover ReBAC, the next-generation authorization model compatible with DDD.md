---
link: https://medium.com/@tpierrain/discover-rebac-the-next-generation-authorization-model-compatible-with-ddd-0d115cea6f2c
byline: Thomas Pierrain. (υѕe caѕe drιven)
site: Medium
date: 2023-12-23T15:27
excerpt: "TL;DR: At Agicap, we’ve started using ReBAC to manage user rights and permissions. ReBAC is an entirely new paradigm for authorization management. It’s so different from what we’re used to that it’s…"
twitter: https://twitter.com/@tpierrain
slurped: 2024-08-26T18:11
title: Discover ReBAC, the next-generation authorization model compatible with DDD
tags:
  - ddd
  - rebac
  - authorization
---

_TL;DR: At Agicap, we’ve started using ReBAC to manage user rights and permissions. ReBAC is an entirely new paradigm for authorization management. It’s so different from what we’re used to that it’s crucial to understand its specificities to use it effectively (and avoid missing out on its benefits by replicating old paradigms with this new approach). The flexibility and granularity of this model also allow us to model authorizations perfectly in line with the code and language of the domain, which suits us just fine 🙂._

_This series of articles should help you grasp the spirit, functionality, and uses of a modern ReBAC system._

_This first episode will serve to position ReBAC in relation to other authorization management paradigms and thoroughly understand its advantages (the WHY)._

_The rest of this series will focus on the implementation and use of a ReBAC system (the HOW), using an open-source solution: OpenFGA._

## Are we talking about Permissions? Authorizations? Roles?

You can consider “permissions” and “authorizations” as synonyms. In literature, “permissions” is often used to refer to finer-grained rights, while “authorizations” is used to characterize a set of rights (coarser-grained).

“Roles” are also groupings (coarser-grained) of permissions (finer-grained) — we’ll delve into this in more detail below.

For this reason, we generally refer to an authorization model in which we define and verify permissions (sometimes using the concept of roles).

## Firstly, what does ReBAC mean?

ReBAC is the acronym for [**Relationship-based access control**](https://en.wikipedia.org/wiki/Relationship-based_access_control). It is a new model where the authorization for an actor to access a resource is defined by the presence of RELATIONSHIPS between these actors and resources (think: graph mode).

## What are the specificities of ReBAC?

- **Expressiveness:** Since all authorizations are based on a simple concept of tuples defining relationships, such as (‘user:alice’, ‘editor’, ‘document:budget’) — meaning that user ‘alice’ can edit the resource named ‘document:budget’ (or, in other words, there exists a relationship ‘editor’ between user ‘alice’ and the resource ‘document:budget’) — **the flexibility is infinite, especially with relationship names being simple strings**.
- **Domain-Driven:** The simplicity and low ceremony of ReBAC concepts (which can be counted on one hand) allow us **to express in a very direct manner**, **the concepts of our domain** that are useful for managing authorizations. There is **no impedance mismatch** between the authorization model and the domain. 👍 We no longer have to adapt to a generic and technical “one-size-fits-all” security terminology, as was often the case with other authorization models and their concepts of groups, roles, ACLs, etc.
- **Fine-grained:** In contrast to the concept of “role” in the previous RBAC model ([**Role-based Access control**](https://en.wikipedia.org/wiki/Role-based_access_control)), the flexibility of ReBAC and its relationships, as numerous as desired, allows for the definition of very fine and precise authorizations. Unlike roles that aggregate and group a set of access rights behind a single concept (the role), ReBAC permits fine-grained authorizations. While this allows for more precise access, it may mean that the application needs to perform more authorization checks. Is that a problem? No, implementations are designed and optimized for this purpose.
- **Composable and versatile:** It is entirely possible to build role-based systems (RBAC) using ReBAC (depending on the authorization model one wants to create). ReBAC is even the best possible way to implement RBAC (we will see why later).
- **Ultra precise and expressive:** With its powerful grammar allowing hierarchies of relationships, transitivity, and more complex models using algebraic operations on relationships such as unions, intersections, and differences.
- **Centralized:** All ReBAC implementations today are centralized (while still being resilient and redundant). This enables quick and efficient action if there is a need to revoke an authorization, remove access, or change the rules if a vulnerability is detected. This is in contrast to having to modify authorization management systems scattered throughout different locations or in the code.
- **Secure:** Like all modern security systems, ReBAC is based on the principle of [**Enforce Least Privileges**](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html#enforce-least-privileges) and [**Deny-by-default**](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html#deny-by-default). While it is possible to express the opposite (discouraged), by default, there is no access or right granted.

## How was authorization handled before ReBAC?

Before ReBAC, there were other authorization models in our industry, such as:

- [**RBAC (Role-based access control)**](https://en.wikipedia.org/wiki/Role-based_access_control): This is coarse-grained access control **based on roles**, permissions, and users. The role-based model is still conceptually interesting, but RBAC implementations were more limited because they did not allow for fine-grained precision in authorizations. This often resulted in implicit permissions in the code behind each role or required additional databases or systems to store fine-grained authorizations separately.
- [**ABAC (Attribute-based access control)**](https://en.wikipedia.org/wiki/Attribute-based_access_control): ABAC, also known as policy-based access control, defines an access control paradigm where access rights are granted to users **based on** the use of policies that combine the presence or absence of **attributes** (think of these as tags). ABAC allows for expressing a set of complex rules to evaluate access based on the presence or absence of attributes. While rich, it is more complex and less domain-driven than ReBAC, meaning it’s a canonical model **more centered on generic terminology** that needs adaptation for different business domains.
- [**NIH (Not Invented Here syndrome)**](https://en.wikipedia.org/wiki/Not_invented_here): This refers to cases where people prefer to reinvent the wheel and manually handle all their authorizations in their local relational databases. In general, it is highly discouraged to reinvent security measures, and authorization management is no exception to this rule.

## Why did we choose this model for Agicap?

We have already listed many interesting features of ReBAC by presenting its specificities above, but let’s specifically look at what ultimately influenced this choice for us:

- **Security:** On the security front, it was important to choose a state-of-the-art solution and avoid falling into the NIH trap, where everyone reinvents the wheel by storing rights and authorizations scattered throughout Agicap’s different components and services. The deny-by-default aspect of ReBAC is infinitely more reassuring than older systems that relied on a restriction-based model. The [**OWASP foundation now recommends ReBAC (and even ABAC) as the default authorization model**](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html), moving away from the old role-based model familiar to all developers. It’s crucial to understand that in recent years, our industry has transitioned from old-school security models (perimeter-based, where defending the external boundaries was thought to be sufficient) to an **In-Depth security model**, where each service must continually verify rights for every incoming request. OAuth even plans to integrate ReBAC and its fine-grained authorization model into OAuth soon (following this [**OAuth 2.0 Rich Authorization Requests RFC**](https://datatracker.ietf.org/doc/html/rfc9396) in particular).
- **Time to Market:** It serves as a long-term accelerator to prevent development teams from repeatedly grappling with the same questions (how to model all of this, how to code it, how to secure the entire system) and allows them to capitalize on this investment. It provides an opportunity to capitalize on a modern approach in terms of expertise and modeling. ReBAC comes with a paradigm that guides everyone’s mental models on this sensitive subject of authorizations. In terms of maintenance, eliminating numerous places in the code that carried role logic allows time-saving when changing an authorization strategy at certain points — you only need to make changes in one location: our ReBAC system.
- **Fostering an All-In-One Experience:** Particularly important for our clients within a multi-product platform like ours. With our ReBAC platform, we have a common foundation for all development teams. It also offers the opportunity to build specific authorizations within each product while also capitalizing on common concepts for all AGICAP products (namely Organizations, Entities, Companies, etc)
- **Auditability:** In terms of auditability, this centralized solution is very interesting for detecting potential fraud attempts or vulnerabilities in our authorization modeling and addressing them.
- **DDD Friendly:** The almost non-existent impedance mismatch between our domain concepts and their ReBAC incarnation for authorization is also a very interesting point. One of the goals of Domain-Driven Design (DDD) practitioners, which includes many at Agicap, is to achieve **maximum alignment between business concepts and code**. We will see that in the upcoming posts but **the ReBAC paradigm is truly one of the first that allows this perfect alignment between our domains concepts/names, and the authorization topic**, without forcing the integration of generic authorization concepts in our code.

Ok. That’s all for this first episode on WHY ReBAC!

Next time we will see HOW to use it (modeling, synchronization with the different Bounded Contexts in your domain, runtime checks).

Stay tuned!

Note: this post is an adaptation of an internal post that I published on the Agicap notion (November 23, 2023).

On this ReBAC topic, our colleagues **Pauline JAMIN** **& Geoffroy BRAUN** have also made a nice introductory talk in french at DevoxxFR : [**Infuser du métier dans les autorisations avec ReBAC**](https://www.youtube.com/watch?v=aJn0v9OR4K8)