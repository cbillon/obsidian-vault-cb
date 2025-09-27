---
link: https://rodolphe.breard.tf/article/introduction-aux-cables-usb/
byline: Rodolphe Bréard
site: Accueil
date: 2025-04-07T02:00
excerpt: Si tout le monde connaît déjà plutôt bien les principaux types de cables USB ainsi que leurs connecteurs, rares sont les personnes qui en maîtrisent les subtilités. Ce billet a donc vocation à rappeler les bases, principalement autours des câbles disposant d’un connecteur USB type A, afin de se préparer à un futur billet qui, lui, sera destiné e à la présentation des câbles disposant d’un connecteur USB type C. Compte tenu de la grande complexité des connecteurs de type C, ce billet d’introduction plutôt simple ne me paraît pas être du luxe, et en prime je suis a peu près certain que nous apprendrez tout de même quelque chose.
slurped: 2025-09-13T20:03
title: Introduction aux câbles USB
tags:
  - cable
  - usb
---

Si tout le monde connaît déjà plutôt bien les principaux types de cables USB ainsi que leurs connecteurs, rares sont les personnes qui en maîtrisent les subtilités. Ce billet a donc vocation à rappeler les bases, principalement autours des câbles disposant d’un connecteur USB type A, afin de se préparer à un futur billet qui, lui, sera destiné e à la présentation des câbles disposant d’un connecteur USB type C. Compte tenu de la grande complexité des connecteurs de type C, ce billet d’introduction plutôt simple ne me paraît pas être du luxe, et en prime je suis a peu près certain que nous apprendrez tout de même quelque chose.

## Contexte historique [#](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/#contexte-historique)

Les plus anciens se souviendront qu’autrefois il y avait quasiment un type de connecteur spécifique à chaque périphérique. Ainsi, on branchait le clavier sur le port PS/2 violet, la souris sur le port PS/2 vert, l’écran sur le port VGA, les enceintes sur le port jack vert, le micro sur le port jack rouge, l’imprimante sur le port parallèle, le joystick sur le game port et ainsi de suite. En fait le seul port vaguement destiné à être générique était le port série, également appelé port COM. Bref, vous étiez rapidement limité par les ports disponibles, malheur à vous si vous achetiez un périphérique nécessitant un port que qui n’est pas présent sur votre carte mère.

Face à cette situation peu tenable, un grand nombre de constructeurs importants se sont regroupés en consortium afin de proposer un port unique pour les gouverner tous et c’est ainsi qu’apparu le désormais célèbre port USB. Forcément, USB a mis un peu de temps pour s’imposer et n’est pas totalement universel car certains périphériques, notamment les écrans, disposent encore de ports dédiés.

L’histoire ayant malheureusement tendance à se répéter, ce nouveau standard universel qu’est l’USB a très rapidement donné naissance à une multitude de connecteurs différents. Ainsi, dès sa publication initiale en 1996, le standard USB 1.0 définissait déjà deux types de connecteurs différents, le type A et le type B. Viendront ensuite s’ajouter le type AB ainsi que les variations mini puis micro des 3 types précédemment cités. Enfin, avec l’arrivée de l’USB 3.0 et le besoin de plus de connecteurs afin d’obtenir toujours plus de performances, de nouvelles variations arrivent encore. Fort heureusement, de nos jours, si le type A est toujours très utilisé, c’est le récent type C qui, aidé de la réglementation Européenne, a sonné le glas pour l’ensemble des autres variantes.

Les types B et AB ainsi que toutes les variations mini/micro n’étant en réalité que des types A avec des formes différentes (ou juste avec un pin d’ID en plus), je n’en parlerai pas dans cet article. Je ne parlerai pas non plus du seul connecteur réellement différent, le type C, qui aura droit à un article à part.

## USB 1.1 et 2.0 [#](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/#usb-11-et-20)

La norme USB 1.0 n’ayant jamais adoptée à large échelle, nous pouvons sereinement commencer avec le premier standard qui a véritablement été répandu auprès du grand public, l’USB 1.1. C’est un standard extrêmement simple qui utilise 4 pins et utilise exclusivement une alimentation de 5V et 1,5A. La capacité de transfert de données est de 12 Mbps en « full-speed » et seulement 1,5 Mbps en « low-speed ».

Arrive ensuite la norme USB 2.0, dit « high-speed » qui reprend exactement la même connectique mais permet une transmission des données allant jusqu’à 480 Mbps. La connectique étant vraiment exactement la même, il est possible d’atteindre les de l’USB 2.0 avec un câble initialement conçu pour de l’USB 1.1, avec toutefois une éventuelle limitation liée à la qualité de fabrication du câble. Mais de nos jours tous les câbles, même les plus mauvais, supportent sans soucis USB 2.0.

Physiquement, il s’agit donc de 4 pins disposés les une à côtés des autres sur un support physique qui empêche de se tromper de sens. Ces 4 pins sont :

