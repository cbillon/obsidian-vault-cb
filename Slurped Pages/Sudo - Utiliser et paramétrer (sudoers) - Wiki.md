---
link: https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers
site: Linuxtricks.fr
excerpt: |-
  Introduction
  sudo (abréviation de substitute user do) est une commande permettant à l'administrateur système d'accorder à...
slurped: 2025-02-22T08:46
title: "Sudo : Utiliser et paramétrer (sudoers) - Wiki"
tags:
  - sudo
  - tutorial
  - linux
  - bash
---

Table des matières

1. [Introduction](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-introduction)
2. [Les avantages de sudo sur Linux](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-les-avantages-de-sudo-sur-linux)
3. [Utiliser sudo](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-utiliser-sudo)
4. [Configurer sudo](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-configurer-sudo)
    1. [Edition de la configuration](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-edition-de-la-configuration)
    2. [Configuration par défaut](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-configuration-par-defaut)
    3. [Déléguer une commande à un utilisateur](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-deleguer-une-commande-a-un-utilisateur)
    4. [Déléguer une commande avec des options ou arguments précis à un utilisateur](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-deleguer-une-commande-avec-des-options-ou-arguments-precis-a-un-utilisateur)
    5. [Déléguer plusieurs commandes à un utilisateur](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-deleguer-plusieurs-commandes-a-un-utilisateur)
    6. [Interdire l'exécution d'autres programmes](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-interdire-l-execution-d-autres-programmes)
    7. [Déléguer une commande en tant qu'un utilisateur particulier](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-deleguer-une-commande-en-tant-qu-un-utilisateur-particulier)
    8. [Ne pas demander de mot de passe](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-ne-pas-demander-de-mot-de-passe)
    9. [Spécifier plusieurs utilisateurs ou groupes](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-specifier-plusieurs-utilisateurs-ou-groupes)
5. [A propos de l'ordre des règles](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-a-propos-de-l-ordre-des-regles)
6. [Paramètres globaux sudo](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-parametres-globaux-sudo)
7. [Et pour plus d'infos ...](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers#paragraph-et-pour-plus-d-infos)

![root](https://www.linuxtricks.fr/upload/root.png "root")

## Introduction

**sudo** (abréviation de **s**ubstitute **u**ser **do**) est une commande permettant à l'administrateur système d'accorder à certains utilisateurs (ou groupes d'utilisateurs) la possibilité de lancer une commande en tant qu'administrateur.

## Les avantages de sudo sur Linux

Histoire de comprendre l'intérêt de sudo, voici un petit pavé explicatif. J'essaie de le faire sur les articles qui sont assez vus pour en améliorer le contenu.

Dans les systèmes de type Unix, tels que Linux, on a 2 types d'utilisateurs:  
- les utilisateurs sans privilèges  
- l'utilisateur root, qui détient tous les droits sur la machine

Aujourd'hui, ça peut poser des problèmes de sécurité, notamment lorsque plusieurs personnes doivent avoir accès aux privilèges root pour effectuer des tâches d'administration. On ne pourra pas créer plusieurs administrateurs comme sous Windows.

Utiliser root directement présente des risques, car :  
- chaque utilisateur ayant besoin d'effectuer des opérations administratives doit connaître le mot de passe root.  
- les actions sont faites en tant que root et il n'y a pas de traçabilité.

Pour remédier à cette situation, la commande **sudo** est une solution intéressante et conseillée d'ailleurs par l'ANSSI (Rubrique [Recommandations de sécurité relatives à un système GNU/Linux](https://cyber.gouv.fr/publications/recommandations-de-securite-relatives-un-systeme-gnulinux) ).

La commande **sudo** va permettre à des utilisateurs spécifiques d'exécuter des commandes avec les droits de root sans avoir à connaître le mot de passe root. Ainsi, un utilisateur peut exécuter une commande administrative en préfixant celle-ci par sudo, ce qui lui demandera son propre mot de passe pour valider l'opération.

  

Code :

`[sudo] password for adrien:    script-de-root.sh`

Le mot de passe est mémorisé par défaut pour une durée de 5 minutes. Ainsi les prochaines commandes lancées avec sudo ne redemanderont pas le mot de passe de l'utilisateur.

Que la commande ait réussie ou échouée, l'action sera tracée dans les journaux système:

Code TEXT :

janv. 29 18:55:10 superlinux sudo[8801]:   adrien : TTY=pts/2 ; PWD=/home/adrien ; USER=root ; COMMAND=/usr/bin/ls -l /root

Pour configurer les permissions d'utilisation de sudo, on le fera à travers le fichier **/etc/sudoers**. Ce fichier permet de définir quelles commandes chaque utilisateur peut exécuter avec des privilèges élevés. On pourra autoriser un utilisateur à exécuter certaines commandes spécifiques sans lui donner un accès complet au compte root ou toutes les commandes pour avoir des actions d'administration complètes.

On va voir dans cet article comment paramétrer tout cela.

Selon les distributions Linux, le premier utilisateur créé peut, par défaut, exécuter toutes les commandes avec sudo :  
- Sur **RHEL et dérivés** : Si la case "Faire de cet utilisateur un administrateur" a été cochée  
- Sur **Fedora Workstation**, **Ubuntu Desktop**, **Linux Mint** : Le premier utilisateur est concerné (pas de mot de passe root saisi dans le processus d'installation)  
- Sur **Fedora Server** (ou spins) : Si la case "Accorder les privilèges d'administration à cet utilisateur" a été cochée  
- Sur **Debian** : Le premier utilisateur uniquement si à l'installation aucun mot de passe root n'a été saisi

## Utiliser sudo

Bien souvent, on utilisera sudo ainsi, permettant d'excuter la commande avec les droits de root :

Cependant, on pourra utiliser des options avec sudo.

L'option **-u** permet de prendre les droits d'un autre utilisateur, on l'a vu dans les tutos d'administration de Nextcloud avec le script occ :

Code BASH :

sudo -u apache php occ upgrade

On exécute la commande php occ upgrade en tant qu'utilisateur apache.

On a aussi l'option **-i**, on l'a vu dans [Différences sudo, su et su -](https://www.linuxtricks.fr/wiki/differences-sudo-su-et-su), qui permet d'ouvrir un shell correctement en tant que root (ou autre utilisateur si utilisé avec -u) :

L'option **-l** permet de lister les droits. Ici, en tant que brooke :

Après la saisie du mot de passe, les droits sont listés :

Code TEXT :

User brooke may run the following commands on superlinux:
    (ALL) /usr/bin/vim /etc/hosts, /usr/bin/systemctl restart httpd.service
    (apache) /usr/bin/vim /var/www/html/config/config.php

Le mot de passe étant mémorisé 5 minutes (par défaut), si on souhaite révoquer cette mémorisation, l'option **-k** sera utile :

Tout de suite après, le mot de passe sera demandé à nouveau

## Configurer sudo

### Edition de la configuration

Pour éditer le fichier **/etc/sudoers**, il est préférable d'utiliser la commande visudo.  
Si on est root :

Si on est utilisateur ayant accès à sudo :

**visudo** éditera le fichier avec l'éditeur par défaut configuré sur le système

En effet, avec **visudo** on aura les avantages suivants par rapport à l'édition directe :  
- lors de l’enregistrement, l’outil vérifie que la configuration est correcte (pour éviter de se couper les accès en cas d'erreur de syntaxe)  
- empêche l'édition si le fichier est déjà en cours d'édition

Le fichier qui se modifie est directement **/etc/sudoers**.

Comme beaucoup de service sous Linux, il sera possible de créer des fichiers de son choix dans le dossier **/etc/sudoers.d**, si dans le fichier sudoers cette ligne est bien décommentée :

Code BASH :

includedir /etc/sudoers.d

### Configuration par défaut

Chaque ligne du fichier est structurée de la même façon.  
Voici 2 lignes présentes sur une RHEL :

Code :

`root    ALL=(ALL)       ALL   %wheel  ALL=(ALL)       ALL`

La première indique que **l'utilisateur root** peut utiliser la commande sudo pour exécuter toutes les commandes.  
La seconde indique que les utilisateurs présents dans **le groupe wheel** peut utiliser la commande sudo pour exécuter toutes les commandes.

Sur Debian, la syntaxe diffère un peu, et c'est le groupe **sudo** qui a les droits sudo, et non wheel :

Code :

`root    ALL=(ALL:ALL)       ALL   %sudo  ALL=(ALL:ALL)       ALL`

Il y a souvent des exemples dans le fichier déjà commentés.

De façon générale :

Code :

`user ALL = (user) commande1,commande2,...   %groupe ALL = (user) commande1,commande2,...`

- **user** indique le nom d'utilisateur auquel le droit sudo s'appliquera ou **%groupe** indique le nom du groupe auquel le droit sudo s'appliquera. Le nom du groupe doit donc être précédé d'un %.  
- **ALL** indique les machines sur lesquelles on peut exécuter ces commandes (généralement c'est toujours ALL). On aura un nom de machine seulement si on pousse massivement les configs sur plusieurs machines et qu'on veut que le droit ne s'applique qu'à certaines.  
- **(user)** désigne l'utilisateur dont on prend les droits (ALL pour tous, y compris root)  
- **commande** représente des commandes pouvant être exécutées par l'utilisateur ou le groupe désigné en début de ligne. (si plusieurs commandes, les séparer par une virgule (ALL pour toutes).

Utiliser le chemin complet de la commande!  
Si on ne connaît pas le chemin de la commande, utiliser which :  

### Déléguer une commande à un utilisateur

Si je souhaite déléguer une commande particulière à un utilisateur particulier (exemple brooke), on va positionner une ligne de ce type :

Code :

`brooke    ALL=(ALL)       /usr/bin/id`

On mettra bien le chemin absolu du programme pour éviter des surprises

Ainsi brooke pourra exécuter la commande id (seule ou avec des options et arguments) :

Ce qui après mot de passe de brooke saisi, la commande est bien exécutée en root :

Code :

`[sudo] password for brooke:    uid=0(root) gid=0(root) groups=0(root)`

Mais brooke ne peut pas lancer d'autre commande :

Le système renvoie :

Code :

`Sorry, user brooke is not allowed to execute '/usr/bin/ls /root' as root on superlinux.`

C'est tracé dans les logs :

Code :

`janv. 29 19:15:58 superlinux sudo[42568]:   brooke : command not allowed ; TTY=pts/0 ; PWD=/home/brooke ; USER=root ; COMMAND=/usr/bin/ls /root`

### Déléguer une commande avec des options ou arguments précis à un utilisateur

Si on veut autoriser une commande avec des options ou arguments spécifiques, on pourra les spécifier :

Code :

`brooke    ALL=(ALL)      /usr/bin/systemctl restart httpd.service`

Ainsi un redémarrage est autorisé :

Code BASH :

sudo systemctl restart httpd.service

Cependant un arrêt ne l'est pas :

Code BASH :

sudo systemctl stop httpd.service

Le système l'indique :

Code :

`Sorry, user brooke is not allowed to execute '/usr/bin/systemctl stop httpd.service' as root on superlinux.`

### Déléguer plusieurs commandes à un utilisateur

Comme vu précédemment, on va pouvoir autoriser plusieurs commandes à un utilisateur ou un groupe. Pour cela, on va les séparer par des virgules :

Code TEXT :

brooke    ALL=(ALL)      /usr/bin/id, /bin/systemctl restart httpd.service

Ainsi, la commande id pourra être exécutée avec sudo, avec ou sans options. Et à cela, on pourra exécuter uniquement un redémarrage du service httpd.

### Interdire l'exécution d'autres programmes

Certains programmes permettent de lancer d'autres programmes dans ceux-ci.  
C'est le cas de vim par exemple.

Ainsi, si je mets :

Code TEXT :

brooke    ALL=(ALL)          /usr/bin/vim /etc/hosts , /usr/bin/systemctl restart httpd.service

On pourra éditer le fichier, Ceci est autorisé :

On l'a vu dans le [Guide de SUR-VI (Utilisation de vi)](https://www.linuxtricks.fr/wiki/guide-de-sur-vi-utilisation-de-vi) on exécute le script actuellement en cours d'édition avec :

Mais on peut lancer n'importe quelle commande depuis vim, par exemple :

Qui renvoie :

Code :

`uid=0(root) gid=0(root) groups=0(root)`

Donc on peut carrément appeler bash (ou toute autre commande) et on peut contourner les restrictions. Exemple :

Pour cela, si on veut autoriser par exemple l'édition d'un seul fichier mais sans pouvoir exécuter de shell ou d'autres commandes avec **NOEXEC** :

Code TEXT :

brooke    ALL=(ALL)          NOEXEC: /usr/bin/vim /etc/hosts , /usr/bin/systemctl restart httpd.service

Ici, on pourra éditer le fichier /etc/hosts avec vim, mais si on lance :

On a une erreur en retour :

Code TEXT :

Cannot execute shell /bin/bash

Attention car cependant, si on autorise l'exécution d'un scipt avec sudo mais un **NOEXEC**, et que ce script lance d'autres commandes, cela peut évidemment poser souci.

### Déléguer une commande en tant qu'un utilisateur particulier

Si on veut autoriser une commande avec un utilisateur particulier, on pourra le spécifier dans les parenthèses :

Code :

`brooke    ALL=(apache)   /usr/bin/vim /var/www/html/config/config.php`

Ainsi on pourra éditer en tant qu'apache (pas root ici) un fichier spécifique, dont apache est le propriétaire :

Code BASH :

sudo -u apache vim /var/www/html/config/config.php

Comme d'hab, c'est consigné dans les logs :

Code TEXT :

févr. 04 19:40:53 superlinux sudo[1941789]:   brooke : TTY=pts/2 ; PWD=/home/brooke ; USER=apache ; COMMAND=/usr/bin/vim /var/www/html/config/config.php

### Ne pas demander de mot de passe

Attention à cet usage, il est à utiliser que dans des cas précis !

Quand sudo est utilisé par root (avec l'option -u pour exécuter la commande en tant qu'autre utilisateur), le mot de passe de root n'est jamais demandé.

Cependant, si un utilisateur classique exécute une commande avec sudo, le mot de passe est demandé.  
Si on veut qu'un utilisateur (hors root) puisse planifier une commande (dans crontab par exemple) lancée en tant qu'autre utilisateur, la saisie du mot de passe empêchera le script de s'exécuter.

Il est alors possible de ne pas demander la saisie du mot de passe via **NOPASSWD** :

Code TEXT :

brooke    ALL=(apache)   NOPASSWD: /usr/bin/vim /var/www/html/config/config.php

Ensuite l'édition se fait sans demande de mot de passe :

Code BASH :

sudo -u apache vim /var/www/html/config/config.php

### Spécifier plusieurs utilisateurs ou groupes

Plutôt que de multiplier les lignes, on peut spécifier plusieurs utilisateurs. On séparera les utilisateurs (login) ou les groupes (%groupe) par des virgules.  
Dans l'exemple d'édition de fichier en tant qu'apache, si on veut donner ce droit à brooke et à christophe :

Code :

`brooke,christophe    ALL=(apache)   /usr/bin/vim /var/www/html/config/config.php`

## A propos de l'ordre des règles

A noter que si on écrit beaucoup de règles, l'ordre a une importance.  
Si plusieurs règles s'appliquent pour un même cas d'usage, c'est la dernière qui correspond au cas d'usage qui s'applique. (On s'arrête pas à la première règle comme un pare-feu)

Ainsi, si on veut permettre de lancer toutes les commandes vim en tant qu'utilisateur apache, mais ne pas demander de mot de passe pour un fichier particulier **/var/www/html/config/config.php** :

Code BASH :

sudo -u apache vim /var/www/html/config/config.php

On devra écrire les règles **impérativement** dans cet ordre :

Code TEXT :

brooke    ALL=(apache)   /usr/bin/vim
brooke    ALL=(apache)   NOPASSWD:/usr/bin/vim /var/www/html/config/config.php

Si on écrit les règles dans l'autre sens :

Code TEXT :

brooke    ALL=(apache)   NOPASSWD:/usr/bin/vim /var/www/html/config/config.php
brooke    ALL=(apache)   /usr/bin/vim

L'édition du fichier **/var/www/html/config/config.php** avec vim demandra le mot de passe car la deuxième s'appliquera en contredisant la première.

## Paramètres globaux sudo

Dans le fichier sudoers, on pourra changer des valeurs par défaut.

On pourra changer le temps en minutes de mémorisation du mot de passe (il sera redemandé à chaque fois avec 0) :

Code TEXT :

Defaults      timestamp_timeout = 0

La première fois, et uniquement la première fois, vous avez un petit laïus qui vous dit de faire attention avec la commande sudo, on pourra l'afficher à chaque fois avec :

Code TEXT :

Defaults      lecture = always

On pourra changer le texte affiché. Au lieu de ce qui est proposé par sudo, afficher le texte dans le fichier /etc/bannersudo :

Code TEXT :

Defaults      lecture_file = /etc/bannersudo

On pourra appliquer un paramètre global uniquement qu'à un seul utilisateur, en le spécifiant après le symbole deux points :

Code TEXT :

Defaults:brooke      lecture = always

_Ici le message s'affichera toujours pour brooke_

Si on veut afficher des étoiles au moment de la saisie de mot de passe (option qui pour des raisons de sécurité est désactivée par défaut), on pourra le faire via :

## Et pour plus d'infos ...

Comme tout concept sous Linux, on a la page de manuel de sudoers qui est très très très ... complète (plus de 2800 lignes sur RHEL 9) :

  

- [Partager sur Facebook](https://www.facebook.com/share.php?u=https%3A%2F%2Fwww.linuxtricks.fr%2Fwiki%2Fsudo-utiliser-et-parametrer-sudoers "Partager sur Facebook")
- [Partager sur Twitter](https://twitter.com/share?url=https%3A%2F%2Fwww.linuxtricks.fr%2Fwiki%2Fsudo-utiliser-et-parametrer-sudoers&text=Sudo%20%3A%20Utiliser%20et%20param%C3%A9trer%20%28sudoers%29%20-%20Wiki "Partager sur Twitter")
- [Partager par email Partager par Email](https://www.linuxtricks.fr/cdn-cgi/l/email-protection#b986caccdbd3dcdacd84eaccddd69c8b899c8af89c8b89eccdd0d5d0cadccb9c8b89dccd9c8b89c9d8cbd8d49cfa8a9cf880cdcbdccb9c8b899c8b81caccddd6dccbca9c8b809c8b89949c8b89eed0d2d09fdbd6ddc084d1cdcdc9ca9c8af89c8bff9c8bffcecece97d5d0d7ccc1cdcbd0dad2ca97dfcb9c8bffced0d2d09c8bffcaccddd694cccdd0d5d0cadccb94dccd94c9d8cbd8d4dccdcbdccb94caccddd6dccbca "Partager par Email")
- [Imprimer Version imprimable](https://www.linuxtricks.fr/wiki/sudo-utiliser-et-parametrer-sudoers# "Version imprimable")