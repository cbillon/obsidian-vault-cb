---
link: https://padlock.argh.in/2024/12/15/container-capabilities.html
site: Padlock
excerpt: Notes about information security, written by Feroz Salam.
tags:
  - slurp/
slurped: 2025-02-08T08:40
title: "Container capabilities: a short tour"
---

A while ago, for reasons that I no longer remember clearly, I spent some time investigating the differences between different configurations of Docker containers and the implications they had on the Linux capabilities the resulting containers would have. I find that advice around Docker security is often a bit muddled – a common occurrence of this confusion is the use of ‘privileged’ and ‘root’ as interchangeable terms, when the implications of either choice for a container are different.

The topic as a whole is surprisingly complex, but I thought it might be useful to compare Linux capabilities across different container configurations. In short, I’m trying to answer the question – what Linux capabilities does a particular configuration combination running on Docker give you? I’m going to consider the following combinations:

- A root container running with the `--privileged` flag
- A root container running without the `--privileged` flag
- A non-root container running without the `--privileged` flag
- A non-root container running with the `--privileged` flag

While not exhaustive, I think the comparison is useful to highlight some interesting nuances of Docker security.

For simplicity, I’m not going to consider containerization engines other than Docker, and I’m also going to (mostly) ignore other ways of setting capabilities. I’m also going to focus exclusively on capabilities, but both the `--privileged` flag and the root user have other security implications.

## A root container running with the `--privileged` flag

To begin, I’m going to start at the highly privileged end of the scale, looking at a containerized process running as root, with the `--privileged` flag enabled. This is probably the most straightforward case. Running a Dockerfile like the following:

```
FROM ubuntu:latest  
RUN apt update && apt -y -q install libcap2-bin iputils-ping python3 python3-pip  
ENV DEBIAN_FRONTEND=noninteractive  
```

with the `--privileged` flag (`docker run -it --privileged my-root-container bash`) puts us in a container with the following capability sets:

```
root@91d6878472c6:/# grep Cap /proc/$$/task/$$/status  
CapInh:	0000000000000000  
CapPrm:	000001ffffffffff  
CapEff:	000001ffffffffff  https://padlock.argh.in/2024/12/15/container-capabilities.html
CapBnd:	000001ffffffffff  
CapAmb:	0000000000000000  

```

