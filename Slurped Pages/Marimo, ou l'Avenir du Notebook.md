---
link: https://blog.ippon.fr/2024/06/26/marimo-ou-l-avenir-du-notebook/
byline: Sébastien VEY
site: Blog Ippon - Ippon Technologies
date: 2024-06-26T18:00
excerpt: Marimo est le Notebook Python qui corrige les défauts de Jupyter et apporte reproductibilité, facilité de maintenance et de déploiement.
twitter: https://twitter.com/@ippontech
tags:
  - marimo
slurped: 2025-05-17T09:21
title: Marimo, ou l'Avenir du Notebook
---

_Alerte_ : sous peu, les Jupyter Notebooks Python pourraient bien disparaître. Merci à eux pour tout ce qu’ils nous ont apporté, nous, Data Engineers, Data Scientists et autres. Mais la relève est là et elle n’amène que du bon : Marimo !

Pas de panique. Marimo, c’est du notebook, et on convertit un notebook Jupyter en notebook Marimo sans difficulté. Mais alors, pourquoi Marimo ?

## Pourquoi Marimo ?

On ne présente plus les notebooks Jupyter, environnements de développement interactifs qui allient le code au visuel.

Mais qui n’a jamais été freiné par ses défauts :

- **non reproductible** : état du _notebook_ dépendant de l’ordre d'exécution des cellules - qui n’a jamais lancé un _notebook_ qui marchait la veille et s’est retrouvé avec un bon gros message rouge ?
- **difficile à maintenir** : bien loin du pur Python, versionable et portable,
- **pas fait pour être déployé** : qui n’a jamais voulu utiliser un _notebook_ en application web ou comme script ?

Marimo m'a convaincu car il corrige ces défauts et apporte bien plus. En se basant sur le principe du DAG (Directed Acyclic Graph), il définit les dépendances entre chaque cellule et génère un _notebook_ reproductible. Cette base solide lui permet de rendre ce _notebook_ exécutable et déployable. En fournissant nativement les éléments nécessaires pour construire une interface, il rend le _notebook réactif et interactif._ Avec Marimo, vous avez :

