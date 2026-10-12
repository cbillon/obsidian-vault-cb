---
link: https://zigazou.dev/shell/fonction-assert
byline: Frédéric BISSON
excerpt: Une assertion en programmation permet de s’assurer d’une condition de fonctionnement avant d’exécuter les instructions suivantes. Bash en étant dépourvu, en voici une !
slurped: 2026-10-12T11:17
title: Fonction assert pour Bash
tags:
  - bash
  - debug
---

Une assertion en programmation permet de s’assurer d’une condition de fonctionnement avant d’exécuter les instructions suivantes. Bash en étant dépourvu, en voici une !

Sommaire

- [Utilisation](https://zigazou.dev/shell/fonction-assert#idm8)
- [Notes](https://zigazou.dev/shell/fonction-assert#idm13)
    - [Utilisation de "$@"](https://zigazou.dev/shell/fonction-assert#idm14)
    - [Utilisation de caller](https://zigazou.dev/shell/fonction-assert#idm22)
- [Code source](https://zigazou.dev/shell/fonction-assert#idm24)
- [Exemple d’utilisation](https://zigazou.dev/shell/fonction-assert#idm26)

Si vous programmez dans des langages plus évolués, la fonction assert ne vous est probablement pas inconnue.

Je vous propose donc une version pour Bash qui présentent quelques astuces dans sa mise en œuvre.

## Utilisation

La fonction s’appelle de la façon suivante :

shell

```
assert "Message d'erreur" assertion
```

Le premier paramètre est une chaîne de caractères qui sera affichée si jamais l'assertion n'était pas vérifiée.

Les paramètres suivants sont compris comme une commande complète à exécuter. Si cette commande retourne un code de retour différent de zéro, la fonction stoppe l'exécution du script en cours, affiche le message d'erreur et retourne le code de retour de l'assertion.

## Notes

### Utilisation de "$@"

On connaît souvent le paramètre spécial $* mais on en oublie tout aussi souvent le paramètre spécial $@. Ces deux paramètres sont très proches l'un de l'autre mais seul le paramètre $@ est intéressant pour la fonction assert.

Quand un script est appelé de la façon suivante :

shell

```
script "Param 1" "Param 2"
```

$1 reçoit "Param 1" et $2 reçoit "Param 2"

Si le script utilise $* dans une commande, la commande recevra à son tour "Param" "1" "Param" "2".

Si le script utilise "$@", la commande recevra à son tour "Param 1" "Param 2", les paramètres seront donc ainsi bien conservés.

Cette particularité est cruciale pour la fonction assert en Bash si on veut passer une commande sans devoir la protéger avec des double quotes ce qui alourdirait son utilisation.

### Utilisation de caller

La commande caller de Bash permet de retourner le numéro de la ligne appelant notre fonction ainsi que le script qui a fait l'appel. C'est très pratique lors de l'affichage du message d'erreur car cela permet d'indiquer la ligne qui a généré l'assertion. Du coup le développeur n'a pas besoin d'indiquer $LINENO dans l'appel de la fonction assert pour indiquer le numéro de ligne courante.

## Code source

shell

```
#!/bin/bash

function assert {
	# First parameter is the message in case the assertion is not verified
	local message="$1"

	# The remaining arguments make the command to execute
	shift

	# Run the command, $@ ensures arguments will remain in the same position.
	# "$@" is equivalent to "$1" "$2" "$3" etc.
	"$@"

	# Get the return code
	local rc=$?

	# If everything is okay, there's nothing left to do
	[ $rc -eq 0 ] && return 0

	# An error occured, retrieved the line and the name of the script where
	# it happend
	set $(caller)

	# Get the date and time at which the assertion occured
	date=$(date "+%Y-%m-%d %T%z")

	# Output an error message on the standard error
	# Format: date script [pid]: message (linenumber, return code)
	echo "$date $2 [$$]: $message (line=$1, rc=$rc)" >&2

	# Exit with the return code of the assertion test
	exit $rc
}
```

## Exemple d’utilisation

shell

```
#!/bin/bash

function assert { ... }

directory="$1"

assert "There must be one argument" test $# -eq 1
assert "Directory $directory does not exist" test -d "$directory"
assert "Directory $directory is not readable" test -r "$directory"
assert "Directory $directory is not writable" test -w "$directory"
assert "Directory $directory is not executable" test -x "$directory"
```