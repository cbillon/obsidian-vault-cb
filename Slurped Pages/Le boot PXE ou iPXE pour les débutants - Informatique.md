---
link: https://www.it-connect.fr/le-boot-pxe-et-le-boot-ipxe-pour-les-debutants/
byline: Florian BURNEL
site: IT-Connect
date: 2023-03-27T16:45
excerpt: Apprenez ce que sont le boot PXE et le boot iPXE, comment ils fonctionnent et quelles sont les différences entre ces deux méthodes de démarrage réseau.
twitter: https://twitter.com/@itconnect_fr
tags:
  - pxe
slurped: 2025-10-12T19:24
title: Le boot PXE ou iPXE pour les débutants - Informatique
---

## I. Présentation

**Dans cet article, nous allons parler de boot PXE ainsi que de boot iPXE puisque ce sont deux notions à connaître en tant qu'administrateur système et réseau et qui sont directement liées à un processus qui concerne toutes les entreprises : le déploiement de postes de travail.**

Traditionnellement, lorsque l'on veut installer un système d'exploitation sur un poste de travail, on va prendre une clé USB et déployer l'ISO du système dessus. Une fois que c'est fait, on va connecter physiquement la clé USB à l'ordinateur pour qu'il charge le contenu de la clé USB afin que l'on puisse procédé à l'installation du système d'exploitation. Dans le même esprit, et même si maintenant c'est plus rare, on peut utiliser un CD ou DVD d'installation, mais là encore il faut avoir les sources sur un support physique.

