---
tags:
  - linux
  - network
---

====== Am I running NetworkManager or networkd? ======

As near as I can tell, there have been 3 approaches to networks in Linux:

1) The oldest uses the /etc/network/interfaces file and ifup/ifdown scripts to manage these interfaces.

2) After that came the network-manager daemon (often written Network-Manager) which has GUI interfaces available.

3) And most recently the systemd-networkd daemon (sometimes abbreviated just 'networkd') which is based on systemd unit files.

To see how your network is being managed, first you must know if you're system is initializing with systemd or the older init as it's first process. (Debian and Ubuntu, for example now use systemd instead of init).

You can check if your system uses systemd with this:

  sudo pidof systemd     &&  echo "systemd"  || echo "other"
  
So if you are NOT running systemd, then clearly you can rule out systemd-networkd.

If you ARE running systemd, then check which network service daemons are running with these two commands:

  sudo service systemd-networkd status
  sudo service network-manager  status   # before Ubuntu V21.10
  sudo service NetworkManager   status   # after  Ubuntu V21.10!

You'll see either Active: active (running) or Active: inactive (dead) reported for each one.

Note that you can also run these newer commands, but obviously if you don't have systemd, they won't work for you:

  systemctl status systemd-networkd
  systemctl status network-manager   # before Ubuntu V21.10
  systemctl status NetworkManager    # after  Ubuntu V21.10!

But you're not done yet...

Finally, even if one of these two daemons are running, that doesn't mean your network hardware interfaces are being managed by them, as there are exceptions.

First any interfaces defined in /etc/network/interfaces are ignored by the network-manager package. (man 5 NetworkManager)

Next, systemd-networkd will only manage network addresses and routes for any link for which it finds a .network file with an appropriate [Match] section. (man 8 systemd-networkd).


