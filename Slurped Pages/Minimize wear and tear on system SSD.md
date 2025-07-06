---
link: https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/
byline: Nalle
site: my Home Lab journey
date: 2025-05-14T16:58
excerpt: Using consumer grade SSD or NVMe may be a short time solution. This post is about what you can do to avoid early death of your system disk. I have always had stand-alone nodes alongside the cluster nodes. And I also have a mini PC with Proxmox for travelling purposes.
twitter: https://twitter.com/@myhomelab
slurped: 2025-07-01T13:16
title: Minimize wear and tear on system SSD
tags:
  - Proxmox
  - ssd
  - log
---

[Proxmox](https://homelab.casaursus.net/tag/proxmox/)

Using consumer grade SSD or NVMe may be a short time solution. This post is about what you can do to avoid early death of your system disk. I have always had stand-alone nodes alongside the cluster nodes. And I also have a mini PC with Proxmox for travelling purposes.

## Basics

What is eating up consumer grade SSDs is the amount of writing going on, logs and cluster stuff, replication and HA.

Using a mirrored disk pair may or may not be a solution. What is clear is that you wear two disks out, not one. I prefer to run XFS or ZFS on all my stand-alone nodes, regardless of what system they run, Proxmox, TrueNAS, Ubuntu, Pop!_OS ...

- Do I use mirrored pairs?
    - Hardly ever. I believe in backups and the 3-2-1.
- Is RAID a substitute or alias of backups?
    - Absolutely not. If you can't afford a backup solution, do **not** enter into virtualization on running servers.
- Do I run desktops on servers for convenience?
    - Absolutely not. A have a minimal Alpine desktop VM on my main network cluster for that
- Do I use PBS (Proxmox Backup Server)?
    - Absolutely! It is one of the reasons Proxmox is so great
    - I run PBS on my main TrueNAS rig (VM and LXC under Incus), Franken PVE&BS and remotely as vanilla PBS
- Is PBS the only backup solution?
    - No I have several other solutions
- Can PBS backup other systems?
    - Yes, see the documentation. Linux and Windows are well documented.

## My script

I have a script on my GitHub to do the essential things. Read the [script](https://github.com/nallej/MyJourney/raw/main/scripts/MinSSDwear.sh)

### Download the script

```
wget https://github.com/nallej/MyJourney/raw/main/scripts/MinSSDwear.sh
```

## The Bootloader

Proxmox VE currently uses one of two bootloaders, depending on the disk setup selected in the installer.

For **EFI** Systems installed with **ZFS** as the root file system, systemd-boot is used, unless Secure Boot is enabled. All other deployments use the standard **GRUB** bootloader, this usually also applies to systems which are installed on top of Debian.

## RAID Controllers

There is a wiki on how to use your RAID card the correct way, see below.

The paramount feature for real raid controller if you want high performance is an onboard cache and "**write back**" mode enabled. This way, the OS doesn't have to wait until data is physically written to disk, since they are immediately written to the cache and the controller will take care of finishing the subsequent write to the disk(s). If you don't want to lose your data in case of black-out, you need a battery backup (BBU).

A RAID controller using write-through instead of write-back behaves very poorly. You can use the command **pveperf** to have a general idea of performance. Look at the FSYNCS/SECOND value.

### The modern ZFS system

Install your RAID Card in IT or HBA mode. See more at this [site](https://hwraid.le-vert.net/). Also read the Proxmox Documentation.

---

---

## References

Proxmox Documentation Index [[1]](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fn1), PVE [[2]](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fn2), Wiki [[3]](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fn3), HowTo [[4]](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fn4), Backup [[5]](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fn5), PBS [[6]](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fn6), HBA Raid Cards[[7]](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fn7)

---

1. Proxmox Documentation [Index](https://pve.proxmox.com/pve-docs/index.html) [↩︎](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fnref1)
    
2. Proxmox Virtual Environment (PVE) Admin [Guide](https://pve.proxmox.com/pve-docs/pve-admin-guide.html) [↩︎](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fnref2)
    
3. Proxmox guides [wiki](https://pve.proxmox.com/wiki/Main_Page) [↩︎](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fnref3)
    
4. Proxmox HowTo [guides](https://pve.proxmox.com/wiki/Category:HOWTO) [↩︎](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fnref4)
    
5. HowTo Backup and Restore [Wiki](https://pve.proxmox.com/wiki/Backup_and_Restore), PVE Backup & Restore [Documenttion](https://pve.proxmox.com/pve-docs/chapter-vzdump.html), Tape Driwe [wiki](https://pve.proxmox.com/wiki/Tape_Drives) [↩︎](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fnref5)
    
6. Proxmox Backup Server (PBS) [Documentation](https://pbs.proxmox.com/docs/), [Download](https://www.proxmox.com/en/downloads/category/proxmox-backup-server), PBS Tape Drive [Documentation](https://pbs.proxmox.com/docs/tape-backup.html) [↩︎](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fnref6)
    
7. RAID Cards [Wiki](https://pve.proxmox.com/wiki/Raid_controller) [↩︎](https://homelab.casaursus.net/minimize-wear-and-tear-on-system-ssd/#fnref7)