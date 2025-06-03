---
link: https://blog.wescale.fr/bash-pro-tips
byline: Matthieu Parisot
site: Wescale
date: 2016-04-07T07:45
excerpt: "Dans ce guide, nous vous proposons de découvrir les fonctionnalités de l’un des shell les plus répandus toutes distributions confondues : Bash."
slurped: 2025-05-11T16:15
title: "Script bash : tous nos conseils"
tags:
  - bash
  - log
---

Les scripts shell traînent une réputation peu glorieuse lorsque l’on parle maintenabilité, pérennité et troubleshooting. Pourtant le shell reste l’un des points de greffe les plus pratiques lorsque l’on aborde une nouvelle machine. Dans un environnement inconnu, c’est le shell qui, le premier, nous accueille et nous fournit de quoi avoir prise sur le système. Pour une instance démarrée dans le Cloud, le shell est parmi les premières options offertes pour bootstrapper la machine. Bref, où il y a un système il y a un shell.

Je vous propose de découvrir ou de redécouvrir des fonctionnalités de l’un des shell les plus répandus toutes distributions confondues : Bash. J’espère vous faciliter la vie et vous permettre de créer des scripts plus robustes et plus maintenables.

## Gestion du logging

Dans un monde idéal, nous serions capables d’écrire au premier jet le programme qui fonctionne systématiquement, gère les erreurs à la perfection et ne nous déçoit jamais. L’expérience nous montre que ce monde n’existe pas, et qu’un regard critique est de rigueur. C’est pourquoi enregistrer systématiquement l’exécution du bootstrap est essentiel pour comprendre ce qui se passe en cas d’erreur.

On rencontre régulièrement des scripts qui procèdent de la façon suivante :

```
#!/bin/bash 
logfile=/var/log/bootstrap.log 
echo "Installation des packages" > $logfile
apt-get install package1 package2 >> $logfile 2>&1
echo "Ajout de l’utilisateur de service" >> $logfile
adduser service >> $logfile 2>&1
ls /fichier_qui_n_existe_pas >> $logfile 2>&1
```

Petits rappels :

Redirige la sortie standard dans le fichier indiqué

Ajoute la sortie standard au fichier indiqué

- 2>&1 Fusionne la sortie standard et la sortie d’erreur

Le script semble efficace car il permet d’enregistrer les étapes et leurs éventuelles erreurs.

Cette approche comporte néanmoins des inconvénients majeurs :

- L’ajout des lignes de logs est fastidieux et pollue le script.
- Lancé depuis la ligne de commande le script est silencieux.
- Il est tout à fait possible d’oublier d’enregistrer une ligne.

Regardons maintenant ce qu’il est possible de faire :

```
#!/bin/bash 
logfile=/tmp/log 
setup() {
 term=$(tty)
 if (( $? != 0 )) 
 then 
  exec 1<&- 
  exec 2<&- 
  exec 1<> $logfile 
  exec 2>&1
  echo "Script is not executed from a terminal" 
 else 
  exec 1<> $logfile
  exec 2>&1 
  tail --pid $$ -f $logfile >> $term & echo "Script is executed from a terminal"
 fi
}

setup echo "Installation des packages" 
apt-get install package1 package2 
echo "Ajout de l’utilisateur de service" 
adduser service ls /fichier_qui_n_existe_pas
```

Tout d’abord, nous créons une fonctions « setup » qui sera appelée au début du script ; si la partie utile du script est identique à l’exemple précédent, nous avons résolu les problèmes identifiés : plus d’ajout systématique, visibilité des logs en mode interactif et aucun oubli possible.  
Voyons comment cela fonctionne :

