---
link: https://linuxfr.org/news/zaibu-une-alternative-libre-pour-les-amateurs-de-degustation
byline: lovasoa
excerpt: L’actualité du logiciel libre et des sujets voisins (DIY, Open Hardware, Open Data, les Communs, etc.), sur un site francophone contributif géré par une équipe bénévole par et pour des libristes enthousiastes
tags:
  - sqlpage
slurped: 2025-03-16T07:00
title: Zaibu, une alternative libre pour les amateurs de dégustation
---

Cette dépêche présente **Zaibu**, une application web auto-hébergeable permettant de conserver un journal structuré de ses dégustations de bières et de vins. Développée avec [SQLPage](https://sql-page.com/), elle met l’accent sur la simplicité, l’indépendance et le respect de la vie privée. Contrairement aux solutions centralisées comme Untappd ou Vivino, Zaibu ne collecte aucune donnée et reste entièrement sous le contrôle de l’utilisateur.

![Logo de Zaibu](app://img.linuxfr.org/img/68747470733a2f2f636f6465626572672e6f72672f6e616e6177656c2f7a616962752f7261772f636f6d6d69742f616236613362663562633066303635613830336431393633396432623839306131383864623861352f7372632f696d672f6c6f676f2e706e67/logo.png "Source : https://codeberg.org/nanawel/zaibu/raw/commit/ab6a3bf5bc0f065a803d19639d2b890a188db8a5/src/img/logo.png")

_Note : n’ayant absolument aucune compétence ni aucun talent en graphisme, le logo a été créé avec Bing Image Creator et retravaillé et vectorisé par mes soins. Je sais, çaymal._

## Sommaire

- - [Pourquoi créer Zaibu ?](https://linuxfr.org/news/zaibu-une-alternative-libre-pour-les-amateurs-de-degustation#toc-pourquoi-cr%C3%A9er-zaibu)
        - [Un besoin personnel](https://linuxfr.org/news/zaibu-une-alternative-libre-pour-les-amateurs-de-degustation#toc-un-besoin-personnel)
        - [Une occasion de tester SQLPage](https://linuxfr.org/news/zaibu-une-alternative-libre-pour-les-amateurs-de-degustation#toc-une-occasion-de-tester-sqlpage)
    - [Une approche libre et auto-hébergeable](https://linuxfr.org/news/zaibu-une-alternative-libre-pour-les-amateurs-de-degustation#toc-une-approche-libre-et-auto-h%C3%A9bergeable)
    - [Une interface simple et accessible](https://linuxfr.org/news/zaibu-une-alternative-libre-pour-les-amateurs-de-degustation#toc-une-interface-simple-et-accessible)
    - [Et maintenant ?](https://linuxfr.org/news/zaibu-une-alternative-libre-pour-les-amateurs-de-degustation#toc-et-maintenant)

> L’alcool est dangereux pour la santé, même en petite quantité. Le vin et la bière, comme les autres alcools, **induisent une dépendance et tuent**. Il est [recommandé](https://www.inserm.fr/dossier/alcool-sante/) de ne pas consommer plus de 2 verres par jour, et de ne pas boire d’alcool au moins 2 jours par semaine. Si vous avez des doutes sur votre consommation, n’hésitez pas à [contacter un professionnel de santé](https://www.ameli.fr/assure/sante/themes/addictions/suivi).

### Pourquoi créer Zaibu ?

Zaibu répond avant tout à un besoin très concret : garder une trace de ses dégustations de boissons (uniquement bières et vins pour l’instant) sans dépendre d’applications trop encombrées ou propriétaires qui exploitent les données de leurs utilisateurs.

Ce projet est en fait l’évolution d’un simple fichier texte mis en forme selon une structure plus ou moins régulière. Il était à l’origine partagé via Nextcloud, un service de stockage et de synchronisation de fichiers, libre et auto-hébergeable. Pour passer de ce fichier brut à une véritable application, plusieurs outils ont été utilisés:

- _Makefile_ : un fichier de configuration pour GNU Make, permettant d’automatiser diverses tâches (ici, la conversion du fichier texte).
- _Gawk_ : une version libre de l’outil AWK, qui lit et transforme le contenu du fichier texte pour l’adapter au format voulu.
- [textql](https://github.com/dinedal/textql) : un utilitaire en ligne de commande qui interprète des fichiers texte (CSV, TSV…) comme des tables SQL, ce qui facilite le chargement des données dans une base SQLite.

Grâce à cette chaîne d’outils, le fichier texte initial a pu être converti en une base de données exploitable, pour ensuite alimenter l’application Zaibu.

Pour ceux qui collectionnent les bouteilles comme d’autres collectionnent les timbres, c’est un outil pratique et léger, conçu pour être maîtrisé de bout en bout : le code source est distribué sous licence libre (AGPLv3), l’application est facile à héberger sur son propre serveur, et consomme très peu de ressources.

Un objectif secondaire était de tester les capacités de l’outil [SQLPage](https://sql-page.com/) pour le développement rapide d’applications de gestion de données.

#### Un besoin personnel

Il peut être difficile de se souvenir d’une bonne bière artisanale goûtée l’année passée ou du vin qui vous a tant plu à un mariage. Un carnet de notes ou un tableau dans un logiciel de bureautique peuvent dépanner, mais on s’y perd vite, et ce n’est pas toujours très pratique à consulter **sur son téléphone** quand on est en pleine dégustation.

Zaibu propose un formulaire simple où vous pouvez renseigner **le nom, le producteur, le style, l’amertume, le taux d’alcool, vos impressions**… Une fois la dégustation terminée, vous conservez une trace précise, consultable à tout moment. En un coup d’œil, vous pouvez comparer vos différents coups de cœur ou vous rappeler pourquoi un vin particulier ne vous avait pas convaincu.

#### Une occasion de tester SQLPage

Zaibu a aussi été conçu comme une démonstration technique. Il a servi de terrain d’expérimentation pour un nouvel outil, SQLPage, qui permet de créer une application web de gestion et d’affichage de données complète sans s’encombrer de milliers de lignes de code. En partant de requêtes de bases de données très simples, on obtient un site fonctionnel rapidement.

Ici il s’agit d’une application de type [CRUD](https://fr.wikipedia.org/wiki/CRUD) dans sa plus simple expression, donc parfaitement adaptée à être écrite en pur SQL. Même si certains traitements nécessitent de se creuser un peu plus la tête quand rien d’autre n’est disponible, il existe généralement une manière d’arriver à ses fins (et on découvre parfois avec bonheur des subtilités du langage qu’on ignorait !).

C’est le _framework_ parfait pour créer rapidement ses propres outils tout en gardant la maîtrise complète de sa donnée, en utilisant une base de données que l’on peut héberger soi-même facilement.

### Une approche libre et auto-hébergeable

De nombreuses applications existent déjà, mais elles imposent souvent la création d’un compte, exploitent les données des utilisateurs et monétisent leur activité via la publicité ou des abonnements. Zaibu prend le contre-pied en offrant une solution **entièrement libre**, légère et indépendante.

L’application repose sur _SQLite_, un système de gestion de base de données qui se distingue des bases de données traditionnelles comme _MySQL_ ou _PostgreSQL_. Contrairement à ces dernières, qui nécessitent un serveur dédié fonctionnant en arrière-plan pour gérer les requêtes et stocker les informations, _SQLite_ est une base de données embarquée.

Cela signifie que toutes les données sont enregistrées directement dans un fichier unique sur l’ordinateur ou le serveur où l’application est installée. Il n’y a donc pas besoin d’installer et de configurer un logiciel supplémentaire pour gérer la base de données. Cette approche simplifie considérablement l’installation et l’utilisation de l’application, surtout pour des utilisateurs qui ne sont pas familiers avec l’administration de serveurs.

Et puis bien sûr, son code est **ouvert**. C’est comme une bière artisanale : vous savez exactement quels ingrédients sont utilisés, comment ils interagissent, et si l’envie vous prend, vous pouvez modifier la recette pour l’adapter à vos préférences. Vous pouvez la brasser tel quel, y ajouter une touche personnelle, ou même la partager améliorée avec d’autres passionnés. Ici, **tout est transparent et modifiable**.

### Une interface simple et accessible

Pensée pour une utilisation mobile et desktop, l’interface de Zaibu permet d’**ajouter rapidement une dégustation**, sans fioritures. Sur smartphone, il devient facile de consulter ses notes en magasin ou chez un caviste pour retrouver une référence appréciée ou éviter une déception.

![capture d'écran du menu principal de l'interface mobile](app://img.linuxfr.org/img/68747470733a2f2f636f6465626572672e6f72672f6e616e6177656c2f7a616962752f7261772f636f6d6d69742f616236613362663562633066303635613830336431393633396432623839306131383864623861352f73637265656e73686f74732f696e6465785f6d6f62696c652e706e67/index_mobile.png "Source : https://codeberg.org/nanawel/zaibu/raw/commit/ab6a3bf5bc0f065a803d19639d2b890a188db8a5/screenshots/index_mobile.png")

![capture d'écran du formulaire de saisie de dégustation de bière de l'interface desktop](app://img.linuxfr.org/img/68747470733a2f2f636f6465626572672e6f72672f6e616e6177656c2f7a616962752f7261772f636f6d6d69742f616236613362663562633066303635613830336431393633396432623839306131383864623861352f73637265656e73686f74732f626565722e666f726d2e73716c5f69642e706e67/beer.form.sql_id.png "Source : https://codeberg.org/nanawel/zaibu/raw/commit/ab6a3bf5bc0f065a803d19639d2b890a188db8a5/screenshots/beer.form.sql_id.png")

### Et maintenant ?

Zaibu est encore jeune et perfectible. L’application pourrait évoluer avec des fonctionnalités comme le partage entre utilisateurs ou l’intégration d’une base collaborative… N’hésitez pas à faire vos retours dans les commentaires !

Et si le principe vous intéresse, vous pouvez aussi découvrir [Mon petit potager](https://codeberg.org/nanawel/monpetitpotager) du même auteur et construit sur le même framework, cette fois pour suivre les récoltes de son jardin et la pluviométrie.

## Aller plus loin

- [Code source de Zaibu (AGPLv3)](https://codeberg.org/nanawel/zaibu "https://codeberg.org/nanawel/zaibu") (211 clics)
- [SQLPage, l'outil libre de création de d'interfaces pour SQL (MIT)](https://sql-page.com/ "https://sql-page.com") (99 clics)
- [Tester Zaibu en ligne](https://zaibu-demo.lanterne-rouge.info/ "https://zaibu-demo.lanterne-rouge.info/") (300 clics)
- [Tester SQLPage en ligne](https://editor.datapage.app/ "https://editor.datapage.app") (74 clics)