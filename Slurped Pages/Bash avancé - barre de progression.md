---
link: https://xieme-art.org/post/bash-avance-barre-de-progression/
byline: ephase
site: Xieme-Art
date: 2024-06-19T22:04
excerpt: Après les messages et les signaux, voici enfin un nouvel article dans la série Bash avancé. Avec presque deux ans de retard, il serait...
twitter: https://twitter.com/@ephase
tags:
  - bash
  - script
  - printf
  - substitution
slurped: 2024-12-19T08:31
title: "Bash avancé: barre de progression"
---
#bash 
Après [les messages](https://xieme-art.org/post/bash-avance-gerer-les-messages/) et [les signaux](https://xieme-art.org/post/bash-avance-its-a-trap/), voici enfin un nouvel article dans la série **Bash avancé**. Avec presque deux ans de retard, il serait temps me direz-vous! Mais mieux vaut tard que jamais non?

Cette article me servira de prétexte pour utiliser massivement la commande interne `printf` et vous montrer quelques cas d’usages. Nous verrons aussi la _substitution de processus_, la _substitution de paramètre_ et d’autres mécanismes offerts par _Bash_.

## Un peu de théorie

Il faut d’abord définir ce que j’entends par _barre de progression_: un élément affiché dans notre terminal représentant **la progression** d’une tâche exécutée par un script. Voici un exemple:

`Task 2/10 ████▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒ 20%`

Cette barre se compose donc:

- d’une légende / titre;
- de la barre de progression représentant graphiquement l’avancée de notre tâche;
- de la progression en pourcentage.

### Découpage en segments

Afin de préparer notre implémentation, découpons notre barre de chargement en segments. Ce découpage nous permettra de faciliter le passage au code.

![Définition des segments d'une barre de progression](https://xieme-art.org/post/bash-avance-barre-de-progression/images/barre_progression.svg)

Voici la légende:

1. segment d’informations pour la légende;
2. segment d’actions effectuées;
3. segment d’actions en attentes;
4. segment complémentaire qui peut être le pourcentage de progression, ou toute autre information annexe.

## Une commande comme base

Pour concevoir notre barre de progression il nous faut analyser la sortie d’une commande. Pour notre tutoriel, je vous propose d’utiliser la commande `ping`.

`ping -c4 aquilenet.fr PING aquilenet.fr (185.233.100.8) 56(84) bytes of data. 64 bytes from hestia.aquilenet.fr (185.233.100.8): icmp_seq=1 ttl=60 time=29.1 ms 64 bytes from hestia.aquilenet.fr (185.233.100.8): icmp_seq=2 ttl=60 time=56.3 ms 64 bytes from hestia.aquilenet.fr (185.233.100.8): icmp_seq=3 ttl=60 time=52.8 ms 64 bytes from hestia.aquilenet.fr (185.233.100.8): icmp_seq=4 ttl=60 time=29.1 ms  --- aquilenet.fr ping statistics --- 4 packets transmitted, 4 received, 0% packet loss, time 3003ms rtt min/avg/max/mdev = 29.055/41.806/56.258/12.764 ms`

Ci-dessus, la commande a été lancée en indiquant le nombre de requêtes à envoyer via le paramètre `-c4` et chacune de ces requêtes affiche le numéro d’ordre (`icmp_seq=`).

Maintenant que nous savons où aller, il est temps de commencer à écrire notre script:

`#!/usr/bin/env bash PING_ITER=16  # shellcheck disable=2317 command() {     local -r host=${1?"first parameter must be an host"}     local -r iteration=${2?"second parameter must be an iteration number"}     local -r cmd=(ping -c "${iteration}" "${host}")      "${cmd[@]}" }  main() {     command "aquilenet.fr" "$PING_ITER" }  main exit 0`

[Télécharger le script](https://xieme-art.org/post/bash-avance-barre-de-progression/files/script1_ping.sh)

Il se compose d’une fonction `command` qui nécessite deux paramètres : un hôte et un nombre d’itérations. Attardons-nous un instant sur deux points importants.

- Dans la mesure du possible il faut utiliser des variables locales dans les fonctions et en lecture seule pour s’assurer que la variable ne sera pas modifiée. Exemple: `local -r mavariable` déclare la variable locale `mavariable` en lecture seule avec le paramètre `-r`.
- La commande et ses arguments sont enregistrés dans un tableau. Avec cette méthode plus besoin de gérer avec le gobbing et l’interprétation des paramètres lors de son appel.

Lors de son lancement, nous pouvons observer l’affichage des différentes requêtes:

`./bash script1_ping.sh PING aquilenet.fr (185.233.100.8) 56(84) bytes of data. 64 bytes from hestia.aquilenet.fr (185.233.100.8): icmp_seq=1 ttl=60 time=29.1 ms 64 bytes from hestia.aquilenet.fr (185.233.100.8): icmp_seq=2 ttl=60 time=56.3 ms # [...]`

## Capturer les messages de sortie

Il faut maintenant capturer la sortie de la commande ping et en extraire l’information pertinente : **le numéro de séquence**. [Télécharger le script complet](https://xieme-art.org/post/bash-avance-barre-de-progression/files/script2_getoutput.sh)

### Récupérer la sortie standard …

Il est tout à fait possible de rediriger la sortie de la fonction `command` vers une autre fonction qui traitera le flux de données. C’est exactement le même principe utilisé par l’enchaînement de commande avec un _pipe_ comme dans `dmesg | grep 'failure'`. Plusieurs techniques existent pour faire ceci en _Bash_, ici nous allons utiliser la **substitution de processus**.

`parse_output() {     while read -r line; do         printf "out: %s\n" "${line}"     done }  main() {     command "aquilenet.fr" "$PING_ITER" > >(parse_output) }`

Ici la sortie standard de notre fonction `command` est envoyée vers la fonction `parse_output`.

Notez que cette substitution peut aussi se faire dans l’autre sens avec la notation `<(commande)`, la sortie standard de la commande substituée sera envoyée vers l’entrée de la commande à gauche comme par exemple:

``# Affiche les éléments trouvés par la commande `ls` en les préfixants de  # 'éléments: ', la sortie de notre substitution (ls) sera traitée par la boucle while read -r line; do     printf "élément: %s\n" "$line" done < <(ls)``

Sous le capot, le flux entre la sortie standard de `command` et l’entrée de `parse_output` se fait grâce à un descripteur de fichier.

### … et la traiter

Pour traiter le flux, `parse_output` place chaque ligne qui arrive dans la variable `$line` grâce à la bocle `while` couplée à la commande `read`.

Ensuite `$line` est affichée en y ajoutant `out:`. Voici ce que donne l’exécution de ce script:

`bash script2_getoutput.sh out: PING aquilenet.fr (2a0c:e300::8) 56 data bytes out: 64 bytes from hestia.aquilenet.fr (2a0c:e300::8): icmp_seq=1 ttl=55 time=20.0 ms out: 64 bytes from hestia.aquilenet.fr (2a0c:e300::8): icmp_seq=2 ttl=55 time=19.8 ms [...]`

## Récupérer le numéro de séquence

Maintenant que le flux arrive dans `parse_output`, essayons de récupérer le numéro de séquence. Nous pourrons alors déterminer l’état d’avancement de notre commande. [Télécharger le script complet](https://xieme-art.org/post/bash-avance-barre-de-progression/files/script3_rematch.sh)

Pour l’extraire du reste de la ligne, utilisons l’opérateur de comparaison `=~` qui permet de vérifier la présence d’un motif dans une chaîne de caractères via **une expression rationnelle** comme par exemple:

`if [[ "$nom" =~ ^(Y|y)orick$ ]]; then     printf "oui c'est moi!\n" fi`

Il est possible de récupérer le motif capturé entre parenthèses ayant “matché” grâce à la variable `$BASH_REMATCH`, utilisée dans `parse_output`:

`parse_output() {     while read -r line; do         if [[ "$line" =~ icmp_seq=([[:digit:]]{1,}).*time=(.*) ]]; then             printf "séquence: %s sur %s temps:%s\n" \                 "${BASH_REMATCH[1]}" \                 "$PING_ITER" \                 "${BASH_REMATCH[2]}"         fi     done }`

En plus du numéro de séquence, le temps est aussi capturé afin d’illustrer le fonctionnement de la variable `$BASH_REMATCH` qui est en fait **un tableau**:

- l’indice `0` permet de récupérer entièrement la chaîne qui correspond au motif;
- l’indice `1` au premier élément capturé, ici le numéro de séquence qui se trouve après `icmp_seq=`
- et l’indice `2` à tout ce qui se trouve après `time=`.

Voici ce que donne l’exécution de ce script:

`bash script3_rematch.sh séquence: 2 sur 16 temps:20.2 ms séquence: 3 sur 16 temps:20.5 ms séquence: 4 sur 16 temps:19.7 ms [...]`

Nous avons maintenant tout les éléments utiles pour la construction de notre barre de progression.

## Première barre de chargement

Il faut commencer doucement, par une barre de chargement aussi simple que possible, sans fioriture. Nous allons ajouté la fonction `draw_progressbar` qui se charge du dessin à l’écran. [Télécharger le script complet](https://xieme-art.org/post/bash-avance-barre-de-progression/files/script4_progress.sh)

`draw_progressbar() {     local -r progress=${1?"progress is mandatory"}     local -r total=${2?"total is mandatory"}     local progress_segment     local todo_segment      printf -v progress_segment "%${progress}s" ""     printf -v todo_segment "%$((total - progress))s" ""     printf >&2 "%s%s\r" \         "${progress_segment// /${PROGRESS_BAR_CHAR[0]}}" \         "${todo_segment// /${PROGRESS_BAR_CHAR[1]}}"  }`

`draw_progressbar` attend deux paramètres: le nombre total d’éléments et celui en cours. Cette fonction est appelée depuis `parse_output` et elle utilise `printf` pour dessiner les deux segments nécessaires (mais pas que…).

### `printf` vers une variable

Ne nous attardons pas pas sur les deux premières variables locales de notre fonction `draw_progressbar`. Les deux suivantes — `progress_segment` et `todo_segment` sont par contre plus intéressantes.

`# [...]     local progress_segment     local todo_segment      printf -v progress_segment "%${progress}s" ""     printf -v todo_segment "%$((total - progress))s" "" # [...]`

`printf -v progress_segment` permet **d’affecter le résultat de la commande `printf`** à la variable locale `progress_segment` au lieu de l’afficher sur la sortie standard.

#### `printf`: format, arguments, spécification

Le premier argument d’une commande `printf` est le _format_, les suivants sont les _paramètres_. Les arguments sont affichés par le format par l’intermédiaire de _spécifications_.

`number=2 drink="water" printf "I have %d bottle of %s.\n" "$number" "$drink"`

Dans l’exemple suivant:

- `"I have %d bottle of %s.\n"` est le format,
- `$number` et `$drink` sont les arguments
- `%d` et `%s` sont des spécifications:
    - `%d` indique que le premier argument doit être affiché comme un nombre entier;
    - `%s` indique que le second argument doit être affiché comme une chaîne de caractère.

Il est possible de définit la taille **minimale d’une spécification**, les caractères manquants seront remplacés par des espaces, comme par exemple:

`printf "bonjour %10s, bienvenue\n" "monsieur" "madame" "Linus Torvalds" bonjour   monsieur, bienvenue bonjour     madame, bienvenue bonjour Linus Torvalds, bienvenue`

Vous remarquerez que la taille de 10 caractères a bien été réservée pour les deux premiers éléments. Le dernier par contre contient plus de 10 caractères: il dépasse de cet espace.

Vous remarquez aussi une caractéristique particulière de `printf`. Une seule spécification est utilisée pour trois arguments, le format est donc répété trois fois.

#### Et dans notre script…

`# [...]     printf -v progress_segment "%${progress}s" ""     printf -v todo_segment "%$((total - progress))s" ""`

Dans le script la taille des deux segments est définie:

- par **la variable** `progress` pour le _segment des actions effectuées_
- par le résultat d’une soustraction pour le _segment des actions en attente_.

_Bash_ réalise les expansions avant de passer dans la commande `printf`, pratique n’est ce pas!

Pour ces deux `printf` le formats a une spécification avec une taille minimale égale à la taille du segment que nous voulons obtenir (car les arguments sont vides). Nous obtenons **deux variables qui contiennent un nombre d’espace égal à la taille des segments représentés**, il ne reste plus qu’à assembler.

### On passe au dressage

La commande `printf` suivante sert à afficher effectivement la barre de progression. D’abord elle est redirigée sur `stderr`, cette sortie est utilisée pour les erreurs, mais aussi pour afficher les informations de diagnostic.

`# [...]     printf >&2 "%s%s\r" \         "${progress_segment// /${PROGRESS_BAR_CHAR[0]}}" \         "${todo_segment// /${PROGRESS_BAR_CHAR[1]}}"`

Son format contient deux spécifications de type chaîne de caractère et **un retour chariot**. Ce dernier permet au curseur de revenir au début de la ligne une fois dessinée. Lors du prochain passage dans la fonction `draw_progressbar`: la barre sera alors réécrite, **pas besoin de gérer l’effacement de la ligne**.

Pour les deux arguments, nous utilisons _l’expansion de paramètre_ afin de remplacer les espaces par les caractères choisis pour symboliser les deux segments de notre barre.

Comment fonctionne le remplacement de motif avec _l’expansion de paramètre_? Voici deux exemples:

``variable="voici une plante, jolie plante non?"  # remplacer plante par fleur dans `variable` # remplace la première occurrence trouvée  echo "${variable/plante/fleur}" # affiche "voici une fleur, joli plante non?"  #remplace toutes les occurrences echo "${variable//plante/fleur}" # affiche "voici une fleur, joli fleur non?"``

Les caractères choisis pour remplacer les espaces sont affecté au tableau `PROGRESS_BAR_CHAR` initialisé en tout début de script.

C’est la fonction `parse_output` qui fait appel à `draw_progressbar`

`parse_output() {     while read -r line; do         if [[ "$line" =~ icmp_seq=([[:digit:]]{1,}).*time=(.*) ]]; then             draw_progressbar "${BASH_REMATCH[1]}" "$PING_ITER"         fi     done }`

### Exécution

Maintenant que vous avez compris comment le script est construit, il est temps de passer à l’exécution:

`bash script4_progress.sh █████▒▒▒▒▒▒▒▒▒▒▒`

Notre barre de progression fonctionne, mais elle ne prend qu’une petite part de l’écran (les 16 caractères qui correspondent aux nombres de requêtes totales). Mais surtout, **aucune information n’est affichée**.

## Ajouter des informations

Elle est bien belle notre barre de progression, mais là on ne sais pas du tout ce que fait le script. L’angle d’attaque: modifier les fonctions `parse_output` et `draw_progressbar` pour ajouter des informations sur la sortie standard. [Télécharger le script complet](https://xieme-art.org/post/bash-avance-barre-de-progression/files/script5_informations.sh)

### La fonction `parse_output`

Elle contient 2 conditions de plus qui vérifient la ligne en cours envoyée par `command`.

`parse_output() {     while read -r line; do         if [[ "$line" =~ icmp_seq=([[:digit:]]{1,}).*time=(.*) ]]; then             draw_progressbar \                 "${BASH_REMATCH[1]}" \                 "$PING_ITER" \                 "Ping in progress (time: ${BASH_REMATCH[2]}) "         elif [[ "$line" =~ ^PING(.*\(.*\)).* ]]; then             printf "Launch ping command to %s with %d iterations\n" \                 "${BASH_REMATCH[1]}" \                 "$PING_ITER"         elif [[ "$line" =~ .*packets\ transmitted.* ]]; then             printf "%s\n" "$line"         fi      done }`

#### Afficher des information en introduction …

La première condition ajoutée récupère, dans la sortie de la commande `ping`, la ligne qui commence par `PING` comme dans l’exemple ci-dessous:

`PING aquilenet.fr (2a0c:e300::8) 56 data bytes`

De cette ligne est récupérée **le nom d’hôte** et **l’adresse IP associée** afin de l’afficher avant notre barre de progression.

`elif [[ "$line" =~ ^PING(.*\(.*\)).* ]]; then`

L’information utile est capturée et réutilisée via la variable `BASH_REMATCH`.

#### … une conclusion …

La seconde condition ajoutée permet de récupérer la ligne qui contient le récapitulatif de ce qui a été fait par commande `ping` et de la renvoyer tel quelle sur la sortie standard.

#### … et ajouter le segment d’information

Enfin un paramètre de plus est passé à la fonction `draw_progressbar`: une chaîne de caractère pour le _segment d’information_ contenant la dernière mesure de temps retournée par `ping` (histoire d’utiliser `${BASH_REMATCH[2]}`)

`if [[ "$line" =~ icmp_seq=([[:digit:]]{1,}).*time=(.*) ]]; then             draw_progressbar \                 "${BASH_REMATCH[1]}" \                 "$PING_ITER" \                 "Ping in progress (time: ${BASH_REMATCH[2]}) "`

### la fonction `draw_progress`

Le troisième paramètre passé à cette fonction est affecté à la variable `info_segment`. Son contenu est ajouté à l’affichage avant la barre de chargement.

`draw_progressbar() {     local -r progress=${1?"progress is mandatory"}     local -r total=${2?"Total elements is mandatory"}     local -r info_segment=${3:""}     local progress_segment     local todo_segment      printf -v progress_segment "%${progress}s" ""     printf -v todo_segment "%$((total - progress))s" ""     printf >&2 "%s%s%s\r" \         "${info_segment}" \         "${progress_segment// /${PROGRESS_BAR_CHAR[0]}}" \         "${todo_segment// /${PROGRESS_BAR_CHAR[1]}}"  }`

### lancement de notre script

Notre barre de progression est toujours là mais avec un peu de contexte.

`bash script5_informations.sh               Launch ping command to  aquilenet.fr (2a0c:e300::8) with 16 iterations Ping in progress (time: 20.7 ms) ███████▒▒▒▒▒▒▒▒▒`

Et une fois terminé la barre de progression disparait et laisse place au récapitulatif:

`bash script5_informations.sh               Launch ping command to  aquilenet.fr (2a0c:e300::8) with 16 iterations 16 packets transmitted, 16 received, 0% packet loss, time 15019ms` 

## Responsive design inside

Maintenant que l’affichage des informations est en place, attardons nous sur le côté _responsive design_ en adaptant la taille de notre barre à la largeur de l’écran.[Télécharger le script complet](https://xieme-art.org/post/bash-avance-barre-de-progression/files/script2_getoutput.sh)

La première question est bien évidement comment obtenir la largeur de la fenêtre de terminal? Lorque _Bash_ est lancé en **mode interactif** cette largeur — exprimée en nombre de colonnes [1](https://xieme-art.org/post/bash-avance-barre-de-progression/#fn:colonne) — est disponible via la variable `$COLUMNS`. Mais dans le cadre d’un script, cette variable n’est pas disponible si vous n’utilisez pas _Bash_ comme interpréteur de commande.

Il est possible d’utiliser la command interne `shopt` pour activer l’option `checkwinsize` mais elle pose deux problèmes:

1. Je n’ai pas réussi à activer l’option avec la commande `shopt -s checkwinsize` dans mes tests (mais un simple `shopt` rend les variables `COLUMNS` et `LINES` disponible);
2. la récupération des ces deux variables ne se fait **qu’après exécution d’une commande externe**, or il y en a peu dans notre script (`ping`), ce qui posera problème un peu plus loin dans cet article.

la commande `tput`, qui permet de récupérer des informations sur le terminal, fait très bien le job comme vous pouvez le voir dans ce petit bout de code ajouté dans la fonction `main()`:

`main() {     COLUMNS=$(tput cols)     command "aquilenet.fr" "$PING_ITER" > >(parse_output) }`

### le dessin de la barre de progression

Maintenant que nous avons le nombre de colonnes, passons au dessin de notre barre de progression. Voici la nouvelle version de `draw_progressbar`:

`draw_progressbar() {     local -r progress=${1?"progress is mandatory"}     local -r total=${2?"total elements is mandatory"}     local -r info_segment=${3:""}     local progress_segment     local todo_segment      local -r bar_size=$((COLUMNS - ${#info_segment}))     local -r progress_ratio=$((progress * 100 / total))     local -r progress_segment_size=$((bar_size * progress_ratio / 100))     local -r todo_segment_size=$((bar_size - progress_segment_size))      printf -v progress_segment "%${progress_segment_size}s" ""     printf -v todo_segment "%${todo_segment_size}s" ""      printf >&2 "%s%s%s\r" \         "${info_segment}" \         "${progress_segment// /${PROGRESS_BAR_CHAR[0]}}" \         "${todo_segment// /${PROGRESS_BAR_CHAR[1]}}"  }`

Afin de déterminer la place disponible pour la barre de chargement (`$bar_size`) — qui sera la partie responsive de notre ligne — il faut soustaire la taille de notre segment d’information (`$info_segment`) à la largeur (`COLUMNS`).

La progression doit maintenant s’exprimer par le ratio (`$progress_ratio`) entre les éléments traités (`progress`) et le nombre total d’éléments (`total`).

L’expansion arithmétique de **_Bash_ ne prend pas en charge les nombres flottants**, alors ce ratio s’exprime en pourcentage qui sera arrondi à l’entier le plus proche.

Il suffit ensuite de calculer le nombre de colonnes occupées par le _segment d’actions effectuées_ (`$progress_segment_size`) avec les deux valeurs obtenues précédemment (`$bar_size` et `$progress_ratio`).

Pour obtenir le nombre de colonnes occupées par le _segment d’actions en attentes_. il suffit de soustraire la taille du segment d’actions effectuées (`$progress_segment_size`) à la taille de la barre (`$bar_size`).

La suite est simple, dans les deux instructions `printf -v` qui permettent d’affecter le bon nombre d’espaces, il suffit d’utiliser `progress_segment_size` et `todo_segment_size` et le tour est joué.

### Exécution du script

Il est temps de passer à l’exécution de cette version de notre script:

`bash script6_responsive.sh Launch ping command to  aquilenet.fr (2a0c:e300::8) with 16 iterations Ping in progress (time: 23.1 ms) █████████████████████████████▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒`

La ligne prend maintenant **toute la place disponible**, la barre de progression ne fait plus juste 16 colonnes. Mais un problème subsiste sur cette version.

Changer la taille de la fenêtre de terminal ne change pas la taille de la barre. Si agrandir la fenêtre ne pose pas trop de problèmes — la barre ne prend pas toute la place disponible — la réduction de la taille mène à ceci:

![Capture d'écran montrant l'affichage de la barre de progression cassé lors de la réduction de la taille de la fenêtre](https://xieme-art.org/post/bash-avance-barre-de-progression/images/responsive_corruped.png)

L’explication est simple: la valeur de `$COLUMNS` n’est pas rafraichie! Il serait tout à fait possible de récupérer le nombre de colonnes au début de la fonction `draw_progress`, mais ce ne serait pas très optimisé pour une fonctionnalité utilisée rarement. **Une meilleure solution existe…** (mais elle n’est pas si simple).

## SIGWINCH à la rescousse

Figurez-vous qu’un signal existe pour signifier un changement de taille d’une fenêtre de terminal. Mais l’implémentation dans notre script est plus épineuse qu’il n’y parait et nécessite quelques changements sur des fonctions d’affichage **pour que notre code soit compatible avec le plus d’émulateurs de terminal** possibles. [Télécharger le script complet](https://xieme-art.org/post/bash-avance-barre-de-progression/files/script7_sigwinch.sh)

D’abord ajoutons la fonction `change_column_size` qui se charge d’appliquer le changement du nombre de colonnes de la fenêtre de terminal.

`change_column_size() {     printf >&2 "%${COLUMNS}s" ""     printf >&2 "\033[0K\r"     COLUMNS=$(tput cols) }`

Cette fonction se charge:

1. de remplir la ligne courante sur `stderr` d’espace pour l’effacer;
2. ensuite d’effacer les caractères de la ligne entière grâce à la séquence `\033[0K\r` passée à `printf`;
3. enfin de récupérer le nouveau nombre de colonnes.

Elle sera exécutée dès le changement de géométrie de la fenêtre de terminal (c’est ici que `SIGWINCH` intervient). Cependant une petite subtilité se cache dans la gestion des signaux en _Bash_: ils ne sont pas transmis au _sub-shell_ et notre fonction `parse_output` est justement exécuté comme tel via la substitution de processus.

Pour notre la capture de notre signal `SIGWICH`, il est alors préférable d’ajouter la commande `trap` au début de notre fonction `parse_output`. Si vous voulez plus d’information sir les signaux en Bash, je vous coneille la lecture de mon [précédent article](https://xieme-art.org/post/bash-avance-its-a-trap/).

`parse_output() {     trap change_column_size WINCH     while read -r line; do         if [[ "$line" =~ icmp_seq=([[:digit:]]{1,}).*time=(.*) ]]; then`

### Exécution du script

Notre barre de progression se redimensionne correctement, mais deux problèmes persistent.

1. Il faut attendre le rafraichissement de la barre de progression pour qu’elle prenne la bonne dimension une fois la fenêtre redimensionnée.
2. La diminution de la taille de la fenêtre de terminal crée des lignes “vides”

![Des lignes vides apparaissent lors de la diminution de la taille de la fenêtre](https://xieme-art.org/post/bash-avance-barre-de-progression/images/responsive_blanklines.png)

Un programme avec une véritable TUI [2](https://xieme-art.org/post/bash-avance-barre-de-progression/#fn:TUI) utilise une boucle et des buffers pour gérer spécifiquement le rafraichissement de l’affichage. Dans le cadre d’un script et de cet article, nous garderons ces deux bugs mineurs et nous en resterons là.

## Bonus stage: un script plus aboutit

Maintenant que tous nous avons une barre de progression fonctionnelle, il est temps de l’améliorer pour la rendre réutilisable et paramétrable — et de mettre en place le segment manquant. [Télécharger le script complet](https://xieme-art.org/post/bash-avance-barre-de-progression/files/script8_bonus.sh)

D’abord il est possible d’ajouter des variables globales pour paramétrer les différents segments de la fonction `draw_progressbar`,

`# Progress bar  dedicated variables # We can control how progress bar work PROGRESS_BAR_CHAR=(█ ▒)  # control which information is displayed" PROGRESS_BAR_DISPLAY_INFO=1 PROGRESS_BAR_DISPLAY_COMPL=1  # Display template # We can control how informations is displayed PROGRESS_BAR_INFO_TEMPLATE=' %s [%2d/%2d] ' PROGRESS_BAR_COMPL_TEMPLATE=' %2d%% ' PROGRESS_BAR_TEMPLATE='\033[1m%s%s\033[0m%s%s\r'`

Les variables finissant par `_TEMPLATE` servent de _templates_ aux segments d’information et complémentaires. Elles contiennent des définitions de **formats** pour `printf` qui sont utilisée dans la fonction d’affichage comme par exemple la mise en gras du segment d’information.

`draw_progressbar() { # ...     if [[ ${PROGRESS_BAR_DISPLAY_INFO:-1} -eq 1 ]]; then         printf -v info_segment "${PROGRESS_BAR_INFO_TEMPLATE}" \             "$info" "$progress" "$total"     fi      local -r progress_ratio=$((progress * 100 / total))     if [[ ${PROGRESS_BAR_DISPLAY_COMPL:-0} -eq 1 ]]; then         printf -v compl_segment "${PROGRESS_BAR_COMPL_TEMPLATE}" \             "$progress_ratio"     fi # ...     printf >&2 "$PROGRESS_BAR_TEMPLATE" \         "$info_segment" \         "${progress_segment// /${PROGRESS_BAR_CHAR[0]}}" \         "${todo_segment// /${PROGRESS_BAR_CHAR[1]}}" \         "$compl_segment"`

Dans `draw_progressbar`, les affichages du segment _d’information_ et celui _complémentaire_ sont conditionnels. Ensuite les formats des différentes commandes `printf` sont aussi donnés par les variables globales définies plus haut.

Nous avons maintenant une fonction `draw_progressbar` et les variables associées nous permettant de personnaliser son apparence en fonction du contexte.

## en conclusion

Au final, voici ce que que donne notre barre de progression:

![GIF animé de la barre de progression en fonctionnement](https://xieme-art.org/post/bash-avance-barre-de-progression/images/video.gif)

Nous avons abordé beaucoup de choses dans cet article un peu plus long que d’habitude. Principalement autour de la commande `printf` et ses nombreuses possibilités (mais nous n’avons pas tout vu…).

J’espère vous avoir montré que par rapport à la commande `echo` habituellement utilisées dans les script, `printf` est réellement bien plus **puissante** et **utile**.

Merci à [Heuzef](https://heuzef.com/), Yishan et [RavenRamirez](https://aloisbouny.fr/) pour les nombreuse relectures, corrections et conseils.