---
link: https://guillaume-richard.fr/le-design-pattern-facade-en-php/
byline: Guillaume RICHARD
site: Portfolio Guillaume RICHARD
date: 2024-10-14T11:00
excerpt: Le pattern Facade simplifie l'utilisation de systèmes complexes en offrant une interface unique pour interagir avec plusieurs sous-systèmes.
slurped: 2026-06-15T08:05
title: le Design Pattern Facade en PHP - Portfolio Guillaume RICHARD
tags:
  - php
---

Le **design pattern Facade** fait partie des modèles de conception structuraux, qui visent à simplifier les interactions avec un système complexe en fournissant une interface plus simple. Ce pattern permet de masquer la complexité des sous-systèmes en regroupant des appels de méthodes à des composants internes sous une interface unique.

### Objectif principal du pattern Facade

Le pattern **Facade** a pour but de :

- **Fournir une interface unifiée** et simplifiée à un ensemble d’interfaces dans un sous-système.
- **Réduire les dépendances** entre les classes et les sous-systèmes.
- Faciliter l’utilisation d’un système complexe en offrant un point d’entrée unique.

Ce modèle est particulièrement utile lorsque :

- Il y a un ensemble complexe de classes ou de systèmes qui sont difficiles à manipuler.
- Vous souhaitez limiter l’accès direct à certaines parties du code tout en les encapsulant.
- Vous voulez améliorer la **lisibilité** et **maintenabilité** du code.

### Exemple : Utilisation du pattern Facade en PHP

Prenons l’exemple d’un système de commande en ligne. Vous avez plusieurs sous-systèmes comme le **paiement**, la **gestion de stock** et la **notification**. Chacune de ces opérations est complexe, mais vous souhaitez fournir une interface simple pour le client qui veut passer une commande.

Voici une implémentation en PHP :

#### Étape 1 : Les sous-systèmes

Commençons par définir les sous-systèmes.

![Les sous-systèmes du design](https://guillaume-richard.fr/wp-content/uploads/2024/10/Les-sous-systemes-853x1024.png)

#### Étape 2 : La classe Facade

Maintenant, nous allons créer une classe **Facade** qui simplifie l’interaction avec ces sous-systèmes.

![La classe Facade.](https://guillaume-richard.fr/wp-content/uploads/2024/10/La-classe-Facade-902x1024.png)

#### Étape 3 : Utilisation de la façade

Le client (ou la partie de votre code qui fait appel à la commande) n’aura besoin d’interagir qu’avec la classe **Facade**, ce qui simplifie le processus.

![Utilisation de la Facade](https://guillaume-richard.fr/wp-content/uploads/2024/10/Utilisation-de-la-Facade-1024x368.png)

#### Sortie :

![Sortie de la Facade](https://guillaume-richard.fr/wp-content/uploads/2024/10/Sortie-de-la-Facade-1024x405.png)

### Explication de l’exemple

Dans cet exemple :

- **PaymentSystem**, **InventorySystem**, et **NotificationSystem** sont des sous-systèmes complexes.
- La classe **OrderFacade** joue le rôle d’une interface simplifiée qui regroupe les différentes étapes d’une commande : vérification du stock, traitement du paiement, mise à jour du stock et envoi d’une confirmation par email.
- Le client final n’interagit qu’avec la classe **Facade** sans avoir à connaître ou comprendre les détails de chaque sous-système.

### Avantages et inconvénients du pattern Facade

#### Avantages :

- **Simplicité** : Offre une interface claire et facile à utiliser, masquant la complexité.
- **Faible couplage** : Le client est découplé des sous-systèmes, rendant le code plus modulaire et maintenable.
- **Flexibilité** : Vous pouvez changer les sous-systèmes internes sans affecter le client.

#### Inconvénients :

- **Risque de sur-simplification** : Si vous masquez trop de fonctionnalités, vous pourriez rendre l’accès à certaines fonctions complexes plus difficile.
- **Ajoute une couche supplémentaire** : Bien que cela simplifie l’accès, cela ajoute une abstraction supplémentaire, ce qui peut parfois nuire aux performances ou à la compréhension.

### Conclusion

Le **pattern Facade** est idéal pour structurer des systèmes complexes et améliorer leur lisibilité et maintenabilité. En regroupant des interactions avec plusieurs sous-systèmes sous une interface unique, ce modèle réduit la complexité du code pour les utilisateurs finaux et améliore l’organisation globale du projet.

## Liens Externes

- ****Facade** Pattern** à [designpatternsphp.readthedocs.io](https://designpatternsphp.readthedocs.io/en/latest/Structural/Facade/README.html)