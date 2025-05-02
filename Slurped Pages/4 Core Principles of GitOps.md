---
link: https://thenewstack.io/4-core-principles-of-gitops/
byline: Alex Williams
site: The New Stack
date: 2023-05-11T21:17
excerpt: It's at the point where GitOps is getting enough notice that a brief on its principles is appropriate. Here are four ground rules to keep in mind.
twitter: https://twitter.com/@thenewstack
slurped: 2025-03-16T07:19
title: 4 Core Principles of GitOps
tags:
  - gitops
---

It’s at the point where [GitOps](https://thenewstack.io/does-gitops-provide-the-key-fix-for-kubernetes-complexity/) is getting enough notice that a brief on its principles is appropriate.

Last year, the [OpenGitOps](https://github.com/open-gitops) community released [GitOps Principles 1.0](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md). There’s general support for GitOps and many competing interests in developing GitOps engines, such as with [Argo and Flux](https://thenewstack.io/argo-cd-and-flux-are-cncf-grads-but-what-now/), two graduate projects from the [Cloud Native Computing Foundation](https://cncf.io/?utm_content=inline+mention). But focusing on principles lets everyone know what GitOps is and, even more so, [helps define what it is not](https://github.com/open-gitops/project/discussions/101#discussioncomment-2724356).

OpenGitOps defines GitOps as a set of principles for operating and managing software systems, [wrote](https://hackmd.io/@scottrigby/SJW6721Sd) open source software engineer [Scott Rigby](https://www.linkedin.com/in/scottrigby/). “When using GitOps, the Desired State of a system or subsystem is defined declaratively as versioned, immutable data, and the running system’s configuration is continuously derived from this data. These principles were derived from modern software operations but are rooted in pre-existing and widely adopted best practices.”

With DevOps, operations and development teams collaborate on their own chosen tools. GitOps provides a declarative approach and [complements DevOps](https://codefresh.io/learn/gitops/gitops-vs-devops/). It allows for application delivery and cluster management. 

Here are a few takeaways, from this talk and others at the conference:

## Principle #1: GitOps Is Declarative.

_A [system](https://github.com/open-gitops/documents/blob/v1.0.0/GLOSSARY.md#software-system) managed by GitOps must have a desired state expressed [declaratively](https://github.com/open-gitops)._

## Principle #2: GitOps Apps Are Versioned and Immutable

_The desired state is [stored](https://github.com/open-gitops/documents/blob/v1.0.0/GLOSSARY.md#state-store) in a way that enforces immutability, versioning, and complete version history._

The general viewpoint: rollbacks should be simple.

You go a step further with GitOps providing an auditable trail of all changes to infrastructure and applications
Versioning allows organizations to find gaps in their security and also allows for testing and declarative infrastructure, Ben Ezra said. Using tools like the [Open Policy Agent](https://thenewstack.io/getting-open-policy-agent-up-and-running/), an open source project for establishing authorization policies across cloud native environments, allows for more productivity because once it’s automated, teams spend less time agonizing over whether or not their infrastructure is compliant, which gives them more time for innovation and feature development.

While automation is an important part of DevOps, it’s by no means the only methodology also calls for cross-functional collaboration and the breaking down the silos and knowledge sharing across an org. GitOps builds on these principles of leveraging automation and infrastructures as code to reduce configuration drift, and providing a single source of truth for an entire team or org.

By writing it down, all team members can contribute to the infrastructure code, which promotes shared responsibility across the entire software development lifecycle. Just as importantly, everyone is aware of these changes, so they can speak up if they see something you missed.”

## Principle #3: GitOps Apps Are Pulled Automatically

_Software automation automatically pulls the declared state declaration from the source._

## Principal #4: GitOps Apps Are Continuously Reconciled

_“Software agents [continually](https://github.com/open-gitops/documents/blob/v1.0.0/GLOSSARY.md#continuous) observe the actual system state and [attempt to apply](https://github.com/open-gitops/documents/blob/v1.0.0/GLOSSARY.md#reconciliation) the desired state.”_

