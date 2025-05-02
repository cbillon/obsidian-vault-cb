---
link: https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter
site: Linuxtricks.fr
excerpt: |-
  Introduction
  Je script, je script, mais parfois, j'ai un sacré trou de mémoire ... et je galère à trouver ce que je cherche sur...
slurped: 2024-12-19T09:12
title: BASH - Mémo pour scripter - Wiki
tags:
  - bash
---

Table des matières

1. [Introduction](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-introduction)
2. [Prérequis](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-prerequis)
3. [Les variables](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-les-variables)
    1. [Affectation de variable](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-affectation-de-variable)
    2. [Lire une variable depuis une saisie du clavier](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-lire-une-variable-depuis-une-saisie-du-clavier)
    3. [Définir une valeur par défaut à une variable](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-definir-une-valeur-par-defaut-a-une-variable)
    4. [Utiliser les variables](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-utiliser-les-variables)
    5. [Utilisation avancée](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-utilisation-avancee)
    6. [Variables spéciales](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-variables-speciales)
4. [Calculs](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-calculs)
    1. [Calculs simples](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-calculs-simples)
    2. [Incrémenter et décrémenter](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-incrementer-et-decrementer)
5. [Les structures conditionnelles : If](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-les-structures-conditionnelles-if)
6. [A propos de la syntaxe des tests](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-a-propos-de-la-syntaxe-des-tests)
7. [Les opérateurs de comparaison](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-les-operateurs-de-comparaison)
    1. [Comparaison d'entiers](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-comparaison-d-entiers)
    2. [Comparaison de chaînes](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-comparaison-de-chaines)
    3. [Match de chaine](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-match-de-chaine)
    4. [Les tests sur les chaines](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-les-tests-sur-les-chaines)
    5. [Les tests sur les fichiers](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-les-tests-sur-les-fichiers)
    6. [Comparaison de fichiers](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-comparaison-de-fichiers)
    7. [La négation (inverser le test)](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-la-negation-inverser-le-test)
    8. [Et Ou](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-et-ou)
8. [Les structures conditionnelles Partie 2](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-les-structures-conditionnelles-partie-2)
    1. [For](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-for)
    2. [While](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-while)
    3. [Case ou switch](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-case-ou-switch)
9. [Les tableaux](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-les-tableaux)
10. [Des petits trucs marrants](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-des-petits-trucs-marrants)
    1. [Lire un fichier ligne par ligne](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-lire-un-fichier-ligne-par-ligne)
    2. [Vérifier si on est root](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-verifier-si-on-est-root)
    3. [Vérifier le bon déroulement d'une commande](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-verifier-le-bon-deroulement-d-une-commande)
    4. [Mettre le texte en couleur](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter#paragraph-mettre-le-texte-en-couleur)

## Introduction

Je script, je script, mais parfois, j'ai un sacré trou de mémoire ... et je galère à trouver ce que je cherche sur Internet.  
Comment on incrémente une variable ? Comment on fait un SI, un POUR ?  
Et bien la réponse se trouve ci-dessous !

## Prérequis

Comme tout script BASH, le doit posséder son SHEBANG :

## Les variables

### Affectation de variable

Pour mettre dans la variable **a** la valeur chaîne **abcdefghijklmnopqrstuvwxyz** :

Code BASH :

a="abcdefghijklmnopqrstuvwxyz"

De la même façon, on peut mettre dans des variables des nombres :

On peut mettre dans une variable le résultat d'une commande shell en encadrant la commande dans **$()** :

On pourra veiller à déclarer correctement les variables avec la commande **declare** pour forcer le type :

Code BASH :

declare -r variableEnLectureSeule
declare -i entier
declare -a tableau

On peut faire du 2 en 1 évidemment : déclarer et affecter

### Lire une variable depuis une saisie du clavier

Si vous voulez demander à l'utilisateur de taper du texte, et de récupérer le contenu dans une variable, c'est possible. C'est la commande read :

Ici, stockons le résultat de la saisie dans la variable TEXTE :

Avec l'option -p on pourra même mettre un message avant la saisie (qui se sera pas inclus dans le contenu de la variable) :

Code BASH :

read -p "Saisir un mot : " TEXTE

### Définir une valeur par défaut à une variable

Il est possible de définir une valeur par défaut à la variable si celle-ci est vide ou non définie :

Si la variable choix a déjà une valeur, alors l'expression reste inchangée, et la valeur de choix reste la même.  
Si la variable choix est vide ou non définie, alors elle est assignée à la valeur "N".

### Utiliser les variables

Pour utiliser ces variables, ajouter un **$** devant. Exemple pour afficher la variable a :

On peut l'appeler en l'encadrant avec ${} :

On peut aussi afficher la longueur de cette variable avec ${#var} :

### Utilisation avancée

On peut aussi extraire une sous-chaine à la volée

Depuis une position (première position à 0) jusqu'à la fin :

Depuis une position avec une longueur :

Code BASH :

echo ${VAR:position:longueur}

Exemple pour afficher 6 caractères de la variable a en commençant au caractère 0 (début) :

Exemple pour afficher à partir du caractère 0 mais en retirant les 4 derniers caractères :

On peut aussi faire de la substitution directement dans la variable (sans passer par sed par exemple) :

Code BASH :

echo ${VAR/cherche/remplace}

Exemple, remplacer le "e" par "E" :

Si on veut que cela se fasse de manière globale (comme le /g de sed), on écrira 2 barres obliques :

Code BASH :

echo ${VAR//cherche/remplace}

On pourra aussi modifier la casse.

Forcer en MAJUSCULES :

Forcer en minuscules :

### Variables spéciales

Il existe quelques variables spéciales :

**$$** : PID du shell courant  
**$!** : PID du dernier travail lancé en arrière plan  
**$?** : code retour de la dernière commande

Et d'autres variables prédéfinies, dites variables d'environnement :

**$HOME** chemin du répertoire personnel de l'utilisateur  
**$OLDPWD** chemin du répertoire précédent  
**$PATH** liste des chemins de recherche des commandes exécutables  
**$PPID** PID du processus père du shell  
**$PS1** invite principale du shell  
**$PWD** chemin du répertoire courant  
**$RANDOM** nombre entier aléatoire compris entre 0 et 32767  
**$REPLY** variable par défaut de la commande "read" et de la structure shell "select"  
**$SECONDS** nombre de secondes écoulées depuis le lancement du shell

## Calculs

### Calculs simples

Grosso-modo, pour faire des calculs simples, on utilise la syntaxe suivante :

On peut affecter ce calcul à une variable :

Ou bien l'afficher :

Si on veut faire une addition :

Si on veut faire une soustraction :

Si on veut faire une multiplication :

Si on veut faire une division :

On peut évidemment utiliser des variables dans le calcul :

### Incrémenter et décrémenter

Pour incrémenter ou décrémenter, on peut utiliser la fonction let :

  

## Les structures conditionnelles : If

Le test "Si'" est généralement fait ainsi :

Code BASH :

if [[Docker compose Condition]]
then
    commande 1
    commande2
fi

Pour les exemples, nous avons besoin d'opérateurs de comparaison, voir ci-dessous.

## A propos de la syntaxe des tests

En BASH, il y a deux possibilités pour écrire des tests :

et

La principale différence entre les deux est que le test avec le double crochet [[ ... ]] est propre à BASH et est une amélioration du simple crochet [ ... ], offrant des fonctionnalités supplémentaires telles que la manipulation de chaines avancées telles que les expressions régulières, ainsi que la possibilité d'utiliser des opérateurs logiques tels que && et ||.

Si on cherche à comparer simplement des nombres ou des chaînes le résultat est le même.

Par conséquent, j'utilise systématiquement le double crochet [[ ... ]].

## Les opérateurs de comparaison

### Comparaison d'entiers

**-eq** : est égal à

**-ne** : n'est pas égal à

**-gt** : est plus grand que

**-ge** : est plus grand ou égal à

**-lt** : est plus petit que

**-le** : est plus petit ou égal à

### Comparaison de chaînes

**=** : est égal à

**!=** : n'est pas égal à

**-z string** : Vrai si la longueur de la chaîne est égale à 0 (variable non initialisée ou vide)

**-n string** : Vrai si la longueur de la chaîne est supérieure à zero (variable initialisée et non vide)

### Match de chaine

**=~** : match avec une regexp.

Exemple, si la variable contient "cou" :

Exemple, si ça commence par un a :

### Les tests sur les chaines

**-z $var** : Vrai si la taille de la chaîne est zéro (variable non utilisée ou vide)

**-n $var** : Vrai si la taille de la chaîne est différente de zéro (variable utilisée et non vide)

### Les tests sur les fichiers

**-e file** : Vrai si le fichier existe

**-d file** : Vrai si le fichier existe et que c'est un répertoire

**-f file** : Vrai si le fichier existe et que c'est un fichier ordinaire

**-h file** : Vrai si le fichier existe et que c'est un lien symbolique

**-r file** : Vrai si le fichier possède les droits de lecture

**-s file** : Vrai si le fichier a une taille non nulle

**-w file** : Vrai si le fichier possède les droits d'écriture

**-x file** : Vrai si le fichier possède les droits d'exécution

**-O file :** Vrai si le possesseur est identique à celui qui exécute le test

**-G file :** Vrai si le possesseur qui exécute le test fait partie du groupe du fichier

-N file : Vrai si le fichier existe et qu'il a été modifié depuis sa dernière lecture

### Comparaison de fichiers

**file1 -nt file2** : Vrai si le file1 est plus récent que file2 (date de modification) ou si file1 existe et pas file2

**file1 -ot file2** : Vrai si file1 est plus ancien que file2 ou que file2 existe et pas file1

### La négation (inverser le test)

Le symbole de négation est **!**.  
Si on teste de cette façon su un dossier existe :

Et bien tester s'il existe pas se fera ainsi :

### Et Ou

Si vous voulez combiner des tests (le ET ou bien le OU) c'est avec && et || :

Pour le ET :

Code BASH :

if [[ "$a" -gt "$b" && "$c" -gt "$d" ]]

Pour le OU :

Code BASH :

if [[ "$a" -gt "$b" || "$c" -gt "$d" ]]

## Les structures conditionnelles Partie 2

### For

Pour le for ce n'est pas très compliqué :

Code BASH :

for i in 1 2 3 4 5
do
    echo $i
    commande1
    commande2
    commandeN
done

On peut traiter avec une commande aussi, par exemple la boucle ci-dessus peut être écrite en utilisant la commande **seq** :

Code BASH :

for i in $(seq 1 5)
do
    echo $i
    commande1
    commande2
    commandeN
done

Et on peut aussi utiliser des chaines de caractère :

Code BASH :

for f in fic1 fic2 fic3
do
    echo "On affiche le fichier $f"
    cat $f
done

Pour la boucle infinie, utiliser ceci (fin de la boucle avec **break**) ou **CTRL+C** :

Code BASH :

for (( ; ; ))
do
   echo "boucle infinie"
done

### While

De manière générale, la syntaxe est la suivante :

Code BASH :

while [[Docker compose Condition]]
do
   commande1
   commande2
   commande3
done

Voici un exemple :

Code BASH :

x=1
while [[ "$x" -le 5 ]]
do
  echo "Passage numéro $i"
  x=$(($x+1))
done

La boucle infinie se traduit ainsi :

Code BASH :

while :
do
    echo "boucle infinie"
done

La boucle infinie peut s’interrompre avec **break** :

Code BASH :

while :
do
    read -p "Entrer un chiffre ( -1 pour quitter ) : " a
    if [[ "$a" -eq -1 ]]
    then
        break
    fi
    echo "Vous avez saisi $a"
done

### Case ou switch

Le case (ou parfois dans d'autres langages switch) permet d'éviter l'imbrication de multiples if.

Voici un exemple. Le test peut être fait sur une chaine (ici B KB MB GB TB) ou sur des nombres :

Code BASH :

case $UNIT in
                B)
                        echo "Unité Octet"
                ;;
                KB)
                        echo "Unité KiloOctet"
                ;;
                MB)
                        echo "Unité MegaOctet"
                ;;
                GB)
                        echo "Unité GigaOctet"
                ;;
                TB)
                        echo "Unité TeraOctet"
                ;;
                *)
                       echo "Choix par défaut"
                ;;
                esac
 

