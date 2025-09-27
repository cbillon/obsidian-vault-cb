---
link: https://rodolphe.breard.tf/article/unicode-utf8/
byline: Rodolphe Bréard
site: Accueil
date: 2025-09-03T02:00
excerpt: Depuis quelques années, afin de notamment représenter du texte, nous utilisons quasi-exclusivement les standards Unicode et UTF-8. Cependant, peu de gens connaissent la différence entre les deux ainsi que les subtilités sous-jacentes. Je vous propose donc une petite initiation à cet univers bien trop méconnu.
slurped: 2025-09-13T19:55
title: Unicode et UTF-8
tags:
  - utf-8
  - unicode
---

Depuis quelques années, afin de notamment représenter du texte, nous utilisons quasi-exclusivement les standards Unicode et UTF-8. Cependant, peu de gens connaissent la différence entre les deux ainsi que les subtilités sous-jacentes. Je vous propose donc une petite initiation à cet univers bien trop méconnu.

## Un peu d’histoire [#](https://rodolphe.breard.tf/article/unicode-utf8/#un-peu-dhistoire)

Comme vous le savez, en informatique les données sont stockées dans un format binaire où les bits sont généralement regroupées par groupe de 8 afin de former un byte ou un octet. Il existe des exceptions à ça, mais je ne les aborderai pas ici. Avec ses 8 bits, un byte peut donc contenir 256 valeurs différentes et suivant le contexte dans lequel les données sont utilisées, elles peuvent représenter plusieurs choses, notamment un nombre.

Pour les nombres, on distinguera deux catégories : les nombres signés et les nombres non signés. Commençons par cette seconde catégorie qui est la plus simple : elle représente les nombres de 0 à 255. Les nombres signés, quand à eux, utilisent un bit pour indiquer si le nombre est positif ou négatif, ce qui permet de représenter les nombres de -128 jusqu’à 127. Ces deux représentations sont faites de manière à ce que les nombres de 0 à 127 soient représentés de la même manière.

