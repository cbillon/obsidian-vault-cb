---
link: https://zonetuto.fr/linux/trouver-et-supprimer-les-fichiers-en-doublons-avec-fdupes/
byline: MrTuto
site: ZoneTuto
date: 2025-01-06T18:26
excerpt: Fdupes est un outil puissant et facile à utiliser qui permet de rechercher et d’éliminer les doublons sur votre distribution Linux
slurped: 2025-02-22T09:23
title: Trouver et supprimer les fichiers en doublons avec fdupes
tags:
  - file
  - doublon
---

Identifier et éliminer les fichiers en double peut vite devenir un casse-tête, surtout lorsque l’on accumule des documents, photos ou vidéos sur plusieurs supports. Si je vous écris cet article, c’est encore une fois après avoir eu besoin de mettre en pratique ce que je vais vous expliquer dans ce billet. J’ai une grosse machine sous Linux avec pas mal de fichiers, je dirais même plus, beaucoup de fichiers. J’aime bien collectionner et je dirais qu’en termes de volumétrie, j’ai un peu plus de 10 To de data. Quand j’ai le temps, je fais un peu de tri, mais je pense que je passe largement plus de temps à accumuler des fichiers. Merci la fibre…

Le problème est simple, comment identifier les fichiers que j’ai en doublon, et ce, de manière automatique. Je n’ai clairement pas le temps ni l’envie de le faire à la main de toute façon… Sur un système Linux, fdupes est un outil simple et efficace qui parcourt vos dossiers à la recherche de doublons afin de vous aider à faire de la place et à garder un environnement propre. C’est un programme en ligne de commande, léger et open source, qui s’appuie sur une méthode de comparaison fiable pour détecter les fichiers identiques, même lorsqu’ils sont dispersés sur des partitions différentes ou des disques externes. Rassurez-vous, ce n’est pas parce que c’est en ligne de commande que c’est difficile à utiliser.

fdupes compare d’abord la taille des fichiers pour éliminer rapidement ceux qui ne peuvent pas être identiques. Il calcule ensuite une somme de contrôle (souvent MD5) sur les fichiers de même taille, puis vérifie leur contenu octet par octet pour confirmer qu’ils sont réellement identiques. Cette stratégie confère à l’outil un bon équilibre entre rapidité et fiabilité. Même si le hashing permet une identification rapide des doublons, la relecture complète du contenu assure que deux documents ne sont jamais confondus uniquement parce qu’ils partagent une même signature numérique.

## Installation de fdupes

Pour commencer, vous devez installer fdupes sur votre système. Selon la distribution Linux que vous utilisez, voici comment procéder :

**Sur Debian/Ubuntu et dérivés :**

```
sudo apt install fdupes
```

**Sur Fedora :**

```
sudo dnf install fdupes
```

**Sur Arch Linux :**

```
sudo pacman -S fdupes
```

Une fois installé, vous êtes prêt à explorer vos disques à la recherche de doublons.

## Scanner un répertoire pour chercher les doublons

L’utilisation de base de `fdupes` est simple. Supposons que vous souhaitez scanner un répertoire spécifique, comme votre dossier `Documents`, pour y trouver des fichiers dupliqués. La commande est la suivante :

```
fdupes ~/Documents
```

Cette commande liste tous les doublons trouvés dans le répertoire spécifié. Les fichiers seront regroupés par paires ou groupes selon leur correspondance. Chaque groupe représente une série de fichiers ayant un contenu identique.

## Inclure les sous-répertoires dans la recherche

Par défaut, fdupes ne cherche pas dans les sous-dossiers du répertoire spécifié. Si vous souhaitez effectuer une recherche plus approfondie, vous devez ajouter l’option `-r` pour activer la récursivité. Voici un exemple :

```
fdupes -r ~/Documents
```

Avec cette commande, tous les sous-dossiers de `~~Documents~~` seront également analysés pour y détecter des doublons.

## Obtenir l’espace occupé par les doublons

Il peut être utile de savoir combien d’espace disque est gaspillé par les fichiers dupliqués. Pour cela, utilisez l’option `-S` :

```
fdupes -r -S ~/Documents
```

Cette commande affiche non seulement les doublons mais également la taille de chaque fichier, ce qui permet de calculer rapidement l’espace récupérable.

## Supprimer automatiquement les doublons

fdupes permet également de supprimer les fichiers en double pour libérer de l’espace. Cependant, cette opération est délicate, car elle peut entraîner une perte de données si elle n’est pas réalisée avec précaution. Pour supprimer les doublons automatiquement, vous pouvez utiliser l’option `-d` :

```
fdupes -r -d ~/Documents
```

Après avoir détecté les doublons, `fdupes` vous demandera de choisir quel fichier garder dans chaque groupe. Vous devrez confirmer chaque suppression manuellement. Attention ce paramètre doit être utilisé avec précaution. Elle nécessite une interaction utilisateur pour éviter des erreurs accidentelles. Vous pouvez également utiliser l’option `-N` pour désactiver la confirmation, mais cela est risqué.

## Scanner plusieurs répertoires en même temps

Supposons que vous ayez des données similaires réparties sur plusieurs dossiers ou disques. Vous pouvez scanner plusieurs répertoires à la fois en les listant dans la commande :

```
fdupes -r ~/Documents /mnt/external_drive ~/Backup
```

## Créer une liste des doublons dans un fichier texte

C’est clairement ce que je préconise si vous n’êtes pas sur de ce que vous voulez faire des fichiers en double. Vous pourrez toujours travailler à la suppression des doublons en vous basant sur ce fichier. Si vous souhaitez garder une trace des fichiers dupliqués pour une analyse ultérieure, vous pouvez rediriger la sortie de fdupes vers un fichier texte :

```
fdupes -r ~/Documents > doublons.txt
```

Vous pouvez bien évidemment envoyer la sortie vers un autre chemin dans le [système de fichier de Linux](https://zonetuto.fr/linux/comprendre-arborescence-des-dossiers-du-systeme-gnu-unix/) en le précisant explicitement :

```
fdupes -r ./MrTutoFile/ > ~/Desktop/liste_doublons_SSD4ToSamsungV2.txt
```

Fdupes se révèle un allié incontournable lorsqu’il s’agit de traquer et d’éliminer les doublons. Il s’installe en quelques secondes, ne consomme que très peu de ressources et offre un mode d’emploi accessible à tous. Qu’il s’agisse de nettoyer votre répertoire de téléchargements, de trier une vaste bibliothèque de photos ou de repérer des fichiers en trop sur un disque dur externe, il vous donne une vision claire de vos données et vous permet de reprendre le contrôle de votre [espace de stockage](https://zonetuto.fr/windows/comment-verifier-etat-sante-usure-disque-dur-ssd-crystaldiskinfo/).