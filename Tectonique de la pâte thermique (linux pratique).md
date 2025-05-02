---
link: https://grimoire.d12s.fr/2018/tectonique-de-la-pate-thermique-linux-pratique.html
byline: Simon Descarpentries
site: Grimoire-Command.es
date: 2018-07-03T00:00
excerpt: Index 1. Pour quoi faire ? 2. De quoi est-ce fait ? 2.1. Pourquoi toujours du cuivre et de l’aluminium pour les ventirads ? 3. Comment choisir sa pâte thermique ? 4. La pâte thermique, une pièce d’usure 5. Conclusion Assez méconnue on la croirait superflue. Elle est pourtant indispensable aux …
slurped: 2025-04-20T19:24
title: Tectonique de la pâte thermique (linux pratique)
tags:
  - linux
  - hardware
  - repair
---

[](https://grimoire.d12s.fr/2018/tectonique-de-la-pate-thermique-linux-pratique.html)

[hardware](https://grimoire.d12s.fr/tag/hardware.html) [repair](https://grimoire.d12s.fr/tag/repair.html)

Index

- [1. Pour quoi faire ?](https://grimoire.d12s.fr/2018/tectonique-de-la-pate-thermique-linux-pratique.html#_pour_quoi_faire)
- [2. De quoi est-ce fait ?](https://grimoire.d12s.fr/2018/tectonique-de-la-pate-thermique-linux-pratique.html#_de_quoi_est_ce_fait)
    - [2.1. Pourquoi toujours du cuivre et de l’aluminium pour les ventirads ?](https://grimoire.d12s.fr/2018/tectonique-de-la-pate-thermique-linux-pratique.html#_pourquoi_toujours_du_cuivre_et_de_laluminium_pour_les_ventirads)
- [3. Comment choisir sa pâte thermique ?](https://grimoire.d12s.fr/2018/tectonique-de-la-pate-thermique-linux-pratique.html#_comment_choisir_sa_p%C3%A2te_thermique)
- [4. La pâte thermique, une pièce d’usure](https://grimoire.d12s.fr/2018/tectonique-de-la-pate-thermique-linux-pratique.html#_la_p%C3%A2te_thermique_une_pi%C3%A8ce_dusure)
- [5. Conclusion](https://grimoire.d12s.fr/2018/tectonique-de-la-pate-thermique-linux-pratique.html#_conclusion)

**Assez méconnue on la croirait superflue. Elle est pourtant indispensable aux super flux thermiques de nos microprocesseurs. Saviez-vous que c’est également une pièce d’usure de nos machines ?**

Linux Pratique n° 104 | novembre 2017 | CC-By-NC-ND | Simon Descarpentries

## 1. Pour quoi faire ?

Je n’ai que trop rarement lu d’informations à propos de la pâte thermique. Sur la première machine que j’ai montée moi-même, le ventirad était fourni avec un sachet qui aurait contenu 2g de poivre ou de sel si le contexte s’y était prêté. Aucune instruction n’était fournie avec le sachet, alors que ce dernier avait un méchant goût de « pas droit à l’erreur, je suis à usage unique ». Du coup, j’ai d’abord subtilement tenté de m’en passer, mais ce ne fut pas une bonne idée : la machine n’atteignait pas l’invite de _login_ avant que le BIOS ne termine la partie avec une sonnerie qui aurait convenu aux pompiers.

![Cyrix 686 (1995) : Fan / Heatsink Required](https://connect.ed-diamond.com/var/diamond/storage/images/connect/linux-pratique/lp-104/tectonique-de-la-pate-thermique-article/figure12/557479-1-fre-FR/figure1_large.jpg)

_Cyrix 686 PR200 (1995) : Fan / Heatsink Required_

Retour à la case départ donc : le montage du ventirad, avec son gramme de pâte thermique pour maximiser la surface de contact entre microprocesseur et radiateur. Un peu comme une colle, mais visant surtout à s’assurer qu’il n’y a pas une lame d’air entre les deux surfaces impossibles à aligner parfaitement ou encore des bulles emprisonnées entre les surfaces insuffisamment planes et lisses dès qu’on y regarde de plus près.

Et la magie opéra, la machine démarrait désormais, je n’avais pas commis d’autres erreurs.

Sur les machines suivantes, le ventirad livré était carrément pré-enduit, d’une couche bien plus fine que ce que le sachet de la précédente machine ne m’avait permis d’atteindre. On pouvait en déduire plein de choses :

- la pâte thermique vaut cher ;
    
- il n’est pas sérieux de livrer un ventirad sans ;
    
- pas la peine de tartiner trop abondamment…
    

## 2. De quoi est-ce fait ?

![Intel Pentium III (2000) : coulure de pâte thermique](https://connect.ed-diamond.com/var/diamond/storage/images/connect/linux-pratique/lp-104/tectonique-de-la-pate-thermique-article/figure22/557484-1-fre-FR/figure2_large.jpg)

_Intel Pentium III (2000) : coulure de pâte thermique_

Mais c’est quoi ce truc, c’est pas toxique au moins ? Parce que les fluides qu’on livre dans cette gamme de quantité, généralement ça dépote : comme les colles cyanoacrylates (celles qui tiennent le monsieur la tête en bas collé par les semelles de ses chaussures, et qui vous collent les doigts tout aussi fermement en cas de mauvaise manip’…).

Sur le principe, une pâte thermique c’est un liant (graisse ou gel de silicone par exemple) et des particules d’aluminium, de cuivre ou d’argent. Le liant donne des propriétés de liquide au métal choisi, et les particules de métal donnent ses propriétés de conductivité thermique à la pâte. Rien de bien grave donc. On aurait pu trouver du mercure par exemple pour avoir du métal liquide, et là ça aurait été moins joyeux à respirer.

Après, les particules de métal présentent, comme tout ce qui est réduit en poudre au grain nanométrique, une surface de contact exponentiellement augmentée par rapport à un morceau macrométrique. Il en résulte une absorption dans nos tissus et des effets augmentés d’autant. Ici, l’aluminium est déclaré comme cancérogène potentiel et est donc retiré des cuisines et des canalisations, mauvaise pioche. Le cuivre ? C’est le principe actif des stérilets par exemple, ça tue des cellules à n’en pas douter. Et l’argent ? C’est un métal relativement lourd, qui s’accumule dans l’organisme, et c’est par exemple ce qui est utilisé dans les filtres des carafes filtrantes pour éviter que le charbon actif ne soit colonisé par des bactéries. Or, si on résume les cours de biologie du collège, l’humain est un gros tas de bactéries dont il faut prendre soin.

### 2.1. Pourquoi toujours du cuivre et de l’aluminium pour les ventirads ?

Parce que l’argent coûte trop cher ! Mais surtout parce que ces trois métaux présentent une bonne conductivité thermique. Cette dernière s’exprime en Watt par mètre kelvin et représente la quantité d’énergie transférée par unité de surface et de temps sous un gradient de température de 1°C par mètre (merci, Wikipédia). Quelques valeurs :

- Argent : 418 W/m.k
    
- Cuivre : 390 W/m.k
    
- Aluminium : 237 W/m.k
    
- Silicium : 149 W/m.k
    
- Verre : 1,2 W/m.k
    
- Carton : 0,11 W/m.k
    
- Air : 0,026 W/m.k
    

On voit dans cette petite liste que l’argent est un très bon conducteur thermique (d’ailleurs c’est très vite froid au touché dès qu’on ne le porte pas en contact avec la peau), mais que le cuivre n’est pas très loin derrière. Le silicium de nos puces chauffantes n’est pas trop mauvais non plus et l’aluminium suffit bien à prendre le relais. On voit par contre que le verre isole mal à lui tout seul (bien qu’il soit 100x moins conducteur que le silicium pur). Enfin, le carton et l’air isolent sérieusement.

C’est donc pour des raisons évidentes de rapport conductivité thermique/prix que les ventirads sont généralement en aluminium pour les entrées de gamme, et en cuivre au-delà. Les quantités de métal en jeu pour la réalisation d’une pâte thermique étant faibles, on peut se permettre d’en faire à base d’argent. Mais ça se ressent sur le prix.

![Seringue de pâte thermique](https://connect.ed-diamond.com/var/diamond/storage/images/connect/linux-pratique/lp-104/tectonique-de-la-pate-thermique-article/figure62/557499-1-fre-FR/figure6_large.jpg)

_Seringue de pâte thermique, garantie 8 ans, 8 W/m.k_

Précisément en se renseignant sur la conductivité thermique du produit visé. Toutes les pâtes thermiques du commerce ne se valent pas. Leur conductivité peut varier de 0,5 W/m.k à 8 W/m.k (pour les pâtes bon marché) et monter jusqu’à 16 et plus pour les pâtes de luxe.

On voit donc, rien qu’en restant dans les pâtes thermiques bon marché, qu’on trouve des pâtes thermiques seize fois plus efficaces que d’autres, ça vaut le coup de prendre cinq minutes pour choisir non ?

En prenant le problème à l’envers, on peut trouver en vente, sous l’appellation « pâte thermique » de bons isolants… badigeonnez votre fenêtre de pâte à 0,5 W/m.k et vous en doublerez le pouvoir isolant si c’est un simple vitrage ! Le résultat dans un usage de pâte thermique sera toujours moins pire que de laisser de la place à l’air, mais pour le même prix, ne vous faites pas avoir.

Ensuite, pour ce qui est du choix l’autre critère c’est la quantité achetée, à contrebalancer par le ratio du prix au poids. En effet, d’une part, pour un usage ponctuel un gramme c’est déjà trop, et d’autre part il serait dommage d’acheter plus d’emballage que de pâte thermique, surtout vu les efforts à déployer pour en arriver à changer la pâte thermique d’un ordinateur portable moderne par exemple. Mais d’ailleurs, pourquoi vous imposeriez vous une telle torture puisque les machines en sont équipées à la livraison ?

## 4. La pâte thermique, une pièce d’usure

Laissez moi vous conter une deuxième histoire. Elle se situe 10 ans après la première, pendant mon école d’ingénieurs à Polytech’Tours. Au club GNU/Linux de l’école, un membre avait récupéré une machine dans sa famille et la dédiait à nos tests hebdomadaires de distribution. Cette dernière fonctionnait très bien jusqu’au jour où elle avait subitement cessé de démarrer d’après ses anciens propriétaires. Le jour du club, nous étions donc cinq internes fermement décidés à réanimer notre patient. Nous avons passé la connectique en revue, testé le disque dur dans une autre machine, visité le BIOS au cas où… Mais plus on fatiguait sur la question, plus la panne nous échappait, devenant fugace, volatile, voire aléatoire !?

Il m’est alors revenu en tête cette histoire de pâte thermique. J’ai récolté des moues dubitatives, et puis c’est compliqué et dangereux à démonter le microprocesseur… On m’a laissé la main sur ce coup là. Je me suis appliqué : nettoyage de l’ancienne pâte thermique avec un mouchoir, nouvelle couche de pâte toute neuve, je remonte le tout en croisant les doigts et abracadabra ! La machine ronronne comme au premier jour. Regards stupéfaits dans la pièce, crédibilité +10 (c’est pas arrivé tous les jours).

La pâte thermique est une pièce d’usure. Elle sèche sous l’effet de la chaleur du composant en contact duquel elle se trouve. Elle se rétracte alors et craquelle, laissant des canyons d’air se creuser entre la puce et son radiateur. Et d’autres mécanismes complémentaires sont probablement aussi à l’œuvre pour faire perdre à la pâte ses propriétés thermiques.

Un cas isolé ? Faisons un nouveau bond de 10 ans en avant. L’été dernier l’ordinateur portable de ma compagne affichait 86°C au repos, faisant gondoler et presque fondre la tablette plastique sur coussin type poire, d’une grande enseigne suédoise, où il reposait. Je pensais lancer un film avec cette machine, mais vu le bruit qu’elle faisait déjà je me ravisais, ne souhaitant profiter lâchement de ce moment de faiblesse pour l’achever. Depuis 5 ans que ma compagne utilise cet ordinateur, on a fait qu’alléger sa partie logiciels passant de Windows 7 à Ubuntu, puis à GNU/Linux Debian et enfin en passant au bureau Xfce. Et en retour lui n’a fait que chauffer de plus en plus, progressivement, sans la moindre gratitude pour toute l’attention dont il était l’objet.

Grillé pour grillé, cette fois ce fut décidé, j’allai le démonter ! Avec le chat qui traîne toujours autour, il devait y avoir une feutrine de poils tassés dans la grille du ventilo… Et ce fut parti pour une demi-heure de démontage. Il faut s’armer de patiente, de ramequins (pour les vis), consulter les guides sur Internet pour trouver les vis cachées et savoir quand réfléchir et quand forcer… Une fois la machine ouverte, un coup d’air sec et c’est bon ? Pas toujours. Parfois la grille du ventilo est nickel et ce fut mon cas. Quitte à en être rendu là, autant finir le boulot, en creusant un peu plus loin : jusqu’au microprocesseur, et pas au-delà. Car il n’y a rien au-delà. Le microprocesseur c’est la partie la plus centrale, la plus harnachée, celle après laquelle il n’y a plus de vis nulle part à retirer. C’est LE boss de fin de niveau. Autant vous faire à l’idée tout de suite.

Une fois la puce découverte, un coup de mouchoir, une couche de pâte et il faut encore tout remettre à sa place… Résultat, au repos la machine s’est remise à tourner à 36°C. Nous avons gagné 50°C juste en changeant la pâte thermique ! En espérant que la nouvelle se dégrade moins vite.

![Exemple d’application](https://connect.ed-diamond.com/var/diamond/storage/images/connect/linux-pratique/lp-104/tectonique-de-la-pate-thermique-article/5/557504-1-fre-FR/5_large.jpg)

_Exemple d’application, peut être un poil trop généreuse…_

## 5. Conclusion

Depuis j’ai fait acheter une belle seringue de 50g au Repair-Café du coin, et j’y joue régulièrement les magiciens. Depuis je m’interroge aussi… Pourquoi n’ai-je jamais rien lu à ce sujet ? Pourquoi rien n’est fait pour qu’on puisse remplacer facilement cette pièce d’usure ? Pourquoi on vend au même prix des pâtes 16x plus (ou moins) efficaces que d’autres ? Cette société ne cessera de m’étonner.

En tout cas, quand on s’engage sur le démontage du microprocesseur d’un ordinateur portable, mieux vaut avoir choisi une pâte thermique faisant honneur à l’effort consenti. Il faut d’abord vérifier que la machine surchauffe même sans charge processeur. Le diagnostic s’affine si elle a quelques années au compteur. Enfin, il faut y aller si librement tourne son ventilateur.

J’espère vous avoir donné toutes les clés avec cet article, alors n’hésitez pas à acheter de belles seringues de pâte thermique, vous aurez probablement plus d’occasions de vous en servir que vous ne l’imaginez.