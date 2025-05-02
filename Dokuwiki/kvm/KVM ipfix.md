---
tags:
  - kvm
  - ip
---

====== Attribution d'une IP fixe ======

  * Recuperer la mac addresse de la VM

    virsh domifaddr <vm name>

  * Mettre à jour le reseau

    virsh net-edit default
    
{{:kvm:ip_fixe.png?400|}} 

Ajouter la mac adress avec l' ip associée

Puis 

   virsh net-destroy default
   virsh net-start default
   sudo systemctl restart libvirtd
   virsh reboot <vm_name>   
