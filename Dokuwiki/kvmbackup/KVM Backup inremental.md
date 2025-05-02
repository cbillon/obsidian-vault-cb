---
tags:
  - kvm
  - backup
  - incremental
---





====== Backup-begin incremental ======
 

===== How to merge incremental backups generated with virsh backup-begin? =====

[[https://www.spinics.net/linux/fedora/libvirt-users/msg13679.html]]

I'm trying to create incremental backups that I could restore when necessary. All backups are being generated fine but I couldn't find a way to recreate an image using all or some of the backup files.

For example, my domain is called jammy and is running.

  jammy-backup.xml:
  
  <domainbackup>
    <incremental></incremental>
  </domainbackup>

  jammy-checkpoint.xml:
  
  <domaincheckpoint>
    <disks>
      <disk name="vda" checkpoint="bitmap"/>
    </disks>
  </domaincheckpoint>

~# virsh backup-begin jammy jammy-backup.xml jammy-checkpoint.xml

* This command with these files generates backup file in /var/lib/libvirt/images appending the checkpoint timestamp in file name (jammy.qcow2 -> jammy.qcow2.TIMESTAMP).

~# virsh checkpoint-list jammy
 Name         Creation Time
-----------------------------------------
 1666006874   2022-10-17 08:41:14 -0300
 
~# ls -lh /var/lib/libvirt/images
-rw-r--r-- 1 libvirt-qemu kvm  2,6G out 18 14:38 jammy.qcow2
-rw------- 1 root root         1,5G out 17 08:41 jammy.qcow2.1666006874

If I create a new domain using jammy.qcow2.1666006874 as disk, everything works good.

Then, I've created some incremental backups.

jammy-backup.xml:
<domainbackup>
  <incremental>1666006874</incremental>
</domainbackup>

~# virsh backup-begin jammy jammy-backup.xml jammy-checkpoint.xml

~# ls -lh /var/lib/libvirt/images
-rw-r--r-- 1 libvirt-qemu kvm  2,6G out 18 14:38 jammy.qcow2
-rw------- 1 root root         1,5G out 17 08:41 jammy.qcow2.1666006874
-rw------- 1 root root         247M out 17 14:42 jammy.qcow2.1666010735

At this point, if I need to restore the backup with checkpoint 1666010735, how can I create a new image using jammy.qcow2.1666010735 and jammy.qcow2.1666006874? 

I would like to merge the incremental backup jammy.qcow2.1666010735 into the full backup jammy.qcow2.1666006874, to get a new image so I can create a new domain using it as disk. Am I doing it the right way?

Ubuntu 22.04 LTS
KVM
libvirt 8.0.0-1ubuntu7.1
qemu 1:6.2+dfsg-2ubuntu6.4

To restore incremental backups you need to restore the layering of the
qcow2 images in the order they were created. This is needed as a single
incremental image contains only differences from the previous backup
(full or incremental). You need to do that until you reach your original
full backup.

The metadata in question is the backing image and backing image path.
The full backing chain should look like:

incr4.qcow -> incr3.qcow2 -> incr2.qcow2 -> incr1.qcow2 -> fullbackup.qcow2

The '->' represents the backing image relationship, thus 'incr4.qcow2'
whould list 'incr3.qcow2' as it's backing image.

The steps how to fix the image using qemu-img are the same as in the
follwing guide:

https://www.libvirt.org/kbase/backing_chains.html#vm-refuses-to-start-due-to-misconfigured-backing-store-format

After you do that you then should use e.g. 'qemu-img convert' to copy
out the data into a new image and use that new image as base for your
new VM.

Note that using the backup images directly as source for a disk used by
VM invaidates/destroys any further increments in that chain, as that VM
then can write data into the image.