Grâce au **boot PXE ou iPXE**, on va pouvoir **s'affranchir de l'utilisation d'une clé USB, d'un CD ou d'un DVD** puisque l'on va **démarrer sur le réseau**. Si vous êtes débutant en informatique et que vous souhaitez en savoir plus sur le boot PXE (et l'iPXE), alors cet article est fait pour vous !

Le **boot PXE** pour _**P**reboot E**x**ecution **E**nvironment_ est un **protocole de démarrage réseau** qui permet à un ordinateur local de démarrer à partir de données situées sur un serveur distant, plutôt que de démarrer à partir du disque local, ou du contenu de la clé USB, si l'on veut faire référence à l'exemple précédent.

Cette méthode de démarrage est souvent utilisée pour le **déploiement de postes de travail** puisqu'elle va permettre de distribuer le système d'exploitation à installer (Windows 10, Windows 11, [Windows Server](https://www.it-connect.fr/cours-tutoriels/administration-systemes/windows-server/ "Windows Server"), Linux, etc.) à partir du réseau. Ainsi, cette méthode permet de **gagner énormément de temps** et d'automatiser le processus de configuration des postes de travail. On peut affirmer que **le boot PXE joue un rôle essentiel pour industrialiser le déploiement de machines en entreprise.**

En fait, si l'on a besoin d'installer Windows 11 sur 20 machines différentes, **plutôt que d'utiliser 20 clés USB ou de faire les machines une par une si l'on a qu'une seule clé USB**, on va effectuer **un boot PXE sur chaque machine et elles vont récupérer les sources d'installation depuis notre serveur**.

Dans la pratique, **comment cela se traduit d'un point de vue réseau et infrastructure à mettre en place ?** Le boot PXE repose sur deux protocoles importants : le **protocole DHCP** (Dynamic Host Configuration Protocol) pour fournir une configuration IP au client PXE, ainsi que l'adresse IP du serveur qui héberge les sources, ainsi que le **protocole TFTP** (Trivial File Transfer Protocol) pour accéder au serveur PXE et commencer à transférer l'image de démarrage via le réseau. Aujourd'hui, toutes les cartes réseau RJ45 Ethernet supportent le boot PXE, mais sur du matériel ancien, ce ne sera pas forcément une évidence !

**On peut résumer ce processus en quelques étapes :**

1. Le client PXE, à savoir l'ordinateur, démarre et envoie une requête DHCP sur le réseau pour obtenir une adresse IP.
2. Le serveur DHCP répond à la demande en fournissant une adresse IP au client, ainsi que les informations nécessaires pour accéder au serveur PXE (adresse IP ou nom).
3. Le client PXE peut accéder au réseau donc il envoie une demande de boot PXE sur le réseau, en utilisant le protocole TFTP  pour accéder au serveur PXE.
4. Le serveur PXE répond à la demande de l'ordinateur en fournissant les fichiers nécessaires pour le démarrage du système.
5. Le client PXE télécharge ces fichiers à partir du serveur PXE (via le protocole TFTP) et les utilise pour démarrer sur l'image fournie.

L'image envoyée par le serveur PXE peut correspondre à un système d'exploitation ou à **un environnement de démarrage tel que WinPE** qui est l'environnement de pré-installation de Windows. Le boot PXE est souvent utilisé pour déployer un système d'exploitation mais on peut aussi l'utiliser pour charger un environnement de dépannage / récupération.

Puisque l'image du système d'exploitation est envoyée via le réseau, **il y a une forte dépendance avec le réseau avec ce mode de fonctionnement** (latence, bande passante, etc.).

## III. Le boot iPXE

Sur le même principe que le boot PXE, **le boot iPXE est une méthode plus récente** qui permet à un ordinateur de démarrer sur le réseau. **Le protocole iPXE est une extension du protocole PXE** qui offre des fonctionnalités supplémentaires telles que:

- le démarrage à partir d'un serveur web via HTTP
- le démarrage à partir d'un SAN iSCSI, FCoE ou AoE
- le démarrage à partir d'un réseau sans fil ou d'un réseau Infiniband
- le support des scripts et des menus de démarrage personnalisés

Puisque l'on peut effectuer le téléchargement à partir d'un serveur Web via HTTP plutôt qu'en TFTP, on va bénéficier d'un protocole plus performant. Ce qui est intéressant aussi, c'est le fait de **pouvoir récupérer les sources à partir d'Internet** !

Le boot iPXE nécessite qu'**un firmware iPXE soit installé sur la carte réseau de la machine**. De plus en plus de fabricants intègrent la prise en charge de l'iPXE, en complément du PXE. Sur les machines où le PXE standard est supporté, il est possible d'**appliquer la technique du PXE chainloading pour bénéficier de l'iPXE.** Ainsi, on stocke l'iPXE sur un serveur TFTP et on démarre via le mode PXE classique et l'iPXE sera récupéré sur le serveur TFTP.

En résumé, on peut dire que **l'iPXE va plus loin car il supporte de nombreux protocoles et va plus loin dans la personnalisation**, même si les grands principes restent les mêmes. De ce fait, l'iPXE est utilisé aussi pour effectuer du déploiement de postes de travail via le réseau.

Des détails supplémentaires sont disponibles sur le site de ce **firmware de démarrage open source** : [ipxe.org](https://ipxe.org/start).

## IV. Conclusion

Dans de prochains articles, nous verrons **comment mettre en oeuvre le boot PXE et le boot iPXE**. Sur Windows Server, nous pourrons utiliser **les rôles DHCP et WDS** (_**W**indows **D**eployment **S**ervices_) pour mettre ce principe en pratique, et ajouter [MDT](https://www.it-connect.fr/cours-tutoriels/administration-systemes/windows-server/deploiement-mdt-wds/ "MDT") pour aller plus loin. Pour en savoir plus sur l'iPXE et voir un cas pratique, vous pouvez lire [cet article déjà en ligne](https://www.it-connect.fr/comment-deployer-windows-avec-wapt-2-2/).

![author avatar](https://www.it-connect.fr/wp-content-itc/uploads/2025/07/Avatar-FB.jpg)

Ingénieur système et réseau, cofondateur d'IT-Connect et Microsoft MVP "Cloud and Datacenter Management". Je souhaite partager mon expérience et mes découvertes au travers de mes articles. Généraliste avec une attirance particulière pour les solutions Microsoft et le scripting. Bonne lecture.