## Les tableaux

Il est possible d'utiliser les tableaux en bash !

Affectation du premier enregistrement du tableau "tab" :

Contenu du premier enregistrement du tableau "tab" :

ou

Contenu du douzième enregistrement du tableau "tab" (oui ça commence à 0) :

Ensemble des enregistrements du tableau "tab" :

Longueur du douzième enregistrement du tableau "tab" :

Nombre d'enregistrements du tableau "tab" :

## Des petits trucs marrants

### Lire un fichier ligne par ligne

Pour lire les lignes d'un fichier :

Code BASH :

while read ligne
do
    echo  "$ligne"
done < fichier

Si on veut lire un fichier avec des lignes avec un séparateur (exemple passwd, qui a le séparateur : ) on peut stocker direct dans les variables

Code BASH :

while IFS=: read user pass uid gid nom dossperso shell
do
   echo  "$user avec UID $uid a son home dans $dossperso"
done < /etc/passwd

### Vérifier si on est root

Pour vérifier si le script est lancé en root, on peut utiliser ce petit morceau de code qui va vérifier l'id de l'utilisateur (0 = root) :

Code BASH :

if [[ $(id -u) -ne 0 ]]
then
   echo "Doit être lancé en root" 
   exit 1
fi

### Vérifier le bon déroulement d'une commande

