---
tags:
  - doc
  - diataxis
---

Utiliser un framework pour écrire de la documentation ? 📖
Dans le contexte d’une boîte qui grandit, de plus en plus de développeuses et développeurs rejoignent l’équipe technique. Très vite, la nécessité d’un vocabulaire commun, et donc d’une solide documentation se fait sentir. Martin Supiot a tenu à compléter cette documentation qui était très faible et nous donne son retour d’expérience sur le sujet.

Il a pris quelques développeurs et ils ont analysé la documentation : un retour surprenant, la documentation n’était pas seulement difficile à utiliser, elle était également… pas simple à écrire ! 😨

Alors c’est quoi leur processus d’écriture ? Ici on ne va parler que documentation technique, ainsi que de documentation informationnelle, donc qui explique ce qui existe actuellement.

Ils utilisent le framework Dàtaxis, un framework pensé pour les lecteurs. Les documents créés s’articulent autour de quatre axes :

Les explications : elles clarifient un sujet, facilitent la compréhension, expliquent un contexte ;
- Les tutoriels : ils permettent l’apprentissage, apportent de la confiance ;
- Le how-to guide : il est axé sur un objectif précis, souvent une action répétitive ;
- Les références : c’est comme le swagger d’une Api. Elles informent sur tous les paramètres qui existent, les options possibles etc.
Comment nommer ces documents ? Le titre doit être simple, clair, efficace. Quand on lit le nom du document, on doit tout de suite comprendre ce qu’il y a dedans. Pour faire simple, les titres peuvent commencer par :

« Pourquoi … » s’il s’agit d’une explication ;
« Comment … » pour les tutoriels et les how-to ;
« Les différents … » lorsqu’on fait des références.

Ce framework offre également la possibilité d’avoir des templates de documentation. Martin nous donne pas mal de conseils pour un documentation de qualité et bien maintenue :

Ajouter des guidelines afin d’aider ses équipes à documenter ;
Savoir à quelle audience on s’adresse : développeur.se.s front ? Back ? Graphistes ? Product Owner ?
Définir des outils (comme Mermaid, ou draw.io) ;
Faire attention au formatage, à la ponctuation ;
Revenir régulièrement sur la documentation pour la garder à jour, surtout les how-to ;
Trouver un relecteur, et définir des mainteneurs pour chaque fichier, qui seront en charge d’effectuer le point précédent par exemple ;
Supprimer les documents qui ne sont plus à jour.