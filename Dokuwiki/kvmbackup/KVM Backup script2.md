---
tags:
  - kvm
  - backup
  - script
---

====== Efficient live full disk backup ======

[[https://libvirt.org/kbase/live_full_disk_backup.html]]

[[https://events19.linuxfoundation.org/wp-content/uploads/2017/12/Qemu-Backup-Status-Vladimir-Sementsov-Ogievskiy-Virtuozzo.pdf | Qemu Backup status ]]


===== Full disk backup using "push" mode =====

The below approach uses the modern backup API, virDomainBackupBegin(). This requires libvirt-7.2.0 and QEMU-4.2, or higher versions.

Start the guest:

   virsh start vm1

Domain 'vm1' started
Enumerate the disk(s) in use:

   virsh domblklist vm1
   Target   Source
   --------------------------------------
   vda      /var/lib/libvirt/images/vm1.qcow2

Begin the backup:

   virsh backup-begin vm1
   Backup started

Check the job status ("None" means the job has likely completed):

   virsh domjobinfo vm1
   Job type:         None

Check the completed job status:

   virsh domjobinfo vm1 --completed
   Job type:         Completed
   Operation:        Backup
   Time elapsed:     183          ms
   File processed:   39.250 MiB
   File remaining:   0.000 B
   File total:       39.250 MiB

Now we see the copy of the backup:

   ls -lash /var/lib/libvirt/images/vm1.qcow2*
   15M -rw-r--r--. 1 qemu qemu 15M May 10 12:22 vm1.qcow2
   21M -rw-------. 1 root root 21M May 10 12:23 vm1.qcow2.1620642185