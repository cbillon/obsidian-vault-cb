---
tags:
  - linux
  - tips
  - swap
---

====== How to create a swapfile ======

[[https://www.linode.com/docs/guides/how-to-increase-swap-space-in-ubuntu/]]

Some systems automatically configure a swap file during installation. To determine whether the system already has a swap file, use the swapon --show command. If any files are displayed, this means one or more swap files already exist. Proceed to the “How to Increase Existing Swap Space” section for information on how to resize a swap file.

  sudo swapon --show

If a swap file has already been created, it is shown as follows. If there is no swap file, there is no output.

Confirm swap space has not already been allocated using the free command. This command can be used on most Linux systems to verify the swap space size.

  free -h

If the total memory for Swap is 0B, a swap file or partition has not been created yet.

Ensure there is enough space on the hard drive to create the swap space. Use the df -h command and locate the entry for the root directory, which is listed as /. The proposed swap file should fit comfortably within the available disk space with some room to spare. The following example indicates the hard drive has 64G of available space. This is more than adequate.

  df -h


Allocate memory for the swap file using the fallocate command. This command is used to reserve a certain amount of disc space for a file in advance. The following command creates a 1G file named swapfile in the root directory. Always give a swap file a very obvious name to avoid confusion.

  sudo fallocate -l 1G /swapfile

Change the file permissions for swapfile so only root can write to it. This prevents other users from accidentally deleting or overwriting the file.

  sudo chmod 600 /swapfile

Run the ls command to confirm the swap file has been successfully created. Use the -l option to see the size of the file.

  ls -hl /swapfile

Use the mkswap command to designate the new file as a swap file. This means it can be used for volatile memory when RAM space runs low.

  sudo mkswap /swapfile

The file swap has now been created, but it is still disabled. Activate it using the swapon command.

sudo swapon /swapfile

The swapon command can be used with the --show option to confirm the swap space is enabled. If the new swap file is not listed, verify the results of the previous instructions.

  sudo swapon --show

To confirm the amount of swap space available on Ubuntu, use the free command. The total column for the Swap entry should display 1.5Gi of total memory (1.0GB swap file and 0.5GB swap partition).

  free -h
  
===== Making the Swap File Changes Permanent =====

To add an entry to the fstab file, follow these steps:

As a precaution, make a backup copy of the existing fstab file.

  sudo cp /etc/fstab /etc/fstab.bak

Edit the file using a text editor.

  sudo nano /etc/fstab

Add the following line to the end of the file.


  File: /etc/fstab
  /swapfile       swap            swap    defaults          0     0
  
Save and close the file. The swap file should appear automatically after a system reboot.

Adjust the Swappiness Setting
The swappiness property controls how aggressively Ubuntu swaps data out of RAM. This setting extends from 0 to 100. Higher values tell the system to use the swap file more often. Configure a setting at the upper end of this range to ensure RAM is also available for other applications. Unfortunately, a very high setting can compromise performance. A low swappiness setting means the system is less likely to use the swap mechanism. Use a setting closer to zero when application performance is important.

==== To adjust the swappiness setting, use the following procedure. ====


Verify the current swappiness setting. The default value is 60.

  cat /proc/sys/vm/

To temporarily adjust the value of swappiness, use the sysctl utility. The following command lowers the value to 50.

  sudo sysctl vm.swappiness=10
  
To permanently change this value, edit the sysctl.conf file. Add the following line to the bottom of the file.


  File: /etc/sysctl.conf
  vm.swappiness=10
