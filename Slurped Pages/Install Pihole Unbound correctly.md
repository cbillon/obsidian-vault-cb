---
link: https://secure-bits.org/en/posts/privacy/pihole/pihole-unbound/
byline: |-
  Marcel DellmannThursday, October 12, 2023 |
  9 minutes
excerpt: Install Pihole Unbound correctly | In this tutorial, I will show you how to install and configure Unbound correctly for Pihole.
slurped: 2026-06-04T12:12
title: Install Pihole Unbound correctly
tags:
  - pi-hole
  - unbbound
---

## Pihole Unbound and DNS encryption - For more network security

In today’s video tutorial, ‘**Pi-hole Unbound**’, I would like to show you how you can use the additional tool ‘**Unbound**’ to ensure greater security for your DNS queries on the network. You will also find all the commands you need to install and configure Unbound alongside Pi-hole. Table of contents for Pihole Unbound

### Requirements for installing Pihole Unbound

> ### Note
> 
> These instructions for installing and setting up **Unbound** are for **Debian-based distributions such as Debian, Ubuntu, Raspberry Pi OS, etc.**.
> 
> This tutorial refers to the previous tutorials on ‘Installing Pi-Hole on a Raspberry Pi’ and uses this configuration as the basis for this article. In this tutorial, I will start from the basics that you will have learned after completing the tutorial ‘**[Installing Pi-Hole + Firewall on a Raspberry Pi](https://secure-bits.org/en/posts/privacy/pihole/pihole-auf-einen-raspberry-pi-installieren/)**’ on your Raspberry Pi. If this is not the case, please feel free to go through the installation tutorial first. Of course, the installation of **Unbound** should also work on other Linux-based systems!

## Video tutorial on ‘Installing Pihole Unbound’

## Installing Unbound

First, we always need to update the operating system to the latest version.

```
sudo apt update && apt upgrade -y
```

After the update, the system should be restarted.

### Pihole Unbound | Retrieving the root.hints

Now you can install Unbound and the list of current root.hints (root name servers, which are updated every 6 months).

```
# Installation of Unbound
sudo apt install unbound
# Installing the list of root name servers (should be done every 6 months)
sudo wget -O /etc/unbound/root.hints https://www.internic.net/domain/named.root
sudo wget -O /var/lib/unbound/root.hints https://www.internic.net/domain/named.root
```

### Automatically update the list of root name servers (root.hints) | Pi-hole Unbound

To automatically update the root.hints list every 6 months, we create a cron job that automates this task for us. To set up this automation, we proceed as follows.

#### Create an update file with all commands

First, we create a file in which we put all the commands for updating root.hints.

```
# Create a command file for updating the root.hints
nano /root/roothintsupdate
# In the editor window of this file, enter the following commands:
#!/bin/bash
wget -O root.hints https://www.internic.net/domain/named.root ;
 
rm /var/lib/unbound/root.hints
mv root.hints /var/lib/unbound/
service unbound restart
```

#### Assign permissions to execute the created file

To ensure that the created update file can be executed, we must assign the correct permissions here.

```
# Assign permissions for the update file
chmod +x /root/roothintsupdate
```

#### Create a cron job for the automatic update of root.hints

To execute our update file automatically every six months, we create a cron job in which we add the cron job configuration according to our requirements.

```
# Open the cron job configuration
crontab -e
# Enter the following line in the lower free area:
0 0 1 */6 * /root/roothintsupdate &
# Save the file with CTRL + O and Enter and exit the editor window with CTRL + X
```

#### Restart the cron job service

Finally, we need to restart the ‘Cronjob’ service and then check that the service is running properly.

```
# Restart the cronjob service
service cronjob restart
# Check the status of the cronjob service
service cronjob status
# Output of the command (piholetest is the name of my test computer):
* cron.service - Regular background program processing daemon
     
Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2024-01-19 06:57:01 UTC; 15s ago
       Docs: man:cron(8)
   Main PID: 3866 (cron)
      Tasks: 1 (limit: 38127)
     Memory: 340.0K
CPU: 2ms
CGroup: /system.slice/cron.service
`-3866 /usr/sbin/cron -f
Jan 19 06:57:01 piholetest systemd[1]: Started Regular background program processing daemon.
Jan 19 06:57:01 piholetest cron[3866]: (CRON) INFO (pidfile fd = 3)
Jan 19 06:57:01 piholetest cron[3866]: (CRON) INFO (Skipping @reboot jobs -- not system startup)
```

### Pihole Unbound - Create Unbound configuration file

Next, we need to create a configuration file for Unbound, in which we will then enter the configuration for the interaction between Unbound and Pi-hole in the next step.

```
# Create Unbound config file
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf
# In this file, we enter the configuration described in the next paragraph
# and save the config file with CTRL+O and exit the editor with CTRL+x
```

### Contents of the Pihole Unbound configuration file

Here you will find the contents of the configuration file that you must enter for Pi-hole and Unbound to work. I have made some adjustments to this configuration file so that external DNS queries are encrypted and configured some other privacy settings. Descriptions of the individual Unbound configurations are always provided as comments above the instructions in the Unbound config file.

```
# Unbound config file for Pi-hole
server:
chroot: “”
# Do not display Unbound version information - security aspect
server:
hide-identity: yes
hide-version: yes
# If no log file is specified, syslog is used.
logfile: “/var/log/unbound.log”
verbosity: 0
log-time-ascii: yes
log-queries: no
# Specify port for Pi-hole Custom DNS 1:
port: 5335
do-ip4: yes
do-udp: yes
do-tcp: yes
# Can be set to yes if you have IPv6 connectivity
do-ip6: no
# Only use this if you have downloaded the list of primary root servers!
root-hints: “/var/lib/unbound/root.hints”
tls-cert-bundle: “/etc/ssl/certs/ca-certificates.crt”
# Only trust the glue if it is within the authority of the server
harden-glue: yes
# Required DNSSEC data for trusted zones; if this data is missing, the zone will be set to BOGUS
harden-dnssec-stripped: yes
# Do not use uppercase randomization, as it is known to sometimes cause DNSSEC issues
# See https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for more details
use-caps-for-id: no
# Reduce the size of the EDNS composition buffer.
# IP fragmentation is unreliable on the Internet today and can 
# cause transmission errors when large DNS messages are sent over UDP. Even
# if fragmentation works, it may not be secure.
# It is theoretically possible to forge parts of a fragmented DNS message without
# the recipient noticing. There was a recent excellent study
# >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
# by Axel Koolhaas and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
# in collaboration with NLnet Labs examined DNS using real data from the
# RIPE Atlas Probes, and the researchers proposed different values for
# IPv4 and IPv6 and in different scenarios. They recommend that servers be configured
# to limit DNS messages sent via UDP to a size that does not trigger
# fragmentation on typical network connections. DNS servers can
# switch from UDP to TCP if a DNS response is too large to fit in this limited
# buffer size. This value was also proposed on DNS Flag Day 2020.
edns-buffer-size: 1232
# TTL limits for the cache
cache-min-ttl: 3600
cache-max-ttl: 86400
# Perform prefetching of message cache entries that are about to expire
# This only applies to domains that have been queried frequently
prefetch: yes
prefetch-key: yes
# One thread should be sufficient, but can be increased on powerful machines.
num-threads: 1
# Number of slabs in the RRset cache. Slabs reduce lock conflicts between threads.
# Must be set to a power of 2 close to num-threads.
msg-cache-slabs: 2
rrset-cache-slabs: 2
infra-cache-slabs: 2
key-cache-slabs: 2
# Cache memory rrset should be twice as large as msg
msg-cache-size: 50m
rrset-cache-size: 100m
# more outgoing connections
# depends on the number of cores: 1024/cores - 50
outgoing-range: 450
# Speed up UDP with multithreading
so-reuseport: yes
# Ensure that the kernel buffer is large enough to avoid losing messages during traffic spikes 
# (not used with AppArmor enabled)
# so-rcvbuf: 1m
# Ensure privacy for local IP ranges
private-address: 192.168.0.0/16
private-address: 169.254.0.0/16
private-address: 172.16.0.0/12
private-address: 10.0.0.0/8
private-address: fd00::/8
private-address: fe80::/10
# !!! Please adjust the IP address range for your system accordingly !!!
# access-control: 192.168.0.0/16 allow
 
# For me, it is this IP address range
access-control: 172.16.0.0/16 allow
# A list of upstream servers that support TLS requests (DoH) and encrypt requests
forward-zone:
name: “.”
forward-tls-upstream: yes
forward-addr: 9.9.9.11@853#dns11.quad9.net
forward-addr: 176.9.93.198@853#dnsforge.de
forward-addr: 94.140.14.140@853#unfiltered.adguard-dns.com
```

### Creating the configuration file for EDNS

We create the configuration file for determining the maximum ‘edns-buffer-size’ with the following command.

```
# Creating the EDNS configuration file
sudo nano /etc/dnsmasq.d/99-edns.conf
# And enter the following parameter here
edns-packet-max=1232
# Save the config file with CTRL+O and CTRL+X
```

### Restart Unbound

Once the above steps have been completed, the Unbound service must be restarted.

```
# Restart Unbound service
sudo systemctl restart unbound.service
# Check the status of the Unbound service
sudo systemctl status unbound.service
# The output should look like this (piholetest2 only stands for the system I used in this tutorial and will be replaced with the name of your Pi-hole instance):
# root@piholetest2:~# sudo systemctl status unbound.service
* unbound.service - Unbound DNS server
     Loaded: loaded (/lib/systemd/system/unbound.service; enabled; vendor preset: enabled)
     
Active: active (running) since Fri 2023-10-06 08:29:59 UTC; 1min 9s ago
       Docs: man:unbound(8)
    Process: 3190 ExecStartPre=/usr/lib/unbound/package-helper chroot_setup (code=exited, status=0/SUCCESS)
    
Process: 3193 ExecStartPre=/usr/lib/unbound/package-helper root_trust_anchor_update (code=exited, status=0/SUCCESS)
   Main PID: 3196 (unbound)
Tasks: 2 (limit: 38124)
     
Memory: 4.8M
        CPU: 92ms
CGroup: /system.slice/unbound.service
`-3196 /usr/sbin/unbound -d -p
Oct 06 08:29:59 piholetest2 systemd[1]: Starting Unbound DNS server...
Oct 06 08:29:59 piholetest2 unbound[3196]: Oct 06 08:29:59 unbound[3196:0] error: Could not open logfile /var/log/unbound.>
Oct 06 08:29:59 piholetest2 systemd[1]: Started Unbound DNS server.
Oct 06 08:29:59 piholetest2 unbound[3196]: Oct 06 08:29:59 unbound[3196:0] info: start of service (unbound 1.13.1).
lines 1-17/17 (END)
```

#### Enable automatic start of Unbound service

To ensure that Unbound starts again after a restart, we need to execute the following command:

```
#Automatic start of Unbound after a restart
sudo systemctl enable unbound.service
```

### Test Unbound functionality

After configuring Unbound, we need to test whether Unbound responds correctly to DNS queries. We do this with the following command:

```
# Test the function of Unbound using my domain as an example. Can also be replaced with google.com, for example
dig secure-bits.org @127.0.0.1 -p 5335
```

The output should then look like the image below. The ‘Status’ must be ‘NOERROR’ and, of course, the resolved IP address of the requested domain must also be displayed. ![Testing pi-hole unbound](app://obsidian.md/images/posts/pihole/Unbound-Pi-hole-testen.png)

### Disabling the default DNS cache daemon

Since Unbound now handles the caching of DNS requests, we will disable the default cache resolver ‘systemd-resolved’.

```
sudo systemctl disable systemd-resolved
```

### Create log file for Unbound

Create the unbound.log file so that Unbound can write its log here, as specified in the config file.

```
# Create log file for Unbound
sudo touch /var/log/unbound.log
# Assign owner/permissions for the unbound.log file
sudo chown unbound:unbound /var/log/unbound.log
# Restart Unbound
sudo systemctl restart unbound.service
```

### Setting up Pihole Unbound

Last but not least, we need to configure Pihole to use Unbound as its upstream DNS server. To do this, log in to Pi-Hole, go to ‘**Settings->DNS**’ and enter the following entry under ‘**Custom 1 (IPv4)**’: ‘**127.0.0.1#5335**‘. On the left side, under ’**Upstream DNS Servers**‘, uncheck the boxes for the upstream DNS servers you have been using, as they are no longer needed. **Important!** - Once all entries have been adjusted, please scroll down and click ’**SAVE**’ to save the changes.

![Setting up Pi-hole Unbound](app://obsidian.md/images/posts/pihole/Pi-hole-Unbound-einrichten.png)

#### Links on the topic of Unbound

Developers behind Unbound and background information on the recursive DNS server

- [https://nlnetlabs.nl/projects/unbound/about/](https://nlnetlabs.nl/projects/unbound/about/)
    
- Documentation on Unbound - [https://unbound.docs.nlnetlabs.nl/en/latest/](https://unbound.docs.nlnetlabs.nl/en/latest/)
    
- Documentation from Pi-hole.net on the topic of Unbound [https://docs.pi-hole.net/guides/dns/unbound/](https://docs.pi-hole.net/guides/dns/unbound/)
    
- Overview image of DNS query via a root name server [https://upload.wikimedia.org/wikipedia/commons/3/3c/DNS-query-to-wikipedia.svg](https://upload.wikimedia.org/wikipedia/commons/3/3c/DNS-query-to-wikipedia.svg)
    

> ## Questions and answers
> 
> If you have any questions or feedback about this blog or YouTube channel, please use my help page. Here you will find questions and answers about the various posts. You are also welcome to ask your own question here.
> 
> [**Visit the Help-page**](https://help.secure-bits.org/help)