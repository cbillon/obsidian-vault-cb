---
link: https://linuxfr.org/news/transferer-sa-licence-windows-dans-une-vm
byline: nlgranger
excerpt: L’actualité du logiciel libre et des sujets voisins (DIY, Open Hardware, Open Data, les Communs, etc.), sur un site francophone contributif géré par une équipe bénévole par et pour des libristes enthousiastes
slurped: 2024-06-06T16:47:48.371Z
title: Transférer sa licence Windows dans une VM
tags:
  - windows
  - licence
  - transfert
---

N. D. M. : [nlgranger](app://linuxfr.org/users/nlgranger) nous explique dans le journal qui est repris pour cette dépêche comment virtualiser un système pré-installé. Son expérience personnelle est relatée ici à la première personne (_je_). Rappelons aussi à tout hasard que si la [licence de ce système d’exploitation propriétaire](https://www.microsoft.com/en-us/Useterms/OEM/Windows/11/UseTerms_OEM_Windows_11_FRENCH.htm) permet apparemment une utilisation dans le cadre d’une telle virtualisation, celle-ci doit être faite sur une seule instance et sans utilisation comme serveur (ce que rappelle aussi le tutoriel mentionné plus loin, mais sur une version précédente du système). Et qu’il n’est possible de faire qu’une « _seule copie du logiciel à des fins de sauvegarde_ ».

Je viens d’acheter un PC et bien que j’aie fouillé et patienté longtemps, aucune offre sans OS n’arrivait ou ne convenait donc j’ai cédé à la vente forcée d’un PC avec Windows.  
Dans ce petit tutoriel, je vous explique comment déplacer cette licence Windows OEM vers une machine virtuelle (VM) sur le même PC. Si vous avez déjà une licence achetée à part, il vous suffit de la spécifier à l’installation, on s’intéresse ici au cas des licences OEM préinstallées sur la carte mère.  
L’intérêt de déplacer Windows dans une VM, c’est de ne pas bloquer une partie de l’espace disque avec une partition qui ne servira quasiment jamais. Là on peut déplacer l’image disque vers un stockage externe (disque ou clé USB) ou recréer la VM au besoin.  
Dans ce tutoriel j’utilise libvirt via le GUI virt-manager, mais je me suis largement appuyé sur [cet excellent tutoriel](https://www.emaxilde.net/posts/2022/01/28/recuperer-et-virtualiser-sa-licence-windows-sous-proxmox-ve.html) pour Proxmox d’Oliver Poncet que je vous invite à consulter.  
Je précise immédiatement qu’il n’est pas nécessaire d’avoir gardé le Windows préinstallé sur la machine, ni même de l’avoir démarré une seule fois.

## Dépendances

Pour parvenir à vos fins, il vous faudra les dépendances suivantes (en espérant ne rien oublier) :

- dmidecode pour lire les infos de la carte mère
- libvirt
- qemu/KVM
- swtpm pour émuler un TPM
- edk2-ovmf pour émuler un UEFI avec Secure Boot
- Le fichier.iso de Windows 11 disponible sur le [site de microsoft](https://www.microsoft.com/fr-fr/software-download/windows11).

Sous ArchLinux : `pacman – S dmidecode libvirt dnsmasq qemu-desktop swtmp`

J’ai utilisé virt-manager pour me faciliter la vie, j’imagine qu’on peut s’en sortir en ligne de commande directement avec qemu.

## Installation

### Récupérer les informations utiles

Pour valider automatiquement votre licence, Windows utilise des informations disponibles depuis la carte mère.

D’abord, le numéro de série, modèle, etc. :

```
$ sudo dmidecode
…
BIOS Information
    Vendor : LENOVO
    Version : NCCN16WW
    Release Date : 02/02/2024
…
    BIOS Revision : 1.16
    Firmware Revision : 1.16
…
System Information
    Manufacturer : LENOVO
    Product Name : 83E3
    Version : Yoga Pro 7 14AHP9
    Serial Number : 9F5OEMTZ
    UUID : a0a73af8-a886-4fbf-8f0d-5fd32c264a16
    SKU Number : LENOVO_MT_83E3_BU_idea_FM_Yoga Pro 7 14AHP9
    Family : Yoga Pro 7 14AHP9
```

_(j’ai édité le serial et l’uuid)_

Ensuite des informations enregistrées dans des tables ACPI :

```
sudo cat /sys/firmware/acpi/tables/MSDM > ~/VMs/MSDM.bin
sudo cat /sys/firmware/acpi/tables/SLIC > ~/VMs/SLIC.bin
```

### Créer la VM

La procédure démarre comme d’habitude, on suit l’assistant de virt-manager jusqu’au moment où il faut bien demander à modifier la configuration avant de démarrer.

Dans les options du BIOS, choisissez la config avec Secure Boot activé, chez moi le fichier se nomme `OVMF_CODE.secboot.4 m.fd`.

Ensuite il faut éditer directement le code XML qui décrit la configuration de la machine. Si c’est la première fois dans virt-manager, il faut cocher une case dans les paramètres de l’appli pour le rendre éditable.

Pour commencer, modifiez le nœud racine XML pour spécifier le schéma, sinon certaines options seront rejetées :

```
<domain type=“kvm” xmlns: qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
```

Mettez aussi à jour l’uuid pour qu’il corresponde à celui indiqué par dmidecode:

```
<uuid>a0a73af8-a886-4fbf-8f0d-5fd32c264a16</uuid>
```

Ensuite, il faut indiquer à qemu d’intégrer les tables ACPI :

```
<qemu: commandline>
<qemu: arg value='-acpitable'/>
<qemu: arg value='file=/home/ngranger/VMs/MSDM.bin'/>
<qemu: arg value='-acpitable'/>
<qemu: arg value='file=/home/ngranger/VMs/SLIC.bin'/>
</qemu: commandline>
```

Puis il faut ajouter les informations de la carte mère :

```
<sysinfo type=“smbios”>
<bios>
<entry name=“vendor”>LENOVO</entry>
<entry name=“version”>NCCN16WW</entry>
<entry name=“date”>02/02/2024</entry>
<entry name=“release”>1.16</entry>
</bios>
<system>
<entry name=“manufacturer”>LENOVO</entry>
<entry name=“product”>83E3</entry>
<entry name=“version”>Yoga Pro 7 14AHP9</entry>
<entry name=“uuid”>a0a73af8-a886-4fbf-8f0d-5fd32c264a16</entry>
<entry name=“serial”>9F5OEMTZ</entry>
<entry name=“family”>Yoga Pro 7 14AHP9</entry>
<entry name=“sku”>LENOVO_MT_83E3_BU_idea_FM_Yoga Pro 7 14AHP9</entry>
</system>
</sysinfo>
```

### Installation de Windows

La procédure est désormais habituelle.

Pour éviter d’avoir à utiliser un compte Microsoft, vous pouvez couper Internet au moment où Windows redémarre pour la configuration du système. Lorsque l’assistant en arrive à la connexion au réseau, tapez Maj-F10 pour ouvrir le terminal et exécutez la commande `oobe\BypassNRO`. Le PC redémarrera sur un assistant qui rend la connexion facultative.

Au démarrage, vous pourrez remettre Internet et vérifier que la licence est bien activée.

## Aller plus loin

- [Journal à l’origine de la dépêche](https://linuxfr.org/users/nlgranger/journaux/transferer-sa-licence-windows-dans-une-vm "https://linuxfr.org/users/nlgranger/journaux/transferer-sa-licence-windows-dans-une-vm") (44 clics)
- [Le tutoriel d'Oliver Poncet](https://www.emaxilde.net/posts/2022/01/28/recuperer-et-virtualiser-sa-licence-windows-sous-proxmox-ve.html "https://www.emaxilde.net/posts/2022/01/28/recuperer-et-virtualiser-sa-licence-windows-sous-proxmox-ve.html") (42 clics)
- [La documentation du fichier de config XML des VMs libvirt](https://libvirt.org/formatdomain.html "https://libvirt.org/formatdomain.html") (20 clics)
- [Complément pour les argument Qemu](https://libvirt.org/kbase/qemu-passthrough-security.html "https://libvirt.org/kbase/qemu-passthrough-security.html") (13 clics)
- [Un commentaire sur les restrictions de licence pour les VMs (a priori c'est OK)](https://law.stackexchange.com/questions/82404/can-you-transfer-a-windows-license-from-hardware-to-a-vm-running-on-the-hardware "https://law.stackexchange.com/questions/82404/can-you-transfer-a-windows-license-from-hardware-to-a-vm-running-on-the-hardware") (15 clics)
- [La documentation de Microsoft sur les tables ACPI servant à la validation de la licence](https://download.microsoft.com/download/1/3/8/13818231-a8ad-4fe7-b4e1-a63cbc5d6027/microsoft-software-licensing-tables.docx "https://download.microsoft.com/download/1/3/8/13818231-a8ad-4fe7-b4e1-a63cbc5d6027/microsoft-software-licensing-tables.docx") (19 clics)