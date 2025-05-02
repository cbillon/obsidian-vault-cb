---
tags:
  - moodle
  - codebase
  - moodlemoot
  - gitops
---
On parle beaucoup de la digitalisation de la formation, en faisant référence à la production de contenus; mais il y aussi le volet Administration de la plate forme.

Les plugins Moodle sont des composants que vous pouvez installer sur votre plate-forme Moodle pour ajouter une fonctionnalité, une fonctionnalité ou une apparence spécifique à votre site Moodle.
Moodle a une communauté tres active , nous avons pres de 2500 plugins de 30 categories différentes dans le [répertoire officiel](https://moodle.org/plugins/).
## Mais pas d'outils d'installation d'une liste de plugins

Il n'est pas rare de trouver des sites avec plusieurs dizaines voire une centaine de plugins installés et ce sans outil natif pour effectuer cette operation.

## ni pour les mises à jour

Une fous les plugins installés, la mise à jour d'une nouvelle version de Moodle s’avère délicate.
Moodle publient régulièrement des mises à jour (correctifs de sécurité) mais ceux tardent à être installés ,les mises à jour étant longues et non exemptes de risques d'erreurs; 
À chaque nouvelle version, nous devons nous assurer que nos plugins essentiels continuent de fonctionner correctement. Malheureusement, le processus de mise à niveau de Moodle n'est pas des plus simples, ce qui décourage nombre d'entre nous de toujours utiliser les dernières versions. En réalité, une mise à niveau effectuée trop rapidement ou sans vérification approfondie de la compatibilité des plugins peut parfois s'avérer plus problématique que bénéfique.

Automatiser l'administration de la plate forme tache indispensable mais qui est souvent chronophage est un des principaux leviers pour libérer du temps.
Les sources des plugins sont disponibles sous 2 formes :
- les fichiers compressés au format zip
-  un dépôt git

L'installation des plugins avec des fichiers .zip  après avoir été créés, conduit à des sites transformés au fil du temps, lors de mises à jour , de l'installation d'un nouveau plugin;
Il en résulte qu’une **infrastructure est difficilement duplicable ou ré-instanciable** (dans le cas d’une grosse panne par exemple).
Cette approche rend difficile  de retrouver un état antérieur, d'avoir une Traçabilité des mises à jour.
## Un outil pour gérer la base de code

Nous avons besoin d'un outil pour couvrir l'ensemble du cycle de vie :
- installation de plugins
- mise à jour version Moodle
- test 
- ...
- 

L'outil a été développé en suivant 4 principes
### Principe 1 : Git est utilisé pour gérer tous les sources:
  - le fichier de configuration
  - Moodle
  - plugins
  - la base de code produite 

### Principe 2 :  mode déclaratif avec 1 fichier unique de configuration

Le mode déclaratif consiste à préciser l'état demandé, l'outil produisant une base de code à l'état demandé.
Le fichier de configuration est simple

L'outil gère les dépendances des composants en recherchant une version de plugin compatible avec la version de Moodle

### Principe 3:  la base de code est versionnée et immutable

La version de la base de code est gérée par git et représente l'état observé.

### Principe 4:  Reconciliation automatique état demandé état observé

Plusieurs événements modifient l'état demandé de la base de code
- nouvelle version de Moodle
- nouvelle version des plugins
- ajout/suppression de plugins 
-  ...
Cette approche repose sur la **configuration déclarative**. Le fichier de configuration de vos dépôts doivent décrire ce que vous souhaitez déployer, et non comment. Cela facilite la compréhension, la maintenance et l'audit de vos configurations.
L'outil compare en permanence et automatiquement l'état réel d'un système à son état souhaité. Si, pour une raison quelconque, les états réel et souhaité diffèrent, des actions automatisées sont lancées pour les rapprocher.
## Une approche simple, mais pas simpliste

Un seul fichier de configuration
## Utilisation de git des bénéfices immédiats

on **stocke tous les fichiers sources de notre état recherché dans Git**.

Les **bénéfices** sont immédiats :

- Tant d’un point de vue traçabilité
    - Historique des modifications
    - 
    
- … que d’un point de vue méthodologie
    - restorer un état précédent
    - gestion de plusieurs instances de code 

L'outil maintient une copie local des différents dépôts (cache local) avec comme avantages:
- de meilleures performances
-  factorisation des sources présents dans plusieurs instances

