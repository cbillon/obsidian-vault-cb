---
link: https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926
byline: Richard Chamberlain
site: DEV Community
date: 2025-01-05T20:59
excerpt: Learn how to create and manage virtual machines using the Proxmox CLI (`qm` command). This step-by-step guide covers scripting VM configurations, setting up storage, memory, networking, and automating deployments for high-performance workloads. Perfect for developers and IT pros looking to streamline their Proxmox workflows.
twitter: https://twitter.com/@SebosInfo
tags:
  - Proxmox
  - automation
slurped: 2025-03-12T13:12
title: "How to Build and Manage Virtual Machines Using Proxmox CLI: A Step-by-Step Guide"
---

Recently, I needed to set up a series of virtual machines (VMs) in Proxmox for a project. One of the VMs was intended for high-end data processing, and I wasn’t entirely certain about the exact requirements at the outset. To address this, I decided to experiment with various configurations until I found the optimal setup. While the Proxmox web interface is excellent, I needed a quicker, more flexible way to make incremental changes without navigating through multiple steps in the GUI. As someone who spends most of their time in the command line, using the Proxmox `qm` command-line interface felt like a natural fit.

## [](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#table-of-contents)Table of Contents

1. [Building a Virtual Machine with Proxmox CLI](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#building-a-virtual-machine-with-proxmox-cli)
2. [Leveraging the Proxmox CLI](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#leveraging-the-proxmox-cli)
    - [Script Setup](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#script-setup)
3. [Removing Existing VMs](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#removing-existing-vms)
4. [Creating the Base VM](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#creating-the-base-vm)
5. [Configuring Storage](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#configuring-storage)
6. [Configuring Networking](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#configuring-networking)
7. [Why Not Use the Web Interface?](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#why-not-use-the-web-interface)
8. [Why Not Use a Clone or Template?](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#why-not-use-a-clone-or-template)
9. [Adapting the Script for Your Project](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#adapting-the-script-for-your-project)

## [](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#leveraging-the-proxmox-cli)Leveraging the Proxmox CLI

Proxmox provides the `qm` command for managing VMs directly from the command line, offering functionality comparable to the web interface and, in some cases, even more flexibility. You can refer to the official [`qm` man page](https://pve.proxmox.com/pve-docs/qm.1.html) for comprehensive details. To make my workflow more efficient and reusable, I began by setting up some variables in a script. These variables included information about the VM ID, name, storage locations, disk sizes, memory, and CPU configuration. Here's the script snippet:  

```
#!/bin/bash
## Basic VM Info
VMID=990
NAME="ThisVM"

## Linux to Install
ISO_STORAGE="ISOs"           # Replace with your ISO storage name
ISO_FILE="OracleLinux-R9-U4-x86_64-dvd.iso" # Replace with your ISO filename

## Hard Drive Size
DISK_STORAGE="vm_storage"    # Replace with your disk storage name
SYSTEM_DISK="300G"
DATA_DISK="500G"

## Memory Size
MEMORY=32
MEMORY_SIZE=$(( MEMORY * 1024 ))

## Processors
CORES=7
SOCKETS=2

## Network
VLAN_TAG=20
INTERFACE="vmbr0"
```

 

### [](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#removing-existing-vms)Removing Existing VMs

If the VM already existed, I made sure to remove it before proceeding. This allowed me to tweak configurations and rerun the script without issues. Of course, if you plan to use this code, ensure the VM is either backed up or no longer needed.  

```
## If existing, remove
if qm list | awk '{print $1}' | grep -q "^$VMID$"; then
    qm stop $VMID
    qm destroy $VMID
fi
```

 

### [](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#creating-the-base-vm)Creating the Base VM

With the setup cleared, I moved on to creating the base VM using `qm create` and configuring its resources. Here’s the snippet for setting memory, CPUs, and enabling NUMA (Non-Uniform Memory Access):  

```
## Create VM
qm create $VMID --name $NAME 

## Setup Memory and CPUs
qm set $VMID --memory ${MEMORY_SIZE}
qm set $VMID --balloon ${MEMORY_SIZE}
qm set $VMID --cpu cputype=host
qm set $VMID --cores ${CORES} --sockets ${SOCKETS} --numa 1
```

 

**Note:** NUMA support helps optimize memory access on modern multi-CPU servers by aligning memory regions with specific processors. This can be particularly important for high-performance workloads.

### [](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#configuring-storage)Configuring Storage

Next, I set up the boot ISO and the VM’s storage drives. For this project, I added a secondary disk specifically for data storage:  

```
## Install ISO and Hard Drives
qm set $VMID --cdrom $ISO_STORAGE:iso/$ISO_FILE

### OS Drive
pvesm alloc vm_storage $VMID vm-${VMID}-disk-0 300G
qm set $VMID --scsi0 vm_storage:vm-${VMID}-disk-0,iothread=1,cache=writeback

### Data Drive
pvesm alloc vm_storage $VMID vm-${VMID}-disk-1 500G
qm set $VMID --scsihw virtio-scsi-single 
qm set $VMID --scsi1 vm_storage:vm-${VMID}-disk-1,iothread=1

### Boot Order
qm set $VMID --boot order='ide2;scsi0'
```

 

### [](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#configuring-networking)Configuring Networking

For networking, the project required a single network card on VLAN 20. Here’s the configuration I used:  

```
## Network 
qm set $VMID --net0 virtio,bridge=${INTERFACE},tag=${VLAN_TAG},queues=4
```

 

[full code here](https://github.com/richard-sebos/qm-promox/blob/main/data_processing.sh)

### [](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#why-not-use-the-web-interface)Why Not Use the Web Interface?

While there’s absolutely nothing wrong with the Proxmox web interface, for this project, I needed the flexibility to tinker and rebuild quickly. Writing the script allowed me to fine-tune the setup iteratively, and it ensured that future rebuilds would be consistent and straightforward.

### [](https://dev.to/sebos/how-to-build-and-manage-virtual-machines-using-proxmox-cli-a-step-by-step-guide-5926#why-not-use-a-clone-or-template)Why Not Use a Clone or Template?

Cloning or using a VM template would certainly be a valid approach for creating similar VMs in the future. However, as someone who enjoys coding and experimenting, I found scripting this process to be both rewarding and practical. It also gave me complete control over every aspect of the VM’s configuration.

How might you adapt this script for your project? Whether you’re automating VM deployments or just testing out configurations, using the Proxmox CLI can save time and streamline your workflows.