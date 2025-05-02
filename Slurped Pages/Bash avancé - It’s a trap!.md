---
link: https://xieme-art.org/post/bash-avance-its-a-trap/
byline: ephase
site: Xieme-Art
date: 2022-11-15T23:10
excerpt: Pour l’instant, on ne peut pas dire que 2022 soit une année productive côté article de blog. Le seul et unique article de cette année...
twitter: https://twitter.com/@ephase
tags:
  - bash
slurped: 2024-12-19T08:37
title: "Bash avancé: It’s a trap!"
---

Pour l’instant, on ne peut pas dire que 2022 soit une année productive côté article de blog. Le seul et unique article de cette année parlait de _Bash_ qui a eu un peu de succès. Comme toute les séries B un peu populaire, il lui fallait une suite. Et bien la voici!

Dans cette suite, il va être question de signaux, de **“trap”** et bien entendu de _Bash_.

## Dis, c’est quoi un signal

Un signal est une sorte de notification envoyée à un processus lorsqu’un événement particulier a eu lieu. Plus concrètement, lorsqu’un programme ne répond pas dans votre terminal et que vous faites Ctrl+C, le système envoi le signal SIGINT au processus en cours lui signifiant d’interrompre séance tenante toute action. Chaque signal est associé à un numéro

Voici un liste de quelques signaux (norme POSIX) :

|Signal|Valeur|Action|Commentaire|
|---|---|---|---|
|SIGHUP|1|Term|Déconnexion du terminal ou fin du processus de contrôle|
|SIGINT|2|Term|Interruption depuis le clavier `CTRL + C`|
|SIGQUIT|3|Core|Demande ”Quitter” depuis le clavier `CTRL + \`|
|SIGILL|4|Core|Instruction illégale|
|SIGABRT|6|Core|Signal d’arrêt depuis abort(3)|
|SIGFPE|8|Core|Erreur mathématique virgule flottante|
|SIGKILL|9|Term|Signal ”KILL”|
|…|…|…|…|

## Les utiliser dans Bash : trap

Paf, l’intrigue arrive comme ça d’un coup : nous allons utiliser la commande `trap` pour gérer les signaux. C’est une commande interne à _Bash_ simple à utiliser :

`trap "<commande>" <liste_signaux>`

Attention cependant, tous les signaux ne peuvent pas être piégés : `SIGKILL` par exemple ne peut pas donner lieu à exécution d’une commande.

## Dans la vraie vie?

Prenons un cas concret : un script nécessite l’ouverture d’une connexion SSH persistante à l’aide des options `ControlMaster` et `ContolPath` histoire de pouvoir lancer plusieurs connexions successives plus rapidement (et sans se ré-identifier).

Si le script ne se passe pas comme prévu alors il est plutôt conseillé de faire le ménage et terminer la connexion maître.

`#!/usr/bin/env bash  # Changer ces valeurs par celles adapté à votre configuration server="192.168.0.254" user="user"  # Réutilisons ce que nous avons créé lors du précédent article source ./message.sh  ssh_sock="/tmp/$(mktemp -u XXXXXXXXXX)"  connect() {     msg "Create SSH main connection wih socket ${ssh_sock}"     ssh -N -o ControlMaster=yes -o ControlPath="$ssh_sock" -f ${user}@${server} || {         error "Can't connect to $server";         exit 20;     } }  launch_command() {     if [ -z "$1" ]     then         error "Launch command require 1 parameter"         exit 31     fi     local command     command="$1"     ssh -q -t -S "$ssh_sock" ${user}@${server} "$command" || {         error "Error executing $command";         exit 30;     } }  cleanup() {     msg "Close SSH main connection"     ssh -q -o ControlPath="$ssh_sock" -O exit $server || {         error "Can't close SSH master connection, $ssh_sock remain";     }     # Sans ce exit, INT ne termine pas le script     exit }  connect  trap cleanup EXIT  if [ "$1" = "error" ] then     launch_command fi  for (( i=1; i<=5; i++ )) do     launch_command "echo 'Message N°$i from $server'"     sleep 1 done`

