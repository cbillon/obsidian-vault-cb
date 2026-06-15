---
link: https://guillaume-richard.fr/le-design-pattern-observer-en-php/
byline: Guillaume RICHARD
site: Portfolio Guillaume RICHARD
date: 2024-10-28T11:00
excerpt: Le pattern Observer notifie plusieurs objets observateurs lorsqu'un sujet change d'état, assurant un couplage faible et une architecture flexible
slurped: 2026-06-15T08:04
title: le Design Pattern Observer en PHP - Portfolio Guillaume RICHARD
tags:
  - php
---

Le **design pattern Observer** est un modèle de conception comportemental qui définit une relation **un-à-plusieurs** entre des objets. Il est utilisé lorsque vous souhaitez qu’un objet (appelé **sujet** ou **observable**) informe automatiquement plusieurs autres objets (appelés **observateurs**) de tout changement d’état, sans que les objets soient fortement couplés entre eux.

### Fonctionnement du pattern Observer

L’idée centrale est la suivante :

- Le sujet garde une liste d’observateurs qui doivent être informés de ses changements d’état.
- Lorsque l’état du sujet change, il envoie une notification à tous ses observateurs pour leur signaler cette modification.
- Chaque observateur peut alors réagir à ce changement de manière indépendante.

C’est un pattern très utile dans des scénarios où vous avez besoin de mettre à jour plusieurs composants en même temps lorsqu’un événement se produit, comme dans les interfaces graphiques, les systèmes de publication/souscription, ou encore les systèmes de notification.

### Exemple d’utilisation en PHP du pattern Observer

En PHP, on peut implémenter ce pattern en utilisant les interfaces pour représenter les concepts de « Sujet » et « Observateur ».

#### 1. Définition du sujet (Observable)

![Pattern Observer : Définition du sujet (Observable)](https://guillaume-richard.fr/wp-content/uploads/2024/10/Definition-du-sujet-Observable-587x1024.png)

#### 2. Définition de l’observateur

![Pattern Observer : Définition de l'observateur](https://guillaume-richard.fr/wp-content/uploads/2024/10/Definition-de-lobservateur-1024x590.png)

#### 3. Exemple d’utilisation

![Pattern Observer : Exemple d'utilisation](https://guillaume-richard.fr/wp-content/uploads/2024/10/Exemple-dutilisation-1024x689.png)

### Explication du code :

1. **ConcreteSubject** est le sujet observé. Il a une méthode `setState()` qui change son état interne et notifie les observateurs enregistrés via `notifyObservers()`.
2. **ConcreteObserver** est un observateur. Il implémente la méthode `update()` qui est appelée chaque fois que le sujet change.
3. Quand le sujet change d’état via `setState()`, il appelle la méthode `notifyObservers()`, qui parcourt tous les observateurs enregistrés et invoque leur méthode `update()`.

### Avantages :

- **Faible couplage** : Le sujet et les observateurs sont faiblement couplés. Le sujet ne connaît pas la nature exacte des observateurs et inversement.
- **Extensibilité** : Vous pouvez ajouter ou retirer des observateurs sans modifier le sujet, ce qui rend le système extensible.

### Inconvénients :

- **Surcharge de notifications** : Si le sujet change fréquemment et qu’il y a de nombreux observateurs, cela peut entraîner un grand nombre de notifications et ralentir le système.
- **Difficulté de débogage** : En raison de la nature asynchrone des notifications, il peut être difficile de comprendre quelles sont les conséquences de certains changements d’état.

### Utilisation courante

Le pattern Observer est souvent utilisé dans les systèmes de notification, les interfaces utilisateur, les systèmes de gestion d’événements ou les applications qui ont besoin de refléter en temps réel les changements d’état d’une entité dans plusieurs autres composants. Un exemple courant serait un système de **flux de nouvelles** où plusieurs utilisateurs (observateurs) reçoivent des mises à jour dès qu’un nouveau contenu est publié (sujet).

### Conclusion

Le pattern Observer permet une gestion flexible des relations de dépendance entre objets, où un changement d’état dans un sujet est automatiquement répercuté sur tous les observateurs, facilitant ainsi une architecture évolutive et modulaire.

## Liens Externes

- ****Facade** Pattern** à [designpatternsphp.readthedocs.io](https://designpatternsphp.readthedocs.io/en/latest/Behavioral/Observer/README.html)