- Nous commençons par détecter si le script est attaché à un terminal, autrement dit si le script a été appelé depuis la ligne de commande ou par le processus de bootstrap
- S’il est attaché à un terminal : - Nous redirigeons la sortie standard dans le fichier de log
- Puis la sortie d’erreur dans la sortie standard
- Enfin, on exécute « tail –f » sur le fichier de log et on redirige le tout dans le fichier représentant le terminal. À ce stade, on ne peut plus utiliser la sortie standard car elle a été redirigée dans le fichier de logs. L’option –pid est utilisée pour terminer le processus à la fin du script.
- Sinon :
- On ferme la sortie standard
- On ferme la sortie d’erreur
- On redirige les sorties standard et d’erreur dans le fichier de log

## Gestion des erreurs

Bash est votre ennemi : pas de compilation, pas de déclaration nécessaire des variables, comportement parfois subtil… Il ne manquera pas de vous faire payer la moindre erreur d’attention. Pour se protéger, il convient d’adopter une attitude défensive. L’excellent [Unofficial Bash Strict Mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/) présente une méthode permettant de réduire grandement cette agressivité inhérente, au prix de quelques efforts sur la syntaxe et le style de programmation. Celle-ci s’appuie sur des options présentes dans Bash qui ne sont pas activées par défaut. En voici quelques-unes qui devraient améliorer la situation et réduire le nombre d’heures passées à débugger vos scripts :

```
 set -o errexit (-e) : force le script à quitter en cas d’erreur.
```

Si le script que l’on développe doit ne produire aucune erreur pour se comporter correctement, il faut impérativement utiliser cette option. En effet, plutôt que de tester le code de retour de chaque commande, on peut tout à fait n’en tester aucun et compter sur le fait que le script s’arrêtera en cas d’erreur.  
Exemple :

```
set -e ls /tmp ls /djsqlkdjqsl # Grave erreur ! ls /tmp
```

Résultat :

```
server:~$ ls -a /tmp/
 . .. .font-unix .ICE-unix .Test-unix .X11-unix .XIM-unix ls: cannot access /djsqlkdjqsl: No such file or directory # La dernière commande n’est pas exécutée
```

### set -o pipefail : Fait échouer l’intégralité d’un pipeline en cas d’erreur.

Par défaut, un pipeline sort avec le code de retour de la dernière commande appelée. Cela peut créer des comportements incompréhensibles, même si l’on utilise l’option errexit, car les pipelines sont exécutés dans un sous-shell.  
Exemple :

```
server:/tmp $ find /non_existing_path | sort ; echo $? 
find: `/non_existing_path': No such file or directory 0 
server:/tmp $ set -o pipefail 
server:/tmp $ find /non_existing_path | sort ; echo $? find: `/non_existing_path': No such file or directory 1
```

### set -o nounset (-u) : force le script à sortir en cas d’utilisation d’une variable non définie.

L’usage de variables non définies est source d’erreur ; en effet, il est très facile de se tromper en introduisant une faute de frappe lors de l’usage d’une variable, celle-ci étant alors vidée de son contenu.  
Exemple:

server:/tmp $ cat nounset.sh #! /bin/bash logfile="/var/log/boostrap.log" echo "Logs will be in [$logFile]" set -o nounset echo "Logs will be in [$logFile]" server:/tmp $ ./nounset.sh Logs will be in [] ./nounset.sh: line 6: logFile: unbound variable

La variable d’environnement **IFS** est une liste de caractères qui définit le ou les séparateur(s) de champs utilisé par Bash ; elle est composée par défaut de l’espace, de la tabulation et du saut de ligne. Ce séparateur de champs influe sur la façon dont le shell va séparer les éléments quand il consomme du texte, soit à travers une liste, soit à travers la commande **read**. Pour éviter les déconvenues, il est préférable de redéfinir ce séparateur, et de le limiter au saut de ligne.  
Voyons pourquoi à travers un exemple :

