---
tags:
  - kvm
  - windows
  - install
---


====== Installation d'un serveur Windows ======

===== Installation OS =====

A partir de virt-manager, crer la nouvelle VM
Le fichier iso se trouve dans le répertoire iso

paramètres : mémoire 8192 4 CPU disk sata 80 Go


==== Ajout d'un nouveau disque ====


* Arreter la VM

* dans Virt-manager afficher le détail de la configuration
  cliquer Ajout d'un élément hardware
  Sélectionner disk sata
  
* Re démarrer la VM

* clic droit  sur le bouton Windows, sélectionner Gérer les disques
  Initialiser une partition non bootable type NTFS , noter la lettre attribuée 

===== Apres installation d'un serveur Windows =====


Il faut 
  * Activer l'acces au bureau
    dans le pannaeau Systeme Activer l'acces du Bureau à distance
  * Autoriser le ping
    Panneau de configuration > Systéme et sécurité > Pare Windows > Parametres avancés > Régle de trafic entrant1 
    Autoriser  **Partagee de fichiers ... Trafic entrant ICMP V4**
 

==== Installation Guest Agent ====

[[https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers]]

=== Using the ISO ===

You can download the latest [[https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso |stable]] or you can download the most recent build of the ISO. Normally the drivers are pretty stable, so one should try out the most recent release first.

You can access the ISO can in a VM by mounting the ISO with a virtual CD-ROM/DVD drive on that VM.

=== Wizard Installation ===


You can use an easy wizard to install all, or a selection, of VirtIO drivers. (use virt-manager)

Open the Windows Explorer and navigate to the CD-ROM drive.
Simply execute (double-click on) virtio-win-gt-x64 
Follow its instructions.
(Optional) use the virtio-win-guest-tools wizard to install the QEMU Guest Agent and the SPICE agent for an improved remote-viewer experience.
Reboot VM


After that, you have to install the qemu-guest-agent:

Go to the mounted ISO in explorer
The guest agent installer is in the directory guest-agent
Execute the installer with double click (either qemu-ga-x86_64.msi (64-bit) or qemu-ga-i386.msi (32-bit)
After that the qemu-guest-agent should be up and running. You can validate this in the list of Window Services, or in a PowerShell with:

  PS C:\Users\Administrator> Get-Service QEMU-GA

  Status   Name               DisplayName
  ------   ----               -----------
  Running  QEMU-GA            QEMU Guest Agent

If it is not running, you can use the Services control panel to start it and make sure that it will start automatically on the next bo

Pour vérifier 

   virsh qemu-agent-command dev20 '{"execute":"guest-info"}'

  

=== Parametrage ===


  virsh edit <domain>
  
on doit avoir

    <channel type='unix'>
      <target type='virtio' name='org.qemu.guest_agent.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
    </channel>