Tous se passe dans le code retour de la commande.  
Exemple avec ls. La page man indique les différents codes retour :

Code TEXT :

       0      si OK,
       1      si problèmes mineurs (par exemple,  impossible  d'accéder  à  un
              sous-répertoire),
       2      si  erreur  grave  (par  exemple,  impossible  d'accéder aux pa‐
              ramètres de la ligne de commande).

Ce code est stocké dans la variable **$?** :

Voyons un exemple avec un succès (même si je n'ai rien dans /mnt, la commande n'a rencontré aucune erreur, le code vaut donc 0) :

Ici un exemple avec un répertoire inexistant (le code vaut 2) :

Code BASH :

ls /existepas
ls: impossible d'accéder à '/existepas': Aucun fichier ou dossier de ce type
echo $?
2

On peut tester le bon fonctionnement d'une commande en testant cette valeur retour :

Code BASH :

ls /mondossier
if [[ "$?" -ne "0" ]]
then
  echo "Dossier introuvable"
fi

La variable $? est modifiée à chaque exécution d'une commande.

On peut faire le test différemment ainsi :

Code BASH :

if ! ls /mondossier;
then
  echo "Dossier introuvable"
fi

### Mettre le texte en couleur

Pour agrémenter vos scripts, vous pouvez mettre de la couleur.

