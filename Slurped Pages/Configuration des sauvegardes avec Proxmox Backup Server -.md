---
link: https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/
byline: Thomas COLIN
date: 2024-06-09T20:12
excerpt: Découvrez comment sauvegarder vos machines virtuelles avec Proxmox Backup Server, une solution open source. Apprenez à installer, configurer et automatiser vos sauvegardes pour garantir une protection complète et efficace de vos données.
slurped: 2025-02-28T18:45
title: Configuration des sauvegardes avec Proxmox Backup Server -
tags:
  - Proxmox
  - backup-server
  - backup
---

![](https://colinfo.fr/wp-content/uploads/2024/06/Configuration-des-sauvegardes-avec-Proxmox-Backup-Server-1.png)

- [Introduction](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Introduction "Introduction")
- [Configuration du NAS Synology](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Configuration_du_NAS_Synology "Configuration du NAS Synology")
- [Installation de Proxmox Backup Server](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Installation_de_Proxmox_Backup_Server "Installation de Proxmox Backup Server")
- [Configuration du partage « Backup » dans PBS](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Configuration_du_partage_%C2%AB_Backup_%C2%BB_dans_PBS "Configuration du partage « Backup » dans PBS")
- [Ajout d’un datastore vers le NAS](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Ajout_dun_datastore_vers_le_NAS "Ajout d’un datastore vers le NAS")
- [Création d’un compte dédié à la sauvegarde](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Creation_dun_compte_dedie_a_la_sauvegarde "Création d’un compte dédié à la sauvegarde")
- [Configuration des sauvegardes](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Configuration_des_sauvegardes "Configuration des sauvegardes")
- [Connexion avec le serveur de sauvegarde](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Connexion_avec_le_serveur_de_sauvegarde "Connexion avec le serveur de sauvegarde")
- [Sauvegarde manuelle de machine virtuelle](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Sauvegarde_manuelle_de_machine_virtuelle "Sauvegarde manuelle de machine virtuelle")
- [Création des jobs automatisés de sauvegarde](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Creation_des_jobs_automatises_de_sauvegarde "Création des jobs automatisés de sauvegarde")
- [Restauration des sauvegardes](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Restauration_des_sauvegardes "Restauration des sauvegardes")
- [Conclusion](https://colinfo.fr/configuration-des-sauvegardes-avec-proxmox-backup-server/#Conclusion "Conclusion")

## Introduction

Après avoir vu comment [installer et configurer Proxmox](https://colinfo.fr/installation-et-configuration-de-proxmox/), nous allons voir comment sauvegarder nos machines virtuelles !

Proxmox dispose de sa propre solution de sauvegarde open source et elle se nomme Proxmox Backup Server. Elle permet entre autre de sauvegarder et restaurer des machines virtuelles, des conteneurs, des hôtes physiques. Il prend en charge également les sauvegardes incrémentielles, la déduplication, la compression et le chiffrement authentifié, l’exportation des sauvegardes sur bandes, la synchronisation à distance, etc.

Cette solution est utilisable de plusieurs façons, soit en bare-metal, en machine virtuelle sur un NAS par exemple ou même directement sur votre serveur Proxmox en tant que machine virtuelle. C’est la dernière solution que nous allons voir.

## Configuration du NAS Synology

Je possède un NAS qui servira de datastore pour stocker les sauvegardes

Je vais donc créer un partage « Backup » sans activer la corbeille.

![Une image contenant texte, capture d’écran, logiciel, nombre
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-26.png)

Comme je vais utiliser le protocole SMB (qui est déjà activé), je configure les permissions pour autoriser seulement un utilisateur spécifique à avoir accès. Vous pouvez configurez également le protocole NFS si vous le voulez.

![Une image contenant texte, capture d’écran, logiciel, nombre
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-27.png)

## Installation de Proxmox Backup Server

Nous devons nous procurez l’ISO pour l’envoyer dans Proxmox.

Voici le lien pour télécharger l’ISO de Proxmox Backup Server : [https://www.proxmox.com/en/downloads/proxmox-backup-server/iso](https://www.proxmox.com/en/downloads/proxmox-backup-server/iso)

La prochaine étape est donc la création de Proxmox Backup Server sur l’hyperviseur Proxmox.

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-28.png)

On ajoute l’ISO à la machine virtuelle et le type Linux en Guest OS

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-29.png)

Choix des configurations systèmes

![Une image contenant texte, capture d’écran, logiciel, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-30.png)

Configuration du stockage

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-31.png)

Configuration du CPU

![Une image contenant capture d’écran, texte, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-capture-decran-texte-logici-4.png)

Allocation de la mémoire

![Une image contenant texte, capture d’écran, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-police-10.png)

Choix des paramètres réseaux

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-32.png)

