---
link: https://medium.com/@tpierrain/3-id%C3%A9es-re%C3%A7ues-%C3%A0-propos-des-agr%C3%A9gats-85fc2aedeba9
byline: Thomas Pierrain. (υѕe caѕe drιven)
site: Medium
date: 2024-03-17T16:38
excerpt: "TL;DR: Les agrégats ne sont pas l’aboutissement du DDD; ils sont juste un des nombreux outils tactiques mis à disposition par Eric Evans dans son blue book. Outil pratique certes, mais néanmoins…"
twitter: https://twitter.com/@tpierrain
slurped: 2024-08-26T18:09
title: 3 idées reçues à propos des Agrégats - Thomas Pierrain. (υѕe caѕe drιven) - Medium
tags:
  - ddd
---

[

![Thomas Pierrain. (υѕe caѕe drιven)](https://miro.medium.com/v2/resize:fill:88:88/0*MP7PVq_gncDctvyb.jpg)



](https://medium.com/@tpierrain?source=post_page-----85fc2aedeba9--------------------------------)

**_TL;DR:_** _Les agrégats ne sont pas l’aboutissement du DDD; ils sont juste un des nombreux outils tactiques mis à disposition par Eric Evans dans son blue book. Outil pratique certes, mais néanmoins compliqué à appréhender. Et même si la communauté a déjà tout écrit il y a longtemps à leur propos, je voudrais revenir ici sur quelques pièges et idées reçues qui persistent sur le sujet._

## **#1: Si tu n’as pas d’agrégats c’est que tu ne fais pas de DDD**

C’est totalement faux. Beaucoup trop de devs sont tellement obsédés à l’idée de trouver leurs agrégats, qu’ils passent complètement à côté de leur domaine et des discussions et modélisations intéressantes sur le sujet (privilégiant la forme au fond).

Ca fait 2 semaines que je cherche mon agrégat. Franchement c’est chaud !

Quitte à choisir, mieux vaut ne pas utiliser d’agrégat dans son code, mais faire en sorte que ce dernier soit aligné avec les langages des différentes perspectives du métier (cf. Ubiquitous languages et Bounded Context — BC).

Si vous ne savez pas ce qu’est le DDD, j’ai résumé ça pour vous ici : [**Domain Driven Design explained in 3 minutes**](https://medium.com/@tpierrain/domain-driven-design-explained-in-3-minutes-d45ed92fcc1c)**.** Et si vous n’avez pas 3 minutes, vous pouvez retenir que c’est une approche (et une boîte à outil) qui vise à produire des solutions logicielles les plus alignées possibles avec les problèmes métiers que vous devez résoudre.

**Avec ou sans Agrégat**. OSEF.

## #2: Tomber dans le piège du modélisme

Faire du DDD, ce n’est pas modéliser le monde réel ou l’intégralité de ce qu’on comprend du métier (i.e. faire du modélisme), mais plutôt de trouver des modèles utiles pour résoudre des problèmes métiers concrets dans notre logiciel.

Bah pourquoi t’as fait ça Jacques ?!? Bah…vu qu’on doit faire une appli pour réserver des billets de train… je me suis dit que ça pourrait être pratique de…

Il est assez facile de tomber dans le piège de modéliser et d’inclure dans notre code (dans nos classes, nos types, etc) des concepts qui ne sont pas nécessaires pour prendre des décisions métiers. Parce qu’on trouve ça élégant ou parce qu’on risque d’en avoir besoin plus tard. A éviter de manière générale, mais dans nos agrégats en particulier.

Pour bien comprendre ce piège du modélisme avec des exemples, je vous recommande l’excellent talk de l’ami [**Grégory Weinbach**](https://x.com/gweinbach) : [**il n’y a pas de bon modèle métier**](https://www.youtube.com/watch?v=qN43Dy6fGkk) (un des tous meilleurs talks sur le DDD en passant), ou de relire la partie **« Avoiding the « Model the Real-World » Fallacy** de [**l’excellent livre de Scott Millett & Nick Tune**](https://www.amazon.fr/Patterns-Principles-Practices-Domain-Driven-Design/dp/1118714709).

La parade au modélisme ? Se forcer à **concevoir ou utiliser des modèles** (i.e. du code) **utiles pour des usages spécifiques et contextualisés** (au sens Bounded Contexts).

## #3: Embarquer trop de choses inutiles dans son agrégat

C’est une variante de la précédente idée reçue. Pour rappel, le pattern « Agrégat » vise à garantir la cohérence métier d’un groupe d’éléments qui sont associés et doivent évoluer ensembles. Et “évoluer” est clé ici. L’agrégat est une technique pour gérer des changements dans le code et faire des modifications cohérentes sur des ensembles d’éléments. Elle vise à éviter qu’on se fasse avoir par une mauvaise distribution de la logique métier dans le code (bypassant celle-ci dans certains cas), ou des problèmes d’accès concurrents; pas pour être exhaustif et inclure des détails inutiles pour ça.

J’espère que mon client a vraiment besoin de ces 15 paniers, parce que sinon…

N’incluez donc dans vos agrégats **que les données importantes pour prendre des décisions et garantir vos invariants (i.e. règles) métiers**. **Ni plus, ni moins.**

Vous vous éviterez ainsi des problèmes de perfs, de possibles conflits de parallélisation, ou de la surcharge cognitive inutiles.

En d’autres termes: **pensez « cas d’usage » (use case) quand vous modélisez un Agrégat** (ne vous projetez pas sur votre possible modèle de lecture). Et évitez surtout de tomber dans le piège d’avoir une vision trop orientée “donnée”.

En définitive, le concept d’agrégat doit rester un outil à notre service. Pas une technique obligatoire qui viendrait nous détourner de l’essentiel.