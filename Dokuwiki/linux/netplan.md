---
tags:
  - linux
  - network
  - netplan
---

====== Netplan ======

[[https://linuxconfig.org/netplan-network-configuration-tutorial-for-beginners]]

Netplan is an utility developed by Canonical, the company behind Ubuntu. It provides a network configuration abstraction over the currently supported two “backend” system, (or “renderer” in Netplan terminology): networkd and NetworkManager. Using Netplan, both physical and virtual network interfaces are configured via yaml files which are translated to configurations compatible with the selected backend.

On Ubuntu 20.04 Netplan replaces the traditional method of configuring network interfaces using the /etc/network/interfaces file; it aims to make things easier and more centralized (the old way of configuring interfaces can still be used: check our article about How to switch back networking to /etc/network/interfaces on Ubuntu 20.04 Focal Fossa Linux). In this article we will learn the basic principles behind the utility, and, just as an example, how we can use it to configure a static IPv4 address for a network interface.

In this tutorial you will learn:

  * The basic structure of yaml configuration files used by Netplan
  * How to create a simple rule to assign a static IP to a network interface
  * How to apply configurations using generate, try and apply subcommands

Netplan configuration files
There are three locations in which Netplan configuration files can be placed; in order of priority they are:

  * /run/netplan
  * /etc/netplan
  * /lib/netplan

{{ :netplan_design_overview.svg |}}

Inside each of these directories configurations are created using files with the .yaml extension which are processed in lexicographical order, regardless of the directory they are in.

Directory priority has a role only when files with the same name exist: in those cases, only the file contained in the directory with the higher priority is parsed.

If a boolean or scalar parameter is defined in more than one configuration file, it will assume the value defined in the last file that is parsed; if the values are sequences, instead, they are concatenated.

Users are supposed to place their configurations inside the /etc/netplan directory; by default the only file present on a fresh installed Ubuntu 20.04 system is /etc/netplan/01-network-manager-all.yaml. In the next section we will see the instructions it contains, and what is their meaning.

====== The /etc/netplan/01-network-manage-all.yaml file ======

The only configuration file existing /etc/netplan/ directory on a fresh installed Ubuntu 20.04 system is 01-network-manage-all.yaml. Let’s take a look at its content:

# Let NetworkManager manage all devices on this system
  network:
    version: 2
    renderer: networkd
    
As suggested by the comment in the file, the configuration is meant to set all the network interfaces on the system to be managed by the NetworkManager renderer. We can observe that directives are indented inside the main node, network. Since we are dealing with yaml files, the indentation is crucial.

Another two keywords we can find in the file are version and renderer: the former specifies the syntax version in use, the latter the system backend (**networkd** vs** NetworkManager**).

[[https://askubuntu.com/questions/1031439/am-i-running-networkmanager-or-networkd]]


In the next section of this tutorial we will create a slightly more complex configuration example, and we will use it to assign a static IPv4 address to a network interface.

====== A Configuration example – setting a static IPv4 address ======

The configuration file we saw above is quite basic; let’s try something a little bit more complex and see how can we configure a static IPv4 address using Netplan.

The first thing we must to do is to create a new configuration file, to be parsed after the default one: let’s call it /etc/netplan/02-static-ip.yaml. Inside the file, we create a rule to match the network interface(s) we want to setup: we can accomplish the task by using the match stanza.

Inside the match section, we can select a series of physical interfaces on the base of the value of the specified properties. For the settings to be applied all the properties must be matched by the rule.

In the configuration file we write:

# Set static ip address for enp1s0 interface
  
  network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: false
      dhcp6: false
     addresses:
      - 192.168.10.10/24
     routes:
      - to: default
        via: 192.168.10.1
     nameservers:
       addresses: [192.168.10.1]
            
Let’s take a closer look to the new instructions we used in the configuration. Inside the main network node, devices can be grouped by their type:

  * ethernets
  * wifis
  * bridges

Since in our example we our dealing with an ethernet device we used ethernets stanza. Inside the match stanza, we referenced the interface by its name: enp1s0. Match rules can also be based on macaddress and, only when using networkd as the renderer, on driver which is the Linux kernel driver name used for the device(s).

To reach our desired configuration, we used a series of directives. Since we want to assign a static address, we disabled dhcp4 and used the addresses keyword to associate an IPv4 address to the interface. Multiple address can be specified: they must be provided together with the subnet mask.

We also set the addresses of the nameservers in the stanza with the same name. Finally, we set the IPv4 address of the gateway the interface should use with the gateway4 keyword.

nota : **gateway4 deprecated use route instead**

====== Simplifying the configuration ======

The configuration we used in the example above can be slightly simplified. To reference the interface we want to assign the static address to we used the match stanza, however, we could have omitted it. Since we want our settings to be applied to just one specific device, we can reference it directly using its predictable name (enp1s0) as id:

  network:
    version: 2
    renderer: networkd
    ethernets:
        enp1s0:
            dhcp4: false
            addresses:
                - 192.168.122.250/24
            nameservers:
                addresses:
                    - 192.168.122.1
            routes:
               - to: default
               via: 192.168.10.1
               
When the match stanza is used, the id (id0 in the previous example) is arbitrary and its used to reference the configured device(s) from other sections of the configuration file. When the match stanza is omitted, instead, the id must correspond to the device predictable name. When working with virtual devices such as bridges or bonds, the id is not used to reference an existing interface, but represents the name that should be used when the interface is created.

At this point our configuration is ready; all we should do is to save it and test it.

====== Testing and applying a Netplan configuration ======

In the previous section we saw how to create a simple Netplan configuration to provide a static IPv4 address for a network interface. Now it’s time to test the configuration, to see if it works correctly. To achieve our goal we can use the netplan utility and the try subcommand.

The try subcommand of the netplan utility, as its name suggests, is used to try a configuration, and optionally roll it back if the user doesn’t confirm it after a certain amount of time. The default timeout is of 120 seconds but it can be changed using the --timeout option.

As you can see from the output of the ip address command, the current IPv4 address for the enp1s0 interface is 192.168.122.200:

  $ ip address|grep enp1s0
  2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.122.200/24 brd 192.168.122.255 scope global dynamic noprefixroute enp1s0

Let’s apply the configuration:

$ sudo netplan try
Once we run the command, the following prompt appears on screen:

Do you want to keep these settings?


Press ENTER before the timeout to accept the new configuration


Changes will revert in 120 seconds
We have enough time to very if the IP address of the interface changed:

$ ip address|grep enp1s0
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    inet 192.168.122.250/24 brd 192.168.122.255 scope global dynamic noprefixroute enp1s0

As we can see, the IPv4 address changed as expected. In this case, however after the timeout expired, the command failed to revert the configuration. This is a a known problem, reported also in the manpage of the utility. In such cases, to fully revert to the initial state, a reboot should be enough.

Two other commands can be used:

  netplan generate
  
  netplan apply

The netplan generate command converts the settings in the yaml files to configurations appropriate to the renderer in use, but does not apply them. In the vast majority of cases it is not meant to be called directly: it is invoked, for example, by netplan apply which additionally applies the changes without a “revert” timeout.

====== Conclusions ======

In this tutorial we approached Netplan, a utility developed by Canonical, which is active by default on Ubuntu 20.04 Focal Fossa. The purpose of this utility is to abstract configurations for network interfaces using a yaml configuration files.

Those configurations are then translated into configurations for the specified renderer, such as NetworkManager or networkd. In this tutorial we saw how to write a simple rule to set a static IP address for a network interface, we learned some of the nodes that can be used in configuration files, and we saw how to apply changes via the netplan try and netplan apply commands. Here we barely scratched the surface of what can be accomplished using Netplan if you want to know more about it, please take a look at the [[https://netplan.io|NetPlan]], and at the utility manpage.