Un ultime résumé avant la création

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-33.png)

Et on peut démarrer la machine

Sur la console d’administration, on choisit « Install Proxmox Backup Server (Graphical) »

![Une image contenant texte, Police, capture d’écran, logo
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-police-capture-decran-2.png)

On accepte les CGU

![Une image contenant texte, capture d’écran, Police, Site web
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-police-11.png)

Sélection du disque d’installation pour le système

![Une image contenant texte, capture d’écran, Police, Site web
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-police-12.png)

Choix de la localisation et de la Timezone

![Une image contenant texte, capture d’écran, Site web, Page web
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-site-w-3.png)

Sélection d’un mot de passe administrateur

![Une image contenant texte, capture d’écran, Police, Site web
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-police-13.png)

Configuration du réseau

![Une image contenant texte, capture d’écran, Site web, Page web
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-site-w-4.png)

Un récapitulatif est visualiser, puis on installe le serveur

![Une image contenant texte, capture d’écran, logiciel, Page web
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-34.png)

![Une image contenant texte, capture d’écran, Site web, Page web
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-site-w-5.png)

On peut maintenant se connecter à l’interface graphique sous cette URL : https://votre-ip:8007

![Une image contenant capture d’écran, texte, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-capture-decran-texte-logici-5.png)

## Configuration du partage « Backup » dans PBS

En CLI sur votre serveur psb, nous allons ajouter le partage « Backup » en tant que point de montage qui sera notre repository avec le protocole SMB.

Vous pouvez accéder à la CLI de votre serveur dans l’onglet Administration > Shell

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-35.png)

Nous allons créer un dossier qui monter le partage

```
mkdir /mnt/backup
```

Pour monter le partage manuellement en ligne de commande :

```
mount -t cifs -o username=youruser //192.168.1.201/Backup /mnt/backup
```

On saisit le mot de passe et on vérifie que le périphérique est bien monté avec la commande suivante :

df -h

![Une image contenant texte, capture d’écran, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-police-14.png)

Le problème de cette solution est qu’à chaque redémarrage, le montage ne se fera pas automatiquement.

Nous allons donc devoir le rendre persistant en éditant le fichier /etc/fstab

Pour plus de sécurité nous allons créer un fichier qui ne comprend que les credentials, accessible uniquement par l’utilisateur root.

Pour créer et éditer le fichier

```
nano /home/root/.cred
```

Le contenu du fichier :

```
username=youruser
password=yourpassword
```

Pour ne donner les droits que à l’utilisateur root :

```
chmod 600 /home/root/.cred
```

On édite le fichier qui permet d’ajouter les lecteurs montés automatiquement :

```
nano /etc/fstab
```

Et on ajoute cette ligne (à adapté selon votre environnement)

```
//192.168.1.201/Backup /mnt/backup cifs credentials=/home/root/.cred 34 34
```

34 fait référence à l’UID et le GUID de l’utilisateur « backup » nécessaire pour réaliser les sauvegardes. Vous pouvez adaptez les ID suivant le retour de cette commande :

```
id backup
```

![](https://colinfo.fr/wp-content/uploads/2024/06/word-image-30034-22.png)

Ensuite, on redémarre systemctl et on monte le lecteur

```
systemctl daemmon-reload
```

Pour monter les partages dans le fichier fstab, il suffit d’exécuter la commande suivante :

```
mount -a
```

On peut toujours vérifier la bonne exécution avec la commande df -h

![Une image contenant texte, capture d’écran, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-police-15.png)

## Ajout d’un datastore vers le NAS

Le lecteur « Backup » est dorénavant monté. Nous allons maintenant créer, sur l’interface graphique, le Datastore à la solution de sauvegarde.

On se rend dans l’onglet Datastore :

![Une image contenant texte, capture d’écran, Police, conception
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-police-16.png)

Puis on insère le point de montage du NAS

![Une image contenant texte, capture d’écran, Police, logiciel
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-police-17.png)

Et je saisi des options, dans ce cas je souhaite qu’il conserve le dernier instantané, les 3 dernières sauvegardes journalières et les 2 dernières sauvegardes mensuelles.

![Une image contenant texte, capture d’écran, logiciel
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-36.png)

Voilà, notre Datastore est ajouté !

Nous avons plusieurs onglets qui permettent de suivre les statistiques, les jobs effectués, etc.

![Une image contenant capture d’écran, texte, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-capture-decran-texte-logici-6.png)

## Création d’un compte dédié à la sauvegarde

Pour continuer dans les bonnes pratiques de sécurité, il est recommandé de ne pas utiliser le compte root pour effectuer les sauvegardes.

Dans Configuration > Access Control, je clique sur « Add » puis je crée un utilisateur, dans mon cas il se nommera « proxsave »

![Une image contenant capture d’écran, logiciel, Logiciel multimédia, texte
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-capture-decran-logiciel-log-1.png)

J’autorise cet utilisateur dans ma banque de données dans l’onglet Permissions en lui attribuant le rôle « DatastoreAdmin »

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-37.png)

