---
link: https://www.baeldung.com/linux/etc-subuid
site: Baeldung on Linux
date: 2024-03-26T02:39
excerpt: Learn about a special file that can create a hierarchy of available
  user identifiers in the Linux system.
twitter: https://twitter.com/@AraromiTemitope
slurped: 2024-11-19T18:32
title: What Is the /etc/subuid File? | Baeldung on Linux
---

## 1. Overview

Managing users and permissions is important to maintaining a secure and organized Linux system. Further, this is especially important in multi-user and containerized environments. In traditional Linux systems, each user is assigned a unique [user ID (UID)](https://www.baeldung.com/linux/identifiers#what-are-uids) and [group ID (GID)](https://www.baeldung.com/linux/identifiers#what-are-gids) to control access to files, processes, and system resources. However, as the number of users and applications increases, the risk of UID or GID conflicts and security vulnerabilities also increases.

To address these challenges, Linux introduced the concept of subordinate user and group ID ranges, commonly known as _subuid_ and _subgid_ ranges. **The _/etc/subuid_ file is an important component of this user and group management system enabling administrators to allocate and manage UID ranges for individual users or processes**.

In this tutorial, we’ll understand the _/etc/subuid_ file and how we can use it in a Linux system.

## 2. User Namespace in Linux

In Linux, **[namespaces](https://www.baeldung.com/linux/list-namespaces#what-is-a-namespace) are a kernel feature that provides process isolation, enabling processes to have an isolated view of system resources such as process IDs, network interfaces, and file systems**.

There are different types of namespaces, one of which is the user namespace.

**User namespaces are specifically designed to isolate user and group IDs with each process having its own set of UIDs and GIDs independent from the host system or other user namespaces**. This distinction is useful for resource management and containerization.

Now, let’s discuss how the user namespace is related to the the _/etc/subuid_ file.

**The _/etc/subuid_ file defines the subordinate user ID (UID) ranges that we can assign to individual users or processes**. Thus, it enables them to create and operate within an isolated user namespaces.

### 3.1. Location and Purpose

The _subuid_ file is typically located in the _/etc_ directory on Linux systems. Further, its primary purpose is to allocate and manage subordinate UID ranges for users or processes. As a result, users can operate in separate user namespaces without conflicting with the host system or other user namespaces.

### 3.2. File Format and Structure

This file follows a specific format:

`username:start_uid:uid_count`

**Each line represents a single entry that defines a subordinate UID range for a user or process**:

- _username_: username or system account to which the subordinate UID range is assigned
- _start_uid_: starting UID value of the range
- _uid_count_: number of subordinate UIDs in the range

Thus, we can define the subordinate UID range as the range of UIDs from _start_uid_ and spanning _uid_count_ UIDs.

### 3.3. Subordinate UID Range

It’s important to note that the subordinate UIDs defined in the _/etc/subuid_ file are separate from the traditional UID assignments in _/etc/passwd_. Basically, subordinate UID ranges are additional UIDs assigned to users for use in user namespaces. This way, subordinate UIDs enable users to operate as different users within a container or a virtualized environment, providing a layer of isolation and security.

**Let’s see a typical output displaying the content of the _/etc/subuid_ file**:

```
$ cat /etc/subuid
root:231072:512
user1:100000:65536
user2:165536:65536
user3:200000:1000
```

In this example, the _root_ user is assigned a subordinate UID range starting from _231,072_ with a total of _512_ UIDs. Moreover, the _user1_ user is assigned a subordinate UID range starting from _100,000_ with a total of _65,536_ UIDs.

### 3.4. Managing the _/etc/subuid_ File

Typically, **it’s the root user that controls and owns this file**. We usually set the permissions for the file to _0644_:

- read and write for the owner
- read for the group and others

These permissions are necessary to prevent unauthorized modifications.

**To add or modify entries in the _/etc/subuid_ file, we can employ [_usermod_](https://www.baeldung.com/linux/add-user-multiple-groups#usermod)**:

```
$ sudo usermod --add-subuids 100000-165535 user1
```

This command assigns the subordinate UID range from _100,000_ to _165,535_ to the user _user1_.

However, it’s essential to exercise caution when making changes to the _/etc/subuid_ file as incorrect entries or unintended overlapping between UID ranges can lead to conflicts and security vulnerabilities.

### 3.5. Relationship With _/etc/passwd_

The [_/etc/passwd_](https://www.baeldung.com/linux/manually-add-user#etcpasswd) file is the primary user database on Linux systems containing information about user accounts:

- usernames
- UIDs
- GIDs
- home directories
- shell programs

**While the _/etc/passwd_ file defines the primary UIDs and GIDs for user accounts, the _/etc/subuid_ file specifies the subordinate UID ranges that users can have within respective user namespaces**.

When a user creates a new user namespace or runs a process within one, the [Linux kernel](https://www.baeldung.com/linux/kernel-source-code-headers#the-linux-kernel) maps the subordinate UID range in _/etc/subuid_ to the user’s UID within a user namespace. Therefore, this enables the user to operate within the confines of that namespace with access to resources specified by the mapped UIDs.

### 3.6. Interaction With Pluggable Authentication Modules

The [Pluggable Authentication Modules (PAM)](https://www.baeldung.com/linux/pam-ssh-login-notifications#pluggable-authentication-modules-pam) framework is a suite of shared libraries that provide authentication and authorization services for applications and system services on Linux systems.

Generally, **the _/etc/subuid_ file interacts with PAM through the _pam_subuid_ module**. In particular, this module is responsible for managing subordinate UID ranges for user sessions. When a user logs in or starts a session, the _pam_subuid_ module reads the user’s subordinate UID range from the _/etc/subuid_ file and sets up the appropriate user namespace for that session.

## 4. Applications

Now, let’s explore some scenarios where the _/etc/subuid_ file is essential.

### 4.1. Container Orchestration and Management

One of the most prominent application categories that leverage the _/etc/subuid_ file is container orchestration and management systems like Docker and Kubernetes.

**Docker relies on user namespaces and subordinate UID ranges to provide secure and isolated environments for running containers**. When we run a Docker container, the container process is assigned a subordinate UID range from the _/etc/subuid_ file of the user running the container.

Similarly, Kubernetes assigns a unique subordinate UID range to each pod, ensuring that its processes run with their own isolated user and group identities.

### 4.2. Virtualization and Cloud Computing Environments

Subordinate UID ranges and the _/etc/subuid_ file are also essential in virtualization where multiple virtual machines or instances may need to run with their own isolated user and group namespaces.

For instance, in an OpenStack environment, we can configure each Nova compute node with a range of subordinate UIDs for VM instances. When we launch a new VM, it’s assigned a subordinate UID range from the available pool.

## 5. Conclusion

The _/etc/subuid_ file is an essential component of user and group management in Linux systems, particularly in multi-user and containerized environments. By defining subordinate user ID ranges for individual users or processes, this file enables the creation of isolated user namespaces. Therefore, this process enhances security, prevents conflicts, and enables efficient resource management.

In this article, we explored the purpose, structure, and management of the _/etc/subuid_ file and its relationship with _/etc/passwd_. We also discussed its interaction with the Pluggable Authentication Modules (PAM).