---
tags:
  - Proxmox
  - windows-11
  - VM
  - tutorial
  - install
---

[ldugrain](https://ldugrain.xyz/windows11-proxmox/)

note: installation le 26 Février
It works fine!
remarques 
- les fichiers iso sont dans /varlib/vz/template/iso
- lors la création de la VM onglet OS
	- cocher Ad  additional drive for virtio drivers
	- Storage: local
	- Iso image: virtio-wn.0.1.266.iso
lors du démarrage de la machine virtuelle pour l'installation
faire entrée des que le message s'affiche : Press any key to boot from CD or DVD...

pour charger les drivers :
descendre dans l’arborescence selectionner amd64-> w11

## Installation de Windows 11 Proxmox

L’installation d’une machine virtuelle Windows 11 est un peu plus périlleuse que pour un Windows 10 quand on le fait sur Proxmox. En effet les prérequis de l’image de dernier OS de chez Microsoft s’est un peu durci avec le temps et la procédure est un peu différente de celle à laquelle nous sommes habitués. Le problème n’existe pas avec une image customisée, mais il est toujours conseillé de travailler avec un image officielle.

## I. Création de la machine virtuelle

1. Téléchargez l’ISO de Windows 1 à cette adresse : [https://www.microsoft.com/fr-fr/software-download/windows11](https://www.microsoft.com/fr-fr/software-download/windows11)

![ea74e441 257b 42c0 9bea bb09c08679f5](https://mlickn25kmop.i.optimole.com/w:598/h:404/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/ea74e441-257b-42c0-9bea-bb09c08679f5.png "Proxmox")

1. Nommer la machine vrtuelle et faire sa configuration de base :

Guest OS → Microsoft Windows et 11/2022

![d93bc32a c1d1 4222 b1bd 28b1c6825166](https://mlickn25kmop.i.optimole.com/w:721/h:230/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/d93bc32a-c1d1-4222-b1bd-28b1c6825166.png "Proxmox")
-**lors la création de la VM onglet OS
	- cocher Ad  additional drive for virtio drivers
	- Storage: local
	- Iso image: virtio-wn.0.1.266.iso**
	- 
Ensuite, activez « Qemu Agent » et sélectionnez « local-lvm » pour stocker la partition EFI et le module TPM 2.0, vous savez, le module qui vous empêche d’installer Windows 11 sur des postes plus anciens. Ici, vous n’aurez rien à craindre.

![efecac1b c63e 456c 90d8 1f1e5904585e](https://mlickn25kmop.i.optimole.com/w:719/h:330/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/efecac1b-c63e-456c-90d8-1f1e5904585e.png "Proxmox")

Dans « Cache », sélectionnez « Write back » pour de meilleures performances et « SCSI » dans « Bus/Device » car il remplace VirtIO Block qui est déprécié et IDE/SATA ne sont pas assez performants en lecture/écriture,

Sélectionnez également « SSD emulation » pour indiquer à la machine virtuelle quelle tourne sur un SSD et non un disque dur, cela permettra d’allonger la durée de vie du SSD. Pensez aussi à activer « Discard » pour une meilleure gestion du provisionnement dynamique.

![fe974972 4035 4c92 bb3b 4523cea342f1](https://mlickn25kmop.i.optimole.com/w:725/h:398/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/fe974972-4035-4c92-bb3b-4523cea342f1.png "Proxmox")

Dans « Cores, ajoutez 2 ou 4 cores selon votre configuration et dans « Type », changez la valeur par défaut « kvm64 » par « host ». Le mode « host » permet un accès direct au CPU sans couche d’abstraction. Il est obligatoire pour Windows car sans le mode « host », beaucoup d’applications risque de planter.

![5e6025b6 4938 40f0 bb51 702d9f58eea7](https://mlickn25kmop.i.optimole.com/w:721/h:240/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/5e6025b6-4938-40f0-bb51-702d9f58eea7.png "Proxmox")

Dans « Memory », sélectionnez 4096 Mib (4 Go de mémoire vive). Privilégier la mémoire fixe sans balloon pour Windows car Il est notoire que cette option est facteur d’instabilité et pose des soucis de performance (pour Linux, la mémoire dynamique ne pose aucun soucis). Voir le lien ci-dessous pour plus d’informations :

![ca45c969 b646 4ce7 8014 c0c63dab55e6](https://mlickn25kmop.i.optimole.com/w:717/h:230/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/ca45c969-b646-4ce7-8014-c0c63dab55e6.png "Proxmox")

Dans « Model », sélectionnez le pilote de para-virtualisation VirtIO pour la carte réseau. La carte ne sera pas reconnu par défaut par Windows, nous verront comment l’installer à l’avance.

Par défaut, le pare-feu du serveur est activé et il est connecté au pont vmbr0 (pont par défaut de Proxmox VE).

![4dcde047 74cb 4344 b8d4 e71ec298a887](https://mlickn25kmop.i.optimole.com/w:720/h:281/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/4dcde047-74cb-4344-b8d4-e71ec298a887.png "Proxmox")

Validez la configuration en cliquant sur « Finish ».

![4be85af2 5137 42a1 82e2 d5d15dc265e2](https://mlickn25kmop.i.optimole.com/w:724/h:520/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/4be85af2-5137-42a1-82e2-d5d15dc265e2.png "Proxmox")

Ensuite, téléchargez l’image contenant les pilotes de VirtIo à cette adresse :

[https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso?ref=tech2rue.sussudio.ovh](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso?ref=tech2rue.sussudio.ovh)

Et comme l’ISO de Windows 11, ajoutez là dans votre serveur en allant dans **Datacenter\nomduserveur\local\ISO Images\Upload\** Select File et cliquez sur « **Upload**« .

![d41296d1 fed7 4d5a 94cd ee780cbb493e](https://mlickn25kmop.i.optimole.com/w:568/h:396/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/d41296d1-fed7-4d5a-94cd-ee780cbb493e.png "Proxmox")

Il vous faudra également ajouter un lecteur CD pour pouvoir monter les pilotes VirtIO. Allez dans Hardware\Add\CD/DVD Drive

![2ee8452e c6dc 452d 841a 1ce9e85a8aa5](https://mlickn25kmop.i.optimole.com/w:301/h:396/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/2ee8452e-c6dc-452d-841a-1ce9e85a8aa5.png "Proxmox")

Et sélectionnez le lieu de stockage (local par défaut) et votre image de VirtIO.

![227aae12 6b32 44b8 8c89 c4101dff6b4b](https://mlickn25kmop.i.optimole.com/w:560/h:404/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/227aae12-6b32-44b8-8c89-c4101dff6b4b.png "Proxmox")

La préparation est faite, plus qu’à faire l’installation.

## II. Installation de Windows 11

1. Démarrez la machine virtuelle

![84b28d81 92f6 4b42 ba1c f5c11847595d](https://mlickn25kmop.i.optimole.com/w:368/h:354/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/84b28d81-92f6-4b42-ba1c-f5c11847595d.png "Proxmox")

1. Appuyez sur n’importe quelle touche pour démarrer le DVD.

![148bf776 d812 4c13 b9a0 df82e4a49928](https://mlickn25kmop.i.optimole.com/w:432/h:71/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/148bf776-d812-4c13-b9a0-df82e4a49928.png "Proxmox")
**faire entrée des que le message s'affiche : Press any key to boot from CD or DVD...**
1. Faire son installation normalement jusqu’à l’étape du disque dur

![2087c5d2 6c5b 43e3 a66d 68e2ef3ceb8d](https://mlickn25kmop.i.optimole.com/w:509/h:382/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/2087c5d2-6c5b-43e3-a66d-68e2ef3ceb8d.png "Proxmox")

Vous remarquerez que le stockage n’est pas reconnu. Normal car vous avez choisis le bus/device SCSI sous VirtIO et Windows n’embarque pas les pilotes VirtIO en natif. Il va donc falloir installer le pilote lié au stockage via le deuxième lecteur monté dans la machine virtuelle.

1. Cliquez sur « Load driver » puis sur « **Browse**« .

![69ac4b56 eea6 43f0 a445 25568f31d2d4](https://mlickn25kmop.i.optimole.com/w:524/h:395/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/69ac4b56-eea6-43f0-a445-25568f31d2d4.png "Proxmox")

![487fe786 4cfe 403c acea bee83f029b58](https://mlickn25kmop.i.optimole.com/w:417/h:179/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/487fe786-4cfe-403c-acea-bee83f029b58.png "Proxmox")

1. Naviguez dans le lecteur CD et allez dans le dossier « D:\vioscsi\w11\amd64 » et cliquez sur « **OK**« . Cela va charger le pilote.
**l'arboresence est dans mon cas amd64 -> w11
sélectionner w11**

![70f7a1aa 584e 484d ae34 916641ea267b](https://mlickn25kmop.i.optimole.com/w:278/h:289/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/70f7a1aa-584e-484d-ae34-916641ea267b.png "Proxmox")

1. Cliquez sur « Next » pour installer le pilote.

![0905bf77 025e 4cd8 b5aa c99c52edd8e5](https://mlickn25kmop.i.optimole.com/w:527/h:391/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/0905bf77-025e-4cd8-b5aa-c99c52edd8e5.png "Proxmox")

![6a53200c b5aa 4449 aaf9 bdcb77c2c1d3](https://mlickn25kmop.i.optimole.com/w:515/h:111/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/6a53200c-b5aa-4449-aaf9-bdcb77c2c1d3.png "Proxmox")

Enfin, sélectionnez le disque, créez des partitions si vous le souhaitez puis cliquez sur « Next ».

![11429b19 c36d 44c1 934a 773325b1e12e](https://mlickn25kmop.i.optimole.com/w:527/h:397/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/11429b19-c36d-44c1-934a-773325b1e12e.png "Proxmox")

La machine virtuelle ayant redémarré, arrivé à cet écran, sélectionnez « France » comme pays puis à l’écran suivant, sélectionnez « French » si vous avez un clavier AZERTY.

![c24b8d27 8694 4a6e a6f7 8bbbc2b63ba5](https://mlickn25kmop.i.optimole.com/w:841/h:615/q:mauto/f:best/https://ldugrain.xyz/wp-content/uploads/2024/08/c24b8d27-8694-4a6e-a6f7-8bbbc2b63ba5.png "Proxmox")