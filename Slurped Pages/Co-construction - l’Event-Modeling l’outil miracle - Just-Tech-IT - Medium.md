---
link: https://medium.com/just-tech-it-now/co-construction-levent-modeling-l-outil-miracle-f3ba181755b7
byline: Benjamin Seillier
site: Just-Tech-IT
date: 2022-07-07T14:21
excerpt: "Sur ce retour d’expérience, l’équipe doit traiter une fonctionnalité très attendue : « L’upload d’un document lors d’une demande de remboursement Santé. » Pour se remettre dans le contexte de…"
twitter: https://twitter.com/@AXAjobs_fr
slurped: 2025-04-12T16:54
title: "Co-construction : l’Event-Modeling l’outil miracle - Just-Tech-IT - Medium"
tags:
  - co-construction
  - event-modeling
---

## **Co-construction : l’**Event-Modeling **l’outil miracle**


Sur ce retour d’expérience, l’équipe doit traiter une fonctionnalité très attendue : « **L’upload d’un document lors d’une demande de remboursement Santé.** »

Pour se remettre dans le contexte de l’[article précédent](https://medium.com/just-tech-it-now/4-mai-2025-breaking-news-mutation-agile-axa-mod%C3%A8le-%C3%A0-suivre-dff66c302bb5), nous sommes donc lundi matin et la livraison est prévue vendredi à 14 heures. Nous rappelons, les dates sont fixées, et l’équipe est constituée… Le top départ est donné.

## 1ère étape : Comprendre le besoin

Depuis plus d’un an, nous avons systématisé l’utilisation de l’[**_Event-Modeling_**](https://eventmodeling.org/posts/what-is-event-modeling/). Un puissant outil qui s’est imposé très facilement, car disons-nous le franchement… avant il n’y avait rien, ou si au contraire, il y avait “des trucs”, chaque équipe avait son fonctionnement : des fichiers Words, des cahiers des charges, des réunions, “volatiles” ou au mieux avec quelques comptes-rendus, et pour les plus techniques, des diagrammes UML ou de séquence… De toute manière, c’était rarement partagé, rarement lu, rarement compris…

L’**_Event-Modeling_** a donc pour objectif de **formaliser collectivement le besoin**. « Collectivement » représente tous les acteurs du projet :

- l’équipe Métier, celui qui exprime le besoin,
- l’équipe Produit (le Product Owner, les Business Analystes… ),
- l’équipe de développement (l’équipe UX/UI, les développeurs, les testeurs, les OPS…)
- Les architectes, les consultant SSI…

Donc de manière générale, toutes les personnes susceptibles de poser des questions et/ou de donner des réponses.

Nous nous réunissons donc armés de post-its autour d’un tableau blanc. Pour les sessions à distance, nous utilisons l’outil [draft.io](http://draft.io/).

Nous nous donnons une heure 🕑. **Pendant 5 à 10 minutes, le métier exprime son besoin**, les autres sont en charge de **remplir du Post-It**. Puis 30 min pour échanger et réorganiser.

Voici le résultat :

Event-Modeling : Demande de remboursement

L’équipe est aguerrie à cette pratique, au début il faut peut-être plus de temps, mais la courbe d’apprentissage est très rapide. Si vous voulez essayer, [Emmanuel](https://www.linkedin.com/in/emmanuel-lehmann-1340033b/) vous donne un déroulé clé en main 🔑 : suivez [le guide](https://draft.io/fr/example/event-modeling).

**Les forces de cet outil :**

- Prenez 2 minutes pour lire le résultat. C’est magique 🪄, non ? Vous n’étiez pas à la séance, vous ne connaissiez pas l’outil, mais nous sommes certains que vous avez compris le besoin exprimé.
- L’appropriation est fine et collective… Eh oui, car nous avons **tous** mis “la main à la pâte” et “participé aux échanges”. Le livrable est très complet : personas, dépendances aux autres systèmes, grandes étapes etc… Nous utilisons et faisons vivre **cette documentation**. Elle est précieuse dans de nombreux ateliers par exemple pour discuter de résilience, de sécurité, de l’UX/UI _…_

_Vous avez un doute sur la sécurité d’un processus ? Prenez votre_ **_Event-Modeling_** _sous le bras, votre correspondant SSI sera mis dans le contexte en 2 min._

Voici quelques exemples d’utilisation de ce livrable :

Réflexion sur la résilience

Réflexion UX & UI

Bon c’est pas tout ça, il nous reste 20 minutes avant de commencer les développements. Commençons à identifier quelques User-Stories (US). Toute l’équipe a déjà commencé à en “parsemer”🧀 l’**_Event-Modeling_**.

Atelier découpage

## 2ème étape : Découpage itératif

Maintenant que le besoin est clair pour tous les acteurs, nous allons construire nos différentes itérations. Posons-nous d’abord la question :

> “Comment y répondre le plus rapidement et simplement possible ? ”

C’est le moment où l’équipe se met en mode : « **Il nous reste 4 jours pour livrer cette fonctionnalité ».**

Pour nous aider, nous utilisons ce tableau où nous élaborons et illustrons nos différentes itérations.

Itérations

Sachant, que comme le décrit [un article précédent](https://medium.com/just-tech-it-now/l%C3%A9tat-d-esprit-agile-plus-que-la-m%C3%A9thode-agile-aef06d20a6cb), chaque US doit être fonctionnelle, démontrable, “ prodable”, apporter le plus possible de Valeur Business, rassurer techniquement, pouvoir être potentiellement la dernière etc..

Il est important de donner un objectif 🎯 aux itérations pour donner du sens à toute l’équipe.

Voilà le découpage est terminé. :

Itérations avec les 🎯

Nous savons grâce à la vélocité de l’équipe qu’une ou deux de ces itérations seraient largement faisables d’ici vendredi… L’équipe a une vélocité moyenne de 10 US par semaine. Les 2 premières US de la première itération peuvent commencer. La bande passante restante peut-être affectée aux autres fonctionnalités attendues sur cette version.

**Avons-nous oublié quelque chose ?**

Si vous n’êtes pas familiarisés avec l’AGILITE, vous êtes peut-être tombés de votre chaise🪑… Vous vous dites, mais comment ? Comment peuvent-ils prétendre s’engager ou même commencer à développer quelques choses alors qu’ils n’ont rien « spécifié » : nos US comportent à peine un titre, elles ne sont même pas estimées 🧐.

**Pourquoi n’avons-nous pas besoin d’estimer, et pourquoi un simple titre nous suffit pour développer ?**

N’oubliez pas que l’AGILITE nécessite d’inverser complètement les valeurs : date **FIXE**, ressource **FIXE** et scope **VARIABLE**. Nous le répétons constamment : c’est aussi facile à dire que difficile à intégrer.

Au début de sa collaboration, l’équipe a créé, formalisé, rédigé une US de référence : référence en terme de contenu, de complexité, et surtout **d’effort** 💪 (de taille)… Cette «US étalon » nous sert pour “calibrer” toutes les autres US.

Durant l’**_Event-Modeling_**, **_nous tamisons le besoin à la maille de notre “US étalon”_**, pour extraire un ensemble d’US uniformes priorisé par la valeur business . Les US représentants des petits quantum d’efforts 💪 qui nous font avancer vers notre cible 🎯.

Aucune US n’est vraiment spécifiée, alors comment l’équipe sait qu’un seul quanta d’efforts 💪 sera suffisant ? Justement, elle ne sait pas mais elle s’y engage. Elle s’engage en disant : “Sur cette US, pour un quanta d’efforts 💪 maitrisé nous allons apporter une réponse”. Et l’équipe ne dépensera **QUE** (et uniquement) ce quanta d’efforts 💪, en se mettant réellement d’accord sur le contenu qu’au moment de la première phase du développement de la carte, l’atelier d’Example Mapping effectué avec les 3 Amigos. Durant cet atelier, l’équipe adaptera donc le contenu de l’US en fonction de l’effort investi : ressource (quanta )**FIXE** et scope(contenu) **VARIABLE .**

L’”US étalon” permet également aux équipes PRODUIT et METIER présentes d’appréhender le résultat attendu. Il est évident que pour une US, le PRODUIT et le METIER savent à quoi s’attendre : Tout le monde 🌍 est aligné sur le résultat à espérer compte tenu de l’effort 💪 investi. Il y a pas d’ambition d’être exhaustif, il y a l’ambition d’être rapide.

> **« Faisons, Testons, Répétons »**
> 
> **à la place de**
> 
> **« Analysons, Rédigeons, Estimons »**

Nous espérons que ce retour d’expérience vous a “hypé” cet outil magique qu’est l’**_Event-Modeling_**.
