---
tags:
  - linux
  - network
  - configuration
---

====== Network Configuration ======

To see your local IP address, you can run the following command in terminal:
  
  $ ip a

Locate the requested network interface and check for the assigned IP address. Additionally, the above command also reveals the network interface hardware address (also known as the MAC address).

There are also various ways to get your public IP address. 
You can go to a website like ip chicken or execute one of the following commands in terminal:

  # echo $(wget -qO - https://api.ipify.org)
  OR
  # echo $(curl -s https://api.ipify.org)
  
To check for currently used DNS server IP address, execute this command:

  $ systemd-resolve --status | grep Current

To display default gateway IP address, run this command:

  $ ip r
  
===== How to set static IP address=====
You can configure a static IP address on Ubuntu 22.04 Jammy Jellyfish either from command or GUI. First, we will cover the instructions to configure one from GNOME GUI.
 
Locate and edit with administrative privileges the /etc/netplan/50-cloud-init.yaml file with the following configuration. Update your desired static IP address, DNS server and gateway where appropriate. Save and exit the file after you have applied your changes.
  network:
    ethernets:
        enp0s3:
            dhcp4: false
            addresses: [192.168.1.202/24]
            gateway4: 192.168.1.1
            nameservers:
              addresses: [8.8.8.8,8.8.4.4,192.168.1.1]
    version: 2

To apply the new Netplan changes execute:

  $ sudo netplan apply

Alternatively, if you run into some issues run:

  $ sudo netplan --debug apply

Confirm your new static IP address by using the ip a command:
  
  $ ip a

