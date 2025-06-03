---
link: https://xieme-art.org/post/bash-avance-gerer-les-messages/
byline: ephase
site: Xieme-Art
date: 2022-01-30T00:30
excerpt: Dans ce premier article de l’année 2022, nous allons voir comment gérer les messages de sorties de nos scripts bash. L’idée ici est de...
twitter: https://twitter.com/@ephase
tags:
  - bash
  - script
  - tutorial
  - log
slurped: 2024-12-19T08:33
title: "Bash avancé: Gérer les messages de sortie de vos scripts"
---
#bash
Dans ce premier article de l’année 2022, nous allons voir comment gérer les messages de sorties de nos scripts _bash_. L’idée ici est de proposer trois types de messages dans un fichier que nous pourrons ensuite inclure dans nos scripts à l’aide de la commande `source`.

Ces messages seront de 3 types différents:

- **messages standards** envoyés sur la sortie standard
- **messages de débogage** affichés si une variable `DEBUG` est positionnée et envoyés vers la sortie d’erreur
- **message d’erreur** envoyés sur la sortie d’erreur.

Nous utiliserons quelques spécificité de bash nous permettant d’agrémenter nos sorties.

## Une première version

Cette première ébauche de cette librairie contient 3 fonctions répondant à la demande formulée en introduction. Appelons ce fichier `message.sh`.
```
  #!/bin/env bash  
 msg() {
    local message="$*"
    # Si la fonction est appelée sans paramètres, on la quitte
    [ -z "$message" ] && return
    printf "%b\n" "$message"
  }
  debug() {
     local message="$*"
     # si la variable $DEBUG n'est pas définie ou si sa valeur
     # est différente de 1, on quite notre fonction.
     [[ -z "$DEBUG" || $DEBUG -ne 1 ]] && return
     [ -z "$message" ] && return
     >&2 msg "DEBUG: $message"
   }
   
   error() {
     local message="$*"
     [ -z "$message" ] && return
     >&2 msg "ERROR: $message"
  }
  ```
  

Vous remarquez que les fonctions `error` et `debug` utilisent la fonction `msg` mais son appel est précédé de `>&2` afin que la sortie se fasse sur **la sortie d’erreur**.

Le `printf` de notre fonction `msg` utilise `%b` pour afficher le contenu de la variable `message`. Ainsi les séquences échappées par un antislash seront interprétées. Nous aborderons le sujets dans la partie suivante.

Pour les tester, créons un script `test.sh` dans le même répertoire que notre librairie avec le code suivant:

```
  #!/usr/bin/env bash
  # inclusion de notre librairie de message, il ne faut pas oublier
  # d'afficher une erreur et teminer notre script s'il y a un problème
  # lors de son chargement.  
  source message.sh || { >&2 printf "Can't load message.sh"; exit 1; }
  
  debug "We will display a message"
  msg "Test Message" 
  debug "We will display an error"
  error "This is an error"  exit 0
  
```
Pour tester il suffit de lancer la commande :

```
  ./test.sh Test Message ERROR: This is an error

```

Comme il n’y a pas de variable `$DEBUG` de définie, les message de débogage ne sont pas affichés, pour tester ces messages il suffit de faire:

```
  $ DEBUG=1 ./test.sh

```  

DEBUG: We will display a message Test Message DEBUG: We will display an error ERROR: This is an error`

## Ajouter un peu de couleur

Personnellement, **j’aime avoir un peu de couleur dans mon terminal**, les choses apparaissent souvent plus claires. Pour nos messages, nous pouvons faire de même.

La commande `printf` permet d’insérer des code couleur (entre autres), utilisons le rouge pour les messages d’erreur (logique non?) et le bleu pour les messages de débogages

```
  debug() {
    local message="$*"
    [[ -z $DEBUG || $DEBUG -ne 1 ]] && return
    # \e[34m permet de choisit la couleur bleu pour afficher notre message
    # \e[0m  permet de revenir à la normale
    [ -n "$message" ] && >&2 msg "\e[34mDEBUG: $message\e[0m"
  }  
  
  error() {
    local message="$*"
    # \e[31m permet de choisir le rouge
    [ -n message ] && >&2 msg "\e[31mERROR: $message\e[0m"
  }