Voici une liste des couleurs possibles (ici j'affecte à des variables les codes couleur) :

Code BASH :

# Réinitialisation de la couleur après cette balise
COLOR_RESET='\033[0m'
# Codes couleurs à placer avant le texte :
COLOR_NOIR='\033[0;30m'
COLOR_ROUGE='\033[0;31m'
COLOR_VERT='\033[0;32m'
COLOR_JAUNE='\033[0;33m'
COLOR_BLEU='\033[0;34m'
COLOR_VIOLET='\033[0;35m'
COLOR_CYAN='\033[0;36m'
COLOR_BLANC='\033[0;37m'

Dans un code de type **\033[0;31m** pour le rouge, vous pouvez remplacer le 0 après le crochet ouvrant indique le style du texte.  
0 : Standard  
1 : Gras  
2 : Ca semble plus foncé  
3 : Italique  
4 : Souligné  
5 : Clignotant  
7 : Surligné  
9 : Barré

Voici un exemple en rouge et gras (notez l'usage de l'option -e de echo) :

Code BASH :

echo -e '\033[1;31m' ROUGE '\033[0m'

- [Partager sur Facebook](https://www.facebook.com/share.php?u=https%3A%2F%2Fwww.linuxtricks.fr%2Fwiki%2Fbash-memo-pour-scripter "Partager sur Facebook")
- [Partager sur Twitter](https://twitter.com/share?url=https%3A%2F%2Fwww.linuxtricks.fr%2Fwiki%2Fbash-memo-pour-scripter&text=BASH%20-%20M%C3%A9mo%20pour%20scripter%20-%20Wiki "Partager sur Twitter")
- [Partager par email Partager par Email](https://www.linuxtricks.fr/cdn-cgi/l/email-protection#99a6eaecfbf3fcfaeda4dbd8cad1bcaba9b4bcaba9d4bcdaaabcd8a0f4f6bcaba9e9f6ecebbcaba9eafaebf0e9edfcebbcaba9b4bcaba9cef0f2f0bffbf6fde0a4f1edede9eabcaad8bcabdfbcabdfeeeeeeb7f5f0f7ece1edebf0faf2eab7ffebbcabdfeef0f2f0bcabdffbf8eaf1b4f4fcf4f6b4e9f6ecebb4eafaebf0e9edfceb "Partager par Email")
- [Imprimer Version imprimable](https://www.linuxtricks.fr/wiki/bash-memo-pour-scripter# "Version imprimable")