The `capabilities` [man page](https://linux.die.net/man/7/capabilities) explains the algorithm used to calculate the effective permissions set from the various permissions sets above (as well as what the sets mean).

Decoding the _Effective_ set using `capsh` shows a pretty extensive list of permissions:

```
root@91d6878472c6:/# capsh --decode=000001ffffffffff  
0x000001ffffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read,cap_perfmon,cap_bpf,cap_checkpoint_restore  

```

This includes the highly permissioned [CAP_SYS_ADMIN](https://lwn.net/Articles/486306/) capability, but there’s no real surprise here. A simple check using Python shows that we can (using `cap_setuid`), set the process’ UID, for example:

```
root@91d6878472c6:/# python3  
Python 3.12.3 (main, Nov  6 2024, 18:32:19) [GCC 13.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import os  
>>> os.setuid(65534)  
>>> import pwd  
>>> print(pwd.getpwuid(os.getuid()))  
pwd.struct_passwd(pw_name='nobody', pw_passwd='x', pw_uid=65534, pw_gid=65534, pw_gecos='nobody', pw_dir='/nonexistent', pw_shell='/usr/sbin/nologin')  
```

## A root container running without the `--privileged` flag

Dropping the `--privileged` flag, as expected, grants a lower set of default capabilities:

```
root@4a54f2ed2f00:/# grep Cap /proc/$$/task/$$/status  
CapInh:	0000000000000000  
CapPrm:	00000000a80425fb  
CapEff:	00000000a80425fb  
CapBnd:	00000000a80425fb  
CapAmb:	0000000000000000  
root@4a54f2ed2f00:/# capsh --decode=00000000a80425fb  
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap  
```

As with the `--privileged` example, these capabilities are already in the effective set, which means that processes that need the included capabilities will be able to run successfully (although you may need to use `setcap` or similar to set the required file capabilities on the applications you wish to execute – note that the list of effective process capabilities includes `CAP_SETFCAP`).

Due to [the way in which capabilities are calculated](https://linux.die.net/man/7/capabilities), you are absolutely bound by the _Bounding_ set in this situation, so any further capabilities you need must be added via `--cap-add` when executing the container. Adding file-level capabilities that are outside the process’ bounding set and attempting to execute those files will not succeed:

```
root@3c456e81068c:/# getcap /usr/bin/ping  
/usr/bin/ping cap_net_raw=ep  
root@3c456e81068c:/# setcap 'cap_net_raw=ep cap_sys_admin=ep' /usr/bin/ping  
root@3c456e81068c:/# getcap /usr/bin/ping  
/usr/bin/ping cap_net_raw,cap_sys_admin=ep  
root@3c456e81068c:/# ping  
bash: /usr/bin/ping: Operation not permitted  
```

This is all still pretty much as you would expect, although the default list of capabilities provides a relatively significant list of effective permissions that could have security implications depending on your threat model.

## A non-root, non-privileged Docker container

What happens if you then switch away from the root user on a non-privileged container?

My Dockerfile looks like this:

```
FROM ubuntu:latest  
RUN apt update && apt -y -q install libcap2-bin iputils-ping  
ENV DEBIAN_FRONTEND=noninteractive  
USER nobody  
```

Running the container and looking at the capability sets shows us the following:

```
nobody@f1bf44469f4a:/$ grep Cap /proc/$$/task/$$/status  
CapInh:	0000000000000000  
CapPrm:	0000000000000000  
CapEff:	0000000000000000  
CapBnd:	00000000a80425fb  
CapAmb:	0000000000000000  
nobody@8f013c859738:/$ capsh --decode=00000000a80425fb  
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_net_raw,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap  
```

Even though the process’ _Effective_ and _Permitted_ sets are empty, this does not mean that you can’t run processes that require capabilities. For example, `ping` in our Dockerfile installation is again installed with CAP_NET_RAW as a file-level capability:

```
nobody@8f013c859738:/$ getcap /usr/bin/ping  
/usr/bin/ping cap_net_raw=ep  
```

and therefore you can run `ping` without issues:

```
nobody@8f013c859738:/$ ping -c 1 1.1.1.1  
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.  
64 bytes from 1.1.1.1: icmp_seq=1 ttl=63 time=26.4 ms

--- 1.1.1.1 ping statistics ---  
1 packets transmitted, 1 received, 0% packet loss, time 0ms  
rtt min/avg/max/mdev = 26.355/26.355/26.355/0.000 ms  
```

This was initially unintuitive to me (as the _Permitted_ and _Effective_ capability sets are empty for the `bash` process that we examined), but it again comes down to the way in which the _Permitted_ set for a new process is calculated:

```
P'(permitted) = (P(inheritable) & F(inheritable)) | (F(permitted) & cap_bset)  
```

That is, `ping` is permitted to use CAP_NET_RAW because CAP_NET_RAW is both in the bounding set (`CapBnd` above) and the permitted set for the `/usr/bin/ping` file.

If there’s one takeaway from this section, it should be that when using Docker containers, dropping capabilities (potentially at the file level but ideally at the bounding set level), can be quite important in pruning viable attack paths available to an attacker – simply running as non-root and without the `--privileged` flag may not be sufficient, dependent on your threat model.

For example, in a container started with `docker run -it --cap-drop NET_RAW non-root bash`:

```
nobody@f51647bac472:/$ capsh --print  
…  
Bounding set =cap_chown,cap_dac_override,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_net_bind_service,cap_sys_chroot,cap_mknod,cap_audit_write,cap_setfcap  
…  
nobody@f51647bac472:/$ ping  
bash: /usr/bin/ping: Operation not permitted  
```

## A non-root, privileged container (or a non-root container with `--cap-add`)

This was the edge case that initially drove me down this rabbit hole, and the Docker permissions model in this context is curious.

Using the non-root Docker image from the previous example, let’s also use the `--privileged` flag when running the container. My initial expectation would be that the _Permitted_ and _Effective_ capability sets would be the same as in the root container, but that is not the case:

```
nobody@b802f123279c:/$ grep Cap /proc/$$/task/$$/status  
CapInh:	0000000000000000  
CapPrm:	0000000000000000  
CapEff:	0000000000000000  
CapBnd:	000001ffffffffff  
CapAmb:	0000000000000000  
nobody@b802f123279c:/$ capsh --decode=000001ffffffffff  
0x000001ffffffffff=cap_chown,cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_kill,cap_setgid,cap_setuid,cap_setpcap,cap_linux_immutable,cap_net_bind_service,cap_net_broadcast,cap_net_admin,cap_net_raw,cap_ipc_lock,cap_ipc_owner,cap_sys_module,cap_sys_rawio,cap_sys_chroot,cap_sys_ptrace,cap_sys_pacct,cap_sys_admin,cap_sys_boot,cap_sys_nice,cap_sys_resource,cap_sys_time,cap_sys_tty_config,cap_mknod,cap_lease,cap_audit_write,cap_audit_control,cap_setfcap,cap_mac_override,cap_mac_admin,cap_syslog,cap_wake_alarm,cap_block_suspend,cap_audit_read,cap_perfmon,cap_bpf,cap_checkpoint_restore  
```

So the _Bounding_ set gets all the permissions, but the _Permitted_ and _Effective_ sets are completely empty. This effectively means that the capabilities you need have to be defined via `setcap` on the application file you wish to execute, before the container is running. This won’t work, for example:

```
nobody@418e021432ff:/$ python3  
Python 3.12.3 (main, Nov  6 2024, 18:32:19) [GCC 13.2.0] on linux  
Type "help", "copyright", "credits" or "license" for more information.  
>>> import os  
>>> os.setuid(0)  
Traceback (most recent call last):  
  File "<stdin>", line 1, in <module>  
PermissionError: [Errno 1] Operation not permitted  
>>> import prctl  
>>> prctl.cap_effective.setuid = True  
Traceback (most recent call last):  
…  
    return _prctl.set_caps(*_parse_caps(True, *args))  
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  
PermissionError: [Errno 1] Operation not permitted  
>>>  
```

Instead, your Dockerfile needs to looks like this (note the `setcap` step):

```
FROM ubuntu:latest  
RUN apt update && apt -y -q install libcap2-bin iputils-ping python3 python3-prctl  
RUN setcap 'cap_setuid=ep' /usr/bin/python3.12  
ENV DEBIAN_FRONTEND=noninteractive  
USER nobody  
```

And then:

```
nobody@d31b4e5dc664:/$ python3  
…  
>>> import os  
>>> os.setuid(0)  
>>> os.getuid()  
0  
```

It’s non-intuitive to me that using the `--privileged` flag with a non-root user does not immediately grant the user all capabilities. The Docker docs state that “The –privileged flag gives all capabilities to the container.”, which is straightforwardly true for root users but not so much for other users. The `--cap-add` flag works similarly: it only changes the _Bounding_ set rather than the _Effective_ or _Permitted_ sets, so using `setcap` is still required.

I’m not the first person to bump into this, there’s more discussion on the internet:

- [An issue from 2022 on the podman GitHub org](https://github.com/containers/podman/issues/13449), which states that the aim of podman’s (similar) behaviour was Docker compatibility.
- [A similar thread from 2023 on the nomad GitHub repo](https://github.com/hashicorp/nomad/issues/16692), which contains some links to older Docker issues on the matter and a nice table of different permissions settings and the impact they have on specific capabilities.
- [Discussion of the matter on the Docker GitHub repo](https://github.com/moby/moby/issues/8460), which ends on a slightly inconclusive note.

I’m not sure what to make of the security impact other than the behaviour is slightly unintuitive to me and is worth understanding when evaluating a container’s security posture.

## Summary

Running through the various permutations of the root user and the `--privileged` flag was useful for me to understand a lot more about how capabilities work in Linux. In addition, there are a handful of things that stand out to me from this review:

- If you’re installing arbitrary packages into your container from external repos, minimizing the _Bounding_ capabilities set is important, using `--drop-cap`.
- Understanding file-level capabilities is important to fully understand the security posture of your Docker containers.
- Using specific capabilities in a non-root Docker container is slightly more involved than you might initially expect!