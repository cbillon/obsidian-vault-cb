---
link: https://www.aukfood.fr/stockage-nfs-pour-proxmox/
byline: Guillaume Chéramy
site: Aukfood
date: 2024-12-24T15:32
excerpt: Ma problématique ici est de d'avoir accès aux ISO pour n'importe quel serveur d'un cluster Proxmox. Cela permet d'éviter de stocker les isos sur chaque serveur et donc de gaspiller de l'espace disque. Bien sûr ici je parle des isos mais ça pourrait aussi être pour les disques virtuels des vms ou les templates LXC. […]
twitter: https://twitter.com/@aukfood
slurped: 2025-05-28T19:25
title: Stockage NFS pour Proxmox - Aukfood
tags:
  - Proxmox
  - nfs
---

Ma problématique ici est de d'avoir accès aux ISO pour n'importe quel serveur d'un cluster Proxmox. Cela permet d'éviter de stocker les isos sur chaque serveur et donc de gaspiller de l'espace disque.

Bien sûr ici je parle des isos mais ça pourrait aussi être pour les disques virtuels des vms ou les templates LXC.

Pour rappel les stockages sont paramétrés dans **/etc/pve/storage**

```
dir: local
    path /var/lib/vz
    content backup,iso,vztmpl

lvmthin: local-lvm
    thinpool data
    vgname pve
    content images,rootdir

rbd: vms
    content rootdir,images
    krbd 0
    pool vms
```

## Serveur NFS

Vite fait la mise en place d'un serveur NFS sur le réseau local.

```
apt install nfs-kernel-server
```

Exposer le répertoire **/etc/exports**

```
/data/isos  192.168.33.0/24(rw,no_root_squash,no_subtree_check)
```

Relancer la configuration

```
exportfs -rav
```

## Client Proxmox

On monte le partage NFS

```
pvesm add nfs ISOS --server 192.168.33.24 --path /var/lib/vz/template/iso/ --export /data/isos --content iso
```

Ce qui va rajouter le bloc suivant dans **/etc/pve/storage** :-lb

```
nfs: ISOS
    export /data/isos
    path /var/lib/vz/template/iso/
    server 192.168.33.24
    content iso
```

![](https://www.aukfood.fr/wp-content/uploads/2024/12/liste-storage-nfs.png)

## NFS "status : unknow"

En passant par la commande pvesm j'avais mon stockage qui apparaissait comme inactif, je l'ai donc supprimé et créé depuis l'interface :

```
 pvesm status
Name             Type     Status           Total            Used       Available        %
ISOS              nfs   inactive               0               0               0    0.00%
```

![](https://www.aukfood.fr/wp-content/uploads/2024/12/add-nfs.png)

## Plus qu'à utiliser

On peut maintenant utiliser ce stockage comme les autres et les isos seront accessibles par chaque noeud du cluster :

![](https://www.aukfood.fr/wp-content/uploads/2024/12/donwload-iso.png)