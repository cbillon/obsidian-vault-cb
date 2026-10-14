---
link: https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice
site: GitHub
excerpt: Script bash pour corriger l affichage des emojis dans OnlyOffice (Snap)
  sur Ubuntu 24.04 -
  NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice
twitter: https://twitter.com/@github
slurped: 2026-10-14T08:23
title: "GitHub -
  NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice: Script
  bash pour corriger l affichage des emojis dans OnlyOffice (Snap) sur Ubuntu
  24.04"
---

Script bash qui corrige l'absence d'affichage des emojis dans OnlyOffice installé via Snap sur Ubuntu 24.04.

## Problème

[](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice#probl%C3%A8me)

Lorsqu'un document contient des emojis (ex : 💡 🔥 ✅), OnlyOffice les affiche sous forme de point d'interrogation `?` au lieu du caractère graphique attendu.

### Cause

[](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice#cause)

OnlyOffice utilise son propre moteur de rendu de polices. La police emoji installée par défaut sur Ubuntu, `NotoColorEmoji.ttf`, utilise un format bitmap couleur (CBDT/CBLC) **incompatible** avec ce moteur. OnlyOffice la détecte lors du scan mais la rejette silencieusement, sans aucun fallback disponible pour les caractères emoji.

### Solution

[](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice#solution)

Installer la police **Symbola**, une police vectorielle (outline TTF) couvrant l'ensemble des emojis Unicode 9.0, que le moteur de rendu d'OnlyOffice peut utiliser. Puis vider le cache de polices d'OnlyOffice pour qu'il intègre Symbola au prochain démarrage.

> Les emojis s'affichent en **monochrome** (Symbola est une police noir & blanc), mais correctement et sans point d'interrogation.

## Prérequis

[](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice#pr%C3%A9requis)

- Ubuntu 24.04
- OnlyOffice installé via **Snap** (`sudo snap install onlyoffice-desktopeditors`)
- OnlyOffice avoir été lancé **au moins une fois** (pour que le dossier de cache existe)
- Droits `sudo` (pour l'installation de la police)

## Installation

[](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice#installation)

# Télécharger le script
curl -O https://raw.githubusercontent.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice/main/fix-onlyoffice-emojis.sh

# Le rendre exécutable
chmod +x fix-onlyoffice-emojis.sh

# L'exécuter
./fix-onlyoffice-emojis.sh

Puis **relancer OnlyOffice**. Le premier démarrage sera un peu plus lent (recalcul du cache de polices), c'est normal.

## Ce que fait le script

[](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice#ce-que-fait-le-script)

1. Vérifie si `fonts-symbola` est installé, et l'installe si ce n'est pas le cas (`sudo apt-get install fonts-symbola`)
2. Localise automatiquement le dossier de cache de polices d'OnlyOffice dans `~/snap/onlyoffice-desktopeditors/` (toutes révisions snap confondues)
3. Supprime les fichiers de cache (`AllFonts.js`, `font_selection.bin`, `fonts_thumbnail*`, etc.) pour forcer leur régénération au prochain démarrage

## Compatibilité

[](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice#compatibilit%C3%A9)

|Élément|Version testée|
|---|---|
|Ubuntu|24.04 LTS|
|OnlyOffice Desktop Editors|9.3.1 (Snap)|
|fonts-symbola|2.60|

## Licence

[](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice#licence)

MIT — voir [LICENSE](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice/blob/main/LICENSE)

## Auteur

[](https://github.com/NicoBOD/Script-de-correction-d-affichage-des-emojis-dans-OnlyOffice#auteur)

**Nicolas BODAINE**