---
tags:
  - kvm
  - delete
---


====== Delete a VM ======

[[https://www.cyberciti.biz/faq/howto-linux-delete-a-running-vm-guest-on-kvm/]]

  * List les VM
  * List les images
  * stop the VM (shutdown ou destroy)
  * delete VM (undefine)

    virsh list --all
    virsh domblklist <domaine>  -> permet d'avoir les fichier images
    sudo qemu-img <domaine> --backing-chain --force-share <image> | grep backing
    virsh undefine --domain {VM_NAME_HERE} --remove-all-storage
    virsh undefine --domain {VM_NAME_HERE} --delete-snapshots --remove-all-storage
      
Les fichiers images doivent être supprimés manuellement

Vous ne pouvez pas pas détruire une VM ayant des snapshots

Sometimes virsh and qemu see different things. This is occasionally because one of them is blind, and less frequently because one of them is hallucinating. In this case I believe virsh is blind. Try this:

    sudo qemu-img info disk_image
    sudo qemu-img snapshot -d snapshot_id disk_image
    sudo qemu-img info disk_image

Pour forcer la suppression d'un snapshot

    virsh snapshot-delete <nom du domaine> --metadata <nom du snapshot>

Puis:

    virsh shutdown <nom de domaine>    
    virsh undefine --domain <nom de domaine> --remove-all-storage
    
    