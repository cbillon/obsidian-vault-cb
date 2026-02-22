---
link: https://blog.cclaude.rocks/post/2024/07/10.executer-tache-modification-dossier.html
site: cClaude.rocks
date: Invalid date
excerpt: Le but de ce billet est de pouvoir exécuter une tâche (un script bash par exemple) lorsque le contenu d’un répertoire a été modifié.
slurped: 2026-02-08T10:16
title: 🐚 Exécuter une tâche lors de la modification du contenu d’un dossier
tags:
  - inotify
  - bash
---

Le but de ce billet est de pouvoir exécuter une tâche (un script **bash** par exemple) lorsque le contenu d’un répertoire a été modifié.

Pour cela on s’appuiera sur le paquet **[inotify-tools 📦](apt://inotify-tools)**.

---

ඏ

---

## Une solution basée sur **inotify**

Dans la plupart des cas, **inotify** est la solution la plus efficace et la plus raisonnable pour suivre les changements de fichiers dans les répertoires que l'on souhaite surveiller. Il a été intégré au code commun du noyau Linux en 2005, ce qui en fait un standard dans toutes les distributions Linux.

Notez que **inotify** a quelques limitations. Le principal problème est qu’il nécessite que le noyau soit au courant de tous les événements pertinents du système de fichiers, ce qui n’est pas toujours possible par exemple pour **NFS**, et de manière générale pour les répertoires partagés. Vous devez tenir compte de ces limitations.

**inotifywait** et **inotifywatch** sont des commandes du paquet **inotify-tools** qui permettent d’utiliser le sous-système **inotify**. **inotifywait** attend les événements du système de fichiers et agit dès qu’il en reçoit un. **inotifywatch** collecte les statistiques d’utilisation du système de fichiers et donne le nombre de chaque événement configuré. Dans la suite, nous ne nous intéresserons qu’à **inotifywait**.

---

ඏ

---

## Exemple de script **Bash** utilisant **inotify**

Voyons tout de suite notre script :

```
#!/bin/bash

if [ -z "$(which inotifywait)" ]; then
    echo "inotifywait not installed."
    echo "In most distros, it is available in the inotify-tools package."
    exit 1
fi

counter=0;

function execute() {
    counter=$((counter+1))
    echo "Detected change n. $counter"
    eval "$@"
}

inotifywait --recursive --monitor --format "%e %w%f" \
--event modify,move,create,delete ./ \
| while read changed; do
    echo $changed
    execute "$@"
done
```

Nous n’allons pas nous lancer dans une explication de chaque ligne de notre script. Voici une présentation des parties les plus importantes :

- Tout ce qui suit le nom du script est interprété comme la commande à exécuter, grâce à la variable spéciale `$@` entre guillemets passée à la fonction `execute()`
- Chaque ligne de sortie de **inotifywait** (formatée comme spécifié par le drapeau `--format`) est temporairement stockée dans la variable `$changed`, grâce au tuyau entre la commande **inotifywait** et la boucle `while read`.
- Au lieu de se terminer après avoir reçu un seul événement (ce qui est le cas par défaut), **inotifywait** s’exécute indéfiniment grâce à l’option `--monitor`. C’est un gain de performance impressionnant comparé au redémarrage d'**inotifywait** après chaque événement.
- L’option `--event` spécifie les événements à surveiller dans le répertoire courant et les sous-répertoires, en surveillant de manière récursive jusqu’à une profondeur illimitée comme demandé par l’option `--recursive`.
- Les sous-répertoires nouvellement créés seront également surveillés.

Sauvegardons notre script sous le nom de `inotifyTest.sh` dans le répertoire à surveiller, puis ouvrons deux terminaux. Nous utiliserons le premier pour voir comment notre script se comporte et le second pour effectuer des opérations dans le répertoire surveillé.

Démarrons le script dans le premier terminal. Dans ce cas, la commande à exécuter est un simple **echo** :

```
./inotifyTest.sh echo "Running our command..."
```

```
Setting up watches.  Beware: since -r was given, this may take a while!
Watches established.
```

Essayons ensuite d’effectuer quelques opérations sur les fichiers et les répertoires dans le second terminal :

```
touch newFile.txt

echo "Some content" >> newFile.txt

rm newFile.txt

mkdir testDir

cd testDir
touch anotherFile.txt
cd ..

rm -fR testDir
```

Pendant ce temps, le premier terminal a enregistré toutes les opérations correctement.

Incidemment, nous notons que la dernière commande `rm -fR testDir` a en fait effectuée deux opérations :

```
CREATE ./newFile.txt
Detected change n. 1
Running our command...
MODIFY ./newFile.txt
Detected change n. 2
Running our command...
DELETE ./newFile.txt
Detected change n. 3
Running our command...
CREATE,ISDIR ./testDir
Detected change n. 4
Running our command...
CREATE ./testDir/anotherFile.txt
Detected change n. 5
Running our command...
DELETE ./testDir/anotherFile.txt
Detected change n. 6
Running our command...
DELETE,ISDIR ./testDir
Detected change n. 7
Running our command...
```

Tout fonctionne donc comme prévu. Cependant, nous devons faire attention à nos cas d’utilisation réels, comme nous le verrons dans la suite.

---

ඏ

---

## Affinage du script

Le script peut détecter beaucoup plus d’événements que nous ne le souhaiterions. Par exemple, ouvrons un fichier texte préexistant `test.txt` avec **xed**, apportons-y une modification et enregistrons-le. Nous nous attendions à un seul événement, mais notre script en détecte quatre :

```
./inotifyTest.sh echo "Running our command..."
```

```
Setting up watches.  Beware: since -r was given, this may take a while!
Watches established.
CREATE ./.goutputstream-7FUFN1
Detected change n. 1
Running our command...
MODIFY ./.goutputstream-7FUFN1
Detected change n. 2
Running our command...
MOVED_FROM ./.goutputstream-7FUFN1
Detected change n. 3
Running our command...
MOVED_TO ./test.txt
Detected change n. 4
Running our command...
```

Ce comportement inattendu est dû à l’utilisation de fichiers temporaires dont nous ne sommes généralement pas conscients. Le même problème se pose avec d’autres éditeurs de terminaux largement utilisés, tels que **nano**.

Fondamentalement, il existe deux approches pour résoudre ce problème. La première consiste à restreindre le type d’événements surveillés, à savoir ceux indiqués par l’indicateur `--event`. La seconde consiste à exclure les fichiers ou répertoires non pertinents en utilisant l’option `--exclude` ou `--excludei`.

Par exemple, refaisons le même test avec **xed**, mais excluons tous les fichiers et répertoires cachés en ajoutant `--exclude '/\.'` aux paramètres de inotifywait. Ce drapeau accepte une **expression régulière étendue POSIX**, nous devons donc échapper les points. Voici le résultat :

```
./inotifyTest.sh echo "Running our command..."
```

```
Setting up watches.  Beware: since -r was given, this may take a while!
Watches established.
MOVED_TO ./test.txt
Detected change n. 1
Running our command...
```

Des quatre événements précédemment détectés, notre script n’a détecté cette fois que le dernier. C’est ce que nous voulions. En général, nous devons analyser nos cas d’utilisation pour trouver les expressions régulières d’exclusion les plus appropriées.

---

ඏ

---

## Nombre maximum de surveillances « inotify »

Dans la plupart des cas, notre script fonctionnera correctement. Cependant, il peut atteindre la limite du système pour le nombre d’observateurs de fichiers si le nombre de fichiers est considérable.

Essayons `tail -f` sur n’importe quel fichier ancien pour vérifier si notre système d’exploitation a dépassé la limite maximale de surveillance fixée par **inotify** :

```
tail -f /var/log/dmesg
```

L’implémentation interne de `tail -f` utilise le mécanisme **inotify** pour surveiller les changements de fichiers. Si tout va bien, il affichera les dix dernières lignes et fera une pause ; alors, abandonnons avec CTRL + c. Au lieu de cela, si nous n’avons plus de surveillance **inotify**, nous obtiendrons probablement cette erreur :

```
tail: inotify cannot be used, reverting to polling: Too many open files
```

**sysctl** permet de consulter la configuration actuelle :

```
fs.inotify.max_queued_events = 16384
fs.inotify.max_user_instances = 128
fs.inotify.max_user_watches = 65536
```

Voyons ce que signifient ces valeurs :

- `max_queued_events` : le nombre maximum d’événements dans la file d’attente du noyau.
- `max_user_instances` : le nombre maximum d’instances de surveillance, égal au nombre de répertoires racines à surveiller.
- `max_user_watches` : le nombre maximum de répertoires dans toutes les instances de surveillance.

Il est possible de modifier `max_user_instances` et `max_user_watches` mais il est généralement conseillé de conserver la valeur par défaut de `max_queued_events`. L’augmentation de ces valeurs doit se faire avec prudence, en effet chaque observateur occupe 1 ko de mémoire noyau sur les systèmes 64 bits, hors ce type de mémoire n’est pas « swappable ».

Pour modifier la configuration de façon permanente, il faut éditer le fichier `/etc/sysctl.conf` avec les permissions de root (sur les dérivés de Debian/RedHat), en modifiant les lignes suivantes ou en les ajoutant si elles n'existent pas. N’oubliez pas de remplacer **n** par la valeur souhaitée (le maximum est 524288) :

```
fs.inotify.max_queued_events = n
fs.inotify.max_user_instances = n
fs.inotify.max_user_watches = n
```

Rechargeons ensuite les paramètres de **sysctl** (sur les dérivés de Debian/RedHat) :

```
sysctl -p
```

---

ඏ

---

## Étude de quelques exemples

---

### Surveillance d’un seul fichier

La documentation de **inotifywait** donne l’exemple suivant :

```
#!/bin/sh
while ! inotifywait -e modify /var/log/syslog; do
  if tail -n1 /var/log/syslog | grep httpd; then
    kdialog --msgbox "Apache needs love!"
  fi
done
```

L’exemple utilise en réalité `/var/log/messages` mais depuis l’arrivée de **systemd**, ce fichier n’existe plus. Il se peut également que le fichier `/var/log/syslog` puisqu’il n’est en réalité pas nécessaire à **systemd**, mais qu’il permet un portage simple de certaine applications.

Dans ce cas, puisqu’on ne surveille qu’un seul fichier, il est possible de relacer à chaque changement la commande **inotifywait**, cette méthode reste relativement efficace.

---

### Synchronisation d’un répertoire

Créer un fichier `inotify-rsync.sh` dans le dossier à synchroniser et rendez le exécutable :

```
touch inotify-rsync.sh
chmod +x inotify-rsync.sh
```

Inspirons-nous du premier script, pour faire la synchronisation du répertoire à chaque changement.

La variable destination est à adapter en fonction de vos besoins…

```
#!/bin/bash
declare -r DESTINATION='user@192.168.2.73:/sdcard/Movies.local/Videos' # A adapter

if ! which inotifywait >/dev/null ; then
  echo >&2 "inotifywait not installed."
  echo >&2 "In most distros, it is available in the inotify-tools package."
  exit 1
fi

function run_action() {
  rsync --delete-excluded --archive --compress --copy-links --delete --progress \
        --recursive --update --verbose -e 'ssh -p 2223' \
        . \
        "${DESTINATION}"
}

inotifywait --recursive --monitor --format "%e %w%f" \
            --event modify,move,create,delete . | while read changed; do
  echo $changed
  run_action
done
```

Les problèmes liés aux fichiers temporaires n’est pas géré ici, mais ce n’est pas le seul problème.

La configuration laisse penser qu’on synchronise un dossier contenant des vidéos. Le script tel qu’il est écrit risque de relancer la synchronisation si des nouvelles vidéos sont ajoutées alors que la synchronisation n’est pas terminée. Dans ce genre de situation, il faudra donc prévoir un système pour ne pas relancer la synchronisation tant que la précédente n’est pas terminée.

Cela peut se faire par le dépôt d’un fichier `.lock`.

Il faudrait également s’assurer que la machine de destination soit joignable et si elle ne l’est pas prévoir un mécanisme pour traiter la synchronisation ultérieurement.

---

ඏ

---

## Liens

- Ce billet s’inspire de ce post : [Execute a Command Whenever File or Directory Changes](https://www.baeldung.com/linux/command-execute-file-dir-change)
- [Execute a Command Whenever File or Directory Changes](https://superuser.com/questions/181517/how-to-execute-a-command-whenever-a-file-changes)