---
link: https://cylab.be/blog/490/tutorial-building-isolated-network-topologies-in-proxmox
excerpt: This guide outlines how to leverage Proxmox Software Defined Networking (SDN) and an OPNsense virtual router to create isolated, multi-subnet environments.
slurped: 2026-05-19T08:18
title: "Tutorial: Building Isolated Network Topologies in Proxmox"
tags:
  - Proxmox
  - network
---

This guide outlines how to leverage Proxmox Software Defined Networking (SDN) and an OPNsense virtual router to create isolated, multi-subnet environments.

---

## 1. Prerequisites

Before beginning, ensure you have:

- A running **Proxmox VE** installation
- An **OPNsense VM** already deployed on that Proxmox server, with at least one interface connected to your upstream internet/WAN
- A **management workstation** (a VM or physical machine) with browser access to the OPNsense Web GUI

In cylab’s case the above are provided, just have an active account on the Cylab Proxmox server

### Initial Infrastructure State

At this point, your OPNsense router is bridged to the internet, but your private virtual networks have not yet been defined. VMs you create are isolated — they have no gateway, no DHCP, and no internet access until we configure the SDN and router together. [![beginningsetup_v3.png](https://cylab.be/storage/blog/490/files/tiBJ6Rd0RbvhsnHE/beginningsetup_v3.png)](https://cylab.be/storage/blog/490/files/tiBJ6Rd0RbvhsnHE/beginningsetup_v3.png)

---

## 2. Planning the Topology[](https://cylab.be/blog/490/tutorial-building-isolated-network-topologies-in-proxmox#2-planning-the-topology)

The goal of this tutorial is to create a topology with three distinct subnets. VMs within these subnets will be able to communicate with each other and access the internet through the central router.

**Target Architecture:**

- **Subnet A**: vntest, **Subnet B**: iovnet, **Subnet C**: vntest3
- **Routing:** OPNsense handling inter-VNet traffic and DHCP assignment.

[![networknew.png](https://cylab.be/storage/blog/490/files/uQPbigaDaGW3rpw5/networknew.png)](https://cylab.be/storage/blog/490/files/uQPbigaDaGW3rpw5/networknew.png)

---

## 3. Defining the Network Blueprint (Proxmox SDN)[](https://cylab.be/blog/490/tutorial-building-isolated-network-topologies-in-proxmox#3-defining-the-network-blueprint-proxmox-sdn)

First, we must define the virtual “wires” (VNets) in the Proxmox datacenter.

1. **Create a Zone:** Navigate to **Datacenter -> SDN -> Zones**. Create a new **Simple** zone (e.g., `testzone`). This acts as the logical container for your virtual networks.

[![zone1.png](https://cylab.be/storage/blog/490/files/IElJxPQUXJoumSf6/zone1.png)](https://cylab.be/storage/blog/490/files/IElJxPQUXJoumSf6/zone1.png)

2. **Create VNets:** Navigate to the **VNets** tab. Create your required networks (e.g., `vntest`, `Iovnet`) and associate them with the Zone created in the previous step.

[![vnet1.png](https://cylab.be/storage/blog/490/files/jjz9sFTvBt2IKVvw/vnet1.png)](https://cylab.be/storage/blog/490/files/jjz9sFTvBt2IKVvw/vnet1.png)

3. **Define Subnets:** Within each VNet, assign the IP ranges for your subnets (e.g., `192.168.9.0/24`).

[![subnet.png](https://cylab.be/storage/blog/490/files/ss1egKrwlZO5A6oX/subnet.png)](https://cylab.be/storage/blog/490/files/ss1egKrwlZO5A6oX/subnet.png)

> **Note:** At this stage, the network paths exist, but there is no gateway or DHCP server attached. VMs created now will require manual IP configuration and will not have internet access until the router is configured.

---

## 4. Configuring the Router (OPNsense)[](https://cylab.be/blog/490/tutorial-building-isolated-network-topologies-in-proxmox#4-configuring-the-router-opnsense)

To enable internet access and automatic IP assignment, we must bridge the Proxmox VNets to the OPNsense virtual appliance.

#### A. Hardware Level Mapping

1. In the Proxmox interface, select your **OPNsense VM**.
2. Navigate to the **Hardware** tab.
3. Add a new **Network Device** for every VNet you created. Ensure they are linked to the correct VNet bridge.

[![networkdeviceadd.png](https://cylab.be/storage/blog/490/files/libn5gzSHpQoUO2F/networkdeviceadd.png)](https://cylab.be/storage/blog/490/files/libn5gzSHpQoUO2F/networkdeviceadd.png)

#### B. Software Interface Assignment

1. Access your **Management Workstation** and log into the OPNsense Web GUI.
2. Navigate to **Interfaces -> Assignments**.
3. Add the new network ports and provide descriptive names (e.g., `LAN_General`, `LAN_IoT`).

[![interfaceassignment2.png](https://cylab.be/storage/blog/490/files/3EUIll9x0QpNXSL1/interfaceassignment2.png)](https://cylab.be/storage/blog/490/files/3EUIll9x0QpNXSL1/interfaceassignment2.png)

4. Click on each interface to **Enable** it.
5. Set the **IPv4 Configuration Type** to “Static IPv4” and assign the gateway IP for that specific subnet (e.g., `192.168.9.1`).

[![interfaceenable.png](https://cylab.be/storage/blog/490/files/rX1L1XkuKUvz2HLT/interfaceenable.png)](https://cylab.be/storage/blog/490/files/rX1L1XkuKUvz2HLT/interfaceenable.png)

#### C. DHCP Server & DNS Configuration

To automate IP distribution for your VMs:

1. Navigate to **Services -> Dnsmasq DNS & DHCP**.
2. Select the tab for your new interfaces
3. Define the **Range** of IPs to be distributed to clients.

[![DHCP1st.png](https://cylab.be/storage/blog/490/files/tOfzwr8BPVbsE5A0/DHCP1st.png)](https://cylab.be/storage/blog/490/files/tOfzwr8BPVbsE5A0/DHCP1st.png)

[![DHCP2nd.png](https://cylab.be/storage/blog/490/files/g7fX0D08LDzaIWSz/DHCP2nd.png)](https://cylab.be/storage/blog/490/files/g7fX0D08LDzaIWSz/DHCP2nd.png)

4. Under **DHCP Options**, ensure you provide a **DNS Server** so VMs can resolve web addresses.

[![DHCP3rd.png](https://cylab.be/storage/blog/490/files/IzlGiNeK2ZKeyU4K/DHCP3rd.png)](https://cylab.be/storage/blog/490/files/IzlGiNeK2ZKeyU4K/DHCP3rd.png)

---

## 5. Firewall Rules[](https://cylab.be/blog/490/tutorial-building-isolated-network-topologies-in-proxmox#5-firewall-rules)

By default, OPNsense may block traffic on new interfaces. You must ensure rules are in place to allow traffic to flow.

1. Navigate to **Firewall -> Rules**.
2. Select your interface tab and add a rule to allow traffic from the subnet to your desired destinations (or “Any” for full internet access).

## [![firewall.png](https://cylab.be/storage/blog/490/files/KTgOfRfHtcL8LhPV/firewall.png)](https://cylab.be/storage/blog/490/files/KTgOfRfHtcL8LhPV/firewall.png)[](https://cylab.be/blog/490/tutorial-building-isolated-network-topologies-in-proxmox#)

## 6. Deployment[](https://cylab.be/blog/490/tutorial-building-isolated-network-topologies-in-proxmox#6-deployment)

Your environment is now fully configured. When creating new Virtual Machines:

1. Go to the **Network** tab during VM creation.
2. Select the appropriate **VNet bridge**.
3. The VM will automatically receive an IP via DHCP and have connectivity based on your router settings.

---

This blog post is licensed under [CC BY-SA 4.0 ![creative commons](https://cylab.be/images/cc.svg) ![attribution](https://cylab.be/images/by.svg) ![share-alike](https://cylab.be/images/sa.svg)](http://creativecommons.org/licenses/by-sa/4.0/?ref=chooser-v1)