---
link: https://nickjanetakis.com/blog/using-envsubst-to-merge-environment-variables-into-config-files
site: Nick Janetakis
date: 2021-07-13T00:00
excerpt: This command is available on Linux and macOC by default and it lets you
  template out files with ENV vars to create dynamic config files.
twitter: https://twitter.com/@nickjanetakis
slurped: 2025-02-08T17:36
title: Using envsubst to Merge Environment Variables into Config Files — Nick
  Janetakis
---

Updated on July 13, 2021 in [#linux](app://obsidian.md/blog/tag/linux-tips-tricks-and-tutorials)

![using-envsubst-to-merge-environment-variables-into-config-files.jpg](app://obsidian.md/assets/blog/cards/using-envsubst-to-merge-environment-variables-into-config-files-7ba7c67cba95c485c8bb999aee8a7e285758bf52ac10fc6a02f491d93573c3a4.jpg)

## This command is available on Linux and macOC by default and it lets you template out files with ENV vars to create dynamic config files.

**Quick Jump:**

- - [Demo Video](#demo-video)

Let’s say you have an nginx or Kubernetes config file which doesn’t support templating out of the box and you want to dynamically create config files based on 1 or more environment variables. This is what `envsubst` lets you do.

This can help you avoid creating 2 separate config files that are nearly identical. Instead you can define what you want to be different as environment variables and send it to `envsubst` to be processed. We’ll go over a few examples below:

### [#](#demo-video) Demo Video

#### Commands

Trying out `envsubst` on the command line:

```
$ export PERSON=nick
$ echo 'hello ${PERSON}' | envsubst
hello nick

# We're using single quotes above instead of double quotes to avoid the variable
# being interpreted by your shell. Single quotes will ensure it's picked up
# and processed by envsubst.
```

Piping the output of `envsubst` as input to another command:

```
$ source .env
$ envsubst < aws-cluster-tpl.yaml | eksctl create cluster --config-file - --dry-run
```

Outputting a new config file with the results of running it through `envsubst`:

```
$ source .env
$ envsubst < aws-cluster-tpl.yml > aws-cluster-example.yaml
```

#### Timestamps

- 0:12 – envsubst lets you generate templated files with ENV variables
- 0:44 – Trying out envsubst on the command line with a simple echo
- 1:19 – Going over the use case of templating out an eksctl config file
- 2:29 – Taking a look at a templated eksctl config file with ENV vars
- 3:34 – Running the templated config through envsubst
- 4:53 – Exporting the processed config to eksctl as input
- 6:29 – Writing out a new processed config file based on using envsubst
- 7:00 – Being able to define default values using a fork of envsubst

#### Reference Links

- [https://github.com/a8m/envsubst](https://github.com/a8m/envsubst) (optional replacement with default variable support)