```

Ici la commande `\e[34m` permet de choisir la couleur bleue et `\e[0m` de revenir à la normale. C’est ici que le choix de `%b` pour le formatage de la variable `$message` est important dans notre fonction `msg`: **`printf` interprètera nos commandes échappées avec l’antislash**.

La couleur c’est bien, mais si on décide de rediriger une sortie (ou les deux) de notre script dans un fichier voici son contenu ouvert dans `vim`:

`$ DEBUG=1 ./test.sh >error.txt 2>&1 $ vim error.txt ^[[34mDEBUG: We will display a message^[[0m Test Message ^[[34mDEBUG: We will display an error^[[0m ^[[31mERROR: This is an error^[[0m`

Nous allons justement régler ce problème dans le paragraphe suivant.

## Sortie vers un fichier

Il est souvent nécessaire de **rediriger les sorties** d’un de nos script vers un fichier. Surtout lorsqu’il est exécuté en dehors d’une session interactive — dans une tâche _cron_ par exemple.

Dans ce cas il peut être intéressant d’ajouter un horodatage en début de ligne comme dans la plupart des applications qui effectuent de la journalisation. Bash dispose d’un opérateur de test permettant de savoir si la sortie demandée est un terminal interactif ou non: `-t`.

Il est aussi intéressant de ne pas inclure les informations de couleur lors de la redirection de la sortie vers un fichier, le problème évoqué dans la partie précédente est réglé.

Le code prend un peu de poids :

- On ajoute une condition dans chacune de nos trois fonctions d’origine afin de tester le type de sortie.
- On y ajoute une fonction `log` qui se charge de traiter notre sortie lorsque elle est redirigée vers un fichier en ajoutant une information de date / heure au début de la ligne.
- Le format de cette date est paramétrable à l’aide de la variable `$DATE_FMT`, dans l’exemple un [_timestamp_](https://xieme-art.org/post/bash-avance-gerer-les-messages/l_w_timestamp).

Explanations again:

- `-t` is a bash builtin test that checks if a file descriptor is opened and refers to a terminal (`man bash` for more info).
- for normal logs that are supposed to go to _stdout_ (file descriptor number 1), we just check if stdout is connected to a terminal (`[[ -t 1 ]]`). If yes, we can use `echo`. Otherwise, we log to the system logs with `loggers`.
- for error logs, we check if _stderr_ is connected to a terminal (`[[ -t 2 ]]`).

Voici le nouveau code :

```

  #!/usr/bin/env bash
  DATE_FMT="+%s"
  
  msg() {
    local message="$*"
    [ -z "$message" ] && return
    if [ -t 1 ]
    then
      printf "%b\n" "$message"
    else
      log "$message"
    fi
  }
  
  log() {
    local message="$*"
    [ -z "$message" ] && return
    # On veux conserver les sauts de ligne et les tabulation du
    # message, utilisons alors %b ...    
    printf "%s %b\n" "$(date $DATE_FMT)" "$message"
  }
  
  debug() {
    local message="$*"
    [[ -z $DEBUG || $DEBUG -ne 1 ]] && return
    [ -z "$message" ] && return
     message="DEBUG: $message"
     if [ -t 2 ]
     then
       >&2 msg "\e[34m$message\e[0m"
     else
       >&2 log "$message"   
     fi
   }
   
   error() {
      local message="$*"   
      [ -z "$message" ] && return
       message="ERROR: $message"
       if [ -t 2 ]
       then
          >&2 msg "\e[31m$message\e[0m"
        else >&2 log "$message"
    fi
  }
```

## Améliorer les informations de débogage

Lors de la sortie d’information de débogage via notre fonction `debug`, il peut être intéressant d’afficher des **informations supplémentaires** comme par exemple la fonction en cours et le fichier source. Ces informations peuvent être très précieuse surtout dans le cas d’un projet conséquent répartis sur plusieurs fichiers sources.

Ces informations sont disponible via des variables spéciales de bash :

- `BASH_SOURCE`: un tableau reprenant **la pile des fichiers** scripts utilisés. `$BASH_SOURCE[0]` représente le fichier source de la fonction en cours.
- `FUNCNAME`: est un tableau reprenant la **liste des fonctions appelées**, en quelque sorte notre pile d’appel. Cette variable est liée à la précédente, `BASH_SOURCE[n]` représente la source de la fonction `FUNCNAME[n]` et `FUNCNAME[0]` la fonction courante.

Dans notre fonction `debug`, `FUNCNAME[0]` correspond donc à `debug`, pour retrouver la fonction appelante, il faut chercher du côté de `FUNCNAME[1]` (et donc de `BASH_SOURCE[1]`).

Voici donc le nouveau code de notre fonction:

```
  debug() {
     local message="$*"
     [[ -z $DEBUG || $DEBUG -ne 1 ]] && return
     [ -z "$message" ] && return
     # On affiche les informations supplémentaires pour le débogage         message="DEBUG [${BASH_SOURCE[1]}:${FUNCNAME[1]}]: $message"   
     if [ -t 2 ]
     then
       >&2 msg "\e[34m$message\e[0m"
     else
       >&2 log "$message"
     fi
   }

```

Afin de tester le fonctionnement de notre modification, nous allons inclure un autre fichier bash dans notre script de test. Voici notre fichier `include.sh`:

```
  #!/usr/bin/env bash
  myfunct() {
     local a=10
     debug "my a variable is $a"
     msg "value of 'a' 
     squared $(( a * a ))"
  }
```
Et modifions notre fichier `test.sh` afin de _“sourcer”_ notre fichier comme ci-dessous :

 ```
  source message.sh || { >&2 printf "Can't load message.sh"; exit 1; }
  source include.sh || { >&2 printf "Can't load include.sh"; exit 1; }
  # Appel de notre fonction venue de include.sh 
  
  myfunct
  debug "We will display a message" msg "Test Message"`
```
Et voici sa sortie:

```
  DEBUG=1 ./test.sh 
  DEBUG [include.sh:myfunct]: my a variable is 10
  value of 'a' squared: 100
  DEBUG [./test.sh:main]: We will display a message
  Test Message
```

Le fichier source et la fonction appelée sont bien affichés. _Bash_ dispose d’autre variables utiles que vous trouverez dans l’aide : `man bash`.

## Améliorer les sorties d’erreur

Avec ce que nous venons de voir, nous pouvons maintenant améliorer la sortie d’erreur. Pourquoi nous n’afficherions pas, lorsque le mode de débogage est activé, une sorte de _stack trace_?

Pour ce faire, nous pouvons utiliser la variable `${#FUNCNAME[@]}` qui va nous donner le nombre de fonctions appelées. Voici le code de notre nouvelle fonction `error`:

```

  error() {
     local message="$*"
     [ -z "$message" ] && return
     message="ERROR: $message"
     # Nous affichons notre "stack trace si le mode débogage est activé
     if [[ -n $DEBUG && $DEBUG -eq 1 ]]
     then
        message="$message\n\tstack trace:\n"
        # Il nous suffit pour ça de parcourir notre tableau FUNCNAME et 
        # d'afficher le BASH_SOURCE et BASH_LINENO correspondant
        for (( i=1; i<${#FUNCNAME[@]}; i++ ))
        do
            message="${message}\t  source:${BASH_SOURCE[i]}"         message="${message} function:${FUNCNAME[$i]}"
            # Attention, il faut prendre ici la valeur de n-1 pour BASH_LINENO               message="${message} line:${BASH_LINENO[$i-1]}\n"
        done
      fi
      if [ -t 2 ]
      then
         >&2 msg "\e[31m$message\e[0m"
       else
         >&2 log "$message"   
       fi
    }
```

Pour tester notre nouvelle fonction, rajoutons le code suivant dans le fichier `include.sh`:

```
  check_file() {
     if [ ! -f "monfichier.txt" ]
     then
         display_error "File monfichier.txt not found"   
      fi
   }
   display_error() {
      error "$*" 
   }
```

Et enfin modifions notre fichier `test.sh` comme ci-dessous:

```
  #!/usr/bin/env bash
  source message.sh || { >&2 printf "Can't load message.sh"; exit 1; }
  source include.sh || { >&2 printf "Can't load include.sh"; exit 1; } 
  
  myfunct 
  check_file debug "We will display a message"
  msg "Test Message" error 
  "This is a simple error message" 
  exit 0`
```
Lors de l’exécution de notre script de test avec le mode débogage, nous pouvons voir que tous les éléments demandés sont présent:

```
  DEBUG=1 ./test.sh 
  DEBUG [include.sh:myfunct]: my a variable is 10
  value of 100 
  ERROR: File monfichier.txt not found     
    stack trace:    
      source:include.sh function:display_error line:18
      source:include.sh function:check_file line:13
      source:./test.sh function:main line:6  
  DEBUG [./test.sh:main]: We will display a message
  Test Message
  ERROR: This is a simple error message
      stack trace:
         source:./test.sh function:main line:9`

## En conclusion

Nous avons vu tout au long de cet article comment utiliser des fonctions pour afficher vos messages, que se soit pour déboguer, afficher des erreurs ou de simples messages.

Pensez que vous pouvez placer votre “bibliothèque” dans un endroit présent dans la variable d’environnement `$PATH` et ainsi l’inclure dans n’importe quel script.

Les fichiers d’exemple sont disponibles [ici](https://xieme-art.org/post/bash-avance-gerer-les-messages/files/scripts.tar.gz).