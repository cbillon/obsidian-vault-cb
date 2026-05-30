---
link: https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg
byline: Richard Chamberlain
site: DEV Community
date: 2026-01-04T18:53
excerpt: Building a complete SMB environment using 100% open-source tools—no proprietary licensing, no vendor lock-in. Proxmox for virtualization, OPNSense for routing/firewall, Ansible for automation, Rocky Linux for VMs.
twitter: https://twitter.com/@SebosInfo
tags:
  - Proxmox
  - ansible
slurped: 2026-05-28T15:57
title: "Prototyping Enterprise Infrastructure in Proxmox: 11+ VMs, 8 VLANs, and Ansible Automation"
---

After seven years running Proxmox in my homelab, I'm tackling my most complex project yet—prototyping a complete SMB infrastructure with 11+ VMs, 8 network segments, and comprehensive automation.

## [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#why-prototype-virtually)Why Prototype Virtually?

Testing infrastructure in VMs before production deployment catches problems early:

- **Network misconfigurations** discovered in VLANs before buying physical switches
- **Resource constraints** identified before ordering hardware
- **Backup failures** found during testing instead of disasters
- **Automation issues** debugged in isolated environments
- **Disaster recovery** practiced without actual disasters

## [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#the-infrastructure)The Infrastructure

### [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#network-design-8-vlans)Network Design (8 VLANs)

```
10.0.100.0/24 – Management and monitoring
10.0.110.0/24 – Base infrastructure servers
10.0.120.0/24 – Application servers
10.0.130.0/24 – General workstations
10.0.131.0/24 – Manager workstations
10.0.132.0/24 – IT workstations
10.0.140.0/24 – Guest Wi-Fi and IoT
10.0.150.0/24 – Public-facing services
```

 

All inter-VLAN routing handled by an OPNSense VM. This lets me test firewall rules, routing policies, and network segmentation before deploying to physical infrastructure.

### [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#user-management)User Management

Proxmox supports PAM and Proxmox VE users. I use both:

- **PAM admin**: SSH access + web UI (root disabled)
- **PVE users**: Limited permissions, no shell access
- **Ansible user**: API-only access for automation

Pro tip: Shut down pveproxy when not using the web UI:  

```
systemctl stop pveproxy   # Stop when not needed
systemctl start pveproxy  # Start when needed
```

 

### [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#resource-pools-used-correctly)Resource Pools (Used Correctly)

I previously misused resource pools as tags. They're actually for **delegation and access control**:

- `smb-servers`: Core infrastructure
- `smb-workstations`: Desktop/laptop VMs
- `smb-project-admin@pve`: Full access across pools
- `smb-admin@pve`: Server pool only

### [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#backup-strategy)Backup Strategy

Dual-layer backups for redundancy:

**Local (10TB):** 7 daily, 4 weekly, 2 monthly  
**External (4TB):** 1 daily, 2 weekly, 1 monthly

And I actually test restore procedures. Backups are worthless if you've never validated them.

### [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#automation)Automation

Ansible user with Proxmox API access enables infrastructure as code:

- VM provisioning from templates
- Network configuration (VLAN assignments)
- Resource management
- Backup scheduling

Configurations stored in GitHub—destroy everything and rebuild from source.

## [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#is-this-overkill)Is This Overkill?

For a homelab? Yes. For learning? Absolutely not.

Overengineering in the lab teaches enterprise concepts (VLANs, RBAC, disaster recovery, automation) without production risk. When something breaks, fixing it builds troubleshooting skills.

## [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#hardware)Hardware

Dual-socket Lenovo D20 (24 cores) with:

- CPU host passthrough for VMs
- Memory ballooning across VMs
- Initial allocation: 4GB servers, 8GB workstations

## [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#whats-next)What's Next

I'm documenting the full build over 3-6 months:

- SMB infrastructure planning
- Ansible automation setup
- Samba Active Directory deployment
- File and print services
- Linux workstation configuration
- SELinux hardening
- Monitoring and backup automation

## [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#read-the-full-article)Read the Full Article

Complete details on my Proxmox prototyping methodology: [Prototyping a Larger Project with Proxmox](https://richard-sebos.github.io/sebostechnology/posts/Proxmox-Prototype/)

## [](https://dev.to/sebos/prototyping-enterprise-infrastructure-in-proxmox-11-vms-8-vlans-and-ansible-automation-5bkg#proxmox-virtualization-infrastructure-linux-devops-networking-ansible)proxmox #virtualization #infrastructure #linux #devops #networking #ansible