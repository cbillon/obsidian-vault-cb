---
link: https://www.linuxtricks.fr/wiki/autofs-montage-automatique-sous-linux
site: Linuxtricks.fr
excerpt: |-
  Introduction
  AutoFS (pour Automount File System) est un service de montage automatique sous Linux qui permet de monter et de démonter...
slurped: 2025-02-22T08:51
title: "AutoFS : Montage automatique sous Linux - Wiki"
tags:
  - autofs
  - linux
  - mount
  - automount
---

Table des matières

1. [Introduction](https://www.linuxtricks.fr/wiki/autofs-montage-automatique-sous-linux#paragraph-introduction)
2. [Avantages et Inconvénients](https://www.linuxtricks.fr/wiki/autofs-montage-automatique-sous-linux#paragraph-avantages-et-inconvenients)
3. [Installation du service autofs](https://www.linuxtricks.fr/wiki/autofs-montage-automatique-sous-linux#paragraph-installation-du-service-autofs)
4. [Informations sur les fichiers](https://www.linuxtricks.fr/wiki/autofs-montage-automatique-sous-linux#paragraph-informations-sur-les-fichiers)
5. [Configuration](https://www.linuxtricks.fr/wiki/autofs-montage-automatique-sous-linux#paragraph-configuration)
    1. [NFS](https://www.linuxtricks.fr/wiki/autofs-montage-automatique-sous-linux#paragraph-nfs)
    2. [SMB](https://www.linuxtricks.fr/wiki/autofs-montage-automatique-sous-linux#paragraph-smb)

## Introduction

AutoFS (pour Automount File System) est un service de montage automatique sous Linux qui permet de monter et de démonter automatiquement des systèmes de fichiers quand c'est nécessaire.  
Plutôt que de monter manuellement les systèmes de fichiers, AutoFS surveille les demandes d'accès aux répertoires et les monte de manière transparente.

Les points de montage, les options et tous les paramètres sont configurés dans des fichiers de configurations spécifiques, en lieu et place du fichier **/etc/fstab**. C'est ce qu'on va voir dans cet article.

L'ensemble des actions seront réalisées en tant que root.  
Utilisant le point de montage /media pour des montages permanents, on utilisera ici /mnt pour les montages via AutoFS.

## Avantages et Inconvénients

Voici quelques avantages et inconvénients par rapport à un montage classique, histoire de comprendre le rôle d'AutoFS .

Avantages d'AutoFS :

- Montage à la demande : AutoFS monte les systèmes de fichiers à la demande, ce qui signifie qu'ils ne sont montés que lorsque cela est nécessaire. Cela peut éviter d'avoir des erreurs si le système de fichiers de destination n'est pas joignable (Ex: SMB ou NFS), notamment au démarrage du système
- Démontage automatique : AutoFS démonte les systèmes de fichiers de manière automatique après une période d'inactivité.
- Flexibilité : AutoFS offre une grande flexibilité dans la configuration des points de montage automatique. Il prend en charge une variété de systèmes de fichiers et permet une configuration fine des options de montage pour chaque système de fichiers.
- Intégration transparente : Une fois configuré, AutoFS fonctionne de manière transparente pour les utilisateurs et les applications. Les systèmes de fichiers sont montés et démontés automatiquement en arrière-plan sans nécessiter d'intervention de l'utilisateur.  
    

Inconvénients d'AutoFS :

- Complexité de configuration : La configuration initiale d'AutoFS peut être plus complexe par rapport au montage statique avec le fichier fstab.
- Performance : Dans certains cas, le montage à la demande peut entraîner des délais de réponse plus longs lors du premier accès à un système de fichiers, car AutoFS doit monter le système de fichiers sur demande. Cependant, une fois monté, l'accès ultérieur est généralement rapide.  
    

## Installation du service autofs

Passé cette introduction explicative, passons au vif du sujet.

Pour installer AutoFS, sur RHEL, Fedora et dérivés :

Dans le monde Debian :

Et on active le service :

Code BASH :

systemctl enable --now autofs

## Informations sur les fichiers

On va manipuler 2 fichiers :  
Le fichier **/etc/auto.master** qui est le fichier principal de configuration d'Autofs. Il définit les répertoires de base (points de montage) et les fichiers de configuration associés.  
Le fichier **/etc/auto.linuxtricks** qui sera le fichier dans lequel on va définir nos règles et options de montage.

Il ne sera pas utile de créer les points de montage, AutoFS se chargera de le créer si nécessaire.

## Configuration

### NFS

Prérequis sur la machine, avoir les outils NFS :

On va ensuite configurer le fichier principal d'AutoFS :

Ajout de cette ligne (la ligne doit être présente qu'une fois si on veut utiliser le fichier auto.linuxtricks pour plusieurs points à monter) :

Code :

`/mnt /etc/auto.linuxtricks`

Cela indique à autofs de surveiller le répertoire /mnt et d'utiliser le fichier /etc/auto.linuxtricks pour les définitions de montage.

Ne pas utiliser / si on veut monter un répertoire direct à la racine par exemple /nfs.  
On utilisera plutôt un /nfs par exemple dans lequel on créé d'autres dossiers à monter.

Le timeout par défaut (durée au boit de laquelle le montage se démonte) est de 300 secondes. On pourra modifier cette valeur avec l'option **--timeout** :

Code :

`/mnt /etc/auto.linuxtricks --timeout 30`

On va ensuite créer le fichier /etc/auto.linuxtricks :

Code :

`vim /etc/auto.linuxtricks`

Ensuite, on va ajouter cette ligne pour un montage NFS :

Code BASH :

ltnfs -fstype=nfs,rw,sync nfs1.linuxtricks.lan:/bkp/ltbackup/

Cette ligne spécifie que le répertoire **/mnt/ltnfs** sera monté à partir du partage NFS **/bkp/ltbackup** du serveur **nfs1.linuxtricks.lan** avec les options définies **rw,sync**

Après avoir modifié les fichiers de configuration, on recharge le service AutoFS :

Dès qu'on fait un appel :

Le montage se fait :

Code :

`Sys. de fichiers                   Taille Utilisé Dispo Uti% Monté sur   nfs1.linuxtricks.lan:/bkp/ltbackup   3,6T    2,0T  1,7T  54% /mnt/ltnfs`

### SMB

Prérequis sur la machine, avoir les outils SMB :

On va ensuite configurer le fichier principal d'AutoFS :

Ajout de cette ligne (la ligne doit être présente qu'une fois si on veut utiliser le fichier auto.linuxtricks pour plusieurs points à monter) :

Code :

`/mnt /etc/auto.linuxtricks`

Cela indique à autofs de surveiller le répertoire /mnt et d'utiliser le fichier /etc/auto.linuxtricks pour les définitions de montage.

Ne pas utiliser / si on veut monter un répertoire direct à la racine par exemple /smb.  
On utilisera plutôt un /smb par exemple dans lequel on créé d'autres dossiers à monter.

On va ensuite créer le fichier /etc/auto.linuxtricks :

Code :

`vim /etc/auto.linuxtricks`

Ensuite, on va ajouter cette ligne pour un montage SMB :

Code TEXT :

ltsmb -fstype=cifs,credentials=/root/.smbcredentials ://192.168.21.25/linuxtricks

Cette ligne spécifie que le répertoire **/mnt/ltsmb** sera monté à partir du partage SMB **//192.168.21.25/linuxtricks** avec les options définies **credentials=/root/.smbcredentials**

Ici, j'ai choisi de mettre dans le fichier **/root/.smbcredentials** les informations de connexion au partage :

Code BASH :

vim /root/.smbcredentials

C'est du classique :

Code :

`user=linuxtricksuser   pass=Linuxtrickspass123   domain=workgroup`

Mettre ce fichier dans /root, le cacher permet de le rendre inaccessible aux autres utilisateurs du système.

Après avoir modifié les fichiers de configuration, on recharge le service AutoFS :

Dès qu'on fait un appel :

Le montage se fait :

Code :

`Sys. de fichiers            Taille Utilisé Dispo Uti% Monté sur   //192.168.21.25/linuxtricks    50G     15G   36G  29% /mnt/ltsmb`

- [Partager sur Facebook](https://www.facebook.com/share.php?u=https%3A%2F%2Fwww.linuxtricks.fr%2Fwiki%2Fautofs-montage-automatique-sous-linux "Partager sur Facebook")
- [Partager sur Twitter](https://twitter.com/share?url=https%3A%2F%2Fwww.linuxtricks.fr%2Fwiki%2Fautofs-montage-automatique-sous-linux&text=AutoFS%20%3A%20Montage%20automatique%20sous%20Linux%20-%20Wiki "Partager sur Twitter")
- [Partager par email Partager par Email](https://www.linuxtricks.fr/cdn-cgi/l/email-protection#fcc38f899e96999f88c1bd898893baafd9ceccd9cfbdd9ceccb19392889d9b99d9cecc9d898893919d88958d8999d9cecc8f93898fd9ceccb095928984d9ceccd1d9ceccab959795da9e939885c19488888c8fd9cfbdd9cebad9ceba8b8b8bd29095928984888e959f978fd29a8ed9ceba8b959795d9ceba9d8988939a8fd1919392889d9b99d19d898893919d88958d8999d18f93898fd19095928984 "Partager par Email")
- [Imprimer Version imprimable](https://www.linuxtricks.fr/wiki/autofs-montage-automatique-sous-linux# "Version imprimable")