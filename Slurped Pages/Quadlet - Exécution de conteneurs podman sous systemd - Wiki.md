---
link: https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd
site: Linuxtricks.fr
excerpt: |-
  Introduction
  Utilisant exclusivement RHEL dans le cadre de mon travail, et quasiment que RHEL et ses clones en perso,...
slurped: 2025-07-21T09:53
title: "Quadlet : Exécution de conteneurs podman sous systemd - Wiki"
tags:
  - containers
  - systemd
---

Table des matières

1. [Introduction](https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd#paragraph-introduction)
2. [La complexité avant Quadlets](https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd#paragraph-la-complexite-avant-quadlets)
3. [Quadlet](https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd#paragraph-quadlet)
    1. [Conteneurs root vs rootless](https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd#paragraph-conteneurs-root-vs-rootless)
    2. [Fichier de config du container](https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd#paragraph-fichier-de-config-du-container)
    3. [Gestion en mode rootless](https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd#paragraph-gestion-en-mode-rootless)
    4. [Gestion en mode root](https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd#paragraph-gestion-en-mode-root)
4. [Dépendances entre conteneurs](https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd#paragraph-dependances-entre-conteneurs)
5. [Mises à jour des images](https://www.linuxtricks.fr/wiki/quadlet-execution-de-conteneurs-podman-sous-systemd#paragraph-mises-a-jour-des-images)

## Introduction

Utilisant exclusivement RHEL dans le cadre de mon travail, et quasiment que RHEL et ses clones en perso, j'utilise bien plus podman que docker pour les conteneurs.

Quadlet nous permet d'exécuter nos conteneurs podman en tant que services systemd. C'est particulièrement utile pour exécuter des conteneurs en arrière-plan et les démarrer automatiquement après un redémarrage du serveur. Aussi, on pourra bénéficier des journaux système pour tracer les actions.

Exécuter des conteneurs podman sous systemd n'est pas nouveau. En effet, cela a été pris en charge par podman depuis longtemps avec la commande **podman generate systemd** mais cette commande nous invite à migrer vers quadlet.

J'apprécie vraiment quadlet et il n'y a pas beaucoup de doc synthétique et en français sur le sujet !  
Avec quadlet, podman est vraiment une alternative à Docker Compose qui est encore plus flexible et puissante !

Dans cet article, je vais expliquer comment utiliser quadlet avec podman.  
Je pars du principe que vous connaissez déjà podman ![;)](https://www.linuxtricks.fr/images/smileys/6.gif ";)")

## La complexité avant Quadlets

Juste une petite explications pour ceux qui ont géré des conteneurs podman avant quadlet.

Le problème avec l'ancienne méthode est qu'elle nécessitait d'exécuter des commandes pour :  
- créer un conteneur  
- générer un fichier de service  
- déplacer le fichier de service s'il n'était pas déjà dans le répertoire mentionné  
- activer le service

Surtout, la commande pour créer le conteneur est souvent longue.

On pouvait se créer des scripts pour automatiser tout ça mais une solution native est la bienvenue.

Ceux qui utilisent docker compose ont aussi une facilité d'utilisation qu'on n'avait pas dans podman ... jusqu'à quadlet !

## Quadlet

### Conteneurs root vs rootless

Si on fait fonctionner les conteneurs en tant que root, les fichiers seront à placer dans : **/etc/containers/systemd**  
Si on fait fonctionner les conteneurs en tant qu'utilisateur sans privileges, les fichiers seront à placer dans **~/.config/containers/systemd** (à créer éventuellement)

Les fichiers auront une extension en **.container** (on connait déjà avec systemd des .service, .socket, .mount, etc.)

### Fichier de config du container

On va partir sur un conteneur PostgreSQL.  
Bien que je ne sois pas fan (à ce stade) des bases de données conteneurisées, c'est un exemple parfait car on va y retrouver les principaux usages :  
- Une application  
- Des ports exposés  
- Des volumes sortis du conteneur  
- Des variables d'environnement

Au moment de la rédaction du tuto, la dernière version de PostgreSQL est la 17.2, donc je pars sur un conteneur PostgreSQL 17.

Je vais créer un fichier conteneur :

Code BASH :

vim /etc/containers/systemd/bdd-pg17.container

Voici le contenu :

Code BASH :

[Container]
Image=docker.io/library/postgres:17
ContainerName=bdd-pg17
PublishPort=5432:5432
Volume=%h/volumes/bdd-pg17:/var/lib/postgresql/data:Z
Environment=POSTGRES_PASSWORD=SUPERMDP
[Service]
Restart=always
[Install]
WantedBy=default.target

Il s'agit d'un fichier de service systemd comme on a l'habitude de croiser, mais avec la section spéciale **[Container]**.  
Cette section contient de nombreuses options documentées. On retrouvera sur la doc officielle toutes les options possibles : [https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#container-units-container](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html#container-units-container)  
Presque toutes ces options correspondent à des options de ligne de commande pouvant être utilisées pour créer un conteneur avec Podman (podman create).

Dans les options que j'ai utilisé ici, que j'utilise le plus souvent :  
- **Image** spécifie l'image (avec le tag) à utiliser.  
- **ContainerName** spécifie le nom du conteneur. Sans cette option, le conteneur s'appellera systemd-NOM où NOM est le nom du fichier de service sans l'extension .container (ici systemd-bdd-pg17)  
- **PublishPort** correspond à -p.  
- **Volume** correspond à -v. (Le :Z est spécifique à SELinux pour adapter le contexte)  
- **Environment** correspond à -e.

Il est important d'utiliser le spécificateur systemd **%h** au lieu de **~** pour le répertoire personnel de l'utilisateur.  
Plus d'infos : [https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html#Specifiers](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html#Specifiers)

Dans la section **[Service]**, on va utiliser l'option Restart et on la défini sur always pour redémarrer le conteneur en permanence (sauf s'il est arrêté manuellement).

Pour démarrer automatiquement le conteneur au démarrage, on défini l'option WantedBy dans la section **[Install]** sur default.target.

Définir WantedBy sur multi-user.target ne fonctionne pas dans le cas des conteneurs sans privilèges. Donc dans tous les cas, on pourra mettre default.target ![:)](https://www.linuxtricks.fr/images/smileys/1.gif ":)") 

### Gestion en mode rootless

Dans le cas d'un usage rootless, l'activation du linger est nécessaire pour que le conteneur soit automatiquement démarré après un redémarrage du serveur sans que l'utilisateur ait ouvert une session :  

Pour que systemd découvre le nouveau fichier de service, on rechargera le démon :

Code BASH :

systemctl --user daemon-reload

.

On pourra maintenant démarrer le conteneur avec :

Code BASH :

systemctl --user start bdd-pg17

Note : Si l'image n'a jamais été pull, le premier démarrage prend du temps ![:)](https://www.linuxtricks.fr/images/smileys/1.gif ":)")

On pourra vérifier le statut du service avec :

Code BASH :

systemctl --user status bdd-pg17

On pourra également vérifier que le conteneur Podman est en cours d'exécution en exécutant podman ps.

### Gestion en mode root

Si on lance nos conteneurs avec l'utilisateur root, pas besoin d'activer le linger.

Pour que systemd découvre le nouveau fichier de service, on rechargera le démon :

.

On pourra maintenant démarrer le conteneur avec :

Note : Si l'image n'a jamais été pull, le premier démarrage prend du temps ![:)](https://www.linuxtricks.fr/images/smileys/1.gif ":)")

On pourra vérifier le statut du service avec :

Code BASH :

systemctl status bdd-pg17

On pourra également vérifier que le conteneur Podman est en cours d'exécution en exécutant podman ps.

## Dépendances entre conteneurs

On le sait, systemd sait gérer les dépendances de services : lancer tel service après un autre (exemple httpd après NetworkManager)  
On pourra le faire aussi avec quadlet.

Supposons qu'on ait un conteneur d'application qui dépend du conteneur de base de données que nous avons créé.

Prenons un exemple :  
- bdd-pg17 est notre base de données PostrgeSQL 17  
- app-linuxtricks est notre application Web Linuxtricks (un conteneut php 8.3 + httpd)

Voilà à quoi resemblerait notre fichier **/etc/containers/systemd/app-linuxtricks.container** par exemple :

Code BASH :

[Container]
Image=php:8.3-apache
ContainerName=app-linuxtricks
PublishPort=8080:80
Volume=%h/volumes/app-linuxtricks:/var/www/html:Z
[Unit]
Requires=bdd-pg17.service
After=bdd-pg17.service
[Service]
Restart=always
[Install]
WantedBy=default.target

La nouvelle section est **[Unit]**.  
On défini l'option **Requires** sur **bdd-pg17.service** pour ne démarrer l'application que lorsque la base de données est lancée.  
On défini aussi l'option **After** pour s'assurer que les deux conteneurs ne démarrent pas en parallèle.

On utilise **bdd-pg17.service** car on fait référence au service de conteneur et non au quadlet bdd-pg17.container :  
- **bdd-pg17.container** est le nom du fichier.  
- **bdd-pg17.service** est le nom du service.  
- **bdd-pg17** est le nom du conteneur (systemd-bdd-pg17 par défaut sinon)

## Mises à jour des images

Maintenant, qu'on a des conteneurs qui s'exécutent en arrière-plan et qui démarrent automatiquement après un redémarrage du serveur, ne serait-il pas agréable d'avoir une méthode simple pour mettre à jour les images de ces conteneurs sans avoir à exécuter podman pull pour chaque conteneur, puis à redémarrer ceux qui ont été mis à jour ?

Par exemple, si une nouvelle image est téléchargée pour PostgreSQL 17 (avec le tag d'image 17 que qu'on a utilisé), alors l'image devrait être mise à jour et le conteneur devrait être redémarré.

Avec Docker, il existe des outils alternatifs comme Watchtower. Avec podman fournit un outil prêt à l'emploi !

Dans la section **[Container]** on pourra rajouter une ligne AutoUpdate, comme par exemple :

Code BASH :

[Container]
Image=docker.io/library/postgres:17
ContainerName=bdd-pg17
AutoUpdate=registry
PublishPort=5432:5432
Volume=%h/volumes/bdd-pg17:/var/lib/postgresql/data:Z
Environment=POSTGRES_PASSWORD=SUPERMDP
[Service]
Restart=always
[Install]
WantedBy=default.target

ici, **AutoUpdate=registry** correspond à --label "io.containers.autoupdate=registry"

Quand on défini AutoUpdate=registry, on pourra ensuite simplement exécuter :

Podman vérifiera si le dépôt a une image plus récente compatible avec le tag utilisé. Dans ce cas, l'image sera téléchargée et le conteneur sera redémarré. C'est aussi simple que ça !

Cela pourrait être dangereux si on utilise un un tag comme latest au lieu d'une version concrète comme 17.  
En effet, la prochaine version poussée vers le tag latest pourrait inclure un changement incompatible !  
Des migrations manuelles sont toujours nécessaires lors de la mise à niveau de PostgreSQL vers une nouvelle version majeure.

Par conséquent, utilisez toujours un tag qui ne peut pas entraîner de changement incompatible si vous activez cette option !

Je vous invite à exécuter **podman auto-update** manuellement sur le serveur pour voir ce qui a été mis à jour et vous assurer que les conteneurs sont toujours fonctionnels !