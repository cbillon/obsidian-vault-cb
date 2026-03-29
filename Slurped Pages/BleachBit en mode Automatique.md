---
link: https://www.metal3d.org/blog/2024/bleachbit-automatique/
site: Metal3d
date: 2024-04-05T23:26
excerpt: Comment planifier un nettoyage de disque avec BleachBit et systemd. Voilà la méthode !
tags:
  - linux
slurped: 2026-03-28T10:24
title: BleachBit en mode Automatique
---

Comment planifier un nettoyage de disque avec BleachBit et systemd. Voilà la méthode !

BleachBit est un outil fantastique pour nettoyer votre système. Mais, bien que l’interface graphique soit très bien pensée, j’avais envie d’automatiser le processus. Et comme on est sous Linux, évidemment, ça se fait super facilement.

> Je vous préviens d’avance, c’est un article assez simple, rien de techniquement innovant. Mais je vois tellement de gens faire les choses un peu n’importe comment, que je me suis dit que ça valait le coup de l’écrire. Aucune animosité de ma part, enfin… si… mais c’est bon enfant. Ne le prenez pas mal 😄

Cet article est un prétexte pour vous parler de systemd, et notamment les timers, en mode “utilisateur” (donc, sans `sudo`, sans taper dans le système, sans prendre des risques de tout péter). Mais je veux aussi mettre en lumière BleachBit, un outil que j’utilise depuis des années et qui m’a toujours rendu de fiers services. Le fait de pouvoir nettoyer son système aussi efficacement, et de pouvoir automatiser le processus, ça m’a paru une bonne idée de vous montrer “ze way to do it” (en mode “je fais ça bien”).

> Non parce que je suis généralement en tain de grogner devant mon écran en lisant certains articles, rédigés par des gorets qui adorent claquer du `sudo` partout. Alors que non, on ne fait pas ça. On est des gens bien, on ne touche pas au système quand il ne faut pas, on ne le casse pas, on ne le corrompt pas. **Arrêtez de faire ça**. Sudo, c’est pour les opérations qui nécessitent des droits d’administration. Point barre.
> 
> - Installer un paquet: OUI
> - Ajouter un timer pour un truc perso sur mon “home”: NON
> 
> Sérieux, arrêtez de faire ça. C’est pas bien.

Parlons de BleachBit, que j’utilise depuis des années.

J’ai très souvent des problèmes de place sur mon disque dur. J’ai beau avoir 500 Go sur le laptop et utiliser des disques distants pour stocker mes données, je me retrouve souvent à court d’espace. C’est moins le cas sur mon poste “desktop” qui, lui, à 2To de stockage. Mais voilà, j’utilise mon laptop, je télécharge des modèles d’IA, je teste, je code, je fais un peu de Blender… En quelques jours, voire quelques semaines si je me calme, j’atteins 80% d’occupation du disque et ça m’énerve. Et BleachBit me règle ça en quelques secondes.

