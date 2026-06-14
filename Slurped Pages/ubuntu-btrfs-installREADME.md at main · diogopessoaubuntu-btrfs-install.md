---
link: https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md
site: GitHub
excerpt: Ubuntu Installation with Btrfs  and automatic snapshots - diogopessoa/ubuntu-btrfs-install
twitter: https://twitter.com/@github
slurped: 2026-06-12T15:29
title: ubuntu-btrfs-install/README.md at main · diogopessoa/ubuntu-btrfs-install
tags:
  - btrfs
---

## Ubuntu + Btrfs + Automatic Snapshots

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#ubuntu--btrfs--automatic-snapshots)

**Update Note (May 2026):**

_I've migrated to Fedora Silverblue (Fedora Atomic), which offers **native Btrfs support with snapshots and rollback via rpm-ostree** right after installation, without any extra configuration. This Ubuntu + Btrfs repository remains as historical reference, but I recommend Silverblue for modern immutable setups_.

## What the Script Does

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#what-the-script-does)

This script creates Btrfs subvolumes (while still in Live CD/USB mode) for Ubuntu 24.04 (or newer) and compatible derivatives.

- Creates Btrfs subvolumes:
    - `@home` `@log` `@cache` `@tmp` `@libvirt` `@flatpak` `@docker` `@containers` `@machines` `@var_tmp` `@opt`

## Requirements

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#requirements)

- Ubuntu 24.04 or newer installed with:
    - Root filesystem using **Btrfs**
    - Separate **/boot** partition formatted as ext4 (2GB)
    - (Optional) EFI partition for UEFI systems (1GB)
- Run the `ubuntu-btrfs-install` script from the **Live CD/USB** after Ubuntu is installed

This guide uses Ubuntu 25.04 as an example.

### Step-by-Step Guide

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#step-by-step-guide)

1. **Preparation**
    
    - Create a bootable USB drive using the Ubuntu ISO
    - Disable Secure Boot in BIOS/UEFI if needed to avoid installation issues
2. **Start Installation**
    
    - Boot from the USB drive and select your language
    - Choose “Manual installation” (custom partitioning)