```
server:/tmp/s$ ls -1 File1 File2 File3 File with spaces FileZZ 
server:/tmp/s$ for x in $(ls -1); do echo "This is [$x]" ; done 
This is [File1] 
This is [File2] 
This is [File3] 
This is [File] 
This is [with] 
This is [spaces] 
This is [FileZZ] 

server:/tmp/s$ IFS=$'\n' 
server:/tmp/s$ for x in $(ls -1); do echo "This is [$x]" ; done 
This is [File1] 
This is [File2] 
This is [File3] 
This is [File with spaces] 
This is [FileZZ]
```

## Gestion de la fin du script

Il est souvent nécessaire de nettoyer des ressources et d’afficher des informations à la fin d’un script. Pour ce faire, Bash offre la commande **trap** qui permet d’intercepter des signaux et d’y associer du code. La syntaxe générale est la suivante :

trap commande spécification_du_signal

La commande peut être une fonction, éventuellement accompagnée d’arguments ; la spécification du signal peut être un signal Unix « classique », ou un mot clef qui regroupe plusieurs signaux.

Pour notre cas, nous nous intéresserons à un usage particulier de cette commande qui doit être combinée à l’option **errexit** ; le but du jeu est d’avoir une fonction appelée au moment où le script devrait normalement s’interrompre.

Exemple :

```
server:/tmp $ cat exit_handler_demo.sh 
#! /bin/bash 
tmpFile=$(mktemp) 
exit_handler() {
 echo "Script unexpectedly exited with code [$?], cleaning up [$tmpFile]" 
 rm "${tmpFile}" } 
 set -o errexit trap exit_handler ERR 
 echo "So far so good..." 
 ls /no_such_file 
 echo "All success!" 


 server:/tmp $ ./exit_handler_demo.sh 
 So far so good... 
 ls: cannot access /no_such_file: No such file or directory Script unexpectedly exited with code [2], cleaning up [/tmp/tmp.0ybcm43sgN]
```

### Rien ne va plus!

Par moment, on ne comprend plus ce qui se passe. La syntaxe est correcte, les variables sont bien initialisées, la logique semble cohérente, et pourtant le programme ne se comporte pas comme on le souhaiterait. Voici quelques conseils pour affronter ces situations stressantes :

- N’hésitez pas à extraire la section qui ne se comporte pas comme vous le souhaitez, et exécutez la indépendamment du reste du script. Idéalement, retapez les commandes dans le shell une par une, et arrêtez vous juste avant que les problèmes ne se manifestent. A ce stade, inspectez scrupuleusement les variables d’environnement et la façon dont le shell évalue les commandes pour comprendre ce qu’il faut corriger. Si cela est trop fastidieux, créez un petit script qui reproduit la section fautive et ajoutez des traces avec la commande **echo**.
- Pour vous aider dans cette tache, deux options de Bash peuvent s’avérer extrêmement utiles. - **set -o verbose (-v)** : affiche les lignes du script juste avant de les exécuter sans évaluer le contenu, au fur et à mesure de la saisie.
- **set -o xtrace (-x)** : affiche les lignes du script en évaluant leur contenu mais avant de les évaluer.
- Il faut aussi noter que l’on peut les activer et les désactiver à loisir : **set +x** désactive l’option, **set -x** l’active (oui, c’est assez contre intuitif!). Utilisez cette fonctionnalité pour rendre le script verbeux uniquement dans la partie à debugguer.
- Si vous essayez de créer une expression rationnelle et que celle-ci ne réagit pas comme vous le souhaitez, essayer de revenir à une expression plus simple, et lorsque votre expression réagit, procédez de façon incrémentale pour ajouter les spécificités dont vous avez besoin.

Démonstration:

```
bash-4.3$ set -v 
bash-4.3$ x=abcd x=abcd 
bash-4.3$ echo $x # l'option -v n'évalue pas le contenu de l'expression abcd 
bash-4.3$ set +v -x 
bash-4.3$ echo $x + echo abcd ## L'option -x évalue le contenu de l'expression abcd + set +v 
bash-4.3$ for y in {1..3}; do echo $(seq $((2*3)) $((8 - $y))); done 
+ for y in '{1..3}' 
++ seq 6 7 
+ echo 6 7 6 7 
+ for y in '{1..3}' 
++ seq 6 6 
+ echo 6 6 
+ for y in '{1..3}' 
++ seq 6 5 
+ echo 6 5 6 5 

bash-4.3$ set -v + set -v 
bash-4.3$ for y in {1..3}; do echo $(seq $((2*3)) $((8 - $y))); done 
for y in {1..3}; do
 echo $(seq $((2*3)) $((8 - $y))); 
done 

+ for y in '{1..3}' seq $((2*3)) $((8 - $y)) 
++ seq 6 7 
+ echo 6 7 6 7 
+ for y in '{1..3}' seq $((2*3)) $((8 - $y)) 
++ seq 6 6 
+ echo 6 6 
+ for y in '{1..3}' seq $((2*3)) $((8 - $y)) 
++ seq 6 5 
+ echo 6 5 6 5
```

## Fonction d’affichage de la pile d’appels

De nombreux langages offrent la possibilité d’afficher une pile d’appels de fonctions (aussi appelée stacktrace en anglais) en cas d’erreur, ce qui est fondamental pour comprendre la chemin d’exécution du code et la cause de l’erreur. Cette trace contient généralement le nom des fonctions et les numéros de lignes par lesquelles sont passées le programme. Bash offre quelques options et variables qui vont me permettre de générer une fonction affichant une stacktrace, y compris si les fonctions sont étalées sur plusieurs fichiers.

Voyons un exemple d’exécution de la fonction proposée. Le script faily.sh inclus la bibliothèque comprenant la fonction de stacktrace, et l’initialise avec la fonction bash_setup. Pour illustrer la fonctionnalité de trace multi-fichiers, j’ai placé la fonction t2 dans un fichier indépendant.

```
server:/tmp $ cat --number faily.sh
#!/bin/bash 
source stacktrace.lib.sh
bash_setup
source t2.sh 
t1() {
 echo "Called t1" 
 t2
}
t3() {
 echo "Called t3" 
 ls /bogus 
} 
echo "Let's call t1" 
t1 
echo "Done." 

server:/tmp $ cat --number t2.sh 
t2()
{ 
 echo "Called t2" 
 t3 
}
```

Passons maintenant à l’exécution:

```
server:/tmp $ ./faily.sh 
Let's call t1 
Called t1 
Called t2 
Called t3 
ls: cannot access /bogus: No such file or directory 
<--- ERROR: [./faily.sh line 17] - command 'ls /bogus' exited with status: 2 | [./faily.sh] | main:21 | t1:11 | [t2.sh] | t2:4 | [./faily.sh] V t3:17 --->
```

Voyons ce qui se passe sous le capot; j’utilise le mécanisme du exit_handler présenté plus haut, mais les paramètres sont différents:

```
trap exit_handler $? $LINENO "${BASH_LINENO[*]}" "${BASH_SOURCE[*]}" "$BASH_COMMAND" "${FUNCNAME[*]:-empty}"' ERR
```

Les variables **BASH_LINENO**, **BASH_SOURCE** et **FUNCNAME** sont des tableaux qui contiennent respectivement les numéros de lignes, les noms de fichiers et les noms de fonctions de la pile d’appel. Il suffit donc de parcourir ces tableaux pour reconstruire le déroulement de l’exécution. Des tests ont été ajoutés pour n’afficher le nom de fichier que si celui-ci a changé depuis le dernier appel.

## Maintenant, à vous de jouer !

L’ensemble des fonctionnalités présentées dans cet article est disponible sur le compte [github de WeScale](https://github.com/WeScale/WeBash), et je vous invite à en profiter. La bibliothèque peut s’intégrer dans n’importe quel script, comme dans le dernier exemple. Vos commentaires sont les bienvenus !