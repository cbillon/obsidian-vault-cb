---
link: https://guillaume-richard.fr/le-design-pattern-builder-en-php/
byline: Guillaume RICHARD
site: Portfolio Guillaume RICHARD
date: 2024-06-17T10:00
excerpt: Le Design Pattern Builder permet de construire des objets complexes avec de nombreuses configurations possibles.
slurped: 2026-06-15T08:05
title: le Design Pattern Builder en PHP - Portfolio Guillaume RICHARD
tags:
  - php
---

Le **design pattern Builder** est un patron de conception qui permet de construire des objets complexes étape par étape. Il est particulièrement utile lorsque vous devez créer un objet avec de nombreuses configurations possibles.

Ce pattern sépare la construction d’un objet de sa représentation. Cela permet la même construction de processus pour créer différents types et représentations d’objets.

## Exemple Abtrait du Pattern Builder

Voici un exemple simple en PHP :

![Exemple de Design pattern Builder](https://guillaume-richard.fr/wp-content/uploads/2024/06/Design-pattern-Builder-799x1024.png)

Dans cet exemple, `Voiture` est la classe de l’objet complexe que nous voulons construire. `VoitureBuilder` est la classe qui construit l’objet `Voiture` étape par étape. Notez que la méthode `build()` retourne l’objet `Voiture` une fois qu’il a été entièrement construit.

## Exemple concret du Pattern Builder

Un bon exemple d’utilisation du design pattern Builder en PHP est la construction d’une requête SQL complexe.

Par exemple, vous pourriez avoir un `SqlQueryBuilder` qui vous permet de construire une requête SQL étape par étape, en ajoutant des conditions, des jointures, des tris, etc., de manière fluide. Voici un exemple simplifié :

![Exemple concret du Pattern Builder :
SQLQueryBuilder](https://guillaume-richard.fr/wp-content/uploads/2024/06/Exemple-concret-du-Pattern-Builder-1024x851.png)

Dans cet exemple, `SqlQueryBuilder` est le « Builder » qui permet de construire une requête SQL étape par étape. Chaque méthode modifie l’état interne de l’objet et renvoie l’objet lui-même (`$this`), permettant ainsi une interface fluide. Enfin, la méthode `getQuery()` est utilisée pour obtenir la requête SQL finale. C’est un exemple concret de comment le design pattern Builder peut être utilisé pour simplifier la création d’objets complexes.

## Autres design pattern

## Liens Externes

- ****Builder** Method** à [designpatternsphp.readthedocs.io](https://designpatternsphp.readthedocs.io/en/latest/Creational/Builder/README.html)