---
link: https://moodle.org/mod/forum/discuss.php?d=462107
excerpt: Hello, at our company we will be creating several Moodle plugins and I am planning to release them as open source software.
tags:
  - moodle
  - Moodle-in-English/-Coding-best-practices-for-creating-open-source-Moodle-plugins-Moodle-org
slurped: 2024-11-19T16:38
title: "Moodle in English: Coding best practices for creating open source Moodle plugins | Moodle.org"
"tags:": moodle  plugin
---

Hello, at our company we will be creating several [Moodle plugins](https://moodle.org/plugins "Auto-link") and I am planning to release them as open source software.

[Here is one](https://github.com/fulldecent/moodle-local_enrollment_tokens) that lets you create course tokens that work like prepaid gift cards. Anybody can use them to create an account on your LMS and enroll in that course. And they expire after first use. There are a bunch of other goodies, reporting, and extensibility in there too.

I would like to ask help, please, is there any very well maintained existing plugins I can study that are managed well? Or any great resources for best practices when making plugins?

Specifically I am looking at:

- Setting up testing
- Automating testing with GitHub Actions
- Linting/pretifying code to match established community conventions
- Skeleton layout of folders/files in the repo
- Any required features for publishing the plugin to the Moodle plugin directory
- Best practices for supporting the new Moodle versions before they come out (e.g. GitHub Action cron job using a version matrix)

Without these best practices you quickly get a project that is unmaintainable, works for you but not for other people that [download](https://moodle.org/mod/glossary/showentry.php?eid=18&displayformat=dictionary "Glossary of common terms : download") it, or goes stale when a new Moodle is released.

^ So I find it valuable to iron out these details first and document them, and publish a MVP "best practices demo" plugin before seriously releasing my real projects.

Thank you!