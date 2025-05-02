---
tags:
  - kvm
  - backup
  - script
---

====== External snapshot commandes utiles ======

concept :

  * backing_file : the original disk image of a VM (read only)
  * overlay_file : the snapshot image (writeable)

====== Pre requis ======

info sur la vm

  virsh domifaddr <nom du domaine> --full
    
  ---------------------------------------------------------------
  vnet11     52:54:00:f7:61:38    ipv4         192.168.122.243/24
  
info sur les images

  virsh domblklist <nom du domaine>  --details
  
   Type   Device   Target   Source
   --------------------------------------------------------------------------
   file   cdrom    sda      /var/lib/libvirt/images/dev19/dev19-cidata.iso
   file   disk     vda      /var/lib/libvirt/images/dev19/dev19.snap1


===== Creating an external disk snapshot =====


  virsh snapshot-create-as <domaine name> <snap name> --disk-only --quiesce
  

quiesce is a file file system freeze mechanism. This pits the guest file system into a consistent state.
The qemu-guest agent needs to be installed on and running inside

List of snapshots

  virsh snapshot-list <nom de domaine>
    


