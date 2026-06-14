---
link: https://louwrentius.com/sharing-group_var-variables-across-ansible-inventories.html
excerpt: Some time ago I worked with a customer that used Ansible to deploy a multitude of applications across their individual TAP1 environments.
slurped: 2026-06-04T09:08
title: Sharing group_var variables across Ansible inventories
tags:
  - ansible
---

## The problem

Some time ago I worked with a customer that used Ansible to deploy a multitude of applications across their individual TAP[1](https://louwrentius.com/sharing-group_var-variables-across-ansible-inventories.html#fn:dtap) environments.

Unfortunately the Ansible inventory was setup in such a way that `group_vars` variables for a specific application were manually duplicated across each TAP environment.

![[Pasted image 20260604091023.png]]

Changing or adding a `group_vars` variable meant updating the `group_vars` file for each TAP environment. Making changes to `group_vars` in this environment is a cognitive burden: there is a high risk that one of the environments is missed by mistake.

The different TAP environments share the vast majority of all `group_var` variables, except for just a few variables that are environment-specific. There should be no reason to duplicate `group_vars` variables for each environment.

## The solution

### A shared folder symlinked into group_vars

Within the application inventory folder, We first create a folder called _000_shared_ that will contain all `group_vars` YAML files with variables that are shared across all TAP environments.

Next, we symlink this folder within each `group_vars` folder of each environment. Each environment also contains environment-specific configuration in one or more separate files.
![[Pasted image 20260604091137.png]]


With this example, any change within `webserver.yaml` will be applied to all TAP environments of the 'Alpha' application.

### Sharing group_vars variables across applications

We can apply this trick one level higher: we can create a shared folder called _000_shared_global_ containing settings that are shared across all application environments.

![Ansible Inventory Hierarchy](https://louwrentius.com/static/images/ansible-inventory/ansible_global.png)

In this example, we could change the DNS configuration for all environments by changing the settings in one, single file. But the global shared `group_vars` can be used to assure a (security) baseline between unrelated applications that do share similar technology (All Python or PHP applications).

## Evaluation

Implementing this solution in an existing Ansible 'code base' can be a significant undertaking, but the long-term benefits seem worth it.

- It reduces the amount of work required for changes
- It reduces the risk of human error
- It reduces the cognitive load

The good news is that you can migrate to this new style of inventory at your own pace, there is no need for a big-bang deployment. Changes can be made to one environment at a time.

## Q & A

### Why the leading 000 zeros in shared folder names?

Ansible parses `group_vars` YAML files according to their order within a directory. This means that the shared folders should be named with leading zeros, to assure that they are always parsed first.

This allows you to override shared `group_vars` settings for a particular environment, because these environment-specific files will be parsed later and their variable settings will be applied.

### Just use one inventory for DTAP!

You can create a single inventory for test, acceptance and production, and separate the environments with groups. A shared group contains shared settings and environment-specific settings are put in the group_vars of the specific group. Unfortunately there is a risk:

[Ansible documentation:](https://docs.ansible.com/projects/ansible/devel/inventory_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups)

> If you need to manage multiple environments, consider defining only the hosts of a single environment in each inventory. This way, it is harder to, for example, accidentally change the state of nodes inside the “test” environment when you wanted to update some “staging” servers.

Therefore, I would recommend against this approach.

### Changes impact production?

Changing shared `group_vars` variables means that all environments are changed, including production. This requires proper code review and deployment processes that assures that said changes are only deployed to production when test and acceptance environments have been changed first.