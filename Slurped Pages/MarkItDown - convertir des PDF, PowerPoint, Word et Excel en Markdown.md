---
link: https://zonetuto.fr/python/markitdown-convertir-des-pdf-powerpoint-word-et-excel-en-markdown/
byline: MrTuto
site: ZoneTuto
date: 2024-12-30T21:56
excerpt: L'outil MarkItDown en Python vous permet facilement de transformer des fichiers PDF, PowerPoint, Word et Excel et HTML au format Markdown
slurped: 2025-02-22T09:19
title: "MarkItDown : convertir des PDF, PowerPoint, Word et Excel en Markdown"
tags:
  - pdf
  - tools
---

Le langage de programmation Python comporte beaucoup d’avantages. C’est facile à installer sur l’ensemble des systèmes d’exploitation, c’est assez rapide à apprendre et ça marche presque partout. Le Python est quasiment utilisable partout, que ce soit sur une machine Linux et ses différentes distributions, un Raspberry Pi, macOS ou encore sur Windows. Python est même disponible sur certaines architectures exotiques. Au fil des années, il s’est largement imposé et depuis l’essor de l’intelligence artificielle pour le grand public, c’est encore plus vrai. Les différents outils qui ont été créés avec ce langage de programmation dominent largement le domaine de l’intelligence artificielle, mais ce n’est pas le seul sujet pour lequel Python est très répandu.

Je vais sûrement vous reparler de ce langage que j’ai découvert pendant mes études et qui depuis a énormément évolué. Vous pouvez presque tout faire avec du Python. Si vous voulez apprendre la programmation, Python est un très bon choix et il existe un très grand nombre de tutoriels en ligne. Je vais d’ailleurs essayer de continuer à vous en proposer toujours plus, il y a tant de choses à dire.

L’autre avantage, c’est que vous trouverez presque toujours une librairie pour faire ce que vous avez besoin donc pas besoin de réinventer la roue. Surtout qu’en général, c’est bien documenté et assez simple à utiliser. Si vous voulez ajouter une nouvelle fonctionnalité à votre application, quelques lignes seront suffisantes pour utiliser les outils qui ont été développés par la communauté du libre et de l’open source. C’est ce que nous allons voir dans cet article avec MarkItDown.

## C’est quoi MarkItDown ?