Ce script est disponible en suivant [ce lien](https://xieme-art.org/post/bash-avance-its-a-trap/files/premier_script.sh)

La connexion principale est initiée par la fonction `connect`. Juste en dessous de l’appel de cette dernière nous trouvons notre instruction `trap`. Elle appelle la commande `cleanup` lorsque le signal `EXIT` est envoyé.

`EXIT` n’est pas un signal standard du système mais interne à _Bash_ comme `ERR`, `DEBUG` ou `RETURN`.

Ensuite, si l’argument `error` est passé à notre script, `launch_command` est exécutée mais sans paramètre, ce qui va générer une erreur.

Enfin notre script exécute 5 fois en boucle la fonction `launch_command` qui se charge d’afficher un petit message sur la machine distante en utilisant la connexion SSH créée dans notre fonction `connect`.

Nous avons donc de quoi tester trois scénarios

### Laisser le script finir normalement

Appeler le script sans argument et le laisser se terminer sans intervenir est notre premier scénario. Voici le résultat :

`$ ./script.sh Create SSH main connection wih socket /tmp/TvCuLSzkMN Enter passphrase for key '/home/user/.ssh/key.ed25519':  Message N°1 from 192.168.0.254 Message N°2 from 192.168.0.254 Message N°3 from 192.168.0.254 Message N°4 from 192.168.0.254 Message N°5 from 192.168.0.254 Close SSH main connection`

Nous pouvons voir que notre fonction `cleanup` a bien été appelée à la fin de notre script exécutée lors de la réception du signal `EXIT`

### Générer une erreur

Maintenant passons `error` en paramètre et observons le résultat :

`$ ./script.sh error Enter passphrase for key '/home/user/.ssh/key.ed25519':  Create SSH main connection wih socket /tmp/Al77btSXKf ERROR: Launch command require 1 parameter Close SSH main connection`

`launch_command` appelée sans paramètre conduit à la sortie de notre script avec un code supérieur à 0. Cette sortie est capturée par `trap` qui lance aussi la fonction de nettoyage. Vérifions le code de retour de notre script :

Le code de retour est bien celui défini dans notre fonction `launch_command` lorsqu’on l’exécute sans paramètre.

### Interrompre l’exécution

Enfin observons ce qui se passe lorsque l’on interrompt l’exécution du script avec Ctrl+C:

`$ ./script.sh Create SSH main connection wih socket /tmp/0gFlBD6siZ Enter passphrase for key '/home/user/.ssh/key.ed25519':  Message N°1 from 192.168.0.254 Message N°2 from 192.168.0.254 ^CClose SSH main connection $ echo $? 0`

Ici encore tout se passe comme prévu, sauf que le code de retour est 0, il faudrait pouvoir changer ce comportement afin de signifier que le script ne s’est pas terminé comme prévu et retourner un code supérieur à 0.

## Différencier les pièges en fonction du signal

Tout est prévu dans _Bash_ pour contourner ce problème : il est possible de lancer plusieurs commandes `trap`. Modifions un petit peu notre script.

### Gérer le signal `SIGINT`

Nous allons ajouter une fonction spécifique pour le signal juste après `cleanup` :

`process_int(){     error "Script interrupted by user (SIGINT)"     exit 255 }`

### ajouter un piège

Maintenant nous n’avons plus qu’a ajouter un piège :

`# [...] trap cleanup EXIT trap process_int INT # [...]`

Et voilà, lors de l’exécution notre script et son interruption tout fonctionne comme prévu :

`$ ./script.sh Create SSH main connection wih socket /tmp/0gFlBD6siZ Enter passphrase for key '/home/user/.ssh/key.ed25519':  Message N°1 from 192.168.0.254 Message N°2 from 192.168.0.254 ^CERROR: Script interrupted by user (SIGINT) Close SSH main connection $ echo $? 255`

Le message de notre fonction `process_int` s’affiche et la fonction `cleanup` se lance automatiquement.

Le script modifié est disponible en suivant [ce lien](https://xieme-art.org/post/bash-avance-its-a-trap/files/second_script.sh)

## Vérifier la connexion au master avec un signal

Il est possible d’utiliser tout un tas de signaux et de les intercepter avec `trap`. Intéressons nous maintenant à `SIGUSR1`. C’est — avec `SIGUSR2` — un signal servant pour ce que l’on veut. Utilisons le pour afficher l’état de la connexion SSH master. Rajoutons la fonction suivante après `launch_command` :

`# [...] check_conn() {     msg "Check connection on ${server}"     ssh -S "$ssh_sock" -O check $server     sleep 10 } #[...]`

Puis de modifions le début de notre script comme ceci :

`# [...] msg "Current PID: $$" connect trap cleanup EXIT trap process_int INT trap check_conn USR1 # [...]`

Au début nous affichons le PID de notre script, nous allons en avoir besoin pour lui envoyer le signal `USR1` avec `kill` depuis un autre terminal. Et enfin piégeons notre signal avec `trap` pour exécuter `check_conn`.

Afin d’avoir le temps de lancer la commande `kill`, augmentons de 20 le nombre de tour de boucle effectué comme ci-dessous :

`# [...] for (( i=1; i<=20; i++ )) do     launch_command "echo 'Message N°$i from $server'"     sleep 1 done`

Le script modifié est disponible en suivant [ce lien](https://xieme-art.org/post/bash-avance-its-a-trap/files/troisieme_script.sh)

### Lancer le script

D’abord lançons le script et notons le numéros de PID :

`$ ./script.sh Current PID: 9499 Create SSH main connection wih socket /tmp/A4IdltbuxY Enter passphrase for key '/home/user/.ssh/key.ed25519':  Message N°1 from 192.168.0.254`

Puis dans un second terminal il nous suffit d’envoyer le signal :

Retour sur notre premier teminal, nous pouvons voir que le signal est alors reçu par notre script qui va exécuter la fonction attachée :

`[...] Message N°6 from 192.168.0.254 Message N°7 from 192.168.0.254 Check connection on 192.168.0.254 Master running (pid=9502)`

Et continuer son exécution ensuite.

## En conclusion

Tout au long de cet article, nous avons vu ce qu’était un signal et comment l’intercepter dans un script écrit en Bash. Bien entendu il est possible d’utiliser les signaux dans d’autres langages, la façon de procéder est similaire.

**Le nettoyage des traces laissées par un script** est la principale utilisation documentée ci-et-là dans différents tutoriaux. Mais les possibilités offertes par ce système de communication inter-processus dans vos script **vont bien au delà de ce simple usage**.

## Crédits

L’image en entête est tirée du film _Star Wars, le retour du Jedi_ © Lucasfilms Ltd, Disney