3. **Create Partitions in the Correct Order**
    
    - Create a new GPT partition table on the disk
    - Create the **/boot/efi** partition:
        - Size: 1GB
        - Format: FAT32 (vfat)
        - Type: EFI System Partition
        - Mount point: `/boot/efi`
    - Create the **/boot** partition:
        - Size: 2GB
        - Format: ext4
        - Mount point: `/boot`
    - Create the root **/** partition:
        - Use all remaining space
        - Format: Btrfs
        - Mount point: `/`
        

Note: `/boot/efi` partition don't necessarily need to be created first. The `/boot` partition can be created first if the installer requires it.

4. **Final Partition Table Should Look Like:**
    
    - `/boot/efi` as FAT32 (vfat)
    - `/boot` as ext4
    - `/` as Btrfs
5. **Complete Installation**
    
    - Finish the Ubuntu installation, but **DO NOT reboot yet**

## How to Use the Script

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#how-to-use-the-script)

⚠️ **After installing Ubuntu with Btrfs, do not reboot!**

### Identify Your Partitions

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#identify-your-partitions)

Run the following command in the terminal:

Look for identifiers like `sda`, `nvme0n1`, etc. Example output:

```
sda     
├─sda1  vfat   /boot/efi
├─sda2  ext4   /boot
└─sda3  btrfs  /
```

### Download the Script to the Live Session

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#download-the-script-to-the-live-session)

cd ~/Downloads
wget https://raw.githubusercontent.com/diogopessoa/ubuntu-btrfs-install/main/ubuntu-btrfs-install.sh

### Make It Executable

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#make-it-executable)

chmod +x ubuntu-btrfs-install.sh

### Run the Script

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#run-the-script)

The argument order must be: `root` → `boot` → `efi`

sudo ./ubuntu-btrfs-install.sh sda3 sda2 sda1

This example is using `/dev/sda`

> Double-check your partition names using `lsblk -f`

### ✅ Done!

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#-done)

You can now reboot before installing Snapper and Btrfs Assistant for automatic snapshots.

💡 Tip: to view Btrfs subvolumes run:

sudo btrfs subvolume list /

## 📦 Manual Installation of Snapper and Btrfs Assistant (Post-Installation)

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#-manual-installation-of-snapper-and-btrfs-assistant-post-installation)

Snapper is a snapshot manager and Btrfs Assistant is a Snapper GUI.

After rebooting the system, install:

sudo apt update
sudo apt install -y snapper btrfs-assistant

Create Snapper root config:

sudo snapper -c root create-config /

Enable timeline and cleanup timers:

sudo systemctl enable --now snapper-timeline.timer snapper-cleanup.timer

You can now launch **Btrfs Assistant** from your application menu or run:

### Automatic Snapshots Configuration

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#automatic-snapshots-configuration)

1. Now go to **"Snapper Settings"** tab 🟢 **Enable timeline snapshots**:
    - Hourly save: 10
    - Daily save: 10
    - Weekly save: 0
    - Montthly save: 3
    - Yearly save: 1
    - Number save: 10

- System unit settings:
    
    - 🟢 Check **"Enable cleanup enabled"**
    - 🟢 Check **"Snapper timeline enabled"**
    - ❌ Keep unmarked **"Snapper boot"**
        - With a separate /boot on ext4, enabling Boot Snapshots is not recommended because:
        - They will have no real effect.
        - They may cause confusion or failures in system restores (since /boot will not be included in Btrfs snapshots).

2. Click **"Apply systemd changes"**.

### ✅ Done! Your system now has snapshots automatically.

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#-done-your-system-now-has-snapshots-automatically)

### Screenshots

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#screenshots)

- Btrfs Assistant "Snapper" [![Btrfs Assistant Snapper](https://camo.githubusercontent.com/70b3c6b0e33cfe0fdb559a9bbed118e91319ce2aa32cd8995c2e44298361fdf6/68747470733a2f2f6769746c61622e636f6d2f2d2f70726f6a6563742f33323533353438382f75706c6f6164732f36356236303034633332353764363631353438323832353961306665643437642f696d6167652e706e67)](https://camo.githubusercontent.com/70b3c6b0e33cfe0fdb559a9bbed118e91319ce2aa32cd8995c2e44298361fdf6/68747470733a2f2f6769746c61622e636f6d2f2d2f70726f6a6563742f33323533353438382f75706c6f6164732f36356236303034633332353764363631353438323832353961306665643437642f696d6167652e706e67)
    
- Btrfs Assistant "Snapper Settings" [![Btrfs Assistant Snapper Settings](https://camo.githubusercontent.com/89e111e82cf361c62a465e7226a274a5904d39f798a8555bf775d06a52193b9c/68747470733a2f2f6769746c61622e636f6d2f2d2f70726f6a6563742f33323533353438382f75706c6f6164732f34323962653734653966623932303838363937393434643233613164656631642f696d6167652e706e67)](https://camo.githubusercontent.com/89e111e82cf361c62a465e7226a274a5904d39f798a8555bf775d06a52193b9c/68747470733a2f2f6769746c61622e636f6d2f2d2f70726f6a6563742f33323533353438382f75706c6f6164732f34323962653734653966623932303838363937393434643233613164656631642f696d6167652e706e67)
    

## License

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#license)

MIT License — [View License](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/LICENSE) You can use, modify, and contribute!

## Credits

[](https://github.com/diogopessoa/ubuntu-btrfs-install/blob/main/README.md#credits)

- [openSUSE Team](https://github.com/openSUSE/snapper) — Snapper
- [Antynea](https://github.com/Antynea/grub-btrfs) — grub-btrfs
- [Dan Cantrell](https://gitlab.com/btrfs-assistant/btrfs-assistant) — Btrfs Assistant
- [Ubuntu](https://ubuntu.com/download) — Operating System