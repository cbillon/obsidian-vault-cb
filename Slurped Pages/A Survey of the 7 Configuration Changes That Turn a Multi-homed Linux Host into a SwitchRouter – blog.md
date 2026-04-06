---
link: https://patrickmccanna.net/7-configuration-changes-that-turn-a-multi-homed-host-into-a-switch-router/
excerpt: This was written on March 1, 2026
slurped: 2026-04-04T09:53
title: A Survey of the 7 Configuration Changes That Turn a Multi-homed Linux Host into a Switch/Router – blog
tags:
---

This was written on March 1, 2026

## What does it mean to turn a Linux system into networking infrastructure?

I think it is incredibly cool that we can change a Linux system into a networking device. But have you ever wondered:

What are we changing when we turn a Linux system into a router or switch? What are we changing if we make a raspberry pi into a WiFi access point? How significant is the system performance monitoring change? What are the gates we have to change to enable packet forwarding and processing?

I’m going to start out with a narrative explanation of the changes that turn a Linux system into a WiFi access point and then I’ll show the commands for implementing it.

I have a cognitive bias: I think of networking devices and computers as different things. This is because the command line experience on networking gear is different than what you experience on servers/hosts. On servers and workstations: you tend to focus a lot on objects on the file system. On networking gear, you’re spending most of your time working with running processes directly. Commands and interaction objectives on networking gear is very different than those on hosts.

I suspect a lot of other people who have worked in networking have similar feelings about networking appliances versus host operating systems. This might be specific to my journey. But for better or worse, I felt that networking was different than general computing. It isn’t. If you know networking, you can make Linux do networking things if you make 7 changes.

1. Activating IP Forwarding
2. Defining The Bridge
3. Activating nftables policies
4. Stateful Firewalling with conntrack
5. Defining NAT and Masquerade policies
6. Vending DHCP and DNS with dnsmasq
7. Vending WiFi networks with hostapd

To activate packet processing and forwarding in the Linux Kernel, you start by changing the Kernel’s configuration for networking. Every Android device that vends a personal WiFi hotspot makes the same general changes.

## A packet’s journey through the kernel

Let’s assume we have a Linux machine with a single network interfaces. A packet arrives on the externally facing interface. The Network Interface Card (**NIC**) signals an interrupt and the driver pulls the frame into a ring buffer in kernel memory via Direct Memory Access (**DMA**), where the hardware writes data into RAM without Central Processing Unit (**CPU**) involvement. The kernel’s networking stack picks the frame up from there, strips the Ethernet header, and examines the Internet Protocol (**IP**) destination address.

At that point the kernel consults its routing table. If the destination address matches one of the machine’s own interfaces, the packet travels up through the network stack to a listening socket, to a process waiting to handle it. If the destination address matches no local interface and IP forwarding is disabled, the kernel drops the packet and increments a counter in `/proc/net/snmp`.

The default behavior of Linux is the end of the line for a packet: the kernel cannot forward the packet to another host. We need to make changes to the system if we want to enable routing. We also need another nic to send across network interfaces. A workstation is a host, not a router.

Now imagine that same system with two NICs (aka dual-homed)- how do we get closer to routing packets?

A router’s role is to forward the packets our single-homed host drops by default. Let’s explore each of the steps that move the kernel from a workstation’s conservative posture as a host into a router that routes packets, modifies packet headers, and filters traffic between interfaces.

---

## What is a hook?

