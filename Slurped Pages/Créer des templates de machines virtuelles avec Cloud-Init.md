---
link: https://www.bejean.eu/2023/03/24/creer-des-templates-de-vm-avec-cloud-init
site: Béjean Développement
date: 2023-03-24T09:00
excerpt: Créer des templates de machines virtuelles avec Cloud-Init - Béjean Développement - Nicolas Béjean - Expert E-commerce - Freelance - Développeur PHP - Expert Magento, Adobe Commerce et Laravel - DevOps - Adhérent Made In Jura
twitter: https://twitter.com/@nicolasbejean
slurped: 2025-02-28T18:29
title: Créer des templates de machines virtuelles avec Cloud-Init
tags:
  - Proxmox
  - Cloud-Init
  - VM
  - ansible
---

Cet article va vous expliquer comment créer des templates de machines virtuelles avec Cloud-Init.

## Qu'est-ce que Cloud-init ?

Cloud-init va très simplement nous permettre de modifier les paramètres d'une machine virtuelle tels que l'utilisateur, le mot de passe, les clés SSH, le domaine DNS, les serveurs DNS et la configuration réseau.

## Création du template

Avant de procéder à la création du template, il est nécessaire de télécharger une image prête à fonctionner avec Cloud-init. Pour se faire, j'utilise l'image Ubuntu Server Lunar que l'on retrouve sur le site [cloud-images.ubuntu.com](https://cloud-images.ubuntu.com/).

Aussi, pour cet article, j'utilise Proxmox pour le déploiement de mes machines virtuelles. Pour télécharger l'image, cliquer sur le nœud de votre cluster Proxmox, où sera créée la machine virtuelle, pour ouvrir un shell. Ensuite, lancer la commande suivante :

```
wget https://cloud-images.ubuntu.com/lunar/current/lunar-server-cloudimg-amd64.img
```

Nous pouvons désormais créer la machine virtuelle.

### Création de la machine virtuelle

Pour créer un template, il faut passer par la création d'une machine virtuelle. Nous allons réaliser cette opération en passant par la ligne de commande, soit directement dans le shell. Pour ce faire, lancer la commande suivante :

```
qm create 1000 --name template-u2304 --cores 2 --memory 512 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci
```

En exécutant cette commande, nous aurons une nouvelle machine virtuelle nommée `template-u2304` avec l'identifiant 1000, 2 cœurs CPU, 512 Mo de RAM, une carte réseau virtio et un contrôleur SCSI virtio.

Ceci étant, la machine virtuelle ne dispose d'aucun disque dur, nous allons donc créer un disque en l'important depuis l'image cloud que nous avons téléchargé, pour ce faire, exécuter la commande ci-dessous :

```
qm set 1000 --scsi0 local-lvm:0,import-from=/path/to/lunar-server-cloudimg-amd64.img
```

Avant d'exécuter la commande, il est important de modifier les valeurs telles que l'identifiant, le nom du disque de stockage ainsi que le chemin vers l'image cloud.

Ensuite, nous devons modifier l'ordre de démarrage des disques en forçant le boot uniquement sur le disque `scsi0` :

```
qm set 1000 --boot order=scsi0
```

### Ajouter le disque cloud-init

Cloud-init fonctionne comme un lecteur de disque. Il est donc nécessaire d'ajouter le lecteur de disque `cloudinit` à la machine virtuelle. Cela se fait par le biais de la commande :

```
qm set 1000 --ide2 local-lvm:cloudinit
```

Puis, nous allons créer un port série et l'utiliser en tant qu'affichage. Ceci est requis pour certaines images cloud-init.

```
qm set 1000 --serial0 socket --vga serial0
```

### Convertir la VM en template

La dernière étape consiste à convertir la machine virtuelle en template. Pour ce faire, lancer la commande suivante :

```
qm template 1000
```

Vous disposez désormais d'un template de machine virtuelle utilisant cloud-init. Ce template va nous permettre de créer une nouvelle machine virtuelle.

## Configuration de cloud-init

Il est possible de configurer les paramètres cloud-init du template en cliquant sur `Cloud-init` dans les paramètres du template. Depuis cette page, vous aurez la possibilité de modifier les paramètres suivants :

- Nom de l'utilisateur
- Mot de passe de l'utilisateur
- Domaine DNS
- Serveur DNS
- Clé SSH
- Configuration réseau

Une fois le ou les paramètres modifiés, il ne faut pas oublier de regénérer l'image.

## Création d'une machine virtuelle à partir du template

Vous pouvez désormais créer un clone partiel ou complet depuis le template que nous venons de créer.

## Playbook Ansible

Voici le playbook Ansible que j'utilise pour créer un template de machine virtuelle avec Cloud-init.

```
- name: "Create VM from cloud images"
  hosts: <host>
  vars:
      id: 1000
      vm_name: "template-u2304"
      vm_core: 2
      vm_ram: 512
      vm_bridge: vmbr0
      cloudimg_path: "/path/to/lunar-server-cloudimg-amd64.img"
      disk: "local"
      user: ansible
      sshkey_path: "/path/to/ssh/public_key>"
      ipconfig0: "ip=<ip-address>/<cidr>,gw=<ip-gateway>"
  tasks:
    - name: Create VM
      command: qm create {{ id }} --name {{ vm_name }} --cores {{ vm_core }} --memory {{ vm_ram }} --net0 virtio,bridge={{ vm_bridge }},firewall=1 --scsihw virtio-scsi-pci --agent 1

    - name: Import image
      command: qm set {{ id }} --scsi0 {{ disk }}:0,import-from={{ cloudimg_path }}

    - name: Set Boot Order
      command: qm set {{ id }} --boot order=scsi0

    - name: Add Cloudinit Disk
      command: qm set {{ id }} --ide2 {{ disk }}:cloudinit

    - name: Create Serial Port
      command: qm set {{ id }} --serial0 socket --vga serial0

    - name: Configure Cloud-init - User & Network
      shell: |
        qm set {{ id }} --ciuser {{ user }}
        qm set {{ id }} --sshkeys {{ sshkey_path }}
        qm set {{ id }} --ipconfig0 {{ ipconfig0 }}
```

Avant de lancer le playbook, n'oubliez pas de modifier les valeurs suivantes :

- host-proxmox
- ip-address
- cidr
- ip-gateway
- /path/to/lunar-server-cloudimg-amd64.img
- /path/to/ssh/public_key

[Lien vers le dépôt Froggit du playbook](https://lab.frogg.it/bejean-developpement/ansible/playbook/)

## Sources

- [Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support)
- [Proxmox VE : accélérer le déploiement des vm via les templates et cloud-init - Christophe Casalegno](https://www.youtube.com/watch?v=jiV4R4-DyMY&t=598s)