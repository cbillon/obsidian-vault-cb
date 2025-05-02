---
link: https://colinfo.fr/installation-et-configuration-de-proxmox/
byline: Thomas COLIN
date: 2024-03-03T23:09
excerpt: Découvrez l'installation et la configuration de Proxmox, une plateforme de virtualisation open source puissante, dans cet article détaillé. Explorez chaque étape, de la création d'une clé USB bootable à la configuration des réseaux, des mises à jour et du stockage. Apprenez également à résoudre les erreurs courantes et à optimiser votre environnement de virtualisation avec Proxmox.
slurped: 2025-02-28T18:43
title: Installation et configuration de Proxmox -
tags:
  - Proxmox
  - install
  - configuration
---

![](https://colinfo.fr/wp-content/uploads/2024/03/Copie-de-infra-auto-heberge.png)

- [Présentation de Proxmox](https://colinfo.fr/installation-et-configuration-de-proxmox/#Presentation_de_Proxmox "Présentation de Proxmox")
- [Installation de Proxmox](https://colinfo.fr/installation-et-configuration-de-proxmox/#Installation_de_Proxmox "Installation de Proxmox")
- [Configuration de Proxmox](https://colinfo.fr/installation-et-configuration-de-proxmox/#Configuration_de_Proxmox "Configuration de Proxmox")
- [Erreur « No valid subscription «](https://colinfo.fr/installation-et-configuration-de-proxmox/#Erreur_%C2%AB_No_valid_subscription_%C2%AB "Erreur « No valid subscription «") 
- [Gérer les mises à jour](https://colinfo.fr/installation-et-configuration-de-proxmox/#Gerer_les_mises_a_jour "Gérer les mises à jour")
- [Création des réseaux](https://colinfo.fr/installation-et-configuration-de-proxmox/#Creation_des_reseaux "Création des réseaux")
- [Configuration du stockage](https://colinfo.fr/installation-et-configuration-de-proxmox/#Configuration_du_stockage "Configuration du stockage")
- [Conclusion](https://colinfo.fr/installation-et-configuration-de-proxmox/#Conclusion "Conclusion")

## Présentation de Proxmox

Voici le lancement de mon premier article portant sur le déploiement de mon infrastructure en auto-hébergement, où nous explorerons l’installation et la configuration de Proxmox. Pour un aperçu de mon infrastructure et de mes objectifs, je vous invite à consulter le lien suivant : [https://colinfo.fr/presentation-de-mon-infrastructure-auto-heberge/](https://colinfo.fr/presentation-de-mon-infrastructure-auto-heberge/)

Proxmox est une plateforme de virtualisation open source qui offre la possibilité de créer et de gérer des machines virtuelles ainsi que des conteneurs Linux dans un environnement centralisé. Cette solution associe la virtualisation basée sur KVM (Kernel-based Virtual Machine) à la gestion des conteneurs via LXC (Linux Containers), fournissant ainsi une solution polyvalente pour l’hébergement de machines virtuelles et de conteneurs sur un même système. Proxmox propose une interface web conviviale pour la gestion des ressources, la surveillance et la sauvegarde des machines virtuelles, ce qui en fait un choix populaire pour les environnements de virtualisation d’entreprise et les centres de données.

Dans cet article, nous aborderons l’installation de Proxmox à partir de l’ISO via une clé USB bootable, ainsi que la configuration des mises à jour, du réseau et l’initialisation de notre disque de 1To destiné à héberger les machines virtuelles.

Pour installer Proxmox sur mon Intel Nuc, équipé de deux disques – un de 128 Go dédié à l’OS et l’autre de 1 To pour le stockage des machines virtuelles – je vais suivre ces étapes :

- Télécharger l’ISO de Proxmox à partir du lien suivant : https://www.proxmox.com/en/downloads.
- Une fois l’ISO téléchargé, je vais créer une clé USB bootable en utilisant l’utilitaire Rufus. Dans l’onglet « Sélection » de Rufus, je spécifie le chemin de l’ISO de Proxmox.

![Une image contenant texte, capture d’écran, nombre, logiciel
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-nombre-2.png)

Une fois la clé USB prête, je la branche sur l’ordinateur et je démarre celui-ci.

Si tout se passe comme prévu, une interface apparaît avec différentes options.

Je vais choisir « Install Proxmox VE (Graphical) » dans le menu proposé. ![](https://colinfo.fr/wp-content/uploads/2024/03/word-image-29434-2-2.jpeg)

On accepte les conditions d’utilisations.

![](https://colinfo.fr/wp-content/uploads/2024/03/word-image-29434-3-2.jpeg)

Dans cette interface, nous allons sélectionner le disque dur qui va héberger le système d’exploitation. Dans mon cas, je vais choisir le disque de 128 Go. ![](https://colinfo.fr/wp-content/uploads/2024/03/word-image-29434-4-2.jpeg)

Dans la configuration initiale, nous allons sélectionner le pays, le fuseau horaire et surtout le format du clavier en AZERTY, c’est mieux pour écrire le mot de passe !

![](https://colinfo.fr/wp-content/uploads/2024/03/word-image-29434-5-2.jpeg)

Sélection d’un mot de passe robuste. ![](https://colinfo.fr/wp-content/uploads/2024/03/word-image-29434-6-2.jpeg)

Dans cette étape, nous allons configurer le réseau de l’hôte, ce qui nous permettra de nous connecter à l’adresse IP que nous allons choisir. Dans mon cas, je vais attribuer l’adresse IP 192.168.1.200. ![](https://colinfo.fr/wp-content/uploads/2024/03/word-image-29434-7-2.jpeg)

Un dernier résumé avant l’installation de l’hyperviseur.

![](https://colinfo.fr/wp-content/uploads/2024/03/word-image-29434-8-2.jpeg)

  
Après le redémarrage du serveur, un shell s’affiche, nous indiquant que nous pouvons maintenant nous connecter à l’URL https://192.168.1.200:8006. ![](https://colinfo.fr/wp-content/uploads/2024/03/word-image-29434-9-2.jpeg)

L’erreur de certificat qui apparaît est due au fait qu’il n’est pas reconnu comme sûr par le navigateur. Vous pouvez cliquer sur « Paramètres avancés » puis choisir « Continuer vers le site » pour accéder à l’interface de Proxmox. ![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-logici-6.png)

Nous allons saisir comme nom d’utilisateur « root » et le mot de passe que nous avons renseigné lors de l’installation pour accéder à l’interface de Proxmox. ![Une image contenant capture d’écran, Logiciel multimédia, logiciel, texte
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-capture-decran-logiciel-mult-2.png)

## Configuration de Proxmox

### Erreur « No valid subscription « 

Il est probable que vous ayez remarqué une erreur lors de la connexion. Elle se produit chaque fois que vous vous connecterez à l’interface graphique de Proxmox.

Heureusement, il est possible de la résoudre en éditant le fichier proxmoxlib.js.

. ![Une image contenant texte, capture d’écran, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-16.png)

Il existe des scripts prêts à l’emploi, comme celui disponible sur le site linuxtricks.fr : [https://www.linuxtricks.fr/wiki/proxmox-desactiver-le-message-you-do-not-have-valid-subscription](https://www.linuxtricks.fr/wiki/proxmox-desactiver-le-message-you-do-not-have-valid-subscription)

Nous allons créer et éditer un fichier nommé remove-popup.sh directement sur le shell de Proxmox :

```
nano remove-popup.sh
```

Ajouter ce contenu :

```
#! /bin/bash
# Patch du fichier JS
sed -i.bak "s/.data.status.toLowerCase() !== 'active') {/.data.status.toLowerCase() !== 'active') { orig_cmd\(\); } else if ( false ) {/" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
# Restart PVE Web
systemctl restart pveproxy.service
# Restart PBS Web
systemctl restart proxmox-backup-proxy.service
```

On rend le script exécutable

```
chmod +x /root/remove-popup.sh
```

Et nous pouvons maintenant exécuter le script :

```
./remove-popup.sh
```

Ce script va effectuer la bonne modification dans le bon fichier et il sera à exécuter à chaque mise à jour

### Gérer les mises à jour

Par défaut, la configuration des mises à jour n’est pas fonctionnelle car elle est basée sur la version payante. Nous pouvons la désactiver.

On remarque que la commande sudo apt update ne fonctionne pas dans cet état. ![Une image contenant texte, capture d’écran, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-17.png)

La configuration des mises à jour peut être réalisée via l’interface graphique, dans l’onglet « Updates ».

Cela offre une vue sur les dépôts configurés. ![Une image contenant Logiciel multimédia, logiciel, Logiciel de graphisme, capture d’écran
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-logiciel-multimedia-logiciel-2.png)

Nous allons commencer par ajouter le dépôt « no subscription », qui est le dépôt gratuit. ![Une image contenant texte, capture d’écran, Police, logiciel
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-18.png)

Et pour finir, nous allons retirer le dépôt « enterprise » qui n’est pas disponible sans licence. Pour ce faire, nous le sélectionnons et le désactivons (disable). ![Une image contenant logiciel, texte, Logiciel multimédia, Logiciel de graphisme
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-logiciel-texte-logiciel-mult-2.png)

De retour sur le shell, des mises à jour sont disponibles maintenant après avoir exécuté la commande apt update. ![Une image contenant texte, capture d’écran, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-19.png)

### Création des réseaux

Conformément à mon infrastructure personnelle, j’ai besoin de trois réseaux distincts.

Sur l’hôte PVE, dans la section « System » puis « Network », nous avons déjà quelques éléments configurés :

- enp2s0 correspond à ma carte Ethernet physiquement connectée.
- vmbr0 est un pont Linux (Linux bridge) créé automatiquement lors de l’installation. Il est directement connecté à la carte physique.
- wlo1 est la carte Wi-Fi, mais elle n’est pas actuellement connectée.

![Une image contenant logiciel, Logiciel multimédia, Logiciel de graphisme, capture d’écran
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-logiciel-logiciel-multimedia-4.png)

Nous allons créer deux Linux bridges supplémentaires en plus de celui déjà existant. Les Linux bridges agissent comme des cartes réseau virtuelles et nous allons les relier à notre pare-feu pour faciliter la communication entre les différents réseaux.

On commence par créer la carte réseau virtuelle « VM Network ». ![Une image contenant texte, capture d’écran, Police, logiciel
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-20.png)

Ensuite, nous allons maintenant créer la carte réseau « DMZ ».

![Une image contenant texte, capture d’écran, Police, logiciel
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-21.png)

Et je termine en incluant un commentaire « WAN » afin de rendre plus facile l’identification de la carte bridge sur vmbr0.![Une image contenant texte, capture d’écran, Police, nombre
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-22.png)

Toutes nos interfaces réseau ont été créées !

### Configuration du stockage

Dans l’onglet « PVE », qui représente notre serveur Proxmox, nous pouvons visualiser les disques détectés. Nous constatons la présence de notre disque système de 128 Go ainsi que du second disque de 1 To, qui est actuellement vide. ![Une image contenant capture d’écran, logiciel, Logiciel multimédia, Logiciel de graphisme
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-capture-decran-logiciel-log-2.png)

Nous allons configurer le deuxième disque de stockage pour héberger nos machines virtuelles en utilisant le stockage LVM, ce qui offre une grande flexibilité. Vous pouvez trouver une explication simple de son fonctionnement avec Proxmox via ce lien : https://sweworld.net/blog/using_lvm_for_proxmox/.

Cela peut se configurer rapidement et simplement à travers l’interface graphique, dans l’onglet « Disks » et « LVM ».

![Une image contenant texte, capture d’écran, logiciel, Logiciel multimédia
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-logici-7.png)

Nous cliquons sur « Create Volume Group »

![Une image contenant texte, capture d’écran, Police, nombre
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-23.png)

Notre stockage est automatiquement créé dans l’onglet « Storage » avec le nom du volume groupe associé. ![Une image contenant logiciel, Logiciel multimédia, Logiciel de graphisme, Montage
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-logiciel-logiciel-multimedia-5.png)

Ensuite, nous allons également monter un partage SMB car mon NAS stocke mes fichiers ISO. Pour cela, j’ai créé un utilisateur avec des droits uniquement sur le répertoire où sont stockés les ISO, afin de respecter le principe du moindre privilège.

Toujours dans l’onglet « Storage », nous allons sélectionner « Add » puis SMB/CIFS. Nous indiquons le contenu du répertoire distant, qui est « ISO image » dans mon cas, ainsi que les informations d’identification et le nom. Veillez à ne pas simplement mettre « iso » comme ID, car cela pourrait générer une erreur !

![Une image contenant texte, capture d’écran, logiciel, nombre
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-logici-8.png)

Afin que les fichiers ISO soient accessibles dans Proxmox, ils doivent être stockés dans le répertoire distant que vous avez sélectionné, dans le dossier « templates/iso ». Assurez-vous donc de déplacer vos ISO vers cet emplacement pour qu’ils puissent être utilisés dans Proxmox.

## Conclusion

Dans cet article, nous avons exploré le processus d’installation et de configuration de Proxmox, une plateforme de virtualisation open source puissante et polyvalente. Nous avons débuté par la présentation de Proxmox et de ses fonctionnalités. Ensuite, nous avons suivi étape par étape l’installation de Proxmox depuis une clé USB bootable, en configurant le réseau, les mises à jour et les disques de stockage.