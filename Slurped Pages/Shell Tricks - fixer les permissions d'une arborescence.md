---
link: https://www.emaxilde.net/posts/2017/02/01/shell-tricks-fixer-les-permissions-d-une-arborescence.html
byline: Olivier Poncet
site: Olivier Poncet
date: 2017-02-01T11:00
excerpt: |-
  Je constate souvent que les utilisateurs et/ou développeurs non Unixiens ayant besoin de travailler sous UNIX, Linux ou BSD ont tendance à ne pas fixer correctement les permissions des répertoires et des fichiers, parfois par fainéantise, mais le plus souvent par méconnaissance de la marche à suivre.
  Voici donc un petit trick pour ceux qui en ont besoin.
  Pourquoi ? Le plus souvent ce sont des développeurs qui ont besoin de dézipper une archive récupérée de on ne sait où, ou de récupérer des fichier depuis une machine Windows ou un serveur SMB/CIFS, et résultat les permissions sont le plus souvent incorrectes (en général 0777, soit rwxrwxrwx).
tags:
  - linux
  - bash
  - permission
slurped: 2025-07-06T19:28
title: Shell Tricks - fixer les permissions d'une arborescence
---

Je constate souvent que les utilisateurs et/ou développeurs non Unixiens ayant besoin de travailler sous UNIX, Linux ou BSD ont tendance à ne pas fixer correctement les permissions des répertoires et des fichiers, parfois par fainéantise, mais le plus souvent par méconnaissance de la marche à suivre.

Voici donc un petit trick pour ceux qui en ont besoin.

### Pourquoi ?

Le plus souvent ce sont des développeurs qui ont besoin de dézipper une archive récupérée de on ne sait où, ou de récupérer des fichier depuis une machine Windows ou un serveur SMB/CIFS, et résultat les permissions sont le plus souvent incorrectes (en général `0777`, soit `rwxrwxrwx`).

Lorsque je leur demande de fixer les permission des répertoires et fichiers avec des valeurs plus correctes, soit `0755` pour les répertoires et `0644` pour les fichiers, certains sont un peu à la lutte.

### Le trick

Pour arriver au résultat attendu en deux coups de cuillerée à pot, nous allons utiliser la commande `find` couplée à la commande `chmod`.

La première commande demande à `find` de trouver récursivement tous les répertoires et sous-répertoires (avec l’option `-type d`) à partir de `pathname` et d’appeler la commande `chmod 755` (avec l’option `-exec`) avec le nom représenté par les deux accolades `{}`. Le `\;` à la fin signifiant la fin de commande à appeler.

```
find 'pathname' -type d -exec chmod 755 {} \;
```

La seconde commande demande à `find` de trouver récursivement tous les fichiers (avec l’option `-type f`) à partir de `pathname` et d’appeler la commande `chmod 644` (avec l’option `-exec`) avec le nom représenté par les deux accolades `{}`. Le `\;` à la fin signifiant la fin de commande à appeler.

```
find 'pathname' -type f -exec chmod 644 {} \;
```

On peut fixer plus précisemment les permissions en fonction du type de fichier, par exemple rendre exécutable les scripts shell, en spécifiant l’option `-name` à la commande `find`.

```
find 'pathname' -type f -name "*.sh" -exec chmod 755 {} \;
```

Il existe évidemment d’autres façons de faire, notamment en utilisant la commande `xargs`, mais je vous ai exposé la méthode que j’utilise quotidiennement.

### What else ?

Voilà, c’est tellement simple et efficace que même les plus noobs n’auront plus d’excuses concernant des mauvaises permissions.