BleachBit est développé en Python et vous trouverez la page officielle [ici](https://www.bleachbit.org/). Et les sources se trouvent sur leur GitHub [ici](https://github.com/bleachbit/bleachbit). Cet outil est une vraie pépite, et je vous encourage à le tester.

## Installation

Bon, pour commencer, il faut installer BleachBit. C’est un logiciel libre, et vous pouvez le trouver dans les dépôts de votre distribution. Par exemple, pour Ubuntu, ça se fait avec la commande suivante :

```
sudo apt install bleachbit
```

Mais comme vous êtes des gens bien, vous utilisez Fedora n’est-ce pas ? Dans ce cas, vous pouvez installer BleachBit avec la commande suivante :

```
sudo dnf install bleachbit
```

Voilà, passons à la suite…

## Premier nettoyage

On va le faire avec l’interface graphique pour commencer. Il y a une raison pour cela : quand vous modifiez les options de nettoyage, BleachBit enregistre ces options dans un fichier de configuration. Vous le trouverez dans `~/.config/bleachbit/bleachbit.ini`.

Pourquoi je fais ça ? Eh bien en fait on pourrait très bien se passer de cette étape. Il faudrait alors utiliser la ligne de commande pour lancer le nettoyage avec un tas de “cleaners” dont il faut connaitre non seulement le nom, mais aussi ce que ça fait. Et franchement j’avais la flème.

L’UI vous donne plein d’information sur chaque “cleaner”. Alors je vous propose de sélectionner :

- pour tous les navigateurs que vous avez installés (firefox, chromium, brave, etc.), de virer le cache, et optimiser les bases de données
- dans la section “système”, vider le cache et les fichier temporaires. Aussi, de supprimer les “fichier corrompus” et la corbeille.
- et ensuite de faire le nécessaire pour les applications que vous utilisez. Lisez bien ce que ça fait (histoire de ne pas perdre un truc important, comme les mots de passe de votre navigateur par exemple).
- Dans analyse approfondie, j’ai personnellement coché la suppression des fichier de sauvegarde et les fichiers temporaires.

> Bien entendu, ce sont mes choix, à vous de voir ce que vous voulez nettoyer. Faites juste attention à ne pas supprimer des trucs importants, j’insiste.

Aussi, dans les paramètres, j’ai ajouté des répertoires que je veux vraiment ne pas nettoyer. Par exemple, j’ai ajouté “`~/.cache/oh-my-posh/`” (pour ne pas perdre la configuration de mon terminal qui stocke les thèmes dedans)[1](https://www.metal3d.org/blog/2024/bleachbit-automatique/#fn:1).

Bon, normalement, vous pouvez “prévisualiser” le nettoyage pour voir ce qui va être supprimé. Si vous êtes sûr de vous, vous pouvez lancer le nettoyage.

> La première fois, j’ai récupéré près de 120Go d’espace disque. C’est pas mal… allez ne faisons pas les blasés, c’est énorme !

## Automatisation

Fermez BleachBit et regardez le fichier `~/.config/bleachbit/bleachbit.ini`. Vous y trouverez toutes les options que vous avez sélectionnées.

Chez moi, par exemple, j’ai ceci :

```
# ... des trucs et puis:

[list/shred_drives]
0 = /home/metal3d/.cache
1 = /tmp

[preserve_languages]
en = True
fr = True

[tree]
brave.cache = True
brave = True
firefox.cache = True
firefox = True
google_chrome.cache = True
google_chrome = True
journald.clean = True
journald = True
system.cache = True
system = True
system.trash = True
thunderbird.cache = True
thunderbird = True
vlc.mru = True
vlc = True
brave.vacuum = True
firefox.vacuum = True
google_chrome.vacuum = True
thunderbird.vacuum = True
system.desktop_entry = True
system.tmp = True
deepscan.backup = True
deepscan = True
deepscan.tmp = True

[whitelist/paths]
0_type = folder
0_path = /home/metal3d/.cache/oh-my-posh/themes
```

Et c’est pratique, parce que la ligne de commande suivante va reproduire votre nettoyage :

```
# pour nettoyer
bleachbit --preset -c

# pour prévisualiser ce que ça va faire... c'est bien de le faire avant
bleachbit --preset -p
```

On peut donc demander de lancer cette commande tous les lundis matin à 3 h du mat’ (et au pire, si le PC est éteint, ça se passera au démarrage de votre ordinateur, voir [la documentation](https://wiki.archlinux.org/title/systemd/Timers#Realtime_timer) de l’option “Persistent”).

Alors, non, on ne va pas utiliser le vieux “crontab”[2](https://www.metal3d.org/blog/2024/bleachbit-automatique/#fn:2). Non vraiment je l’adore, c’est un vieux copain, mais `systemctl` est tellement plus adapté. Moi, je ne fais pas partie des détracteurs qui ont menacé de mort l’auteur de systemd. J’espère d’ailleurs que ces crétins aient eu ce qu’ils méritaient[3](https://www.metal3d.org/blog/2024/bleachbit-automatique/#fn:3). Bref, je m’égare.

Le “souci” c’est que, quand on ne le sait pas (mais moi je sais, nananère), systemd est “de base” un daemon[4](https://www.metal3d.org/blog/2024/bleachbit-automatique/#fn:4) qui tourne via l’utilisateur “root”. Donc si vous allez claquer des fichiers avec `sudo` dans `/etc/systemd/system/`, vous devriez tout de suite arrêter ce que vous faites. C’est aussi grave que ceux qui vous disent de faire un `sudo pip install` (sérieux, arrêtez de faire ça).

Sauf que systemd est bien pensé, et ouais. Vous avez aussi un espace utilistateur pour y assigner vos `services`, `timers`, `mounts` et `sockets`. C’est dans `~/.config/systemd/user/` que ça se passe. Mais j’ai tendance à oublier ce dossier.

L’astuce, c’est de s’en foutre. On va utiliser une ligne de commande pour éditer les deux fichiers nécessaires.

La première commande :

```
systemctl --user edit --force --full bleachbit.service
```

Bim, un éditeur s’ouvre, et vous collez le contenu suivant :

```
[Unit]
Description=Clean disk space with BleachBit

[Service]
Type=oneshot
ExecStart=/usr/bin/bleachbit --preset --clean

[Install]
WantedBy=default.target
```

Sauvez le fichier, et voilà. Le service existe. Maintenant, on va créer un “timer” pour lancer ce service tous les lundis matin à 3 h.

```
systemctl --user edit --force --full bleachbit.timer
```

Et encore un éditeur qui s’ouvre !

```
[Unit]
Description=Clean the disk space each monday at 3:00 AM

[Timer]
OnCalendar=Mon *-*-* 03:00:00 Europe/Paris
Persistent=true

[Install]
WantedBy=timers.target
```

> Le fonctionnement de systemd est d’avoir un timer qui porte le même nom que le service. C’est pour ça que je vous ai demandé de créer un service `bleachbit.service` et un timer `bleachbit.timer`. Dans les faits, un “timer”, execute un “service” à un temps donné.

Bon et bien allez hop, on les active et voilà !

```
systemctl --user enable --now bleachbit.service
systemctl --user enable --now bleachbit.timer
```

C’est tout, ça va marcher. Et si vous voulez vérifier que tout est bien configuré, vous pouvez lancer la commande suivante :

```
# voir l'état des timers
systemctl --user list-timers

# voir les logs du timer
journalctl --user -u bleachbit.timer
journalctl --user -u bleachbit.service
```

Et vous pouvez lancer le nettoyage manuellement si vous le voulez:

```
systemctl --user start bleachbit.service
```

Bref, c’est vraiment propre, intégré dans votre environnement, c’est centralisé (logs avec `journalctl`, activation avec `systemctl`)… Ce n’est plus du bricolage avec des scripts qui en foutent partout. Propre. Zen.

## Conclusion

BleachBit c’est le pied, et systemd c’est le pied. Alors les deux ensemble, bah, c’est les deux pieds. C’est Jean-Claude Van Damme qui me l’a dit.