- un _notebook_ **reproductible**, que vous pourrez rouvrir demain sans surprise. Il n’y a pas d’état caché, il marchait, il marche,
- un fichier source pur Python, formaté par [_black_](https://github.com/psf/black?ref=blog.ippon.fr), facilement **lisible**, facilement **versionnable,**
- un _notebook_ **déployable** en page web interactive ou script,
- une **interface de développement vraiment aboutie** avec, entre autres, panneaux de logs et graphe des dépendances.

Dans cet article, je vous présente les détails du fondement de Marimo, la manière dont il définit le DAG. Je mets ensuite l'outil à l'épreuve sur un cas d'usage et vous livre mes impressions.

## La Naissance de Marimo

Marimo a été initié par [Akshay Agrawal](https://x.com/akshaykagrawal?ref=blog.ippon.fr) qui, lors de son PhD orienté Machine Learning à Stanford, s’est senti frustré par son utilisation quotidienne de Jupyter. Après un [passage par Google Brain bien formateur](https://www.debugmind.com/2020/01/04/paths-to-the-future-a-year-at-google-brain/?ref=blog.ippon.fr), Akshay s’est convaincu qu’il fallait repenser le _notebook_.

Marimo est jeune, il souffle sa première bougie sous peu, mais coupe déjà le souffle.

## L'Esprit du DAG

Le principe de Marimo est de définir un DAG liant chaque cellule de notre _notebook_. Nous créons et éditons nos cellules depuis l'interface. En arrière plan, Marimo reporte le code de chaque cellule dans une fonction spéciale :

- les variables lues dans la cellule sont définies en paramètres d'entrée de la fonction,
- les variables assignées dans la cellule font partie du résultat renvoyé par la fonction.

Le code de ces fonctions est dans un premier temps analysé par Marimo, qui construit le diagramme en reliant les paramètres d'entrée d'une fonction aux fonctions qui renvoient ces paramètres.

Prenons un cas simple pour illustrer ce mécanisme. Trois cellules sont définies. La figure 1 présente la définition des cellules en partie droite et le DAG résultant en partie gauche. La figure 2 montre le code source des fonctions générées pour ces trois cellules :

![](http://blog.ippon.fr/content/images/2024/07/cells-and-dependencies.png)

__Figure 1 : Dépendances (DAG) et définitions des cellules depuis l'interface Marimo__

![](http://blog.ippon.fr/content/images/2024/07/corresponding-code.png)

__Figure 2 : Définitions des fonctions représentant les cellules dans le fichier source__

Le respect des contraintes suivantes est contrôlé lorsque l'on édite le notebook, afin de conserver un DAG valide :

- une variable ne doit être assignée que dans une seule cellule. D'autres cellules peuvent dépendre de cette variable, son assignation doit être unique,
- les cellules ne doivent pas créer de cycle (cellule 1 : `x = y`/ cellule 2 : `y = x`).

Au démarrage, Marimo commence par exécuter les cellules n'ayant pas de dépendances, puis les cellules dont les dépendances ont été assignées par les cellules précédemment exécutées.

Marimo exécute de nouveau une cellule lorsque :

- une de ses dépendances est de nouveau assignée par une cellule parent,
- un objet d'interface dont la cellule dépend a muté (par exemple un _slider_ `mo.ui.slider(...)`d'une cellule parent a changé de valeur).

Attention, hors éléments de l'interface, Marimo ne ré-exécute pas une cellule lorsqu'une dépendance a muté (eg. un _append_ sur une liste) :  
→ on utilise donc le paradigme fonctionnel, on assigne mais on ne modifie pas,  
→ si l'on a vraiment besoin de gérer des états qui mutent, on creuse. La documentation nous oriente sur le guide [_reactive state_](https://docs.marimo.io/guides/state.html?ref=blog.ippon.fr).

## **Mise à l'épreuve**

![](http://blog.ippon.fr/content/images/2024/07/usecases.png)

__Figure 3 : Exemples d'utilisation, directement tirés du github de Marimo__

Une fois le principe compris, comme les _notebooks Jupyter_, les possibilités sont multiples. Libre à vous de les utiliser pour tous les usages possibles et imaginables. Les cas d’usage classiques sont l’exploration de données et de la construction d'interfaces interactives. Marimo présente un grand nombre d'exemple sur sa page [use cases](https://marimo.io/use-cases?ref=blog.ippon.fr), dont sont tirées les deux images de la figure 3 ci-dessus.

Data Engineer au quotidien, donc moins orienté Data Science et Visualisation, j’ai éprouvé l'outil au travers d'un cas que j’avais en tête : une interface de dépôt de fichiers.

Chez certains clients, nous avons parfois besoin d’ingérer des canards boiteux, fichiers produits manuellement ou récupérés directement par les métiers, donc sujets à problèmes et qui méritent amplement d’être contrôlés avant l'ingestion. Une petite interface web accessible aux métiers s’y prête bien :

- l’utilisateur dépose un fichier à ingérer,
- une batterie de contrôles de conformité est jouée,
- si le fichier est valide, il est déposé sur AWS S3,
- un rapport est affiché à l’utilisateur.

Au programme :

- de l’UI pas chère, merci Marimo,
- pour les contrôles, un module en arrière-plan qui utilise notamment [_polars_](https://pola.rs/?ref=blog.ippon.fr), qui peut être utile pour des contrôles plus poussés,
- [_Poetry_](https://python-poetry.org/?ref=blog.ippon.fr) pour gérer le projet,
- un déploiement Docker.

⇒ Marimo m’a permis de faire une petite interface utilisateur à peu de frais.

Le code est disponible sur [ce repository github](https://github.com/sebvey/marimo-dataentry?ref=blog.ippon.fr).

Avec le fichier pas du tout valide qu’a voulu me déposer le métier :

![](http://blog.ippon.fr/content/images/2024/07/mon-use-case-1.png)

__Figure 4 : Cas d'usage Data Entry : avec échec des contrôles__

Heureusement, mon utilisateur voit le résultat des contrôles, corrige, recharge un fichier valide (et doux espoir : je n’en entends pas parler) :

![](http://blog.ippon.fr/content/images/2024/07/mon-use-case-2.png)

__Figure 5 : Cas d'usage Data Entry : avec réussite des contrôles__

## **Prise en Main**

Je ne ferai pas mieux que Marimo qui met à disposition tout ce qu’il faut pour apprendre vite et bien, ci-dessous de quoi vous orienter pour vos débuts :

- on peut commencer doucement mais surement avec le guide [Coming from Jupyter Notebook](https://docs.marimo.io/guides/coming_from_jupyter.html?ref=blog.ippon.fr) qui pose bien les bases,
- un _playground_ est disponible sur [marimo.app](https://www.notion.so/4b754dacc5da413ab91e875f8bd4a38b?pvs=21&ref=blog.ippon.fr), avec une dizaine de tutoriels et des exemples,
- pour les convaincus, on passe au [getting started](https://docs.marimo.io/getting_started/index.html?ref=blog.ippon.fr) de la documentation pour installer Marimo en local,
- les [guides](https://docs.marimo.io/guides/index.html?ref=blog.ippon.fr) sont d'une grande utilité, présentant entre autres les bonnes pratiques, les interactions avec les graphiques et tableaux de données (dataframes) ainsi que les déploiements,
- les [recettes](https://docs.marimo.io/recipes.html?ref=blog.ippon.fr), multiples de petits cas d’usage sont très utiles, notamment la partie [control flow](https://docs.marimo.io/recipes.html?ref=blog.ippon.fr#control-flow).

## **Pros and Cons**

## **Pros**

Vous l’aurez compris mais je refais une courte liste : _notebook_ reproductible, compatible git, déployable en application web, une belle interface de développement.

Je rajouterai un point : Marimo peut charger automatiquement les modules en cas de changement. Ceci couplé à un projet géré par poetry qui installe nos modules en mode éditable, c’est tellement pratique : on développe des modules dans l’IDE pour ne pas charger le notebook, du côté Marimo la mise à jour est faite de manière transparente.

## **Neutral**

- Un temps de montée en compétence sur les bonnes pratiques (mais le retour sur investissement est bon),
- La documentation est vraiment bien faite mais il faudra fouiller un peu : Marimo est jeune et pour l’heure, il faut oublier Google, Stackoverflow ou autres, on tombe vite sur des images d’étranges organismes aquatiques.

## **Cons (mais ça peut changer vite)**

- Le développement se fait dans l’interface web, un _keymap_ Vim est disponible mais rien d’autre. Il y a une extension VS Code, mais elle se limite pour l'instant à lancer l’interface dans un onglet de l'IDE,  
    ⇒ on est parfois frustré de ne pas pouvoir faire du multi curseur, des nouveaux raccourcis clavier ou de devoir basculer entre Marimo et notre IDE pour développer les modules utilisés en arrière-plan.
- Jupyter est un écosystème qui va bien au-delà de Python et qui a une longue vie devant lui. On ne fera sûrement jamais tourner un noyau Scala/Spark sur Marimo (mais qui le voudrait ?),
- Le logo est rond, alors que Marimo fait des DAGs. Et ça c’est dur. 😡

## Aller Plus Loin

Pour les convaincus, je conseille de prendre le temps de lire le post d’Akshay Agrawal, à l’origine de Marimo : [Lessons learned reinventing the Python notebook](https://marimo.io/blog/lessons-learned?ref=blog.ippon.fr)**.** La genèse du projet est détaillée ainsi que les différents choix d'architecture qui ce sont présentés et les choix qui ont été faits. La lecture est instructive.

Il reste à noter différentes fonctionnalités qui m'ont paru intéressantes mais que je n'ai pas testé :

- les [intégrations déjà existantes](https://docs.marimo.io/integrations/index.html?ref=blog.ippon.fr), dont Google (sheets / Google Cloud BigQuery / Google Cloud Storage),
- l’intégration dans l’interface de développement de l’[IA Copilot](https://docs.marimo.io/getting_started/index.html?ref=blog.ippon.fr#github-copilot),
- l'intégration de Marimo dans une application plus large, l'exemple d'une intégration dans une application FastAPI étant présenté dans [ce guide](https://docs.marimo.io/guides/deploying/programmatically.html?ref=blog.ippon.fr).

C’est maintenant à votre tour de prendre en main Marimo, si ce n’est pas déjà fait. Bons développements !