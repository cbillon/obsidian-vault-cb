---
link: https://towardsdatascience.com/yq-mastering-yaml-processing-in-command-line-e1ff5ebc0823
byline: Martin Heinz
site: Towards Data Science
date: 2021-06-15T15:37
excerpt: Learn to parse and manipulate YAML files more efficiently using yq
  command-line utility and this simple cheat sheet...
twitter: https://twitter.com/@Martin_Heinz_
slurped: 2024-10-26T16:26
title: "yq: Mastering YAML Processing in Command Line - Towards Data Science"
---

## Learn to parse and manipulate YAML files more efficiently using `yq` command-line utility and this simple cheat sheet

Nowadays, YAML is used for configuring almost everything (for better or worse), so whether you’re DevOps engineer working with Kubernetes or Ansible, or developer configuring logging in Python or CI/CD with GitHub Actions — you have to deal with YAML files at least from time-to-time. Therefore, being able to efficiently query and manipulate YAML is essential skill for all of us — engineers. The best way to learn that is by mastering YAML processing tool such as `yq`, which can make you way more efficient at many of your daily tasks, from simple lookups to complex manipulations. So, let's go through and learn all that `yq` has to offer - including traversing, selecting, sorting, reducing and much more!

## Setting Up

Before we begin using `yq`, we first need to install it. When you google `yq` though, you will find two projects/repositories. First of them, at [https://github.com/kislyuk/yq](https://github.com/kislyuk/yq) is wrapper around `jq` - the JSON processor. If you're already familiar with `jq` you might want to grab this one and use the syntax you already know. In this article though, we will use the other - a bit more popular project - from…
