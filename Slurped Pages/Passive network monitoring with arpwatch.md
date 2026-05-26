---
link: https://cylab.be/blog/505/passive-network-monitoring-with-arpwatch
excerpt: arpwatch is a lightweight network monitoring tool used to passively observe ARP (Address Resolution Protocol) packets on a local network. It was developed to to track the mapping between IP and MAC address. A change in this mapping is generally an indicator of a MAC spoofing or arp cache poisoning attack. In this situation, arpwatch can send an email alert to administrators.
slurped: 2026-05-19T08:15
title: Passive network monitoring with arpwatch
tags:
  - monitoring
  - network
---

`arpwatch` is a lightweight network monitoring tool used to passively observe ARP (Address Resolution Protocol) packets on a local network. It was developed to to track the mapping between IP and MAC address. A change in this mapping is generally an indicator of a MAC spoofing or arp cache poisoning attack. In this situation, `arpwatch` can send an email alert to administrators.

The tool can also be used to perform passive network monitoring. It allows to list devices connected on the local network, like `nmap`, but without sending a single packet, only listening on received arp packets…

[!
## Installation[](https://cylab.be/blog/505/passive-network-monitoring-with-arpwatch#installation)

Arpwatch is available in most distributions, so installation is easy:

```
sudo apt install arpwatch
```

However there is however a catch: **`arpwatch` must be manually enabled for each network interface**:

```
sudo systemctl enable arpwatch@<interface>
```

For example:

```
sudo systemctl enable arpwatch@enp0s1
```

Then (re)start the daemon:

```
sudo systemctl restart arpwatch.service
```

## Monitor network devices[](https://cylab.be/blog/505/passive-network-monitoring-with-arpwatch#monitor-network-devices)

After a few seconds, `arpwatch` will populate the file `/var/lib/arpwatch/<interface>.dat` with discovered devices.

For example:

```
sudo cat /var/lib/arpwatch/enp0s1.dat

2c:fa:a2:b3:84:0b	10.67.42.2	1777470939		enp0s1
48:45:e6:b2:79:7b	10.67.42.194	1777470939		enp0s1
2c:fa:a2:a5:c0:70	10.67.42.5	1777381241		enp0s1
e4:e7:49:5b:dc:c9	10.67.42.186	1777462917		enp0s1
```

In this file, the 3rd column is the timestamp.
