---
link: https://grimoire.d12s.fr/2020/smartctl_to_check_ssd_lifetime_and_much_more.html
byline: Simon Descarpentries
site: Grimoire-Command.es
date: 2020-06-02T00:00
excerpt: Contrôle et surveille l’état d’un disque SMART (température, heures de fonctionnement, journaux d’erreurs…) 1. smartctl commands The command generaly comes with the smartmontools package of your GNU+Linux distribution. $ sudo smartctl -H /dev/$device (1) $ sudo smartctl -l error /dev/$device (2) $ sudo smartctl -a /dev/$device …
slurped: 2025-04-20T19:29
title: Smartctl to check ssd lifetime and much more
tags:
  - ssd
  - hardware
  - smartctl
---

[](https://grimoire.d12s.fr/2020/smartctl_to_check_ssd_lifetime_and_much_more.html)

[hardware](https://grimoire.d12s.fr/tag/hardware.html) [storage](https://grimoire.d12s.fr/tag/storage.html) [adminsys](https://grimoire.d12s.fr/tag/adminsys.html)

_Contrôle et surveille l’état d’un disque SMART (température, heures de fonctionnement, journaux d’erreurs…)_

## 1. smartctl commands

The command generaly comes with the `smartmontools` package of your GNU+Linux distribution.

```
$ sudo smartctl -H /dev/$device (1)
$ sudo smartctl -l error /dev/$device (2)
$ sudo smartctl -a /dev/$device (3)
$ sudo smartctl -t short /dev/$device (4)
```

|       |                                                                                            |
| ----- | ------------------------------------------------------------------------------------------ |
| **1** | Display health status ; you may have to add `-d 'scsi'` for such drives, or some USB ones… |
| **2** | Recent errors                                                                              |
| **3** | All info                                                                                   |
| **4** | Start short SMART tests (hint: a long one exists also)                                     |

## 2. Outstanding `smartctl -a /dev/$device` results

The number of hours the device was "on" :

[…]
 9 Power_On_Hours 0x0032  100  100  000 Old_age Always - 8612
[…]

The number of working hours (for disks) :

[…]
 3 Spin_Up_Time 0x0007 222 222 033 Pre-fail Always - 2246
[…]

The remaining lifetime of SSD in term of rewriting capacities :

[…]
 202 Percent_Lifetime_Remain 0x0031 096 096 000 Pre-fail Offline - 4
[…]

Here the SSD consumed 4% of its manufacturer guaranteed lifetime (during 3 years and 9 000 power-on hours).

An SSD can survive 3x (to 10x) its official TBW (TeraByte Written) value, but its warranty wont…