1. VBUS : alimentation +5 V (fil rouge ou orange)
2. D- : données - (fil blanc ou or)
3. D+ : données + (fil vert)
4. GND : masse (fil noir ou bleu)

![Vue de face du connecteur mâle d’un câble USB 2.0 type A. Les pins sont légendés.](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/usb-a-20.png)

Vue de face du connecteur mâle d’un câble USB 2.0 type A. Les pins sont légendés.

Si la plupart des embouts mâles disposent d’une envelope de protection métallique, certains sont totalement ouverts, laissant ainsi clairement apparaître les 4 pins.

![Vue de face du connecteur mâle d’un câble USB 2.0 type A ne disposant pas d’enveloppe de protection. Les pins sont légendés.](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/usb-a-20-no-enclosure.png)

Vue de face du connecteur mâle d’un câble USB 2.0 type A ne disposant pas d’enveloppe de protection. Les pins sont légendés.

## La « capote » USB [#](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/#la-capote-usb)

Dans les années 2011/2012, un type d’attaque nommée « [juice jacking](https://fr.wikipedia.org/wiki/Juice_jacking) » a fait surface. Cette attaque repose sur le fait que la victime recharge son matériel électronique, typiquement un téléphone, en utilisant un port USB préalablement corrompu par un attaquant. En effet, il existe beaucoup d’infrastructure publiques permettant à n’importe qui de recharger son matériel via USB, on en voit régulièrement dans les gares, des aéroports voir meme des [abribus](https://www.20minutes.fr/nantes/2012891-20170216-nantes-ports-usb-ecrans-tactiles-wifi-abribus-nouvelle-generation-vont-arriver).

Dès 2013 Android et iOS ont déployé des contre-mesures logicielles afin d’empêcher de telles attaques, notamment en demandant à l’utilisateur de confirmer toute connexion autre qu’une recharge. De plus, à ma connaissance aucune attaque réelle de ce genre n’a été repérée, il n’y a eu que des démonstrateurs de construits. Malgré cela, beaucoup de personnes refusent de recharger leur matériel sur un port USB public.

Une solution alternative est d’utiliser un bloqueur de données, également surnommé « [capote USB](https://fr.wikipedia.org/wiki/Condom_USB) ». Le principe de ca matériel est simple, il s’agit d’une extension USB dont les pins de données (D+ et D-) ne sont pas connectés, ce qui ne permet que d’obtenir l’alimentation électrique 5 V grace aux pins VBUS et GND. Soit dit en passant, sachez que la CNIL distribue parfois ce genre de capotes.

![Deux préservatifs USB posés l’un sur l’autre et arborant chacun le logo de la CNIL.](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/capote-usb-cnil.png)

Deux préservatifs USB posés l’un sur l’autre et arborant chacun le logo de la CNIL.

Il est également très simple de construire un tel ustensile en utilisant un câble de rallonge USB type A dont on coupe les fils de données (blanc et vert). Il est alors conseillé, après l’opération, de refermer la plaie à l’aide d’une [gaine thermorétractable](https://fr.wikipedia.org/wiki/Gaine_thermor%C3%A9tractable).

Afin d’illustrer le principe, j’ai pris un vieux câble USB en fin de vie et lui ai appliqué [le traitement royal façon 1793](https://fr.wikipedia.org/wiki/Ex%C3%A9cution_de_Louis_XVI) avant de le brancher et de mesurer le voltage entre les fils VBUS et GND. J’ai bien les 5 V donc c’est bon, je ne vous ai pas raconté de bobards.

![Un câble USB coupé dont sortent 4 fils dénudés : un rouge, un vert, un blanc et un noir. Sur les parties dénudées des fils rouge et noirs sont respectivement posées les bornes rouge et noir d’un multimètre. À côté du câble se trouve un multimètre réglé en mode voltmètre et dont l’écran affiche la valeur 5,02.](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/mesure_usb_voltmetre.png)

Un câble USB coupé dont sortent 4 fils dénudés : un rouge, un vert, un blanc et un noir. Sur les parties dénudées des fils rouge et noirs sont respectivement posées les bornes rouge et noir d’un multimètre. À côté du câble se trouve un multimètre réglé en mode voltmètre et dont l’écran affiche la valeur 5,02.

## USB 3.0 et 3.1 [#](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/#usb-30-et-31)

Si l’USB 2.0 est encore largement répandu grâce à son faible coût et son débit largement suffisante pour certains périphériques, l’augmentation de la tailles des données a nécessité un nouveau standard plus performant. C’est ainsi qu’en 2008 arrive l’USB 3.0, dit « SuperSpeed » ou simplement « SS » qui permet d’atteindre un débit de transfert de données de 5 Gbps.

Afin d’atteindre de telles performances, il a été nécessaire d’utiliser non plus 4 mais 8 pins (9 en réalité) et donc de modifier l’ensemble des connecteurs USB. Si le type A a été adapté avec une rétro-compatibilité dans les deux sens, les types B et l’ensemble des types « micro » ont du être physiquement augmentés. Les types « mini » ont, eux, été abandonnés. Afin de différencier les embouts disposant des nouveaux pins des autres, on les appelle souvent « SuperSpeed ». Si dans le langage courant on parle de port USB 2 et de port USB 3, il faudrait donc plutôt parler port USB type A et de port USB type A SuperSpeed.

Comme vous pouvez le voir sur la photo, sur les types A, les 5 nouveaux pins suivants ont été ajoutés en arrière des 4 pins d’origine.

1. Rx- : données -, réception (fil bleu)
2. Rx+ : données +, réception (fil jaune)
3. GND DRAIN : masse pour le signal (pas de fil)
4. Tx- : données -, émission (fil violet)
5. Tx+ : données +, émission (fil orange)

![Vue de face du connecteur mâle d’un câble USB 3.0 type A. Les pins sont légendés.](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/usb-a-30.png)

Vue de face du connecteur mâle d’un câble USB 3.0 type A. Les pins sont légendés.

Notons que la nouvelle masse GND DRAIN, également appelée GND DR, n’a pas de fil, elle n’est utilisée que sur le connecteur lui-même. C’est pour ça que plus haut j’ai écrit que l’on ajoutait 4 pins (les 2 nouvelles paires différentielle pour les données) mais qu’en réalité il y en a 5 nouveaux. Notez qu’afin d’éviter la confusion, la masse que nous avions déjà avec USB 1 et 2 est parfois appelée GND A.

Si l’ajout de 2 nouvelles paires différentielle permet naturellement d’atteindre de meilleurs débit, l’arrivée de l’USB 3.1, dit « SuperSpeed+ » ou « SS+ » a permis, en conservant exactement les mêmes câbles, d’encore monter le débit de données à 10 Gbps.

## USB 3.2 [#](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/#usb-32)

Jusque là, les différentes versions d’USB vivaient en harmonie. Mais un jour l’USB 3.2 décida de passer à l’attaque. Afin d’atteindre un débit de 20 Gbps, de nouveaux pins ont été ajoutés. Ceci fait qu’aujourd’hui l’USB 3.2 ne peut fonctionner qu’avec un seul type de connecteur, le type C.

Mais alors, pourquoi parler d’USB 3.2 dans cet article où on ne parle que des embouts de type A ? Pourquoi ne pas l’avoir gardé pour l’article sur les embouts de type C ? Et bien pour deux raisons. La première est que certains fabricants vendent des contrôleurs USB 3.2 avec des embouts type A SuperSpeed. Cependant, du fait du manque de pins, le débit est bel et bien limité à 10 Gbps, vous n’atteindrez pas les 20 Gbps promis par l’USB 3.2.

La seconde raison que la norme USB 3.2 a décidé de semer la confusion avec le concept de mode d’opération. Ainsi, elle a rétroactivement décrété que USB 3.0 utilisait le mode d’opération « USB 3.2 Gen 1×1 », que l’USB 3.1 utilisait le mode d’opération « USB 3.2 Gen 2×1 » et que l’USB 3.2 utilise le mode d’opération « USB 3.2 Gen 2×2 ».

Maintenant, vous savez pourquoi USB 3.2 ne fonctionne pas réellement sur un port USB type A SuperSpeed malgré ce que certaines appellations peuvent laisser supposer.

## Reconnaître les câbles USB type A [#](https://rodolphe.breard.tf/article/introduction-aux-cables-usb/#reconna%c3%aetre-les-c%c3%a2bles-usb-type-a)

Si vous avez bien suivi ce billet, vous savez désormais qu’il n’existe que 2 types de câbles USB de type A : le simple type A et le SuperSpeed. En général on les différencie grâce à leurs couleurs différentes, les SuperSpeed étant bleus alors que les autres sont blancs ou noir.

En réalité il y a un peu plus de couleurs que ça mais on quite alors le domaine des câbles afin d’arriver dans celui des contrôleurs USB. Je ne vais donc pas spécialement expliquer les couleurs en question, le mieux étant de toujours se référer à la notice du fabricant car l’attribution des couleurs n’est pas toujours bien respecté.

Le moyen le plus fiable pour différencier un câble « USB 2 » d’un « USB 3 » reste encore de regarder si les 5 pins additionnels du SuperSpeed sont présents ou non. Ce point ne ment généralement pas et permet d’éviter de se faire arnaquer par un fabricant/vendeur malhonnête qui vend un USB 2 peint en bleu sous l’appellation USB 3. Oui, ce genre d’arnaque existe et dans le prochain billet vous verrez que c’est encore bien pire avec l’USB de type C. Là franchement c’est gentillet comme arnaque car dans le pire des cas juste vous n’aurez pas le débit espéré, ça ne risque pas de littéralement cramer votre baraque.

Et c’est sur ce cliffhanger de fou que se termine ce billet. Maintenant je sais que vous allez absolument vouloir lire le suivant, surtout qu’il vous sera vraiment immensément plus utile que celui-là.