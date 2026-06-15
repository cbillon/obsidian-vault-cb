---
link: https://guillaume-richard.fr/le-design-pattern-strategy-en-php/
byline: Guillaume RICHARD
site: Portfolio Guillaume RICHARD
date: 2024-05-16T11:00
excerpt: Le Design Pattern Strategy encapsule des algorithmes dans des classes distinctes et les rend interchangeables à l'exécution.
slurped: 2026-06-15T08:06
title: le Design Pattern Strategy en PHP - Portfolio Guillaume RICHARD
tags:
  - php
---

Le **Design Pattern Strategy** est un modèle de conception comportemental qui permet de définir une famille d’algorithmes, de les encapsuler chacun dans une classe séparée, et de rendre leurs objets interchangeables. Ce modèle est souvent utilisé en PHP, en particulier lorsqu’il est nécessaire de changer d’algorithme à l’exécution.

## Exemple Abtrait du Pattern Strategy

Voici comment vous pouvez l’implémenter en PHP :

![Exemple de design Pattern Strategy  en PHP.](https://guillaume-richard.fr/wp-content/uploads/2024/05/Design-Pattern-Strategy-675x1024.png)

Dans cet exemple, `ConcreteStrategyA` et `ConcreteStrategyB` sont des stratégies concrètes qui implémentent une interface commune `Strategy`. `Context` utilise cette interface pour appeler l’algorithme défini par les Stratégies Concrètes. Pour changer la façon dont le contexte effectue son travail, d’autres objets peuvent remplacer l’objet de stratégie actuellement lié par un autre.

## Exemple plus concret du Pattern Strategy

Voici un exemple concret d’utilisation du Design Pattern Strategy en PHP.

Imaginons que nous avons besoin d’un système de filtrage pour les chaînes de caractères. Différents filtres pourraient inclure le retrait du HTML, la censure des gros mots, la détection de combinaisons de caractères pouvant être utilisées pour envoyer du spam, etc. Voici comment nous pourrions utiliser le **Design Pattern Strategy** pour cela :

![Exemple concret de Design Pattern Strategy en PHP](https://guillaume-richard.fr/wp-content/uploads/2024/05/Design-Pattern-Strategy-1-854x1024.png)

Dans cet exemple, **HTMLFilter** et **SwearFilter** sont des stratégies concrètes qui implémentent une interface commune **FilterFormData** utilise cette interface pour appeler l’algorithme défini par les Stratégies Concrètes. Pour changer la façon dont **FormData** effectue son travail, d’autres objets peuvent remplacer l’objet de stratégie actuellement lié par un autre.

## Autres design pattern

## Liens Externes

- **Strategy Method** in PHP par [refactoring.guru](https://refactoring.guru/design-patterns/strategy/php/example)
- ****Strategy** Method** à [designpatternsphp.readthedocs.io](https://designpatternsphp.readthedocs.io/en/latest/Behavioral/Strategy/README.html)