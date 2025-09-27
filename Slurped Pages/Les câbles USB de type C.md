---
link: https://rodolphe.breard.tf/article/les-cables-usb-c/
byline: Rodolphe Bréard
site: Accueil
date: 2025-07-08T02:00
excerpt: Dans le précédent article traitant de l’USB, j’ai parlé des câbles disposant d’un connecteur de type A, ce qui était plutôt simple. Il est désormais temps de nous attaquer à quelque chose de bien plus intéressant, à savoir les câbles disposant d’un connecteur de type C.
slurped: 2025-09-13T20:04
title: Les câbles USB de type C
tags:
  - usb
  - cable
---

Dans le précédent article traitant de l’USB, j’ai parlé des câbles disposant d’un connecteur de type A, ce qui était plutôt simple. Il est désormais temps de nous attaquer à quelque chose de bien plus intéressant, à savoir les câbles disposant d’un connecteur de type C.

## Présentation générale [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#pr%c3%a9sentation-g%c3%a9n%c3%a9rale)

Physiquement, nous avons 2 rangées de 12 pins, soit en théorie 24 pins. Cependant, en pratique, nous allons avoir quelques limitations que j’expliquerai un peu plus loin.

![Schémas d'un embout USB de type C où chaque pin est numéroté de A1 à A12 en haut et de B12 à B1 en bas. Le nom de chaque PIN est indiqué.](https://rodolphe.breard.tf/article/les-cables-usb-c/USB_Type-C_Receptacle_Pinout.svg)

Schémas d’un embout USB de type C. [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/deed.fr), [Chindi.ap](https://commons.wikimedia.org/wiki/User:Chindi.ap), [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:USB_Type-C_Receptacle_Pinout.svg?uselang=fr)

Ne vous prenez pas trop la tête avec ce schémas, nous n’allons pas nous attarder sur tous les pins.

Mais avant d’aller plus loin, il faut que je nous fassions la distinction entre, d’un côté, l’alimentation électrique et, de l’autre, le transfert de données. Avec les embouts de type A la question ne se posait pas vraiment car l’alimentation électrique était (presque) toujours la même. Avec les embouts de type C, en revanche, c’est devenu un sujet à part entière.

## USB Power Delivery [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#usb-power-delivery)

Afin de facilement expliquer les choses, nous ferons la distinction entre, d’une part, les équipements qui fournissent de l’énergie électrique, donc une alimentation (« source » en anglais), et ceux qui consomment de l’énergie électrique (« sink » en anglais). Les équipements qui peuvent, suivant la situation, agir comme une alimentation ou un sink sont nommés dual-role power (DRP).

Enfin, pour rappel, la puissance P, exprimée en watts, correspond à la tension U, exprimée en volts, multipliée par l’intensité I, exprimée en ampères.

\(P = U × I\)

Avec les câbles de type A dont je parlais dans mon précédent article, la situation était assez simple car il n’était possible que d’avoir une tension de 5 V et une intensité limitée. Ainsi, les spécifications USB 2.0 autorisent jusqu’à 500 mA et celles de l’USB 3.2 autorisent jusqu’à 900 mA. Nous avons donc des maximum de 2,5 W et 4,5 W, donc vraiment très peu. Avec de telles limites, nous n’avons pas à nous poser la question de savoir si les câbles vont supporter le courant ou bien prendre feu.

Les embouts de type C ayant été conçus pour devenir plus universels et donc remplacer les câbles d’alimentation de matériels électroniques assez gourmands en puissance électrique, notamment des ordinateurs portables, il a fallu revoir tout ça. C’est ce qui nous mène à un mécanisme important, le USB Power Delivery, abrégé USB PD ou parfois simplement PD. Ce mécanisme permet à l’alimentation et au sink de négocier dynamiquement une intensité et une tension dans les limite de ce qu’autorisent les spécifications de l’USB PD. Au moment où j’écris ces lignes, il est possible d’avoir des tensions de 5 V à 48 V et des intensité allant jusqu’à 5 A, soit un maximum de 240 W.

En réalité, USB PD existait avant le type C et fonctionnait, de manière optionnelle, sur le type A afin d’autoriser des limites supérieures à celles définies par les spécifications USB 2.0 et USB 3.2. Cependant, non seulement ces nouvelles limites restaient relativement basses, mais en plus la version 3 des spécifications USB PD a déprécié son utilisation avec les cables qui ne sont pas de type C.

D’ailleurs, avant même USB PD, un autre mécanisme nommé USB Battery Charging (USB BC) permettait déjà de négocier des intensités et tensions supérieures aux limites définies par les spécifications USB 2.0 et USB 3.2. Encore une fois, ces limites sont faibles.

Ce mécanisme est particulièrement intéressant car il permet à l’alimentation de ne fournir au sink que ce qui est strictement nécessaire pour son fonctionnement. On évite donc ainsi les pertes liées aux conversions dans tous les sens. Dans le cas du rechargement d’une batterie, notamment celle d’un téléphone portable, l’USB PD permet au sink (par exemple le téléphone) de mettre en œuvre certains mécanismes de réduction de la puissance, et donc d’allongement du temps de charge, afin de préserver la batterie.

![Capture d'écran des paramètre de contrôle de la charge sous LineageOS.](https://rodolphe.breard.tf/article/les-cables-usb-c/controle_charge_hu_ea84c1094fba2a07.png)

Capture d’écran des paramètre de contrôle de la charge sous LineageOS.

La négociation de la puissance entre l’alimentation et le sink a également pour objectif de réduire le nombre de déchets électroniques. En effet, jusque là le modèle était, pour chaque matériel électronique, de fournir un adaptateur secteur dédié. Grace à USB PD nous avons désormais des adaptateurs secteur génériques capables de charger tous nos équipements pouvant être chargés via un port USB de type C. La seule limitation à ça est que le chargeur doit pouvoir fournir suffisamment de puissance. Maintenant que l’USB de type C commence à être très répandu, je pense qu’il faut nous préparer à voir de plus en plus de produits vendus avec le chargeur en option.

Pour ma part, en voyage, j’apprécie beaucoup n’avoir besoin de prendre qu’un seul chargeur pour mon ordinateur, mon téléphone et tout autre gadget USB-C.

![Photographie d'un chargeur USB se branchant sur le secteur. Les inscriptions indiquent que pour l'USB-C il supporte jusqu'à 65 W, que pour l'USB-A il supporte jusqu'à 18 W et que le total USB C + A ne peut pas dépasser 65 W.](https://rodolphe.breard.tf/article/les-cables-usb-c/chargeur_hu_5e546b161024d1a0.jpg)

Ce chargeur USB type A et C supporte jusqu’à 65 W pour l’USB-C et 18 W pour l’USB-A, il a donc sans doute été conçu avant la dépréciation de l’USB PD pour l’USB-A.

Bon, c’est bien joli tout ça, mais il y a un problème que nous n’avons pas encore résolu. Si vous avez eu une lecture particulièrement attentive, j’indiquais les les limites maximales de tension et d’intensité étaient régulièrement augmentées. Dans ces conditions, comment est-il possible de garantir que les câbles USB fabriqués avant ces nouvelles limites maximales supportent de telles puissances ? Il est donc temps de parler du eMarker.

## Le eMarker [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#le-emarker)

Dans le monde de l’USB de type A, nous n’avions véritablement que deux types de câbles, ceux à 4 pins (dits USB 2) et ceux à 9 pins (dits USB 3). Maintenant que nous avons plongé dans le monde de l’USB de type C, il va falloir nous habituer à une certaine complexité supplémentaire.

Si en apparence tous les câbles USB-C semblent être similaires car tous dotés d’embouts comportant exactement le même nombre de pins, en réalité il n’en n’est rien. Deux câbles USB-C d’apparence strictement identique peuvent en réalité être extrêmement différents. Par exemple, comme nous venons de le voir en parlant de l’USB PD, suivant leur conception les câbles peuvent supporter des valeurs maximales de tension et d’intensité différentes.

La raison pour laquelle nos appareils USB ne se trompent pas et refusent d’envoyer 240 W dans un câble conçu pour n’en supporter que 15 est simple. Les cables USB sont dotés d’un marqueur électronique contenant divers informations sur le câble, notamment la puissance maximale supportée. Comme vous l’aurez deviné, ce marqueur électronique se nomme tout simplement eMarker (parfois stylisé E-Marker).

Cependant, l’eMarker n’est pas toujours obligatoire. Dans le cas où il est absent, afin d’éviter les incidents, des limites de sécurité s’appliquent. Dans le cadre d’USB PD, ces limitations prennent la forme de l’interdiction de dépasser les limites d’intensité et de tensions définies dans les spécifications USB 2.0 et, le cas échéant, USB 3.2.

En réalité, il est parfois possible d’avoir des limitations de sécurité plus élevées (le plus souvent 7,5 W), mais expliquer ceci nous ferait rentrer dans des considérations beaucoup trop complexes pour cet article.

Mais le eMarker ce n’est pas uniquement pour définir la puissance que le câble est capable d’encaisser sans broncher. Par exemple, il sert également, de manière optionnelle à indiquer le fabricant. Notons que pour obtenir un identifiant de fabriquant à mettre dans un eMarker il faut [payer une redevance à l’USB-IF](https://www.usb.org/getting-vendor-id). Bien que le coût de cette redevance soit négligeable pour une entreprise produisant de grandes quantités de câble, énormément de fabricants refusent tout de même de franchir cette étape. Ça en dit long sur le fait qu’ils assument la qualité de leurs produits. Dans la mesure où, en règle générale, ils ne commercialisent pas directement leurs câbles mais produisent pour le compte de marques qui revendent leur production, il est alors quasiment impossible, même avec du matériel de test spécialisé, d’identifier le producteur d’un câble.

Certains vendeurs font croire que le eMarker sert à « stabiliser » le courant ou autres choses du genre. C’est de la publicité mensongère, je vous conseille de ne rien acheter chez de tels vendeurs. Alors oui il existe des câbles qui embarquent de l’électronique afin d’améliorer le signal, ça s’appelle des cables actifs, mais ce n’est pas le eMarker qui fait ça.

Un autre rôle intéressant du eMarker, c’est d’indiquer les protocoles supportés par le câble. Voyons donc les principaux protocoles en question. Spoiler alert : il existe en réalité d’autres protocoles que nous ne verrons pas ici.

## USB 2.0 [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#usb-20)

Comment ? Qu’est ce que j’apprends ? Maintenant qu’on a un super connecteur de 2 fois 12 pins on s’en sert pour faire de l’USB 2.0, un protocole qui ne nécessite que 4 pins ?

Et bien oui. Dans le domaine de l’USB la rétro-compatibilité est prise très au sérieux. Le principe du « qui peut le plus peut le moins » a pour conséquence que les câbles de bonne qualité qui supportent des protocoles modernes vont également supporter les protocoles plus anciens.

Il est cependant très fréquent de rencontrer des câbles bas de gamme qui supportent uniquement USB 2.0. Dans la mesure où nous verrons les câbles plus haut de gamme un peu plus loin, concentrons nous ici sur ceux qui ne font que de l’USB 2.0. Dans un tel cas, seuls les pins suivants sont connectés :

- GND (A1, B1, A12, B12)
- VBUS (A4, B4, A9, B9)
- CC (A5 ou B5)
- D+ (A6 ou B6)
- D- (A7 ou B7)

Normalement, à ce stade vous commencez à voir la symétrie qui est utilisée afin que l’on puisse brancher un câble dans les deux sens. C’est encore mieux visible lorsque l’on utilise un testeur de câble afin de vérifier quels sont les pins qui sont effectivement reliés.

![Photographie d'un testeur de cable BLE caberQU affichant les pins connectés : A1, A4, A5, A6, A7, A9, A12, B1, B4, B9, B12.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-2.0-up.jpg) ![Photographie d'un testeur de cable BLE caberQU affichant les pins connectés : A1, A4, A9, A12, B1, B4, B5, B6, B7, B9, B12.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-2.0-down.jpg)

Observez les pins CC, D+ et D- qui, en fonction du sens de branchement du câble, sont sur la face A ou la face B. C’est pour cela que l’on aime dire que les périphériques USB-C savent dans quel sens vous avez branché votre câble.

Si vous avez prêté attention à l’article précédent, vous avez sans doute remarqué que le pin CC n’existait pas dans le monde de l’USB type A. C’est en effet un ajout de l’USB type C, il s’agit d’un pin de configuration qui est très utile. Notez bien ce point, nous en reparlerons dans un prochain article.

## USB 3.2 et USB 4 [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#usb-32-et-usb-4)

Maintenant que nous savons qu’il existe des câbles USB de type C qui ne font que de l’USB 2.0, montons progressivement en gamme. Dans le précédent article je vous disais que l’USB 3.2 Gen 2×2 ne pouvait atteindre son débit réel de 20 Gbps que si l’on ajoutait des pins qui n’existaient que sur les câbles de type C. Nous allons donc vérifier ce point.

En observant un câble USB 3.2 Gen 1×1 (aka USB 3.0), on s’aperçoit qu’il y a maintenant plus de pins de connectés que pour sur le cable USB 2.0 que nous avons vu précédemment. Jusque là rien de surprenant, avec l’USB de type A il y avait également des pins supplémentaires.

![Photographie d'un testeur de cable BLE caberQU dans lequel est branché un cable USB 3.2 Gen 1×1.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-3.0-disp.jpg) ![Photographie d'un testeur de cable BLE caberQU dans lequel est branché un cable USB 3.2 Gen 1×1.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-3.0-up.jpg) ![Photographie d'un testeur de cable BLE caberQU dans lequel est branché un cable USB 3.2 Gen 1×1.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-3.0-down.jpg)

Si avec l’USB type A les câbles USB 3.0 et USB 3.1 étaient identiques, vous serez surpris d’apprendre que ce n’est pas tout à fait le cas avec le type C. La raison est un peu complexe, donc si vous voulez savoir pourquoi la réponse se trouve dans la section 4.5.1.1 des spécifications de l’USB type C. Voici donc ce que donne un câble USB 3.2 Gen 2×1 (aka USB 3.1).

![Photographie d'un testeur de cable BLE caberQU dans lequel est branché un cable USB 3.2 Gen 2×1.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-3.1-disp.jpg) ![Photographie d'un testeur de cable BLE caberQU dans lequel est branché un cable USB 3.2 Gen 2×1.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-3.1-up.jpg) ![Photographie d'un testeur de cable BLE caberQU dans lequel est branché un cable USB 3.2 Gen 2×1.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-3.1-down.jpg)

Enfin, nous pouvons vérifier que pour USB 3.2 Gen 2×2 et USB 4 nous avons un pin supplémentaire de connecté. Je sais, il s’agit du pin de configuration, mais comme je l’écrivait un peu plus haut il s’agit d’un sujet compliqué qui est assez bien expliqué dans les spécifications avec de jolis schémas (n’y voyez aucune ironie, c’est vraiment le cas).

![Photographie d'un testeur de cable BLE caberQU dans lequel est branché un cable USB 4.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-4-disp.jpg) ![Photographie d'un testeur de cable BLE caberQU dans lequel est branché un cable USB 4.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-4-up.jpg) ![Photographie d'un testeur de cable BLE caberQU dans lequel est branché un cable USB 4.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c-4-down.jpg)

## La câble d’alimentation pur [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#la-c%c3%a2ble-dalimentation-pur)

Il est parfois possible de trouver des câbles exclusivement destinés à la charge électrique, ne permettant ainsi aucun transfert de données. L’intérêt de ces câbles est assez limité, mais certains commerciaux ont tout de même flairé le bon plan et essayent de vous vendre à prix d’or ces câbles très basiques sous prétexte que ce sont donc des bloqueurs de données et que donc « La SéKuRiTaY ».

![Capture d'écran du site marchand ldlc.com présentant la fiche produit d'un câble USB-C de la marque MicroConnect labellisé « bloqueur de données ». Ce cable de 1,5 mètre de long est vendu 26,95 €.](https://rodolphe.breard.tf/article/les-cables-usb-c/ldlc_bloqueur_de_donnees_hu_1de9d10db1fb4d87.png)

Si je me fais pirater après avoir utilisé ce câble hors de prix, Crom rira de moi et me jettera hors du Valhalla.

Si vous comptiez acheter ce genre de câble, posez vous la question suivante : lorsque vous rechargez votre matériel électronique avec un véritable câble USB-C, vous le branchez sur quoi ?

1. un chargeur branché sur le secteur ;
2. une batterie externe ;
3. un autre matériel électronique dont vous vous méfiez (y compris si c’est le votre).

Pour ma part je n’ai fait que les options 1 et 2. L’option 3 je ne l’ai fait qu’avec un câble adaptateur USB A vers C, donc pas un véritable câble USB-C comme ce qui est vendu. Vous aurez remarqué que l’option 3 est la seule qui justifie d’éventuellement avoir recours à ce genre de câble. C’est dommage, c’est je pense l’option la moins utilisée.

Si vous vous dites que je suis quand même un peu dur dans mon jugement, sachez que ce câble là ne supporte qu’au maximum 60 W, ce qui n’est quand même pas grand chose comparé aux 240 W que l’on peut obtenir avec l’USB PD. Donc même si on considère juste le critère de charge, c’est médiocre. You had one job. Le pire c’est que pour exactement le même prix vous pouvez avoir un câble supportant 40 Gbps et 240W, avec également la finition textile tissé et la même longueur. Et avec une finition plastique lisse vous l’avez pour encore moins cher.

![Capture d'écran du site marchand ldlc.com présentant la fiche produit d'un câble USB-C de la marque Goobay présenté comme supportant 40 Gbps et 240W. Le câble dispose d'une surface externe en textile tissé.](https://rodolphe.breard.tf/article/les-cables-usb-c/ldlc_cable_usb_4_textile.png) ![Capture d'écran du site marchand ldlc.com présentant la fiche produit d'un câble USB-C de la marque Goobay présenté comme supportant 40 Gbps et 240W. Le câble dispose d'une surface externe en matière plastique lisse.](https://rodolphe.breard.tf/article/les-cables-usb-c/ldlc_cable_usb_4_classique.png)

Je sais que je remue le couteau dans la plaie, mais si jamais vous pensiez me contredire en argumentant qu’il faudrait se méfier du chargeur branché sur le secteur ainsi que de la batterie externe car ce sont des équipement électroniques potentiellement corrompus, et bien j’ai une mauvaise nouvelle pour vous. Un câble USB peut lui aussi contenir de l’électronique, c’est ce qu’on appelle un câble actif. Et je ne parle pas du eMarker qui, soit dit en passant, est obligatoire sur les câbles actifs et optionnel sur les câbles passifs. Je parle vraiment d’électronique miniaturisée notamment destinée à reproduire le signal afin d’augmenter les performances du câble. Vous ne devriez donc pas avoir plus confiance en votre câble qu’en votre chargeur ou votre batterie externe. D’ailleurs on trouve dans le commerce des câbles USB spécifiquement conçus pour les attaquants, par exemple le [O.MG Cable](https://shop.hak5.org/products/omg-cable?variant=41850108412017). Allez donc regarder en détail ce que fait ce câble, c’est véritablement impressionnant, vous ne regretterez pas.

Alors, qu’est-ce qui vous garantit que votre câble bloqueur de données n’est pas en réalité un câble produit par des services de renseignement afin justement de vous attaquer ? Ce serait quand meme bien plus logique que les bornes publiques de recharge car non seulement ça ne ciblerait que les paranos de la sécurité et non les personnes lambda, mais en plus ça permettrait d’atteindre des cibles jusque dans les pays où l’on ne peut pas déployer des bornes et ce de manière régulière et non pas juste occasionnelle.

Bref, si vous vous souciez des attaques via du matériel corrompu, votre câble est bien plus suspect que la prise de votre hall de gare. Révisez vos priorités avant de claquer votre argent dans des gadgets inutiles qui, de surcroît, ne respectent pas les spécifications USB. Parce que oui, je ne le précise que maintenant, mais les spécifications USB n’autorisent pas ce genre de câble. Mais je ferai un article à part sur ces câbles non-standards.

## Le marquage des câbles [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#le-marquage-des-c%c3%a2bles)

Dans la mesure où il est impossible de connaître les capacités d’un câble USB par une simple observation visuelle, il est donc nécessaire d’avoir recours au marquage des câbles. Et vu que nous avons pu voir tout au long de cet article que le monde de l’USB-C c’est un peu le far-west, je vous propose donc de rester dans la thématique en reprenant le titre du plus célèbre film de Sergio Leone.

### Le bon [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#le-bon)

La bonne nouvelle c’est que l’USB-IF a défini un logo officiel permettant d’identifier rapidement et simplement les capacités d’un câble USB de type C. Ainsi, chaque câble indique à la fois le débit et la puissance maximale qu’il supporte. Une exception cependant, pour les câbles ne supportant que l’USB 2.0 le débit maximal n’est pas indiqué.

Notons que ce logo dispose d’une variante colorée destinée à être apposée sur l’emballage du câble ainsi qu’une variante monochrome simplifiée destinée à être apposée directement sur les embouts des câbles.

![Les différentes variantes du logo officiel des câbles USB de type C.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c_official_logo_hu_ffcf77d0715634e4.png)

Les différentes variantes du logo officiel des câbles USB de type C.

Ce système est simple et efficace. Il ne nécessite pas de connaître toutes les subtilités de l’USB que nous avons pu voir au cours de ces articles. Il est également agréable de voir que l’USB-IF a bien pris en compte les critiques des logos officiels des autres types de connecteurs USB, ces derniers étant tellement mal fichus que je n’ai même pas souhaité aborder le sujet dans l’article précédent.

En plus de ça, l’USB-IF impose que, pour pouvoir apposer le logo officiel, le fabricant doit faire certifier ses câbles par un processus de test également défini par l’USB-IF et dispose d’un identifiant de fabricant. Plus qu’un simple indicateur des capacités du câble, le logo officiel est donc également un gage de qualité.

Seulement voilà, les test qualité ont un coût. En plus de ça, il faut également payer une petite redevance à l’USB-IF afin d’avoir le droit d’apposer le logo officiel. Vous voyez où je veux en venir…

### La brute [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#la-brute)

Malheureusement, afin de réduire les coûts de production, beaucoup de fabricants refusent de payer des redevances à l’USB-IF ainsi que de se soumettre à des contrôles de qualité pouvant être coûteux. Il leur est donc impossible d’apposer le logo officiel de l’USB-IF sur leurs produits. Lorsqu’un tel fabricant est de bonne volonté, il apposera tout de même un marquage similaire au logo.

![Photographie des deux connecteurs d'un câble USB de type C. Sur l'un des connecteurs est indiquée la mention « 40Gbps » et sur l'autre est indiquée la mention « 240w ».](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c_cable_unofficial-logo_hu_3475a9acbb4442a5.jpg)

Apposition d’un marquage non-officiel sur les connecteurs d’un câble USB de type C.

Bref, ça reste pas mal, mais vous voyez à nouveau où je veux en venir…

### Et le truand [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#et-le-truand)

Quand le fabricant veut réduire un maximum les frais et, en plus, n’en a strictement rien à faire du consommateur qui utilisera son produit, il ne s’embête même pas à indiquer quoi que ce soit d’utile sur ses câbles. Au mieux on trouve le logo de la marque qui a revendu le câble.

![Photographie d'un câble USB de type C sans aucun marquage.](https://rodolphe.breard.tf/article/les-cables-usb-c/usb-c_cable_no-logo_hu_f2045996198c293a.jpg)

Un câble USB de type C sans aucun marquage.

Autant vous dire que dans ce genre de cas de figure il ne faut pas attendre grand chose du câble en question. Parfois les capacités du câble sont inscrites sur l’emballage que vous avez très certainement déjà jeté. Mais meme si c’était le cas, personnellement je n’aurai tout de même qu’une confiance limitée dans ces indications.

Face à ces câbles non marqués, quelqu’un a proposé une méthode de marquage basée sur les couleurs et des bares, le [USB-C Cable Colour Codes](https://sa.lj.am/usbccccc/). Bien que l’idée soit intéressante, pour ma part je ne la recommande pas pour deux raisons :

1. ce code couleur est difficile à retenir et n’est pas intuitif ;
2. le projet n’est pas maintenu et n’inclue donc pas les ajouts des spécifications les plus récentes.

Bref, pour moi le mieux reste encore de se tenir le plus éloigné possible des câbles USB de type C ne présentant aucun marquage. Et si vous n’avez pas le choix, étiquetez juste vos câbles avec les mêmes informations que le marquage officiel.

![Photographie d'un câble USB-C non marqué. À côté de lui est posé une étiquette comportant les mentions « 15 W » et « 20 Gbps ».](https://rodolphe.breard.tf/article/les-cables-usb-c/etiquetage_01.jpg) ![Même photographie que la précédente, sauf que cette fois l'étiquette n'est plus posée à côté de câble mais sur le câble.](https://rodolphe.breard.tf/article/les-cables-usb-c/etiquetage_02.jpg)

## Tester des câbles [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#tester-des-c%c3%a2bles)

Bien qu’il soit impossible de visuellement déduire les capacités d’un câble USB de type C non marqué, il est cependant possible de repérer certains câbles peu sophistiqués, en particulier ceux ne supportant qu’au maximum l’USB 2.0. En effet, ce type de câble nécessite moins de fils et moins de blindage, ce qui fait que ces câbles sont généralement plus fin que les câbles plus performants. Attention, si un cable fin est forcément peu performant, la réciproque est fausse : un cable épais n’offre pas forcément de bonnes performances.

La notion de fin et d’épais étant toute relative, dans le cas présent nous pouvons considérer 3,6 mm de diamètre comme fin et 4,6 mm comme épais. 1 mm d’écart peut sembler peu, mais en réalité c’est beaucoup. Si vous mettez côte-à-côte un câble fin et un câble épais, la différence vous sautera aux yeux.

![Photographie d'un câble USB-C dont l'épaisseur est mesurée à l'aide d'un pied à coulisse électronique. La valeur affichée est de 3,57 mm.](https://rodolphe.breard.tf/article/les-cables-usb-c/epaisseur_cable_bof.jpg) ![Photographie d'un câble USB-C dont l'épaisseur est mesurée à l'aide d'un pied à coulisse électronique. La valeur affichée est de 4,58 mm.](https://rodolphe.breard.tf/article/les-cables-usb-c/epaisseur_cable_ok.jpg)

Bien entendu, vous n’avez pas besoin de faire comme moi et de mesurer avec un pied à coulisse. Vous pouvez vous faire une idée à l’aide d’une simple règle graduée au millimètre en regardant juste si le diamètre est en dessous ou au dessus de 4 mm. Une astuce pour améliorer la précision est de ne pas prendre la mesure depuis le bord de la règle mais depuis une grosse graduation un peu plus loin.

Un autre point à garder en tête est que l’habit ne fait pas le moine. Si les câbles avec une couche extérieure en nylon ou kevlar tissé donnent un aspect très qualitatif (ou « premium » comme disent les anglophones), les fabriquant sans scrupules ont très vite vu le truc arriver et se sont mis à utiliser ça (ou du simili) pour recouvrir leurs câbles de piètre qualité.

Bon, maintenant que nous avons joué aux apprentis sorciers avec nos mesure de tailles de câbles, passons aux choses plus sérieuses. Actuellement, le meilleur outil qui soit pour analyser un câble USB de type C est le [BLE caberQU](https://caberqu.com/content/8-ble-caberqu). En plus de lire le eMarker, il permet de contrôler la connexion des pins, la résistance du câble et a quelques autres fonctionnalités intéressantes. Très simple à prendre en main et particulièrement puissant, c’est mon chouchou. Dans la mesure où vous l’avez déjà suffisamment vu à l’œuvre durant tout cet article, je ne vais pas remettre une nouvelle photo de cet appareil.

En complément, j’utilise le [POWER-Z KM003C](https://www.chargerlab.com/category/power-z/power-z-km003c-km002c/). L’intérêt de ce testeur est surtout de mesurer en direct l’intensité, la tension et la puissance. Il est également utile pour détecter le support de certains protocoles et, dans une moindre mesure, il peut également lire des eMarkers. Son point noir est sa difficulté de prise en main et sa documentation très médiocre malgré des vidéos explicatives en anglais. Un PDF de documentation en anglais est également trouvable, mais est tout aussi médiocre.

![Photographie d'une batterie externe en train d'être chargée à l'aide d'un câble USB-C. Entre la batterie et le câble se trouve un POWER-Z KM003C qui indique une tension de 4,7 V, une intensité de 2,28 A et donc une puissance de 10,7 W. Il est également indiqué l'usage d'USB PD et une flèche indique que le courant va du câble vers la batterie.](https://rodolphe.breard.tf/article/les-cables-usb-c/power-z_charge_hu_262bc90549d2bb83.jpg)

Chargement d’une batterie externe.

## Pour aller plus loin [#](https://rodolphe.breard.tf/article/les-cables-usb-c/#pour-aller-plus-loin)

Si vous avez l’esprit aventureux et beaucoup de temps à perdre, vous pouvez télécharger les spécifications sur [le site de l’USB-IF](https://usb.org/documents). Afin de vous épargner une recherche dans leur bibliothèque, voici les liens direct vers les spécifications en vigueur à date de publication de cet article.

- [USB 2.0](https://usb.org/document-library/usb-20-specification)
- [USB 3.2](https://usb.org/document-library/usb-32-revision-11-june-2022)
- [USB 4](https://usb.org/document-library/usb4r-specification-v20)
- [USB Power Delivery](https://usb.org/document-library/usb-power-delivery)
- [USB Type-C](https://usb.org/document-library/usb-type-cr-cable-and-connector-specification-release-24)

À l’inverse, si pour vous l’USB est un sujet qui attise votre curiosité mais que vous voyez ça plus comme de la culture technique et avez envie de creuser le sujet sans pour autant vous manger plusieurs milliers de pages de spécification écrites dans un langage abscons, alors je vous recommande les vidéos, en anglais, de [Quiescent Current](https://www.youtube.com/@quiescentcurrent). Vous y trouverez plein de tests de câbles USB commentés de très bonnes explications. Pour ma part je trouve ça très divertissant, mais c’est sans doute juste que je suis beaucoup trop geek.

Je vous recommande également toute [la série d’articles consacrés à l’USB-C](https://hackaday.com/series_of_posts/all-about-usb-c/) publiée sur le magazine en ligne Hackaday.

Pour terminer sur une touche plus accessible au grand public, je vous recommande également [la vidéo, en anglais, d’Andreas Spiess sur l’USB-C](https://www.youtube.com/watch?v=qV03FfdPHOw).

Ces sources sont les plus fiables que j’ai pu trouver. En cherchant vous en trouverez beaucoup d’autres, mais leur fiabilité est parfois aléatoire. Et oui, je me suis tapé la lecture de plusieurs parties des spécifications afin de valider les sources que je viens de vous citer et exclure les autres sources moins fiables que j’ai pu trouver.