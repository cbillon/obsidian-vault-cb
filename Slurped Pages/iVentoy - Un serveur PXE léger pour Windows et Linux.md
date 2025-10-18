---
link: https://www.it-connect.fr/tuto-iventoy-serveur-pxe-windows-linux-vmware-esxi/
byline: Florian BURNEL
site: IT-Connect
date: 2024-05-08T18:00
excerpt: Dans ce tutoriel, nous allons utiliser le serveur PXE iVentoy pour déployer Windows 11, Debian, Ubuntu, ou encore VMware ESXi par le réseau. Simple et pratique.
twitter: https://twitter.com/@itconnect_fr
tags:
  - linux
  - pxe
slurped: 2025-10-12T19:30
title: iVentoy - Un serveur PXE léger pour Windows et Linux
---
amitmerchant
Sommaire

- [I. Présentation](https://www.it-connect.fr/tuto-iventoy-serveur-pxe-windows-linux-vmware-esxi/#I_Presentation)
- [II. Télécharger et installer iVentoy](https://www.it-connect.fr/tuto-iventoy-serveur-pxe-windows-linux-vmware-esxi/#II_Telecharger_et_installer_iVentoy)
- [III. Configurer le serveur iVentoy](https://www.it-connect.fr/tuto-iventoy-serveur-pxe-windows-linux-vmware-esxi/#III_Configurer_le_serveur_iVentoy)
- [IV. Déployer une machine via le réseau](https://www.it-connect.fr/tuto-iventoy-serveur-pxe-windows-linux-vmware-esxi/#IV_Deployer_une_machine_via_le_reseau)
- [V. Conclusion](https://www.it-connect.fr/tuto-iventoy-serveur-pxe-windows-linux-vmware-esxi/#V_Conclusion)

## I. Présentation

**Dans ce tutoriel, nous allons découvrir iVentoy, un serveur PXE très léger et simple à utiliser capable de déployer des images Windows, Linux, mais aussi VMware ESXi. À l'heure actuelle, il prend en charge de nombreux systèmes et 170 images différentes ont été testées !**

- [Cliquez ici pour regarder la vidéo sur YouTube](https://youtu.be/ZsIzD_qcxiQ)

iVentoy a été développé par la même personne que Ventoy, le célèbre outil de création de clés USB bootable. Sauf qu'iVentoy est **un serveur PXE clé en main** qui intègre à la fois le serveur DHCP et le boot PXE, tout en proposant un ensemble d'options et fonctionnalités. iVentoy est une alternative à d'autres solutions comme le rôle **Windows Deployment Services (WDS) de Windows Server**.

Le principe est simple : vous stockez sur votre serveur iVentoy différentes images ISO (Windows 11, Debian, Ubuntu, Windows Server, etc.) et vous démarrez iVentoy de façon à pouvoir déployer ces images ISO via le réseau local. Autrement dit, **l'installation du système d'exploitation est effectuée par le réseau** et vous pouvez installer plusieurs machines en même temps.

**iVentoy est gratuit pour une utilisation personnelle et il est facturé 49 dollars pour une utilisation commerciale.** C'est un tarif plus que raisonnable et ceci permettra de soutenir le projet. La version gratuite permet de déployer jusqu'à 20 machines simultanément, tandis que la version commerciale n'a aucune limite. Vous pouvez [consulter le site d'iVentoy pour en savoir plus](https://www.iventoy.com/en/index.html).

**Pour approfondir le sujet du boot PXE et de WDS, voici des lectures recommandées :**

- [Le boot PXE et iPXE pour les débutants](https://www.it-connect.fr/le-boot-pxe-et-le-boot-ipxe-pour-les-debutants/ "Le boot PXE et le boot iPXE pour les débutants")
- [Boot PXE sur Windows Server avec WDS et DHCP](https://www.it-connect.fr/serveurs-dhcp-wds-boot-pxe-bios-et-uefi/ "Serveurs WDS et DHCP – Comment faire du boot PXE BIOS et UEFI sur le même réseau ?")

## II. Télécharger et installer iVentoy

**iVentoy peut être installé sur Linux, mais également Windows et Windows Server.** Dans cet exemple, nous allons l'installer sur un poste de travail Windows 11 : l'outil fonctionnera parfaitement et n'a aucune dépendance vis-à-vis de Windows Server.

**L'installation est vraiment simple !** Rendez-vous sur le GitHub officiel et téléchargez la dernière version correspondante à votre système d'exploitation. Ici, le fichier "**iventoy-1.0.220-win64-free.zip**" est téléchargé.

- [GitHub - iVentoy](https://github.com/ventoy/PXE/releases)

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/Telecharger-iVentoy.jpg)

Quand c'est fait, décompressez l'archive ZIP sur votre PC. Puis, accédez au dossier obtenu afin d'exécuter le fichier nommé "**iVentoy_64**". Il va permettre de lancer le serveur iVentoy ! Une notification apparaîtra à l'écran pour vous demander d'**autoriser iVentoy dans le pare-feu : acceptez**. Il y a également une alerte SmartScreen parce que l'exécutable n'est pas signé, mais vous pouvez l'exécuter (ce serait bien que le développeur améliore ce point).

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/Installer-iVentoy-sur-Windows-11.png)

Si nécessaire, cliquez sur "**Open Link**" pour accéder à l'interface de gestion d'iVentoy. Il s'agit d'**une interface web accessible en locale** à l'adresse suivante :

```
http://127.0.0.1:26000/
```

Voici un aperçu :

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/Lancement-iVentoy-800x489.jpg)

Voilà, le serveur iVentoy n'attend plus qu'une chose : être configuré pour vous permettre de déployer vos premières machines !

## III. Configurer le serveur iVentoy

La première chose à effectuer, c'est **alimenter la bibliothèque d'images ISO d'iVentoy**. C'est très simple, il vous suffit de **copier-coller vos images ISO** dans le répertoire "**iso**" de l'application. Dans cet exemple, deux images ISO pour Windows 11 et Debian sont copiées. La liste des images ISO testées est disponible sur [cette page](https://www.iventoy.com/en/isolist.html).

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/iVentoy-Ajouter-une-image-ISO.png)

Ensuite, à partir de l'interface web, cliquez sur "**Image Management**". Vous devriez voir apparaître vos images ISO. En cliquant sur une image ISO, nous accédons à **plusieurs options,** notamment pour :

- Définir l'image sélectionnée comme image de démarrage par défaut ("**Set as default boot file**").
- Protéger le démarrage sur cette image par un mot de passe ("**Boot Password**").
- Injecter des pilotes ou des scripts ("**Injection File**")
- Utiliser un script d'auto-installation, c'est-à-dire un fichier **Unattend.xml** pour Windows, un fichier **preseed.cfg** pour Debian, un modèle **Cloud-init** pour Ubuntu, etc. ("**[Auto Install Script](https://www.iventoy.com/en/doc_autoinstall.html)**")

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/iVentoy-Liste-des-images-ISO-800x438.jpg)

Basculer vers l'onglet "**Configuration**" qui donne accès **aux options pour les serveurs TFTP et DHCP**. iVentoy intègre son propre serveur DHCP, mais vous pouvez utiliser un serveur DHCP externe, sur Windows Server, par exemple. Si vous utilisez un autre serveur DHCP, veillez à bien le configurer. Pour cette démonstration, le serveur DHCP intégré à iVentoy est utilisé.

Pour utiliser un serveur DHCP externe, vous devez configurer l'option "**DHCP Server Mode**" et ne pas utiliser le mode "**Internal**". Vous devez alors configurer l'**option DHCP 66** avec l'**adresse IP du serveur iVentoy** et l'**option DHCP 67** avec la valeur "**iventoy_loader_16000**". Pour information, le "**16000**" dans le nom correspond au numéro de port par défaut de l'interface web.

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/iVentoy-Configuration-DHCP-800x456.png)

Basculez vers l'onglet "**Boot Information**" afin de démarrer le serveur iVentoy. Mais avant cela, nous devons configurer la partie "**IP Configuration**" afin de définir un masque de sous-réseau, une adresse IP de passerelle, un serveur DNS, ainsi qu'une plage d'adresses IP à distribuer aux machines clientes à déployer.

Voici un exemple :

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/iVentoy-Configurer-Boot-800x315.jpg)

Une fois que c'est fait, vous pouvez cliquer sur le bouton vert "Lecture" pour démarrer le serveur iVentoy.

> **Remarque** : la section "**MAC Filter**" du menu permet de configurer le filtrage MAC pour autoriser uniquement certaines machines, via leur adresse MAC.

**Désormais, nous allons pouvoir tester notre serveur iVentoy, car il est - déjà - prêt !**

## IV. Déployer une machine via le réseau

Pour tester le déploiement via le réseau, vous pouvez utiliser un ordinateur physique ou une machine virtuelle. Pour ma part, je vais utiliser une machine virtuelle sur VMware Workstation. Elle a bien **démarré sur le réseau en iPXE** et j'arrive sur une interface avec le logo iVentoy où je vois bien mes deux images : Debian et Windows 11 !

Avec les flèches directionnelles du clavier, il suffit de choisir l'image correspondante au système que nous souhaitons déployer et d'appuyer sur "**Entrée**".

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/Boot-PXE-avec-iVentoy-800x314.png)

La machine va charger l'image ISO par le réseau. Nous arrivons sur le programme d'installation de Debian 12 :

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/iVentoy-Debian.jpg)