MarkItDown est une bibliothèque Python open source développée par Microsoft, conçue pour convertir divers types de fichiers en format Markdown, facilitant ainsi leur indexation et analyse. Elle prend en charge une large gamme de formats, notamment les documents Word (.docx), Excel (.xlsx), PowerPoint (.pptx), PDF, ainsi que les images (avec extraction des métadonnées EXIF et reconnaissance optique de caractères), les fichiers audio (avec transcription automatique), le HTML et d’autres formats texte tels que CSV, [JSON](https://zonetuto.fr/python/comment-traiter-fichier-json-volumineux/) et XML.

Cette polyvalence permet aux développeurs d’intégrer facilement des documents existants dans des workflows basés sur [Markdown](https://fr.wikipedia.org/wiki/Markdown), améliorant ainsi l’efficacité et la cohérence du traitement de l’information. De plus, MarkItDown offre une interface en ligne de commande et une API Python, s’adaptant aux différents besoins des utilisateurs. Son installation est simple via pip, et son utilisation intuitive en fait un outil précieux pour quiconque souhaite automatiser la conversion de documents en Markdown.

## Installer MarkItDown avec pip

Je vais partir du principe que vous avez déjà installé Python sur votre machine donc je n’aborderai pas cette partie. De toute façon, vous avez sûrement une version récente de Python déjà installée non ? Pour installer MarkItDown sur Windows avec pip vous pouvez faire la commande suivante :

```
py -m pip install markitdown
```

Par contre sur Ubuntu, la distribution [Linux que j’utilise sur mes serveurs VPS](https://zonetuto.fr/linux/comment-creer-gratuitement-un-serveur-vps-sur-digitalocean/), c’est encore plus court :

```
pip install markitdown
```

D’ailleurs, pour éviter tout problème, je vous conseille d’avoir, si vous utilisez cette distribution, un [Ubuntu bien à jour](https://zonetuto.fr/linux/faire-migration-ubuntu-lts-vers-version-superieure/). L’installation va commencer avec la récupération de dépendances, c’est que vous êtes sur la bonne voie ! Vous devriez avoir quelque chose comme ça qui s’affiche si l’installation de MarkItDown se lance bien :

```
Collecting markitdown
  Downloading markitdown-0.0.1a3-py3-none-any.whl.metadata (4.8 kB)
Collecting beautifulsoup4 (from markitdown)
  Downloading beautifulsoup4-4.12.3-py3-none-any.whl.metadata (3.8 kB)
Collecting charset-normalizer (from markitdown)
  Downloading charset_normalizer-3.4.0-cp313-cp313-win_amd64.whl.metadata (34 kB)

etc ...
```

## Utiliser MarkItDown pour transformer des fichiers au format Markdown

Dans cette dernière partie nous allons voir comment utiliser MarkItDown très simplement. Vous allez voir, c’est vraiment facile, merci Python ! L’API de MarkItDown est très facile à prendre en main et permet de convertir rapidement un fichier au format PDF en Markdown à l’aide de quelques lignes de code Python. Pour commencer, il suffit d’importer le module, puis de créer une instance de la classe MarkItDown et enfin d’utiliser la méthode convert. Après l’installation de MarkItDown, vous pouvez ainsi transformer un fichier-zonetuto.pdf en Markdown en appelant convert et en affichant le contenu grâce à la propriété text_content. Avec du Python ça donne par exemple :

from markitdown import MarkItDown
markitdown = MarkItDown()
result = markitdown.convert("zonetuto.pdf")
print(result.text_content)

Si vous préférez employer MarkItDown en ligne de commande, vous pouvez également exécuter la commande markitdown suivie de votre fichier-zonetuto.pdf. Cette commande affichera le Markdown généré directement dans votre terminal. Pour enregistrer ce résultat dans un fichier-zonetuto.md, il suffit de rediriger la sortie standard vers un nouveau fichier. Vous obtiendrez alors un fichier Markdown prêt à être édité et partagé. Voici un exemple :

```
markitdown fichier-zonetuto.pdf > fichier-zonetuto.md
```

Sur Windows, vous pouvez exécuter MarkItDown en invoquant Python directement et en indiquant le module à utiliser. Cette commande fonctionne de la même manière que la version Linux ou macOS et vous pouvez également rediriger la sortie vers un fichier Markdown. Vous obtiendrez ainsi le contenu converti dans fichier-zonetuto.md grâce à la commande suivante :

```
py -m markitdown fichier-zonetuto.pdf > fichier-zonetuto.md
```

Si vous souhaitez envoyer un flux de données en entrée pour générer le Markdown, vous pouvez simplement omettre l’argument du fichier et utiliser une commande comme cat. Cela permettra d’utiliser MarkItDown à partir d’un fichier-zonetuto.pdf sans devoir préciser directement le nom du fichier dans la commande. Cette méthode est très pratique pour enchaîner plusieurs traitements ou manipulations de fichiers dans un même flux. La commande ressemblera donc à ceci :

```
cat fichier-zonetuto.pdf | markitdown
```

Vous pouvez aussi faire la commande suivante si vous rencontrez des problèmes d’encodage de caractères :

```
markitdown fichier-zonetuto.pdf -o document.md
```

Cette commande utilise un paramètre intégré à l’outil qui précise directement à MarkItDown le chemin du fichier de sortie. Techniquement, le résultat est le même, mais cette forme ne repose pas sur la redirection du shell, ce qui peut être pratique dans certains contextes. J’espère que vous allez maintenant pouvoir convertir sans le moindre problème vos fichiers en Markdown à l’aide de MarkItDown. Pour suivre les évolutions du logiciel, je vous conseille d’[aller voir ce qu’il se passe sur le dépôt GitHub](https://github.com/microsoft/markitdown), car il évolue rapidement avec une petite communauté qui s’est déjà construite autour.