In the Linux kernel, a hook is a designated interception point in a code path where external functions can register themselves to execute. Think of it as a slot in an assembly line: the main process pauses at predefined points and runs every function that has registered at that slot, in priority order. Each registered function can inspect, modify, accept, or drop the item passing through. Hooks let the kernel separate its core packet-processing logic from policy decisions like filtering and address translation. The kernel defines where the hooks are; administrators and tools like [nftables](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page) decide what code runs at each one. The kernel implements hooks as arrays of function pointers stored in structures like [`struct nf_hook_entries`](https://elixir.bootlin.com/linux/v6.8/source/include/linux/netfilter.h). At each hook point, the kernel iterates the array via [`nf_hook_slow()`](https://elixir.bootlin.com/linux/v6.8/source/net/netfilter/core.c), passing each registered callback a pointer to the packet’s [`sk_buff`](https://elixir.bootlin.com/linux/v6.8/source/include/linux/skbuff.h) structure.

---

Earlier, I made reference to “The kernel’s networking stack.” Just what does that mean?

A packet arrives at the NIC. The driver places it in memory and the kernel’s networking stack processes it through several ordered stages. At defined points along this path, the kernel passes the packet through [**netfilter**](https://www.netfilter.org/), a hook-based framework built directly into the kernel’s networking code.

Netfilter hooks are function pointer arrays registered inside the kernel’s packet processing path. At each hook point, the kernel iterates through every registered function in priority order, passing a pointer to the packet’s socket buffer (`sk_buff`). Each registered function can accept, drop, modify, or queue the packet. Userspace tools like [nftables](https://wiki.nftables.org/wiki-nftables/index.php/Main_Page) register callback functions at these hooks by sending commands through a [**netlink**](https://man7.org/linux/man-pages/man7/netlink.7.html) socket, a kernel-userspace Inter-Process Communication (**IPC**) channel designed for networking configuration.

You can observe netfilter’s activity at runtime. `nft list ruleset` shows all currently registered tables and chains. [`conntrack -L`](https://conntrack-tools.netfilter.org/) shows the live connection tracking table. For deeper inspection, [`perf trace`](https://perf.wiki.kernel.org/index.php/Main_Page) or [`bpftrace`](https://github.com/bpftrace/bpftrace) can attach probes to kernel functions like [`nf_hook_slow`](https://elixir.bootlin.com/linux/v6.8/source/net/netfilter/core.c) (the function the kernel calls when it iterates hook callbacks), letting you watch individual packet decisions in real time.

The five standard hook points are:

|Hook|Position in the packet path|
|---|---|
|**PREROUTING**|Immediately on arrival, before any routing decision|
|**INPUT**|For packets destined for a local process|
|**FORWARD**|For packets passing _through_ the machine to another host|
|**OUTPUT**|For packets generated by local processes|
|**POSTROUTING**|Just before a packet leaves an interface|

After PREROUTING, the kernel makes its routing decision. Packets addressed to the machine itself travel up through INPUT. Packets addressed to other hosts, when forwarding is enabled, move to FORWARD and then out through POSTROUTING. Every configuration step either registers code on one of these hooks or changes how the routing decision behaves.

---

## Change 1: Activating IP Forwarding

IP forwarding is the first gate for enabling transport of packets across interfaces. Without it, the FORWARD hook might exist, but the kernel never sends packets to it. Packets arriving for foreign destinations die after the routing lookup. With it open, the kernel hands those packets to FORWARD, and every other piece of the router configuration takes effect.

You manage ip forwarding through the /etc/sysctl.d/10-forward.conf file:

```
/etc/sysctl.d/10-forward.conf

net.ipv4.ip_forward=1
```

/etc/sysctl.d/ is a drop-in configuration directory for kernel runtime parameters. At boot, [systemd-sysctl.service](https://www.freedesktop.org/software/systemd/man/latest/systemd-sysctl.service.html) reads every *.conf file in that directory (plus /etc/sysctl.conf) and writes each parameter to its corresponding path under /proc/sys/.

The kernel exposes a virtual filesystem at /proc/sys/ where every tuneable parameter appears as a file. The dotted sysctl notation is just a path translation: net.ipv4.ip_forward maps to /proc/sys/net/ipv4/ip_forward. Writing `1` to this file tells the IPv4 stack to send packets with non-local destinations through the FORWARD hook rather than discarding them. The kernel implements this decision in [`ip_forward()`](https://elixir.bootlin.com/linux/v6.8/source/net/ipv4/ip_forward.c) in `net/ipv4/ip_forward.c`.

Writing 1 to sysctl.d/10-forward.conf makes those writes persistent across reboots.

`systemd-sysctl.service` reads all files under `/etc/sysctl.d/` at boot and applies them in lexicographic order. Restarting the service applies them immediately without requiring a system reboot. You can verify the active value at any time:

```
cat /proc/sys/net/ipv4/ip_forward
```

`1` means forwarding is live. `0` means the gate is closed, and the rest of the router configuration is inert regardless of what else is configured.

**Our first change** is setting the kernel’s ip_forward parameter to 1.

---

## Change 2: Defining The Bridge: Collapsing Two Interfaces Into One Segment

A home network serves both wired and wireless clients on the same subnet. The configuration creates a network bridge, `br0`, and attaches `eth0` and `wlan0` to it as member ports. For details on Linux bridge interfaces, see the [kernel bridge documentation](https://www.kernel.org/doc/html/latest/networking/bridge.html).

**Our second change** is defining a bridge and adding interfaces to it that bind them for passing packets.

A bridge operates at Layer 2, the Ethernet layer. The kernel’s bridge module maintains a Media Access Control (**MAC**) address forwarding table. When a frame arrives on `eth0`, the bridge looks up the destination MAC address in that table and forwards the frame to the port where that address was last seen. If the address is unknown, the bridge floods the frame to all member ports. The bridge expires learned associations after a configurable aging time. To the rest of the network, `br0` appears as a single unified switch, one shared Layer 2 segment across both wired and wireless interfaces. The kernel implements bridge forwarding logic in [`br_forward()`](https://elixir.bootlin.com/linux/v6.8/source/net/bridge/br_forward.c) in `net/bridge/br_forward.c`.

This matters for routing because the kernel assigns IP addresses to interfaces, not to physical ports. Assigning `192.168.1.1` to `br0` means the router holds a single Local Area Network (**LAN**) address regardless of whether a client is wired or wireless. Both interfaces carry traffic on the same subnet and communicate at Layer 2 without any routing decision required between them.

One important distinction: a wired interface like eth0 is enslaved to the bridge directly with a single command (`ip link set eth0 master br0`), and the kernel’s bridge module immediately begins learning MAC addresses from frames arriving on it. **A wireless interface (wlan0) cannot be enslaved to the bridge this way.**

The 802.11 protocol requires an association and authentication lifecycle that standard Ethernet bridging doesn’t account for. Instead, hostapd manages this relationship: the `**bridge=br0**` directive in `**hostapd.conf**` instructs hostapd to attach wlan0 to the bridge once the interface is in AP mode. Wireless clients that associate with the AP are then visible to the bridge as if they were on a wired port. The result is the same unified L2 segment, but the path to get there is different for wired and wireless members.

Per [https://wireless.docs.kernel.org/en/latest/en/users/documentation/hostapd.html](https://wireless.docs.kernel.org/en/latest/en/users/documentation/hostapd.html):

> The mac80211 subsystem moves all aspects of master mode into user space. It depends on hostapd to handle authenticating clients, setting encryption keys, establishing key rotation policy, and other aspects of the wireless infrastructure. Due to this, the old method of issuing `iwconfig <wireless interface> mode master` no longer works

On a standard Ethernet bridge port, any device that sends a frame gets its MAC learned — there’s no prior handshake required at L2. On an 802.11 AP, the MAC layer itself enforces that a client must complete authentication and association (State 3) before the AP will accept or forward its data frames. The AP’s MAC (managed by the driver via mac80211) is the gatekeeper, and it needs a userspace daemon (hostapd) to handle the authentication exchanges. The kernel’s bridge module has no knowledge of 802.11 states — it just sees frames — so it can’t manage this lifecycle on its own.

The [`bridge-utils`](https://wiki.linuxfoundation.org/networking/bridge) package provides `brctl` for inspecting bridge state. The kernel handles all forwarding logic through the `br_netfilter` and bridge modules.

**Aside: bridges and packet capture.** A bridge port is an excellent place to insert a packet capture. Attach a third interface to `br0` and mirror traffic to a tap device (for more on tap/tun virtual interfaces, see the [kernel tuntap documentation](https://www.kernel.org/doc/html/latest/networking/tuntap.html)), or use a standalone bridge with a port set to promiscuous mode feeding a capture daemon like [`tcpdump`](https://www.tcpdump.org/) or [`Zeek`](https://zeek.org/). Because the bridge sees all frames on the segment before any routing or filtering decision, a capture at this layer sees the complete pre-Network Address Translation (**NAT**), pre-firewall traffic picture. Tools like `tcpdump -i br0` or an [AF_PACKET](https://man7.org/linux/man-pages/man7/packet.7.html) socket bound to the bridge interface work at line rate for most home and small-business traffic volumes. These tools max out on a default Linux kernel at around 18 Gbps (at least they did when I last tested them, around 2023). Higher line rates require tools with hardware-based filtering like the Data Plane Development Kit (**[DPDK](https://www.dpdk.org/)**) or eXpress Data Path (**[XDP](https://www.kernel.org/doc/html/latest/networking/af_xdp.html)**).

---

## Change 3: Activating nftables policies: Installing Code on the Hooks

Now that we have a bridge, we need to define packet processing rules via [netfilter’s](https://www.netfilter.org/) nftables.

Netfilter is the broader kernel-level packet filtering framework that provides the hooks into the network stack, while nftables (via `nf_tables`) is the modern packet classification engine that operates on top of those hooks. It replaced iptables as the preferred interface, but both ultimately rely on the same netfilter hook infrastructure in the kernel. The kernel implements the nf_tables subsystem in [`nf_tables_api.c`](https://elixir.bootlin.com/linux/v6.8/source/net/netfilter/nf_tables_api.c) in `net/netfilter/`.

The firewall and NAT rules in `/etc/nftables.conf` are callback registrations. nftables sends them to the kernel through a netlink socket, and the `nf_tables` subsystem installs them at the specified hooks. Each chain declaration names its hook and priority explicitly:

```
chain forward {
    type filter hook forward priority 0; policy drop;
    iifname "eth0" oifname "br0" ct state { established,related } counter accept
    iifname "br0"  oifname "eth0" ct state { new,established,related } counter accept
    counter
}
```

This chain controls traffic forwarding between interfaces, the core job of a router. Here’s what’s happening:

### The chain definition:

`type filter hook forward priority 0; policy drop;`

This attaches to netfilter’s forward hook, meaning it only sees packets that aren’t destined for the router itself but need to pass through it. The default policy is drop, so anything not explicitly allowed is silently discarded. This is a deny-by-default posture.

In this WiFi AP setup, eth0 is the WAN-facing interface — the uplink to your ISP or upstream router. br0 is the LAN-facing bridge, which aggregates traffic from wired clients (if any are directly attached) and wireless clients managed by hostapd. All LAN traffic enters and exits through br0, regardless of whether it originated from a wired or wireless device. With that topology in mind, the two rules in the FORWARD chain map directly to the two directions of traffic flow across the router.

### Rule 1: Wide Area Network (WAN) to LAN (return traffic only):

`iifname "eth0" oifname "br0" ct state { established,related } counter accept`

Traffic arriving from eth0 (the WAN/internet side) heading toward br0 (the LAN bridge) is only accepted if conntrack (ct state) shows the connection was already initiated from the LAN side. This means unsolicited inbound connections from the internet are blocked, exactly what you want from a NAT router/firewall.

### Rule 2: LAN to WAN (outbound traffic):

`iifname "br0" oifname "eth0" ct state { new,established,related } counter accept`

Traffic from br0 heading out to eth0 is accepted for new connections as well as existing ones. This lets LAN clients freely initiate connections to the internet.

### The trailing counter:

This is a catch-all counter with no action; it just counts packets that matched neither rule above (and will therefore be dropped by the policy). It’s useful for monitoring how much traffic is being rejected.

This is a classic “stateful” firewall pattern. LAN devices can reach the internet freely, but the internet can never initiate connections inward. The related state also allows things like Internet Control Message Protocol (**ICMP**) errors and File Transfer Protocol (**FTP**) data channels that are associated with an existing connection to pass through.

When `nftables.service` loads or reloads the configuration, it flushes the existing ruleset and installs the new one atomically through the netlink interface. No packet sees a partial ruleset during the transition. Reload with:

```
sudo systemctl reload nftables.service
```

Validate a configuration file before applying it:

```
sudo nft -c -f /etc/nftables.conf
```

If you are gonna dive deep into netfilter, [this blog](https://blogs.oracle.com/linux/introduction-to-netfilter) is outstanding

**Our third change** was defining nf_tables rules for processing packets.

---

## Change 4: Stateful Firewalling with conntrack

The rule fragments `ct state { established, related }` and `ct state { new, established, related }` reference [**conntrack**](https://conntrack-tools.netfilter.org/), the kernel’s connection tracking subsystem. Conntrack is what makes two simple rules sufficient to handle all legitimate traffic. The kernel implements the connection tracking core in [`nf_conntrack_core.c`](https://elixir.bootlin.com/linux/v6.8/source/net/netfilter/nf_conntrack_core.c) in `net/netfilter/`.

Conntrack watches traffic as it passes through netfilter and maintains a table of active flows. Each entry stores the source and destination addresses, ports, protocol, and current connection state. When a LAN client opens a Transmission Control Protocol (**TCP**) connection to a server on the internet, conntrack creates an entry and marks the flow `new`. Once the three-way handshake completes, conntrack marks it `established`. Reply packets from the internet match `ct state established` in the FORWARD chain and pass through automatically.

The firewall allows outbound connections from `br0` to `eth0` when they carry state `new` or `established`. Return packets arriving on `eth0` match as `established`. Conntrack holds the bookkeeping; the firewall rules consult the table.

The `related` state covers secondary flows. Protocols like FTP open a control connection and then negotiate a separate data connection on a different port. ICMP error messages tie back to existing TCP or User Datagram Protocol (**UDP**) flows. Conntrack understands these relationships and marks the secondary flows accordingly, so the firewall accepts them without explicit rules for every protocol variant.

**Our fourth change** is an expansion of network connection tracking in the Kernel’s connection tracking subsystem. We have begun tracking packets for systems beyond just our own host.

---

## Change 5: Defining NAT and Masquerade policies: Rewriting Addresses at the Border

Home networks use Request for Comments (**RFC**) 1918 private address space: `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`. The public internet carries routes to none of these ranges. Every packet leaving the LAN needs its source address replaced with the router’s public IP before it exits. Without that replacement, the originating host will never receive replies from the internet.

The `postrouting` chain at the POSTROUTING hook replaces each outbound packet’s private source address with the router’s public address:

```
chain postrouting {
    type nat hook postrouting priority 100; policy accept;
    oifname "eth0" counter masquerade
}
```

The term _masquerade_ relates to the act of disguising oneself. The router pretends to be the original sender of a request bound for the internet, but it remembers which node on the internal network made the original request. The resource on the internet responds to the router as if it’s connecting with the original sender, but the router modifies the packet and sends it on to the original requester. The router presents the LAN client to the outside world under a different identity, the WAN IP, concealing the private address behind a public one. The client appears to the remote server as the router itself. The router hides the client’s original address. The kernel implements the masquerade action in [`nf_nat_masquerade.c`](https://elixir.bootlin.com/linux/v6.8/source/net/netfilter/nf_nat_masquerade.c) in `net/netfilter/`.

Conntrack stores the translation as part of each flow’s entry. The tuple `(private IP, private port, public IP, public port, protocol)` lives in the conntrack table for the lifetime of the connection. You can inspect it directly:

```
sudo conntrack -L
```

Each line shows the original and reply tuples for a live flow, along with the connection state and a timeout countdown. Flows that have been idle long enough age out, and conntrack removes their entries, a key mechanism for preventing the NAT table from growing without bound. TCP connections time out after the session closes or after a configurable idle period. UDP entries use shorter timers because UDP carries no close signal.

The `masquerade` action reads `eth0`’s current IP address at the moment the packet is processed, rather than at configuration time. This makes it the correct choice for a WAN interface that acquires its address via Dynamic Host Configuration Protocol (**DHCP**), where the public IP may change without notice. When the address changes, new connections use the new address automatically. Conntrack retains entries for established connections under the old address until they expire.

**Our fifth change** is defining rules that modify the sender and recipient addresses in packets processed by the host.

---

## Change 6: Vending DHCP and DNS with dnsmasq: Announcing the Router to New Clients

Every computer on the Internet needs to know three things to work: their IP address, their default gateway to the internet, and their Domain Name System (**DNS**) server.

A router must introduce itself to clients on their network. New clients arrive without an IP address, without a default gateway, and without a DNS resolver. [`dnsmasq`](https://thekelleys.org.uk/dnsmasq/doc.html) vends these values to clients on their network through DHCP.

When a device joins the network, it broadcasts a DHCP discovery. `dnsmasq` listens on `br0` and responds with an offer containing an IP address, subnet mask, lease duration, and two DHCP options: option 3 (default gateway, `192.168.1.1`) and option 6 (DNS server, `192.168.1.1`). Option 3 tells the client where to send packets destined for addresses outside the local subnet. Option 6 tells the client which resolver to query. `dnsmasq` caches upstream responses locally, reducing query volume and accelerating repeat lookups.

`dnsmasq` binds to `br0` so it serves only the LAN. It never listens on `eth0`.

**NetworkManager as an alternative:** [NetworkManager](https://networkmanager.dev/) can handle both DHCP server and DNS functions through its built-in `dnsmasq` integration, activated by setting `dns=dnsmasq` in `/etc/NetworkManager/NetworkManager.conf`. NetworkManager launches its own `dnsmasq` instance and manages its configuration dynamically as interfaces come and go.

There are significant tradeoffs for each approach. NetworkManager’s approach reduces manual configuration and handles interface lifecycle events automatically. This is useful on a laptop or a machine where interfaces appear and disappear. On a dedicated router, you generally will want greater control. NetworkManager may reconfigure `dnsmasq` or restart it in response to network events, interrupting DHCP leases in unpredictable ways. A static `dnsmasq` configuration launched by systemd gives you deterministic startup order, explicit binding, and straightforward log inspection via `journalctl -eu dnsmasq.service`. You know exactly what the daemon is configured to do because you wrote the configuration file.

From a kernel perspective, both paths land in the same place: a userspace process bound to a UDP socket on port 67, servicing DHCP requests arriving on the bridge interface. The kernel doesn’t distinguish between the two arrangements. The difference is in how the daemon is launched, configured, and supervised. This is a service management and operational tradeoff, not an architectural one.

**Our sixth change** is deploying a new daemon (dnsmasq) for vending DHCP and DNS services to clients on the system’s network(s).

---

## Change 7: Vending WiFi networks with hostapd: Switching the Wireless Card into Access Point (AP) Mode

Wireless interfaces operate in one of several modes. In managed mode, a card scans for access points and associates as a client. In AP mode, the card broadcasts beacons, accepts association requests, and manages the full authentication lifecycle for connecting devices.

The kernel’s [`mac80211`](https://elixir.bootlin.com/linux/v6.8/source/net/mac80211/) subsystem provides a unified programming interface for 802.11 hardware across different driver implementations. [`hostapd`](https://w1.fi/hostapd/) communicates with `mac80211` through the [`nl80211`](https://wireless.wiki.kernel.org/en/developers/documentation/nl80211) netlink interface, the same socket-based kernel-userspace channel that nftables uses, applied here to the wireless subsystem. Through `nl80211`, `hostapd` commands the driver to enter AP mode, sets the Service Set Identifier (**SSID**), channel, and Wi-Fi Protected Access 2 (**WPA2**) encryption parameters, and takes ownership of authentication frames.

The `bridge=br0` directive in `hostapd.conf` attaches the AP interface to the bridge as a member port. Wireless clients, once associated, enter the same Layer 2 segment as wired clients. Their traffic arrives on `br0`, the kernel applies the same netfilter decisions, and packets travel the same forwarding path as everything else on the LAN.

Debian ships `hostapd` masked by default. Systemd registers the service but blocks it from starting. This blocking prevents an unconfigured instance from launching and broadcasting an open network. `systemctl unmask hostapd` removes that block, after which `systemctl enable --now hostapd` starts it and registers it for future boots.

**Our seventh change** is deploying a new daemon (hostapd) for vending WiFi networks from the device’s WiFi card.

## The Result: A WiFi Router!

Each configuration step activates a different layer of the kernel’s networking architecture. Together, they build a complete forwarding system:

|Step|Kernel mechanism|Layer|
|---|---|---|
|`ip_forward=1` via sysctl|IPv4 stack enables FORWARD path|L3|
|`br0` bridge *|L2|L2 *|
|nftables FORWARD chain|Netfilter hook, packet policy|L3/L4|
|conntrack|Stateful connection table|L3/L4|
|masquerade|Source NAT at POSTROUTING|L3|
|dnsmasq DHCP|Gateway and DNS announcement|Application|
|hostapd via nl80211|AP mode through mac80211|L2 wireless|

_**Note on the bridge row:** Adding a wired interface to br0 is a direct kernel operation — the bridge module immediately takes over frame forwarding for that port. Adding a wireless interface is indirect: hostapd’s_ `bridge=br0` _directive handles the attachment after the wireless card enters AP mode and a client associates. Both result in the same logical L2 segment, but the mechanism differs. If you are debugging bridge membership,_ `**brctl show**` _(or_ `**ip link show master br0**`) _will show wired members directly; wireless clients appear as learned MAC entries in the bridge’s forwarding table once they associate, which you can inspect with_ **`brctl showmacs br0`.**

Start with a Linux machine in its default state: a workstation that receives packets for itself, forwards nothing, and drops traffic addressed to any IP it doesn’t own. Its IP forwarding gate is closed. Its netfilter FORWARD chain is empty. Its wireless card listens for beacons rather than broadcasting them. It has no DHCP server, no NAT table, and no bridge.

- IP forwarding opens the gate for the possibility of routing.
- The bridge collapses the wired and wireless interfaces into a single addressable domain.
- The nftables chains install policy at the FORWARD hook, deciding what passes and what drops.
- Conntrack feeds state information into those policy decisions, making simple rules work for complex traffic patterns.
- Masquerade hides the LAN behind the router’s public identity and keeps a translation table in memory.
- dnsmasq announces the router’s presence and hands every new client the information it needs to reach the outside world.
- hostapd converts a client-mode radio into an access point.

These are the changes that transform a Linux system into a WiFi router. You can evaluate and inspect them through 6 commands:

- **`cat /proc/sys/net/ipv4/ip_forward`** for forwarding state,
- **brctl show** `for bridge membership,`
- **nft list ruleset** `for the active firewall policy,`
- **conntrack -L** for live flows and NAT mappings,
- **journalctl -eu dnsmasq.service** for DHCP lease activity,
- **[`iw`](https://wireless.wiki.kernel.org/en/users/documentation/iw) `dev`** for wireless interface mode.