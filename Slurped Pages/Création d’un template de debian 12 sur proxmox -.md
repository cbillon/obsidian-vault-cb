---
link: https://colinfo.fr/creation-dun-template-de-debian-12-sur-proxmox/
byline: Thomas COLIN
date: 2024-03-11T12:00
excerpt: Découvrez comment créer efficacement un template Debian 12 sur Proxmox en intégrant une image cloud-init pour une personnalisation avancée des instances. Suivez les étapes détaillées de téléchargement de l'image cloud-init, de création de la machine virtuelle, et de conversion en template via la ligne de commande. Cette approche permet une gestion rapide et reproductible des instances, idéale pour le déploiement à grande échelle.
slurped: 2025-02-28T18:40
title: Création d’un template de debian 12 sur proxmox -
tags:
  - Proxmox
  - template
  - Cloud-Init
---

![](https://colinfo.fr/wp-content/uploads/2024/03/Copie-de-infra-auto-heberge-2.png)

- [Introduction](https://colinfo.fr/creation-dun-template-de-debian-12-sur-proxmox/#Introduction "Introduction")
- [Installation](https://colinfo.fr/creation-dun-template-de-debian-12-sur-proxmox/#Installation "Installation")
- [Initialisation à cloud-init](https://colinfo.fr/creation-dun-template-de-debian-12-sur-proxmox/#Initialisation_a_cloud-init "Initialisation à cloud-init")
- [Création du template](https://colinfo.fr/creation-dun-template-de-debian-12-sur-proxmox/#Creation_du_template "Création du template")
- [Conclusion](https://colinfo.fr/creation-dun-template-de-debian-12-sur-proxmox/#Conclusion "Conclusion")

## Introduction

Dans ce guide, nous explorerons la création d’un template Debian 12 sur Proxmox en utilisant la ligne de commande et en intégrant une image cloud-init pour la personnalisation avancée des instances.

Cloud-init est un outil largement utilisé dans les environnements cloud. Nous allons comprendre son principe à travers cet article.

L’objectif final est de créer rapidement un template prêt à être déployé à grande échelle. Nous utiliserons Terraform pour provisionner Proxmox lors du prochain chapitre.

## Installation

### Initialisation à cloud-init

Nous allons donc créer une machine virtuelle à partir de l’image cloud-init debian12 en ligne de commande sur le serveur proxmox, et nous allons convertir cette image en template.

Cloud-init permet de gérer l’initialisation d’une machine virtuelle dans le cloud au premier démarrage. Elle permet de gagner du temps et d’éviter toutes les étapes d’installation du système d’exploitation.

Grâce à cloud-init, nous pouvons spécifier automatiquement certaines informations au premier démarrage de la machine :

- Nom de l’utilisateur
- Mot de passe de l’utilisateur
- Configuration réseau (IP, masque, passerelle et DNS) de la carte réseau
- Clé SSH

### Création du template

Nous allons créer un template depuis la ligne de commande du serveur proxmox. Vous pouvez vous connecter soit en SSH, soit depuis l’interface graphique.

La première étape consiste à récupérer l’ISO cloud-init (genericcloud pour mon besoin) de debian12 à cet URL : [https://cloud.debian.org/images/cloud/bookworm/20240211-1654/](https://cloud.debian.org/images/cloud/bookworm/20240211-1654/)

Dans mon cas, je souhaite que l’ISO soit stocké sur mon NAS alors je me place dans le répertoire ou est monté mon lecteur qui contient mes ISO :

```
cd /mnt/pve/nas_iso/template/iso 
```

Je lance le téléchargement avec l’utilitaire wget :

```
wget https://cloud.debian.org/images/cloud/bookworm/20240211-1654/debian-12-genericcloud-amd64-20240211-1654.qcow2
```

Nous pouvons créer une machine virtuelle avec cette commande, on va affecter certains paramètre de configuration comme le nombre de processeur, la mémoire, le disque, etc.

```
qm create 499 --name template-debian --cores 1 --memory 1024 --net0 virtio,bridge=vmbr0 --scsihw virtio-scsi-pci
```

On crée et importe le disque cloud-init au template :

```
qm set 499 --virtio0 vg_proxmox:0,import-from=/mnt/pve/nas_iso/template/iso/debian-12-genericcloud-amd64-20240211-1654.qcow2 
```

Cloud-init fonctionne comme un lecteur de disque, il faut donc l’ajouter en tant que lecteur

```

qm set 499 --ide2 vg_proxmox:cloudinit
```

On configure le template pour autoriser seulement le boot depuis le disque virtio :

```
qm set 499 --boot order=virtio0
```

Coud-init a également besoin d’une interface serial0 pour fonctionner, on l’ajoute :

```
qm set 499 --serial0 socket --vga serial0
```

Notre machine est prête, nous allons la convertir en template.

```
qm template 499
```

Dans les paramètres « Cloud-Init » du template, nous pouvons choisir de définir certaines valeurs comme le nom de l’utilisateur, le mot de passe, etc. Je vais laisser ces paramètres vierges car ils seront dynamiquement approvisionnés avec Terraform.

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-logici-27.png)

## Conclusion

La mise en place d’un modèle Debian 12 sur Proxmox, avec l’incorporation de l’image cloud-init, offre une approche efficace pour personnaliser les instances de manière avancée et les déployer à grande échelle. Nous avons vu de manière détaillée comment déployer le modèle en ligne de commande.