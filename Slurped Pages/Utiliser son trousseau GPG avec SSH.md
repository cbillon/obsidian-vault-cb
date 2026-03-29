---
link: https://medspx.fr/blog/Sysadmin/gnupg_ssh/
byline: Médéric Ribreux
excerpt: |-
  Posted by Médéric Ribreux
         🗓 2021-06-01
          In 
  blog/
  Sysadmin/
tags:
  - ssh
  - gpg
slurped: 2026-03-19T15:56
title: Utiliser son trousseau GPG avec SSH
---

## Introduction

Depuis la rédaction de mon article sur [Enigma](https://medspx.fr/blog/Sysadmin/enigma_roundcube/), je suis toujours tenté d'aller plus loin avec GnuPG. En effet, une fois qu'on a pris le temps de créer un trousseau de clefs, on a toujours envie de le garnir d'un maximum de clefs différentes, histoire de pouvoir ouvrir le plus de portes possibles sans se prendre la tête.

Dans le monde de l'informatique, c'est pareil. En ce qui me concerne, je tape le plus souvent une phrase de passe lorsque je lance des commandes via ssh. Mais, plutôt que d'avoir un trousseau GPG d'un côté et des clefs privées/publiques pour SSH, ne peut-on pas tout faire avec un seul outil ?

Après quelques recherches, il semble que c'est possible depuis peu. En effet, on peut gérer l'authentification SSH par clef en utilisant GnuPG. Voici comment j'y suis parvenu en utilisant GnuPG en version 2.1, sous Debian Stretch.

## Comprendre ce qu'on fait, c'est mieux

Avant de se lancer, il faut toujours prendre le temps de comprendre ce qu'on fait plutôt que de suivre bêtement une suite de commandes. Ça prend plus de temps, mais vous deviendrez un meilleur administrateur système en suivant cette voie.

Je ne vais pas rentrer dans la théorie du chiffrement asymétrique mais juste faire quelques rappels sur le fonctionnement de GPG et de SSH.

Commençons par SSH… SSH permet de se connecter de manière sécurisée à une machine distante qui dispose d'un serveur SSH en écoute sur un port précis (le 22 par défaut). Si SSH permet d'ouvrir une session en utilisant un login/mot de passe, il encourage surtout l'authentification par clefs privée/publique. En règle générale, la clef privée est protégée par une phrase de passe (un long mot de passe).

SSH dispose d'ailleurs d'un agent qui gère ces clefs privées/publiques. En effet, si on utilise souvent SSH, on doit taper sans cesse la phrase de passe de la clef privée. L'agent SSH permet de disposer d'une espèce de cache sécurisé de phrase de passe. Cela évite de taper tout le temps la phrase de passe, le tout, avec une méthode sécurisée sur le poste de travail.

En ce qui concerne GPG, il existe également un agent (gpg-agent). Son rôle est de gérer une espèce de cache de clefs (privées et publiques) pour éviter de vous demander de taper votre mot de passe de clef privée sans arrêt. Il permet aussi de transmettre les clefs publiques à d'autres programmes (comme ssh par exemple). Finalement l'agent GPG fait presque la même chose que l'agent SSH sauf que chacun des programmes dispose de son jeu de clefs à lui, avec ses formats spécifiques.

Enfin, comme l'agent GPG a besoin que vous tapiez votre mot de passe de clef privée de temps en temps, il existe un programme qui permet de saisir ce mot de passe (ou cette phrase de passe bien souvent). Ce programme se nomme pinentry et peut être servi par plusieurs binaires selon l'environnement. En effet, si vous avez une session une session Gnome, vous aurez droit à une belle fenêtre adaptée au style de ce gestionnaire de bureau. En revanche, si vous êtes sur un terminal pur, il vous faudra un pinentry capable de s'afficher sur un terminal (pinentry-curses ou pinentry-tty). Pour ma part, comme j'utilise i3-wm et beaucoup de Qt (grâce à QGIS), j'utilise pinentry-qt4.

## Création d'une sous-clef pour l'authentification SSH

Bon, le premier truc à faire, c'est de générer une sous-clef (subkey) dédiée à la fonction d'autorisation. En effet, SSH n'accepte pas n'importe quel type de clef et l'agent GPG non plus.

Voici quelles sont les étapes à suivre. Ce n'est pas très compliqué, il suffit d'utiliser la fonction `--edit-key` de gpg avec l'option `--expert` qui seule donne accès à la création de clef d'authentification:

gpg --edit-key --expert mon_compte
gpg (GnuPG) 2.1.18; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

La clef secrète est disponible.

sec  rsa4096/1234567890123456
 créé : 2017-07-22  expire : 2018-07-22  utilisation : SC
 confiance : ultime        validité : ultime
ssb  rsa4096/1234567890123456
 créé : 2017-07-22  expire : 2018-07-22  utilisation : E
ssb  rsa2048/1234567890123456
 créé : 2017-07-22  expire : 2018-07-22  utilisation : S
ssb  rsa2048/1234567890123456
 créé : 2017-07-22  expire : 2018-07-22  utilisation : E

gpg> addkey
Sélectionnez le type de clef désiré :
(3) DSA (signature seule)
(4) RSA (signature seule)
(5) Elgamal (chiffrement seul)
(6) RSA (chiffrement seul)
(7) DSA (indiquez vous-même les capacités)
(8) RSA (indiquez vous-même les capacités)
(10) ECC (signature seule)
(11) ECC (indiquez vous-même les capacités)
(12) ECC (chiffrement seul)
(13) Clef existante
Quel est votre choix ? 8

Actions possibles pour une clef RSA : Signer Chiffrer Authentifier
Actions actuellement permises : Signer Chiffrer

(S) Inverser la capacité de signature
(C) Inverser la capacité de chiffrement
(A) Inverser la capacité d'authentification
(Q) Terminé
Quel est votre choix ? A
…
Quel est votre choix ? C
…
Quel est votre choix ? S
Actions possibles pour une clef RSA : Signer Chiffrer Authentifier
Actions actuellement permises : Authentifier
(S) Inverser la capacité de signature
(C) Inverser la capacité de chiffrement
(A) Inverser la capacité d'authentification
(Q) Terminé

Quel est votre choix ? Q
les clefs RSA peuvent faire une taille comprise entre 1024 et
4096 bits.
Quelle taille de clef désirez-vous ? (2048) 4096
La taille demandée est 4096 bits
Veuillez indiquer le temps pendant lequel cette clef devrait être
valable.
	 0 = la clef n'expire pas
  <n>  = la clef expire dans n jours
  <n>w = la clef expire dans n semaines
  <n>m = la clef expire dans n mois
  <n>y = la clef expire dans n ans
Pendant combien de temps la clef est-elle valable ? (0)
La clef n'expire pas du tout
Est-ce correct ? (o/N) o
Faut-il vraiment la créer ? (o/N) o
…

Vous disposez maintenant d'une clef GnuPG pour l'authentification. Reste à pouvoir l'utiliser correctement dans SSH…

## Substitution de ssh-agent par gpg-agent

Depuis quelques versions maintenant, l'agent de gestion des passphrases de GnuPG (v2 pour rappel) peut servir d'agent SSH en lieu et place de celui fourni avec OpenSSH.

Sur un système moderne comme Debian Stretch, il faut juste le configurer dans le fichier de configuration de l'agent GPG pour lui indiquer l'option `enable-ssh-support` dans son fichier de configuration. Ce dernier se nomme `~/.gnupg/gpg-agent.conf`. S'il n'existe pas, vous devrez le créer et lui ajouter une ligne avec `enable-ssh-support`.

Ensuite, il faut juste recharger l'agent GPG. Néanmoins, il faut également désactiver l'agent SSH. Ce n'est pas une mince affaire parce qu'il est géré par systemd. Le moyen le plus simple et le plus efficace consiste à fermer et rouvrir la session graphique. En effet, l'unité systemd qui gère l'agent SSH se lance uniquement lors de l'ouverture d'une session graphique.

Bien entendu, si vous avez du temps et que vous souhaitez personnaliser un peu plus le comportement de l'agent GPG, je vous conseille de lire la documentation de ce dernier via `man gpg-agent`.

## Modification du fichier sshcontrol

Pour que l'agent GPG transmette bien des clefs d'authentification à ssh, il faut encore lui indiquer quelle clef utiliser. En effet, si vous avez un trousseau GnuPG étoffé, vous ne voudrez sans doute pas autoriser toutes les clefs à être utilisées par ssh. L'agent GPG dispose d'un fichier dédié dans `~/.gnupg/sshcontrol`.

Nous allons donc simplement indiquer notre clef d'authentification que nous avons créé auparavant dans ce fichier. Mais attention, ce dernier exige non pas un uid de clef mais un keygrip. Vous pouvez l'obtenir à l'aide de l'option `--with-keygrip`:

gpg --list-public-keys --with-keygrip mon_compte
/home/mon_compte/.gnupg/pubring.gpg
-------------------------------
pub   rsa4096/879357560768796E 2017-07-22 [SC] [expire : 2018-07-22]
  1C848291661BD7E20B873A27879357560768796E
  Keygrip = 1801833AA0573C2372AD196729C6B004A4BF891B
uid                [  ultime ] mon_compte <mon_compte@example.com>
sub   rsa4096/792D62F3336393FA 2017-07-22 [E] [expire : 2018-07-22]
  Keygrip = F8894EEFCF321067DA881E2AC471C9BACF0B8067
sub   rsa2048/A44D23DA9C451C04 2017-07-22 [S] [expire : 2018-07-22]
  Keygrip = 8582CAA7B033E8B775809BA05ED37B3463CB0F30
sub   rsa2048/B040BDF79DB1058C 2017-07-22 [E] [expire : 2018-07-22]
  Keygrip = 60B08365121D1B34E911C9EA8138946DB1FE8FCA
sub   rsa4096/519BE81D08A417C7 2017-11-11 [A] [expire : 2018-07-22]
  Keygrip = EB994608FCAB196BF0D79602D3A04F043F5DEE9F

Dans mon cas, la sous-clef utilisée est la dernière (elle a l'option [A] pour authentification). Son keygrip est `EB994608FCAB196BF0D79602D3A04F043F5DEE9F`. Vous avez juste à renseigner cette valeur dans le fichier `~/.gnupg/sshcontrol` avec votre éditeur de texte préféré ou une redirection shell.

Pour vérifier que ça fonctionne, la commande `ssh-add -l` doit maintenant indiquer la clef d'authentification rentrée en amont.

## Export de la sous-clef dans les clefs autorisées de SSH

Maintenant que tout est configuré sur la machine qui va se connecter à distance, il faut quand même indiquer à la machine d'en face quelle clef publique doit être acceptée. Dans le temps, il fallait utiliser un programme tiers de la suite monkeysphere mais, depuis GnuPG 2.1, il existe une commande d'export qui permet de le faire assez simplement. Il s'agit de l'option `--export-ssh-key`. Elle exporte une clef publique au format accepté par SSH. Il n'y a qu'à l'utiliser de la manière suivante:

gpg --export-ssh-key mon_compte >> ~/.ssh/authorized_keys

Par défaut, cette commande exporte la première clef d'authentification au format SSH, directement prêt à être intégrée dans un fichier d'autorisation SSH.

Pour exporter la clef à distance, vous pouvez simplement utiliser la commande ssh dédiée: `ssh-copy-id mon_compte@machine_distante`. En effet, cette commande utilise le résultat de `ssh-add` pour envoyer les clefs à distance. Comme `ssh-add` utilise maintenant GnuPG, il n'y a pas de problème pour l'utiliser.

## Que faire si ça plante ?

Déjà, vous devez voir si l'agent SSH est toujours actif. Si c'est le cas, c'est mauvais signe, vous devez trouver le moyen de le gicler. Le plus simple est de lancer une commande du type `set | grep SSH`. Elle doit vous retourner uniquement la variable SSH_AUTH_SOCK qui contient la socket de l'agent SSH. Ce denier doit être l'agent GPG. C'est facilement reconnaissable dans le contenu de la variable qui doit afficher un truc avec gpg-agent.ssh.

Autre point qui peut poser problème: il manque une sous-clef privée d'authentification ou une clef d'authentification est inutilisable. Ça m'est arrivé assez facilement en exportant une sous-clef d'authentification en oubliant d'incorporer la sous-clef privée. Après l'import dans un autre trousseau sur une autre machine, la commande gpg --list-secret-keys m'indiquait une sous-clefs inutilisable (ssb#). J'ai dû procéder à une exportation complète en utilisant l'option `--export-secret-subkeys` de GnuPG.

Autre symptôme lié au point précédent: si `ssh-add -l` n'affiche rien ou un message d'erreur, je vous conseille de jeter un coup d'oeil à votre journal système utilisateur (via `journalctl --user -xe`) pour voir si vous avez des erreurs concernant une clef manquante dans le fichier sshcontrol. En effet, pour le keygrip d'une clef située dans le fichier `sshcontrol` soit pris en compte, il faut absolument qu'il y ait le fichier correspondant dans `~/.gnupg/private-keys-v1.d/`. Si ce n'est pas le cas, vous avez sans doute un problème avec une sous-clef privée.

## Addendum du 01/06/2021

Parfois, le problème se situe plutôt du côté de gpg que de SSH, notamment dans l'interface qui permet de saisir la phrase de passe de la clef privée (pinentry). J'ai remarqué que sous le terminal Kitty et aussi sur une session à distance, pinentry avait du mal à détecter correctement l'environnement et ne permettait pas de saisir la phrase de passe. Après avoir galéré pas mal de temps avec ça, j'ai trouvé un contournement: lancer la commande suivante avant d'utiliser une commande qui demanderait le mot de passe:

gpg-connect-agent updatestartuptty /bye

## Conclusions

Avec cette recette, vous pouvez enfin vous passer des clefs SSH classiques et de son agent. Ce mode de gestion unifié qui passe par GnuPG vous permettra de renforcer le rôle de cette solution de chiffrement qui se révèle finalement assez complète.

De plus, cela vous encouragera à avoir GnuPG un peu partout sur vos machines et à vous organiser pour gérer votre trousseau au mieux.

Prochaine étape ? Peut-être utiliser une clef de sécurité ou une carte OpenPGP pour gérer le trousseau. Ou encore mettre en place [Pass](https://www.passwordstore.org/) qui utilise nativement GnuPG pour sécuriser les autres mots de passe…