---
tags:
  - kvm
  - script
  - backup
---


====== Backup with external snapshot Old method ======

[[https://libvirt.org/kbase/live_full_disk_backup.html]]

This is the alternative in case you cannot use libvirt-7.2.0 and QEMU-4.2 for some reason. But this assumes you're using at least QEMU 2.1 and libvirt-1.2.9.

This backup approach is slightly more involved, and predates the virDomainBackupBegin() API: Assuming a guest with a single disk image, create a temporary live QCOW2 overlay (commonly called as "external snapshot") to track the live guest writes. Then backup the original disk image while the guest (live QEMU) keeps writing to the temporary overlay. Finally, perform the "active block-commit" operation to live-merge the temporary overlay disk contents into the original image — i.e. the backing file — and "pivot" the live QEMU process to point to it.


1. Check the current acitve disk image in use:

   virsh domblklist vm1


2. Create an external live snapshot:

   virsh snapshot-create-as --domain vm1 sn1 \
   --diskspec vda,file=/export/images/sn1.qcow2 \
   --disk-only --atomic --no-metadata

Note if you have QEMU guest agent installed in your virtual machine, try --quiesce option with
'snapshot-create-as' to ensure you have a consistent disk state.

Now, you have a disk image chain:

  base <-sn1 (live QEMU)


3. Take a backup of the original disk in backround:

   rsync -avh --progress /export/images/base.img \
   /export/images/copy.img


4. Commit the sn1 contents back into base:

   virsh blockcommit vm1 vda --active --verbose --pivot

Now, the chain is:

   base (live QEMU)

with contents from sn1 live committed into base.


5. Check the current acitve disk image in use again:

   virsh domblklist vm1
   
Now you can safely discard the overlay image — it is no longer valid; and its contents are now fully merged into the base image.