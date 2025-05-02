---
tags:
  - kvm
  - disk
---



[[https://www.cyberciti.biz/faq/how-to-add-disk-image-to-kvm-virtual-machine-with-virsh-command]]


===== Step 1 – Create the new disk image =====


Type the following command on KVM host to create a new disk image called ubuntu-box1-vm-disk1-5G with 5G size:

   cd /var/lib/libvirt/images/
   sudo qemu-img create -f raw ubuntu-box1-vm-disk1-5G 5G

Or you can use the dd command to create file using the following command

   sudo dd if=/dev/zero of=ubuntu-box1-vm-disk1-5G bs=1M count=5120 status=progress   
   
 
if : input file

of: output file

bs : blocksize

count: 5120

  * **Preallocated**
  * **thin-Provisionned** : space will be allocated as needed **seek=5120 count=0**

A note about qcow2 format

Raw disk image format is default. This format has the advantage of being simple and easily exportable to all other emulators. However, QEMU image format (qcow2) the most versatile format. If you need to take VM snapshots or AES encryption. Try qcow2 format. The syntax is as follows:
  
   sudo qemu-img create -f qcow2 ubuntu-box2-vm-disk1-5G 5G
   
===== Step 2 – Attach the disk to the virtual machine =====

Before you attache the disk to your VM, find out current disk name. Login to your VM and type the following command:

   df

OR
   sudo fdisk -l | grep '^Disk /dev/vd[a-z]'
   

  virsh attach-disk {vm-name} \
  --source /var/lib/libvirt/images/{img-name-here} \
  --target vdb \
  --live \
  --persistent

===== Step 3 – Partitioning the disk drive in VM =====

Now the guest named ‘ubuntu-box1’ now has a hard disk device called /dev/vdb. Login to your VM and type the following command to verify the same:

   sudo fdisk -l | grep '^Disk /dev/vd[a-z]'

Next, start fdisk for the new device:

   sudo fdisk /dev/vdb

Type 
  * n for a new partition. 
  * p for a primary partition. 
  * Choose an available partition number 1. 
  * Enter the default first cylinder by pressing Enter. 
  * Choose the entire disk is allocated by pressing Enter. 
  * Finally type p to verify new partition. 
  * Enter w to write changes and quit.

To format the new partition with the ext4 file system, enter:

   sudo mkfs.ext4 /dev/vdb

To verify:
   sudo lsblk /dev/sdb
  
Finally, you need to create a mount directory:

   sudo mkdir /disk2/

And mount the disk on the guest:

   sudo mount /dev/vdb1 /disk2/

Edit the file /etc/fstab
  
   sudo vi /etc/fstab

And update it as follows so that /dev/vdb1 get mounted across reboot persistently:

  /dev/vdb1    /disk2    ext4     defaults    0 0
