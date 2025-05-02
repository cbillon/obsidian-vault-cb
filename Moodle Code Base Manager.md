---
tags:
  - gitops
  - moodlemoot
---

## Moodle un LMS avec un ecosystem riche

Moodle est un (LMS) open source largement adopté, connu pour sa flexibilité, ses nombreuses options de personnalisation et son importante communauté.
Les fonctionnalités de Moodle peuvent être étendus par des plugins.
Nous avons pres de 2500 plugins dans 40 categories différentes, référencés dans le [répertoire officiel](https://moodle.org/plugins/) développés par la communauté.
Il n'est pas rare de trouver des sites avec plusieurs dizaines,  voire une centaine de plugins installés.
## Mais..

Il manque un outil pour installer une liste de plugins, et installer de nouvelles versions de Moodle.
Il est possible d'installer des plugins de façon unitaire (interface graphique ou en ligne de commande ), mais il n'y a pas d'outils pour installer une base de code avec des dizaines de plugins.
Autre point : Moodle publient régulièrement des mises à jour (corrections, nouvelles fonctionnalités, correctifs de sécurité). Le process de mise à niveau de Moodle n'est pas des plus simple, ce qui décourage nombre d'entre nous de toujours utiliser les dernières versions: les correctifs de sécurité ne sont pas installés ou installés avec retard.
En réalité, une mise à niveau effectuée trop rapidement ou sans vérification approfondie de la compatibilité des plugins peut parfois s'avérer plus problématique que bénéfique.

Les mises à jour de Moodle utilisant des sources sous format .zip  conduisent à accumuler des mises à jour sur une version existante; ceci est problématique :
- difficile de restorer un version antérieure
- par de vue claire sur l'historique des mises à jour

en résumé :
- installation des plugins : tâche fastidieuse, coûteuse en temps
-  une configuration de site difficilement reproductible
-  la mise à jour d'une nouvelle version de Moodle loin d'être simple

## La proposition : 

Moodle Code Base Manager est un script bash qui permet d'installer une liste de plugins et d'installer de nouvelles versions de Moodle.

Le but est d'automatiser l'ensemble du process :
- génération d'une nouvelle base de code (build)
-  test avant livraison manuel et/ou automatisé
- déploiement sur différents environnements (run)

Le projet a été développé ePourquoi ?:n suivant 4 principes :

### . 1 Séparer la génération de la base de code de son environnement de déploiement

Pourquoi ?:

- une même base de code doit pouvoir être déployée dans différents environnements : local, pre-prod, prod,...
-  avec des environnements très différents :
  -  au niveau des matériels 
  - de l'infrastructure: serveur bare metal, conteneurs docker, ...
  - de la configuration logicielle (os, logiciels base de données, librairies de test, ...)

Les éléments de configuration seront injectés dans la base de code au moment du déploiement. 
### . 2 git comme unique unique de vérité

Tous les sources seront versionnés sous git:

- le fichier de configuration
- Moodle
- Plugins
- La base de code produite 

dans un but de simplification, les fonctions git sub modules ne seront pas utilisés.

Pourquoi ?

Le versionnage des sources apporte de nombreux avantages notamment:

- une traçabilité de toutes les modifications
- la possibilité de restorer un état antérieur 
- utilisation d'un cache en local  des sources (moodle, plugins) :
  - une amélioration des performances lors des traitements
  - factorisation des sources : permet d'éviter de multiplier les copie de code lors la gestion de plusieurs instances 

### . 3 un fichier unique de configuration

ce fichier donne l'état attendu de la base de code
cet état est défini par :
- la version de Moodle
- la version des plugins

Pourquoi ?

Cela simplifie le fonctionnement : le role de l'outil est de réconcilier l'état attendu de la base de code avec celui observé (fonctionnement en mode déclaratif).
### . 4 un résultat versionné et immuable

Une nouvelle version complète de la base de code est générée  à chaque fois itération.

Pourquoi ? :

Ce résultat est versionné sous git cela permet à tout moment de restorer un état antérieur

## En conclusion

L'automatisation des tâches d'installation et de maintenance offre de nombreux avantages :

- Réduction des erreurs, risques et coûts associés aux tâches et processus manuels
- Possibilité de revenir à un état antérieur
- Diminution des délais : des opérations de mise à jour qui se chiffraient en heures se mesurent en minutes
- Documentation automatique de la configuration installée









