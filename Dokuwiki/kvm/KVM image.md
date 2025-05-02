---
tags:
  - kvm
  - image
  - qemu
---

====== qemu-img info without exclusive access ======


By default, ‘qemu-image info’ will throw an error if it cannot get exclusive access to the disk file it is trying to read.

Although it is not listed in the man page, you can use the ‘**force-share**’ flag to run the info command without exclusive access

   sudo qemu-img info mydisk.qcow2 --force-share