Dans le même temps, côté serveur iVentoy, la section "**Device List**" présente dans "**Boot Information**" liste toutes les machines en cours de déploiement. La colonne "**Status**" est intéressante, car elle indique où en est la machine et notamment l'image de démarrage sélectionnée.

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/iVentoy-Liste-des-appareils-800x78.jpg)

Par ailleurs, il est tout à fait possible d'installer Windows 11, toujours via le boot iVentoy !

![](https://www.it-connect.fr/wp-content-itc/uploads/2024/04/iVentoy-Windows-11.jpg)

Il ne reste plus qu'à patienter pendant l'installation du système !

## V. Conclusion

**L'application iVentoy est à la fois simple à utiliser et très efficace !** Elle offre la possibilité de déployer facilement des images Linux, Windows, etc... Très facilement car il suffit de copier-coller l'image ISO dans le répertoire "iso" de l'application ! Si vous en avez assez de vos clés USB Ventoy, vous pouvez vous orienter sur iVentoy pour effectuer du déploiement par le réseau !

![author avatar](https://www.it-connect.fr/wp-content-itc/uploads/2025/07/Avatar-FB.jpg)

Ingénieur système et réseau, cofondateur d'IT-Connect et Microsoft MVP "Cloud and Datacenter Management". Je souhaite partager mon expérience et mes découvertes au travers de mes articles. Généraliste avec une attirance particulière pour les solutions Microsoft et le scripting. Bonne lecture.