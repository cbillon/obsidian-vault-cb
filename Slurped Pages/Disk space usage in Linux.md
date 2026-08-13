---
link: https://peteris.rocks/blog/disk-space-usage-in-linux/
byline: Pēteris Ņikiforovs
site: peteris.rocks
excerpt: How to check which files and directories take up the most disk space in Linux
slurped: 2026-07-05T11:35
title: Disk space usage in Linux
tags:
  - linux
---

## 

Total free and used disk space

Here is how to check how much disk space is used and how much is free and available:

```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda        24G  5.7G   19G  24% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
devtmpfs        492M  4.0K  492M   1% /dev
none             99M  216K   99M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            494M     0  494M   0% /run/shm
none            100M     0  100M   0% /run/user
```

We are using the `df` program that is part of the `coreutils` package.

If you don't have you can install it with `sudo apt-get install coreutils`.

- The `-h` parameter stands for human readable output and it converts bytes to gigabytes and adds the `G` suffix.
- `df` will also display network file systems, you can see the local file systems with the `-l` or `--local` flag.
- To see the size in SI units (i.e. powers of 1000 not 1024), use the `-H` or `--si` flag.
- Add the `--total` flag for a grand total.
- Add `-T` to see file system types and use the `-x` parameter to hide some file systems types.

Here is a handy command that hides the junk:

```
$ df -h --local -x tmpfs -x devtmpfs
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda        24G  5.7G   19G  24% /
```

## 

Find large files and directories

To list files in a directory and see their size in a human readable form, use the `-h` and `-l` flags:

```
$ ls -lh /boot
total 13M
-rw-r--r-- 1 root root 1.2M Sep  3  2014 abi-3.13.0-36-generic
-rw-r--r-- 1 root root 162K Sep  3  2014 config-3.13.0-36-generic
drwxr-xr-x 2 root root 4.0K Oct 18 14:25 grub
-rw-r--r-- 1 root root 2.7M Oct 18 14:26 initrd.img-3.13.0-36-generic
-rw------- 1 root root 3.3M Sep  3  2014 System.map-3.13.0-36-generic
-rw------- 1 root root 5.6M Sep  3  2014 vmlinuz-3.13.0-36-generic
```

`du` estimates disk space usage for files and directories. It is also a part of `coreutils`.

Here is how much disk space is taken up by `/boot` and its subdirectories:

```
$ du -sh /boot
16M     /boot
```

Here is an explanation of the flags used:

- `-h` stands for human readable output
- `-s` stands for summarize i.e. hide subdirectories

To also see subdirectories, don't use the summarize `-s` flag:

```
$ du -h /boot
2.4M    /boot/grub
16M     /boot
```

If you also want to see files in the output, add the `-a` or `--all` flag:

```
$ du -ah /boot
2.7M    /boot/initrd.img-3.13.0-36-generic
8.0K    /boot/grub/grub.cfg
4.0K    /boot/grub/gfxblacklist.txt
4.0K    /boot/grub/grubenv
2.4M    /boot/grub/unicode.pf2
2.4M    /boot/grub
3.3M    /boot/System.map-3.13.0-36-generic
168K    /boot/config-3.13.0-36-generic
1.2M    /boot/abi-3.13.0-36-generic
5.6M    /boot/vmlinuz-3.13.0-36-generic
16M     /boot
```

To see the disk space usage of a directory and its files and subdirectories but not subsubdirectories, use the following command:

```
$ du -sh /boot/*
1.2M    /boot/abi-3.13.0-36-generic
168K    /boot/config-3.13.0-36-generic
2.4M    /boot/grub
2.7M    /boot/initrd.img-3.13.0-36-generic
3.3M    /boot/System.map-3.13.0-36-generic
5.6M    /boot/vmlinuz-3.13.0-36-generic
```

But you'll notice that the files are not sorted by size. Let's fix that.

```
$ du -sh /boot/* | sort -h
168K    /boot/config-3.13.0-36-generic
1.2M    /boot/abi-3.13.0-36-generic
2.4M    /boot/grub
2.7M    /boot/initrd.img-3.13.0-36-generic
3.3M    /boot/System.map-3.13.0-36-generic
5.6M    /boot/vmlinuz-3.13.0-36-generic
```

The `sort` program understands the human readable suffixes when the `-h` flag is used. You can use the `-r` flag to see the largest files at the top.

To see just the largest three files, you can use the `tail` command:

```
$ du -sh /boot/* | sort -h | tail -n 3
2.7M    /boot/initrd.img-3.13.0-36-generic
3.3M    /boot/System.map-3.13.0-36-generic
5.6M    /boot/vmlinuz-3.13.0-36-generic
```

If you try `du -sh /` you'll get a lot of `Permission denied` errors. The solution is to run it as root.

```
$ sudo du -sh /
du: cannot access ‘/proc/20643/task/20643/fd/3’: No such file or directory
du: cannot access ‘/proc/20643/task/20643/fdinfo/3’: No such file or directory
du: cannot access ‘/proc/20643/fd/4’: No such file or directory
du: cannot access ‘/proc/20643/fdinfo/4’: No such file or directory
5.7G    /
```

`/proc` is a file system that lets you access kernel structures. These are not real files, don't take up any physical space on your hard drive and may exist in one millisecond and be gone in another millisecond. You can just hide them from the output:

```
$ sudo du -sh / 2> /dev/null
5.7G    /
```

### 

ncdu

There is a very nice ncurses interface for `du` which lets you interactively explore the disk space usage. The name of the program is `ncdu`.

On Ubuntu Linux, install it like this:

```
sudo apt-get install ncdu
```

Specify the directory you want to analyze as the first parameter:

```
$ ncdu /usr
```

![ncdu](https://peteris.rocks/blog/disk-space-usage-in-linux/ncdu.png)

## 

Summary

- `df -h`
- `ls -lh /usr`
- `du -sh /usr/*`
- `ncdu /usr`

Top 10 largest files and directories on your computer:

```
sudo du -ah / 2> /dev/null | sort -h | tail -n 10
```

Top 10 largest files on your computer:

```
(sudo find / -type f | xargs sudo du -h | sort -h | tail -n 10) 2> /dev/null
```