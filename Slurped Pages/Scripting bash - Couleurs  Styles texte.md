---
link: https://slash-root.fr/scripting-bash-couleurs-styles-texte/
site: slash-root.fr
date: 2022-12-06T18:13
excerpt: Mémo pour mettre de la couleur et du style sur le texte dans le scripting bash.
slurped: 2026-09-23T07:02
title: "Scripting bash : Couleurs / Styles texte"
---

[![Featured image of post Scripting bash : Couleurs / Styles texte](app://obsidian.md/scripting-bash-couleurs-styles-texte/cover.jpg)](app://obsidian.md/scripting-bash-couleurs-styles-texte/)

[Bash](app://obsidian.md/categories/bash/)

### Mémo pour mettre de la couleur et du style sur le texte dans le scripting bash.

## [](#couleurs)Couleurs

### [](#variables)Variables

```
neutre='\e[0;m'
noir='\e[0;30m'
gris='\e[1;30m'
rougefonce='\e[0;31m'
rose='\e[1;31m'
vertfonce='\e[0;32m'
vertclair='\e[1;32m'
orange='\e[0;33m'
jaune='\e[1;33m'
bleufonce='\e[0;34m'
bleuclair='\e[1;34m'
violetfonce='\e[0;35m'
violetclair='\e[1;35m'
cyanfonce='\e[0;36m'
cyanclair='\e[1;36m'
grisclair='\e[0;37m'
blanc='\e[1;37m'
```

### [](#utilisation)Utilisation

```
echo -e "${rougefonce}Hello${neutre} ${jaune}World${neutre}"
```

> **-e** : enable interpretation of backslash escapes

## [](#styles)Styles

### [](#variables-1)Variables

```
normal='\033[0m'
gras='\033[1m'
fin='\033[2m'
italic='\033[3m'
souligne='\033[4m'
flash='\033[5m'
inverse='\033[7m'
invisible='\033[8m'
```

### [](#utilisation-1)Utilisation

```
echo -e "${gras}Hello${normal} ${flash}World${normal}"
```

> **-e** : enable interpretation of backslash escapes