Dernière étape, on récupère le fingerprint qui nous sera utile pour se connecter au serveur de backup depuis l’hyperviseur.

On peut, sur la page « Dashboard », sélectionner « Show Fingerprint » pour avoir la valeur de ce dernier.

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-38.png)

### Connexion avec le serveur de sauvegarde

De retour sur l’hyperviseur Proxmox, nous allons faire une jonction avec le serveur de sauvegarde. Pour cela, nous allons nous rendre dans l’onglet « Datacenter » puis « Storage », sélectionner « Add » et enfin « Proxmox Backup Server »

On renseigne les différentes informations de connexion demandées

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-39.png)

Le second onglet « Backup Retention » a déjà été configuré dans le PBS, et vous avez également la possibilité de chiffrer vos données dans l’onglet « Encryption » si vous le souhaitez

![Une image contenant texte, capture d’écran, logiciel, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-40.png)

Le stockage pbs est ajouté !

![Une image contenant Logiciel multimédia, logiciel, capture d’écran
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-logiciel-multimedia-logiciel-1.png)

### Sauvegarde manuelle de machine virtuelle

Il est possible d’effectuer une sauvegarde manuelle d’une machine virtuelle, dans l’onglet « Backup » de cette dernière.  
Je sélectionne « Backup now »

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-41.png)

Je choisi mon storage « pbs »

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-42.png)

L’événement TASK OK signifie que la sauvegarde s’est correctement déroulée

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-43.png)

### Création des jobs automatisés de sauvegarde

Afin d’éviter à avoir à faire les sauvegardes manuellement pour chaque machine virtuelle, je vais créer des jobs de backup qui travailleront automatiquement.

Cela se configure dans l’onglet « Datacenter » puis « Backup » et je sélectionne « Add »

![Une image contenant logiciel, Logiciel multimédia, texte, Logiciel de graphisme
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-logiciel-logiciel-multimedia-1.png)

Ensuite, j’ajoute plusieurs jobs :

- 1 hebdomaire
- 1 mensuel
- 1 annuel

![Une image contenant texte, capture d’écran, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-police-18.png)

Voici un exemple pour le job hebdomadaire :

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-44.png)

Votre environnement est désormais protégé !

## Restauration des sauvegardes

Pour restaurer les sauvegardes, c’est très simple.

Vous pouvez consulter le volume de donnée dans l’onglet à votre gauche et sélectionner « Backup » ensuite. Les sauvegardes existantes seront affichées.

![Une image contenant logiciel, Logiciel multimédia, texte, Logiciel de graphisme
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-logiciel-logiciel-multimedia-2.png)

Dans mon exemple, je n’ai que celle que j’ai effectué tous à l’heure.

Je peux décider de faire de la restauration de fichier uniquement, ou restaurer entièrement la machine virtuelle à l’aide de l’option proposée en haut.

Je sélectionne « Restore » pour restaurer l’entièreté de la VM. Je sélectionne le volume de stockage, et je peux modifier à mon aisance certains paramètres tels que le nom, le CPU, etc.

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-texte-capture-decran-logici-45.png)

Une fois la restauration terminée, une fenêtre de confirmation d’état s’affiche.

![Une image contenant Appareils électroniques, texte, capture d’écran, ordinateur
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/06/une-image-contenant-appareils-electroniques-texte.png)

Je peux constater que la VM « pfsense-restored » a bien été créé.

![](https://colinfo.fr/wp-content/uploads/2024/06/word-image-30034-43.png)

## Conclusion

Grâce à ce tutoriel, nous avons vu comment mettre en place Proxmox Backup Server pour sauvegarder et restaurer facilement vos machines virtuelles, conteneurs et hôtes physiques Proxmox. En configurant un NAS Synology comme repository final et en créant des jobs de sauvegarde automatisés, nous disposons désormais d’une solution de sauvegarde robuste et complète. En cas de perte de données, vous pouvez facilement restaurer vos machines virtuelles à partir des sauvegardes stockées sur votre NAS.