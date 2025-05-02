---
link: https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md
site: GitHub
excerpt: 📑 Lasting documents from the OpenGitOps project which are versioned and released together (including the GitOps Principles and Glossary) - open-gitops/documents
twitter: https://twitter.com/@github
slurped: 2025-03-16T07:31
title: documents/PRINCIPLES.md at v0.1.0 · open-gitops/documents
tags:
  - gitops
  - principles
---

## Summary

[Gitops Principles](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#summary)

GitOps is a set of principles for operating and managing software systems.

When using GitOps, the _Desired State_ of a system or subsystem is defined declaratively as versioned, immutable data, and the running system's configuration is continuously derived from this data.

These principles were derived from modern software operations but are rooted in pre-existing and widely adopted best practices.

## Principles

[](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#principles)

1. **The principle of declarative desired state**
    
    A system managed by GitOps must have its _Desired State_ expressed declaratively as data in a format writable and readable by both humans and machines.
    
2. **The principle of immutable desired state versions**
    
    _Desired State_ is stored in a way that supports versioning, immutability of versions, and retains a complete version history.
    
3. **The principle of continuous state reconciliation**
    
    Software agents continuously, and automatically, compare a system's _Actual State_ to its _Desired State_. If the actual and desired states differ for any reason, automated actions to reconcile them are initiated.
    
4. **The principle of operations through declaration**
    
    The only mechanism through which the system is intentionally operated on is through these principles.
    

## Notes

[](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#notes)

### Principle 3 Notes

[](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#principle-3-notes)

- These differences could be due to the actual state drifting from the desired state, or the desired state changing intentionally.
- The source of drift doesn't matter. Contrary to CIops, _any_ drift will trigger a reconciliation

### Principle 4 Notes

[](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#principle-4-notes)

- We talk here about "regular operations." In an emergency, other modes of operations, e.g. manual intervention, should be considered - followed by a reconciliation of the "tainted" system with the declared state. → resolve the conflict between "GitOps principle" and "I need to deal with problems that GitOps doesn't cover"

## Glossary

[](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#glossary)

- ### Continuous
    
    [](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#continuous)
    
    By "continuous" we adopt the industry standard term to mean reconciliation continues to happen, not that it must be instantaneous.
    
- ### Declarative Description
    
    [](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#declarative-description)
    
    Describing the desired state or behavior of a system without specifying how that state will be achieved, thereby separating between configuration - the desired state - and implementation - the commands, API calls, scripts ... that actually achieve the desired state described in the declarative description.
    
- ### Desired State
    
    [](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#desired-state)
    
    The aggregate of all configuration data for a system form its _Desired State_ which is defined as data sufficient to recreate the system so that instances of the system are behaviourally indistinguishable.
    
- ### Software System
    
    [](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#software-system)
    
    One or more Runtime environments consisting of resources under management. In each Runtime, management Agents to act on resources according to security policies. One or more software Repositories for storing deployable artifacts that may be loaded into the runtime environments, eg. configuration files, code, binaries and packages. One or more Administrators who are responsible for operating the runtime environments ie. installing, starting, stopping and updating software, code, configuration, etc. A set of policies controlling access and management of repositories, deployments, runtimes.
    
- #### State Store
    
    [](https://github.com/open-gitops/documents/blob/v0.1.0/PRINCIPLES.md#state-store)
    
    A system for storing versioned, immutable Desired States that provides access control and auditing on the changes to the Desired State. Git may be configured as a State Store, but [special precautions must be taken](https://github.com/open-gitops/documents/blob/main/PRINCIPLES.md).