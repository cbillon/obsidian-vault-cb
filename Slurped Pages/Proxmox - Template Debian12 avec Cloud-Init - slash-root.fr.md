---
link: https://slash-root.fr/proxmox-template-debian12-avec-cloud-init/
byline: Julien LOUIS
site: slash-root.fr
date: 2023-11-09T14:23
excerpt: Création d'un template de VM Debian 12 avec Cloud-Init sous Proxmox.
slurped: 2026-02-24T10:16
title: "Proxmox : Template Debian12 avec Cloud-Init - slash-root.fr"
tags:
  - Proxmox
  - Cloud-Init
---

Cette procédure va permettre de créer un template de VM Debian 12 pour Proxmox avec Cloud-Init.

L'intérêt sera de cloner ce template, puis de pouvoir se connecter directement en ssh au 1er démarrage de la VM. Le nom d'hôte, l'adressage ip, la(es) clef(s) ssh seront configurés à l'initialisation de la VM grâce à cloud-init.

Ce template pourra éventuellement être utilisé par la suite avec une solution telle que Terraform / OpenTofu.

## Création du template

- Se connecter sur un shell `root` de l'hôte Proxmox `pvehost` :

```
ssh root@pvehost
```

- Télécharger la dernière image cloud de Debian 12 :

```
wget https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2
```

- Installer `libguestfs-tools` si nécessaire et ajouter `qemu-guest-agent` à l'image cloud précédemment téléchargée :

```
apt install -y libguestfs-tools
virt-customize -a debian-12-generic-amd64.qcow2 --install qemu-guest-agent
```

- Créer une VM avec l'ID 666 qui sera le template :

```
qm create 666 --name debian12-cloudinit --net0 virtio,bridge=vmbr1 --scsihw virtio-scsi-single
```

Elle sera nommée `debian12-cloudinit` et la 1ère carte réseau sera rattachée au `vmbr1` du serveur Proxmox.

- Ajouter un nouveau disque dur, importé depuis l'image cloud, de 20 G à la VM :

```
qm set 666 --scsi0 local:0,iothread=1,backup=off,format=qcow2,import-from=/root/debian-12-generic-amd64.qcow2
qm disk resize 666 scsi0 20G
```

- Définir le disque dur en 1er boot sur la VM :

```
qm set 666 --boot order=scsi0
```

- Définir le nombre de coeurs et la mémoire RAM de la VM :

```
qm set 666 --cpu host --cores 2 --memory 4096
```

- Attacher un disque Cloud-Init à la VM :

```
qm set 666 --ide2 local:cloudinit
qm set 666 --agent enabled=1
```

- Ajuster les paramètres dans les options `Cloud-Init` qui figureront dans le template (si nécessaire) :

![cloudinit](https://i0.wp.com/slash-root.fr/wp-content/uploads/2023/11/01-cloudinit.png?w=696&ssl=1)

Ces options sont disponibles dans la WebUI du serveur Proxmox au niveau de la VM.

Plus de détails ici [https://pve.proxmox.com/wiki/Cloud-Init_Support](https://pve.proxmox.com/wiki/Cloud-Init_Support) sur les différents paramétrages.

- Convertir la VM en template :

```
qm template 666
```

## Utilisation du template

- Cloner entièrement le template en VM.

![clone](https://i0.wp.com/slash-root.fr/wp-content/uploads/2023/11/02-clone.png?w=696&ssl=1)

- Ajuster les paramètres `Cloud-Init` de la VM si nécessaire.
    
- Démarrer la VM et récupérer l'IP (si nécessaire) :
    

![ip](https://i0.wp.com/slash-root.fr/wp-content/uploads/2024/05/03-ip.png?w=696&ssl=1)

- Se connecter en SSH sur la VM avec la clef ssh renseignée dans cloud-init :

```
julien@mypc:~$ ssh -i .ssh/demo admin@172.16.20.15
...
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.16.20.15' (ED25519) to the list of known hosts.
Linux vm-deb12-01 6.1.0-13-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.55-1 (2023-09-29) x86_64
...
admin@vm-deb12-01:~$ sudo -l
...
User admin may run the following commands on vm-deb12-01:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
```

Par défaut l'utilisateur définit par cloud-init appartient au groupe `sudo` et peut exécuter absolument tout sans saisir son mot de passe.