Afin de représenter du texte, il faut donc définir quelle série de bits est associée à quel caractère, c’est ce que l’on appelle l’encodage. Afin de faire simple, il est possible de stocker un caractère sur un byte. Dans la mesure où nous avons déjà une représentation de nombres pour ce byte, à la place de réfléchir sur la position exacte des bits, nous pouvons plutôt associer un nombre à un caractère. C’est ainsi qu’est né le standard [ASCII](https://fr.wikipedia.org/wiki/American_Standard_Code_for_Information_Interchange) qui associe les nombres de 0 à 127 à des caractères. Ce jeu de caractères contient des caractères affichables (lettres chiffres, symboles de ponctuation, etc.) ainsi que des caractères non-affichables (null, sonnerie, etc.).

Seulement voilà, l’ASCII étant un standard nord-américain, il a été conçu pour représenter les caractères utilisés en anglais et rien d’autre. Si l’on souhaite écrire en français ou en espagnol, il n’est alors pas possible de représenter certains caractères tels que les caractères accentués, et la situation est encore pire pour les langues utilisant un alphabet différent. Face à cette situation, chaque langue a eu droit à son standard personnalisé. Pour la plupart des langues utilisant l’alphabet latin, il a été créé des standards se basant sur l’ASCII et l’étendant en utilisant les possibilités non-utilisées pour représenter les caractères supplémentaires dont on a besoin. C’est ainsi que sont nés les standards [ISO/CEI 8859-1](https://fr.wikipedia.org/wiki/ISO/CEI_8859-1) et [ISO/CEI 8859-15](https://fr.wikipedia.org/wiki/ISO/CEI_8859-15).

## Un standard unique pour les gouverner tous [#](https://rodolphe.breard.tf/article/unicode-utf8/#un-standard-unique-pour-les-gouverner-tous)

Le problème avec le fait que chacun utilise un encodage différent, c’est qu’avec internet qui permet d’échanger des informations, il ne faut pas se tromper. Un exemple type sont les salons IRC dans lesquels tout le monde doit configurer son client afin d’utiliser le même encodage que les autres. Dans les années 90 et 2000 où les encodages pullulaient, il était fréquent de voir des personnes arriver dans un salon et parler avec un encodage différent, ce qui causait des problèmes de compréhension.

Face à cette situation qui embêtait tout le monde et demandait beaucoup d’efforts, il a été décidé de créer un standard unique qui supporterait tous les alphabets de toutes les langues et bien plus encore. C’est ainsi qu’ont notamment été créés Unicode et UTF.

À ce stade, vous devez normalement avoir une question : pourquoi deux standards, Unicode et UTF ? Pourquoi ne pas avoir simplement fait un nouveau standard qui soit un système d’encodage ? Et bien la réponse est tout simplement qu’il existe beaucoup plus de 256 caractères, qu’il n’est donc pas possible de tout représenter sur 8 bits et qu’afin de s’intégrer au mieux dans tous les environnements il a fallu conserver certaines souplesse d’encodage.

Pour faire simple, il s’agit d’une grosse base de données de l’ensemble des caractères possibles. On y retrouve donc l’ensemble des différents alphabets, signes de ponctuation, mais également des symboles divers, les emojis et bien d’autres choses. À chacun de ces caractères est associé un numéro.

Le terme caractère est cependant assez imprécis car il est possible de l’associer à des notions différentes. Avec Unicode, on va donc plutôt parler de « points de code ». Retenez donc qu’Unicode est la base de données qui, à chaque point de code, associe un numéro. Ce numéro est représenté par le préfixe `U+` suivi du numéro en [hexadécimal](https://fr.wikipedia.org/wiki/Syst%C3%A8me_hexad%C3%A9cimal). Ainsi, la lettre `A` capitale se voit attribuer le numéro U+0041 et la lettre `a` minuscule le numéro U+0061.

On confond souvent majuscule et capitale. La majuscule est une notion de grammaire tandis que la capitale est une représentation particulière d’une lettre. Par exemple, la phrase « JE SUIS LÀ. » ne contient qu’une seule majuscule, le `J`, mais est intégralement écrite en lettres capitales. Le fait que l’on représente généralement les majuscules en capitales et le reste en minuscules n’aide pas à faire la distinction.

Notons que cette base de donnée est gérée par un consortium et est évolutive. Régulièrement une nouvelle version est publiée, ce qui ajoute, modifie ou enlève des points de code. Dans sa [version 16.0.0](https://www.unicode.org/versions/Unicode16.0.0/) du 10 septembre 2024, Unicode contient 1 114 112 points de code possibles mais seulement 292 531 sont assignés, dont 154 998 sont associés à ce que l’on peut généralement appeler des caractères (cf. [décompte du nombre de caractères Unicode](https://www.unicode.org/versions/stats/charcountv16_0.html)).

## Universal Character Set Transformation Format [#](https://rodolphe.breard.tf/article/unicode-utf8/#universal-character-set-transformation-format)

Le « Universal Character Set Transformation Format », abrégé UTF, est un standard permettant d’encoder l’ensemble des points de code Unicode. Ce standard définit trois manière de s’y prendre : UTF-8, UTF-16 et UTF-32.

Commençons par UTF-32 qui est la méthode la plus simple des trois. Elle utilise 32 bits (donc 4 bytes), ce que l’on appelle une **unité de point** afin de représenter chaque point de code. Le point de code ayant le plus grand numéro possible étant U+10FFFF, il n’y a besoin que de 21 bits pour le représenter. En utilisant 32 bits par point de code, UTF-32 permet de représenter chaque point de code en une seule unité de point. Si cette méthode est la plus simple, elle a deux désavantages majeurs : elle n’est pas rétro-compatible avec l’ASCII et nécessite 4 fois plus d’espace de stockage que l’ASCII.

Afin de palier à ces deux problèmes, il est possible d’utiliser UTF-8. Afin de représenter l’ensemble des points de code, UTF-8 utilise 1, 2, 3 ou 4 unités de point de 8 bits chacune. Afin d’être rétro-compatible avec ASCII, si le bit de poids fort est à 0 (valeurs de 0 à 127) alors c’est qu’il n’est fait usage que d’une seule unité de bloc. À l’inverse, si le bit de poids fort est à 1, c’est qu’il est fait usage de plusieurs unités de bloc. Le même principe est utilisé pour savoir s’il est fait usage de 2, 3 ou 4 unités de bloc. Avec ce nombre variable d’unité de blocs, il est possible de non seulement être rétro-compatible avec ASCII, mais en plus représenter l’ensemble des points de code en un minimum d’espace. Pour plus de détails sur la manière d’enchaîner plusieurs unités de point, je vous laisse consulter [la page Wikipedia dédiée à UTF-8](https://fr.wikipedia.org/wiki/UTF-8#Description).

Vous pouvez utiliser l’outil en ligne [Unicodey](https://unicodey.com/) afin d’analyser une chaîne de caractère, un point de code particulier ou des données en hexadécimal. Chaque point de code est détaillé ainsi que sa représentation UTF-8.

Et comme vous l’aurez deviné, UTF-16 permet de représenter l’ensemble des points de code en 1 ou 2 unité de bloc de 16 bits. Cette méthode présentant tous les désavantages d’UTF-8 et UTF-32 mais aucun de leurs avantages, elle ne présente pas vraiment d’intérêt. D’ailleurs, la simplicité d’UTF-32 due à l’utilisation d’une seule unité de bloc est son seul avantage, elle ne présente donc pas non plus beaucoup d’intérêt. En conséquent, UTF-16 et UTF-32 ne sont généralement utilisées que dans des contextes techniques très particuliers où il y a des raisons spécifiques poussant à utiliser de telles tailles d’unité de points.

## Les graphèmes [#](https://rodolphe.breard.tf/article/unicode-utf8/#les-graph%c3%a8mes)

Vous pensiez vraiment qu’il était possible de représenter l’ensemble des caractères possibles dans une liste de points de code ? Surprise, ce n’est pas le cas. Afin d’éviter les confusions on ne va d’ailleurs pas parler de caractère mais de graphème, un graphème étant une séquence d’un ou plusieurs points de code.

Il y a de multiples raisons pour regrouper plusieurs points de code en un seul graphème. L’une de ces raisons est l’ajout de signes diacritiques (accents, cédille, etc.) afin de modifier la prononciation. Ainsi, le graphème `à` est obtenu par la combinaison du point de code U+0061 pour la lettre `a` et du point de code U+0300 pour l’accent grave combinatoire (à ne pas confondre avec le point de code U+0060 qui est l’accent grave seul `` ` ``). La représentation UTF-8 de l’accent grave combinatoire étant sur 2 bytes, on obtient donc un total de 3 bytes : `61 CC 80`.

Mais ce n’est pas tout car Unicode intègre tout de même le point de code U+00E0 pour représenter un `à`. En UTF-8 ce point de code se représente avec seulement deux bytes : `C3 A0`. Bref, un unique graphème peut être représenté de plusieurs manières différentes.

S’il n’est possible d’obtenir certains graphèmes qu’en utilisant une combinaison de points de code (par exemple certains emojis), j’ai choisi cet exemple afin de vous parler de la normalisation Unicode. En effet, le choix d’utiliser une forme ou une autre peut varier en fonction du terminal que l’on utilise. Ceci peut poser des problèmes, par exemple dans la saisie d’un mot de passe. Afin de parer à ça, on utilise alors un algorithme de normalisation Unicode qui va tout remettre sous une unique forme. Il existe 4 algorithmes de normalisation : NFD, NFKD, NFC et NFKC. Pour plus de détails dessus, je vous laisse lire les articles Wikipedia sur les [équivalences Unicode](https://fr.wikipedia.org/wiki/%C3%89quivalence_Unicode) ainsi que sur la [normalisation Unicode](https://fr.wikipedia.org/wiki/Normalisation_Unicode).

Il est à noter que certaines personnes abusent de ces signes diacritiques afin d’écrire dans une forme parfois qualifiée de maudite : le [texte Zalgo](https://fr.wikipedia.org/wiki/Texte_Zalgo). Par pitié ne faites pas ça, c’est très mauvais pour l’accessibilité des personnes malvoyantes.

## Les glyphes [#](https://rodolphe.breard.tf/article/unicode-utf8/#les-glyphes)

Maintenant que vous savez ce que sont les points de code, les unités de code et les graphèmes, vous maîtrisez la plupart des notions utiles à la bonne compréhension d’Unicode et UTF-8. Rajoutons-en cependant une dernière, le glyphe.

Dans ce contexte, un glyphe est une image permettant de représenter tout ou partie d’un graphème. Les glyphes se trouvent donc dans les polices de caractères et il est possible de combiner plusieurs glyphes. À noter qu’une police de caractère peut avoir plusieurs glyphes pour un même graphème suivant le style (gras, italique, etc.).

On m’a fait remarquer le cas intéressant des caractères de plusieurs langages asiatiques qui ont été unifiés dans Unicode (projet [Unihan](https://en.wikipedia.org/wiki/Han_unification)). Ainsi, si les points de code sont identiques pour ces langages, les glyphes servant à représenter les graphèmes les utilisant sont différents en fonction du langage. Ceci pose donc problème si le langage du texte n’a pas correctement été défini car pour l’afficher il faut alors faire un choix par défaut qui n’est pas nécessairement le bon.

## Conclusion [#](https://rodolphe.breard.tf/article/unicode-utf8/#conclusion)

Maintenant, vous aussi vous pouvez hausser les épaules et soupirer lorsque quelqu’un vous demande de limiter la longueur d’une entrée textuelle à un certain nombre de caractères. En effet, par nombre de caractères on peut entendre :

- le nombre de bytes ;
- le nombre de points de code ;
- le nombre de graphèmes.

Mais en plus ces trois valeurs peuvent varier en fonction des équivalences Unicode. Bref, la consigne en apparence simple devient soudainement beaucoup plus compliquée qu’il n’y parait.

Afin de ne pas vous laisser avec trop de questions métaphysique sur ce sujet sachez que, s’agissant de la méthode de calcul de la longueur d’un mot de passe afin de savoir s’il respecte les longueurs minimales et maximales, la [publication spéciale 800-63B du NIST](https://pages.nist.gov/800-63-3/sp800-63b.html) préconise de normaliser en utilisant NFKC ou NFKD puis de compter le nombre de points de code. À titre personnel, pour cet usage précis je recommande NFKC.