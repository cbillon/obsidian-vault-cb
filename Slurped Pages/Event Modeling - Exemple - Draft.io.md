---
link: https://draft.io/fr/example/event-modeling
byline: |-
  Emmanuel Lehmann
  						
  							Consultant Software Craftsmanship & Agile chez Axa
excerpt: L’Event Modeling est une méthode utilisée pour modéliser le fonctionnement d’un système d’information sous l’angle des interactions.
slurped: 2025-04-12T16:57
title: Event Modeling - Exemple - Draft.io
tags:
  - event-modeling
---

## Qu’est-ce que l’Event Modeling ?

L’Event Modeling est une méthode utilisée pour modéliser le fonctionnement d’un système d’information sous l’angle des interactions que celui-ci entretient avec des utilisateurs ou d’autres systèmes. L’outil fut introduit en 2019 par Adam Dymitruk, expert en développement logiciel et en Behavior-Driven Development, avec comme objectif d’apporter une vision globale et multidimensionnelle d’un système d’information, et de rendre celui-ci plus prévisible, plus flexible et plus sûr.

Plus précisément, l’Event Modeling s’articule autour d’une suite d’événements, mise en lien avec des commandes, des processus et des interfaces d’interaction pour les utilisateurs. Pour matérialiser ces interactions, la méthode fait intervenir du wireframing qui nourrit la conception des commandes et des vues avec lesquelles les différentes catégories de personas sont amenées à interagir.

## Comment se déroule un atelier d’Event Modeling ?

Avant de démarrer un atelier d’Event Modeling, il est nécessaire d’effectuer quelques travaux au préalable :

- Documenter les [Personas](https://draft.io/fr/example/persona) impliqués ;
- Recueillir leurs attentes et les irritants ;
- Cartographier le parcours des utilisateurs dans le système, par exemple à l’aide d’un [User Story Mapping](https://draft.io/fr/example/user-story-mapping) high-level ; et
- Définir les objectifs métier et indicateurs qui seront utilisés pour mesurer l’atteinte de ces objectifs.

En s’appuyant sur les informations collectées en amont, un atelier classique d’Event Modeling se déroule en sept étapes :

1. #### 1. Lister les événements
    
    La première étape consiste à brainstormer pour lister tous les événements impliqués dans le système dont on souhaite décrire le fonctionnement visé ;
    
2. #### 2. Définir la chronologie des événements
    
    Une fois cette liste établie, il s’agit alors de trier ces événements par ordre chronologique et de les placer sur un axe horizontal. Cette frise constitue la colonne vertébrale de la modélisation ;
    
3. #### 3. Dessiner les wireframes
    
    Ensuite, il faut dessiner, sous forme de wireframes ou de simples mockups, les interfaces avec lesquelles les utilisateurs interagissent, que ce soit pour entrer des données dans le système ou simplement consulter des informations. A noter qu’il est également nécessaire de représenter les processus qui doivent être exécutés par le système, en particulier dans le cas où ceux-ci font appel à des systèmes externes qui traitent et renvoient des données.
    
4. #### 4. Identifier les commandes
    
    Cette quatrième étape consiste à écrire précisément les données ou changements d’état que l’utilisateur est amené à entrer dans le système. Chaque commande doit être reliée à une interface.
    
5. #### 5. Définir les vues
    
    La cinquième étape est similaire est la précédente, sauf qu’ici il s’agit de décrire les vues qui doivent être apportées aux utilisateurs. Au même titre que les commandes, chaque vue doit être reliée à une interface.
    
6. #### 6. Classer les événements et les wireframes
    
    Un système fait généralement intervenir plusieurs types de personas ainsi que des domaines ou d’autres systèmes. Créer des lignes de nage et nommer explicitement les catégories impliquées permet de distribuer les événements et les wireframes dans l’espace et d’identifier plus facilement les scénarios et éventuelles dépendances.
    
7. #### 7. Identifier les scénarios et fonctionnalités à implémenter
    
    Une fois la modélisation terminée, en regroupant un ensemble d’événements, de commandes et de vues, il ne reste qu’à identifier les fonctionnalités et user stories à développer. Celles-ci viennent directement nourrir votre backlog produit.
    

## Quelques conseils d’un professionnel du Software Craftsmanship et de l’Agilité

Emmanuel Lehmann, Consultant Software Craftsmanship & Agile chez Axa, connaît bien l’Event Modeling pour avoir eu l’opportunité de le mettre en application à plusieurs reprises dans son organisation. Il a pu dégager de son expérience un certain nombre de recommandations :

- Utiliser un User Story Mapping en amont et en aval de l’atelier d’Event Modeling. En amont, pour esquisser le flux narratif du système avec les métiers. En aval, pour prioriser les user stories avec une logique de MVP ;
- Ne pas se perdre dans les détails en essayant de définir les règles associées aux différents scénarios. Ce travail peut parfaitement être effectué ultérieurement, par exemple avec des ateliers The Three Amigos d’[Example Mapping](https://draft.io/fr/example/example-mapping) ;
- Au cours de l’étape 6 (Classer les événements et les wireframes), mettre en évidence les événements qui font appel à des systèmes tiers pour identifier les éventuelles dépendances et anticiper le travail qui pourrait être nécessaire d’effectuer, en parallèle, par d’autres équipes. L’identification des dépendances offre également l’opportunité d’aborder la résilience du système dans son ensemble (“Resilience by Design”) ;
- Se servir de l’Event Modeling comme source d’information commune entre les équipes développement et les UX/UI Designers, notamment pour assurer l’alignement entre eux et éviter de solliciter inutilement les métiers à plusieurs reprises pour discuter des mêmes sujets.