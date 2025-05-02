---
link: https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part3/
byline: Enix
excerpt: Cette série de trois articles présente des solutions pour optimiser la taille des images Docker. On parle dans ce dernier article de standardisation d'image de base, d'optimisation d'assets, de systèmes de build alternatifs comme DockerSlim ou Bazel ou encore de la distribution NixOS.
slurped: 2024-05-13T13:41:04.707Z
title: Chérie, j'ai rétréci Docker - part 3/3
tags:
  - docker
  - image
  - size
---

La voilà ! La **dernière partie** de notre série sur l’**optimisation de la taille des images Docker** !

Si vous avez raté les deux premiers épisodes, vous pouvez les retrouver ici :

- [“Chérie, j’ai rétréci Docker !”, Part 1/3](https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part1/)
    
- [“Chérie, j’ai rétréci Docker !”, Part 2/3](https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part2/)
    

## Introduction

Dans les **deux premières parties**, on avait couvert les méthodes les plus courantes pour optimiser la taille de nos images Docker. On avait vu que les améliorations les plus convaincantes sont généralement obtenues grâce aux builds multi-stage avec des images basées sur Alpine, et qu’on pouvait faire encore mieux (au détriment de la maintenabilité) avec des builds statiques.

Dans cette **dernière partie**, on va voir comment on peut aller encore plus loin. On va parler de **standardisation d’images de base**, de **“stripping” de binaires**, d’optimisation d’assets, de **systèmes de build alternatifs** comme _DockerSlim_, _Bazel_ ou encore de la distribution _NixOS_.

On parlera aussi de petits détails mis de côté jusqu’à présent mais qui peuvent être importants, comme les fichiers de **timezone** ou les **certificats**.

## Partir sur de bonnes bases

Voilà une méthode qui va permettre de faire des économies significatives dès que vos nœuds font tourner plusieurs conteneurs en parallèle : le **partage de couches**.

Les images Docker sont constituées de _layers_ (des couches, en bon français). Et chaque couche peut ajouter, supprimer ou modifier des fichiers, exactement comme les _commits_ dans un dépôt de code ; ou encore, comme une classe qui hérite d’une autre classe. Lorsqu’on exécute un `docker build`, chaque ligne du Dockerfile va générer une couche. Et **lorsqu’on transfère une image, on transfère seulement les couches qui n’existent pas encore** sur la destination.

Si de nombreuses images partagent les mêmes couches, Docker n’a besoin de transférer et stocker la couche qu’une seule fois pour l’ensemble des images. Cela permet d’**économiser de la bande passante réseau et de l’espace de stockage**.

Suivant le _storage driver_ utilisé par Docker, les couches peuvent également permettre d’économiser des I/O disques et de la mémoire. Lorsque de nombreux conteneurs ont besoin de lire les mêmes fichiers d’une couche, le système va en effet lire et mettre en cache ces fichiers une seule fois. (C’est le cas notamment avec les drivers _overlay2_ et _aufs_).

On voit assez vite comment on peut **optimiser les ressources réseau, le disque et la mémoire sur les noeuds qui hébergent les conteneurs** : il faut que nos conteneurs (plus précisément, leurs images) partagent ces couches au maximum. Là où les choses se compliquent, c’est que cette méthode peut aller totalement à l’encontre des recommandations que je vous ai données dans les deux premiers articles de cette série !

Prenons un exemple où on build de superbes images ultra optimisées avec des binaires statiques. Ces binaires peuvent prendre au final 10 fois plus d’espace que leurs équivalents dynamiques. Je vous mets quelques scénarios où on fait tourner 10 conteneurs, chacun utilisant une image différente avec l’un de ces binaires :

Scénario 1: binaires statiques avec l’image `scratch`

- Taille de chaque image : 10 Mo
- Taille des 10 images: 100 Mo (quel calcul !)

Scénario 2: binaires dynamiques avec l’image `ubuntu` (64 Mo)

- Taille individuelle de chaque image : 65 Mo
- Répartition de la taille de chaque image : 64 Mo pour `ubuntu` + 1 Mo pour le binaire spécifique
- Utilisation disque totale : 74 Mo (10 x 1 Mo pour les couches spécifiques + 64 Mo pour les couches partagées)

Scénario 3: binaires dynamiques avec l’image `alpine` (5.5 Mo)

- Taille individuelle de chaque image : 6,5 Mo
- Répartition de taille de chaque image : 5.5 Mo pour `alpine` + 1 Mo pour le binaire spécifique
- Utilisation disque totale : 15.5 Mo

A première vue, les binaires statiques optimisés étaient une superbe idée. Mais on voit bien ici que dans un contexte avec de nombreux conteneurs, ils sont en fait totalement contre productifs. Les images utilisent plus d’espace disque, elles sont plus longues à transférer et elles utilisent plus de RAM. On peut difficilement faire pire par rapport à notre objectif initial !

Un détail important : pour que ces scénarios de partage des couches fonctionnent, on doit s’assurer que toutes les images utilisent exactement la même image de base. Si on a plusieurs images qui utilisent `centos` et d’autres qui utilisent `debian`, on n’optimise rien du tout. Ce n’est pas mieux si on utilise `ubuntu:16.04` et `ubuntu:18.04`. Et ce n’est pas mieux non plus si on utilise deux versions différentes de `ubuntu:18.04` !

Cela signifie également qu’à chaque fois qu’une image de base est mise à jour, on doit rebuild toutes nos images pour conserver une cohérence entre nos conteneurs.

Ça veut aussi dire qu’on va avoir besoin d’une **bonne organisation et d’une bonne communication entre les équipes**. Vous vous dites peut-être que ça n’est pas un problème technique, ça. C’est vrai, mais justement : c’est d’autant plus difficile à résoudre ! Il ne suffit plus de trouver la meilleure solution pour _son_ image. Si on veut avoir les meilleurs résultats, il faut se mettre d’accord avec ses collègues pour adopter une image qui convienne à tout le monde. Par exemple, peut-être que vous ou votre équipe voulez absolument utiliser une image de base Debian, alors que d’autres équipes préfèreraient utiliser une image de base Ubuntu. Il va donc falloir impliquer les autres et parler ensemble pour faire le meilleur choix possible. Dans ce genre de discussion, le plus important n’est pas toujours d’avoir les meilleurs arguments, mais d’être prêt·e à changer d’avis … Car si chacun·e campe sur ses positions, il n’y aura pas de consensus. Moralité : si vous voulez vraiment optimiser la taille de vos images au maximum, les solutions les plus efficaces seront aussi **obtenues grâce à des compétences interpersonnelles et pas uniquement techniques**.

Enfin, il y a un cas spécifique où les images statiques continuent d’avoir de l’intérêt. C’est lorsqu’on sait à l’avance que nos images vont être exécutées dans des environnements hétérogènes, ou lorsqu’elles seront la seule chose qui tourne sur un nœud donné. Dans ce cas, aucun partage de couche n’est possible entre les conteneurs et donc aucune optimisation de ce type.

## Nettoyage de binaires et compression

Dans ce chapitre, on aborde des **techniques complémentaires** qui ne sont pas spécifiques aux conteneurs mais qui peuvent nous faire économiser quelques mégaoctets (mais parfois seulement quelques kilooctets).

### Nettoyage de vos binaires

Par défaut, la plupart des compilateurs génèrent des binaires avec des symboles de debug. C’est pratique en cas d’incident, mais ça n’est pas strictement nécessaires à l’exécution. L’outil `strip` supprime ces symboles. Je vous accorde que l’économie réalisée n’est pas transcendante, mais ça peut être utile si vous êtes dans une situation où chaque octet compte.

### Et pour les fichiers médias dans nos conteneurs ?

Il y a quelques questions utiles à se poser dans le cas où notre image de conteneur contient des fichiers média (images, animations, sons, etc). Peut-on réduire leur taille, par exemple en utilisant des formats de fichier ou des codecs différents ? Peut-on les héberger à l’extérieur de l’image pour diminuer la taille de l’image à distribuer ? Cette dernière option est particulièrement intéressante si le code change souvent par rapport aux assets. Dans ce cas précis, c’est malin d’**éviter de redistribuer les assets à chaque nouvelle release de code**.

### Compression : la fausse bonne idée

Une solution qui nous vient toujours à l’esprit pour optimiser la taille de fichiers est la compression. Si on veut diminuer la taille de nos images, alors pourquoi ne pas compresser nos fichiers ? Des assets HTML, javascript ou autres CSS devraient bien se compresser avec zip ou gzip. Il existe même des méthodes plus efficaces comme bzip2, 7z ou encore Izma.

A première vue, c’est un moyen simple pour réduire la taille des images. Et si on prévoit de servir ces assets de manière compressée, alors pourquoi pas. Mais si on a besoin de décompresser ces assets pour les utiliser, on va au contraire gâcher des ressources. Voyons pourquoi…

**Les couches Docker sont déjà compressées** avant d’être transférées, on ne gagnerait donc rien en jouant sur la compression de nos images. Et si on compte décompresser les fichiers au lancement du conteneur, l’utilisation du disque sera encore pire : nous allons stocker à la fois les versions compressées et décompressées ! Pire encore, on ne peut pas optimiser la taille via le partage de couche qu’on a vu juste avant : ces fichiers n’étant décompressés que lorsqu’on exécute les conteneurs, ils ne seront pas partagés.

Et qu’en est-il de [UPX](https://upx.github.io/) ? Si vous n’êtes pas familier avec UPX, c’est un très bon outil pour réduire la taille des binaires. Il compresse le binaire et y ajoute une petite surcouche afin de le décompresser de façon transparente lors du lancement. Résultat : des binaires plus petits, en échange d’une courte attente au démarrage (à cause de la décompression).

Mais si on veut réduire la taille de nos conteneurs, UPX sera (lui aussi) contre productif.

Tout d’abord, comme pour la compression classique, le disque et l’usage réseau ne vont pas être optimisés car UPX n’apportera rien en complément de la compression native des couches Docker.

Ensuite, lorsqu’on lance un binaire classique, seules les portions nécessaires sont chargées en mémoire (comme lorsqu’on utilise `mmap()`). En revanche, lorsqu’on lance un binaire compressé avec UPX, tout le binaire doit être décompressé en mémoire. Ceci induit une utilisation mémoire plus élevée et des temps de démarrage rallongés, en particulier avec des runtimes comme Go qui tendent à générer des binaires plus gros.

(Quand je préparais l’examen pour devenir Certified Kubernetes Administrator, j’ai monté un lab de machines virtuelles KVM, dans lesquelles j’utilisais le binaire [hyperkube](https://stackoverflow.com/questions/33953254/what-is-hyperkube/33967582#33967582). Comme c’est un très gros binaire, j’ai voulu gagner de la place en le passant à la moulinette avec UPX. Le résultat à été catastrophique : j’ai réussi à diminuer la taille disque de mes VMs, mais l’utilisation mémoire a explosé.)

## Quelques techniques exotiques !

Il y a pas mal d’autres outils qui peuvent nous aider à réduire la taille de nos images Docker. La liste ci-dessous présente quelques uns de ces outils, mais elle n’est pas exhaustive.

### DockerSlim

[DockerSlim](https://github.com/docker-slim/docker-slim) s’appuie sur une technique de réduction de la taille des images qui est presque … magique. Je ne sais pas exactement comment ça marche dans le détail (au delà de la [description du design](https://github.com/docker-slim/docker-slim#design) dans le README), du coup je vais faire des suppositions avec vous.

J’ai l’impression que DockerSlim exécute notre conteneur, observe quels fichiers ont été consultés pendant son exécution, et supprime ceux qui n’ont pas été accédés. En se basant sur cette hypothèse, je me suis dis qu’il fallait vraiment faire attention lorsqu’on utilise DockerSlim, car de nombreux runtimes et frameworks chargent des fichiers dynamiquement, ou à la volée, seulement la première fois qu’ils sont nécessaires.

Pour vérifier cette hypothèse, j’ai essayé DockerSlim pour une application Django très simple. DockerSlim a réduit la taille de 200 Mo à 30 Mo. Pas mal ! La _home_ de l’application fonctionnait ; en revanche, de nombreux liens étaient cassés. Je suppose que les _templates_ (qui sont typiquement chargés à la demande par Django, sauf configuration spécifique) n’ont pas tous été détectés par DockerSlim et n’ont donc pas été inclus dans l’image finale. Pour la petite histoire, même les pages d’erreurs étaient cassées, probablement car les modules utilisés pour envoyer et afficher les exceptions avaient été zappés eux aussi. Je pense que n’importe quel code Python avec des modules dynamiques rencontrerait le même problème.

Est-ce que ça veut dire que je déconseille DockerSlim ? Non ! Dans certains cas, il peut faire des merveilles. **Comme pour tous les outils puissants, il faut comprendre comment il fonctionne pour savoir quand et comment l’utiliser** sans mauvaise surprise.

### Distroless

Les images [Distroless](https://github.com/GoogleContainerTools/distroless) sont une collection d’images minimales construites sans les _package managers_ classiques de distributions Linux et avec des outils extérieurs. On a **des images très petites**, mais un peu spartiates à mon goût : il manque les outils comme `ps`, `ls`, etc., qui sont bien pratiques en phase de debug.

Pour ma part, je préfère avoir un _package manager_ et une distribution familière. Qui sait si je ne vais pas avoir besoin d’un outil supplémentaire pour investiguer en live un problème de conteneur ? Alpine fait seulement 5,5 Mo et me permet d’installer tout ce dont je peux avoir besoin. Pour 5,5 Mo, je ne pense pas que ça vaille la peine de se priver de cette flexibilité.

De votre côté, si vous maîtrisez des méthodes permettant d’analyser vos conteneurs sans jamais avoir besoin d’exécuter des programmes au sein du conteneur, vous pouvez clairement optimiser vos images en utilisant des bases Distroless.

Au passage, les images basées sur Alpine sont souvent plus petites que leurs équivalentes Distroless. On peut alors se demander quel est l’intérêt de Distroless … Il y en a au moins deux.

Tout d’abord, la **sécurité**. Les images Distroless sont minimales. Moins de choses dans l’image signifie de facto moins de vulnérabilités potentielles.

Ensuite, les images Distroless sont construites avec **Bazel**. Si vous voulez apprendre et expérimenter avec Bazel, ces images sont donc de solides exemples pour démarrer. Vous vous demandez peut-être ce qu’est Bazel exactement ? Et bien, j’ai justement prévu d’en parler dans la section d’après !

### Bazel (et autres builders alternatifs)

Il existe des systèmes de build qui n’utilisent même pas de Dockerfiles. [Bazel](https://bazel.build/) est l’un d’entre eux. La force de Bazel est de **permettre de définir des dépendances complexes entre notre code source et les binaires qu’il crée**, un peu comme un Makefile. Ça nous permet de re-build uniquement ce qui est nécessaire, que ce soit dans notre code (suite à une petite modification locale) ou dans nos images de base (de sorte qu’on puisse appliquer un patch ou mettre à jour une bibliothèque sans devoir rebuild toutes nos images). Bazel permet aussi de réaliser des tests unitaires avec la même efficacité, et de n’exécuter que les tests des modules affectés par nos changements de code.

**Bazel est particulièrement efficace pour les programmes très gros et très complexes**. Quand on atteint une certaine complexité, le build et les tests peuvent prendre des heures. Puis des jours. Arrivés à ce stade, typiquement, on déploie des fermes de build et de test pour régler le problème. On retrouve des temps plus raisonnables, mais ça peut quand même prendre des heures, et surtout, ça demande énormément de ressources, et ça n’est souvent plus possible de faire tourner tous les tests en local. C’est à ce moment là que ça vaut le coup de dégainer un outil comme Bazel, parce qu’il est **capable de compiler et de tester uniquement le strict nécessaire**. Et au lieu de prendre des heures ou des jours, ça ne prend que quelques minutes.

Parfait ! Ça a l’air mortel ce truc, allez, on migre tout sur Bazel !

Hop hop hop. Pas si vite.

Utiliser Bazel nécessite d’apprendre un système de build complet et complexe. Beaucoup plus complexe que les Dockerfiles ; même en comptant les astuces de builds multi-stage dont on a parlé dans les premiers articles avec les subtilités sur les bibliothèques statiques et dynamiques.

**Maintenir ce système de build et les recettes associées n’est pas une mince affaire et nécessite un travail conséquent**. À titre personnel, je n’ai pas de retour d’expérience à faire sur Bazel, mais d’après ce que j’ai vu autour de moi, la maintenance d’un système de build et de test basé sur Bazel peut facilement occuper une personne senior à temps plein.

Pour les organisations qui comptent des centaines de développeurs qui perdent un temps considérable à cause de leurs build et de leurs tests, c’est probablement une bonne idée d’investir dans Bazel. A l’inverse, pour une petite start-up ou une équipe de taille modeste, ça risque d’être une décision peu pertinente à moins d’avoir la chance de disposer de quelques ingénieurs qui connaissent très bien Bazel et qui peuvent le gérer pour les autres.

## Nix

Après la publication des parties 1 et 2 de la série, plusieurs personnes sont venues me voir pour évoquer avec enthousiasme le [gestionnaire de package Nix](https://nixos.org/nix/). J’ai donc décidé de me pencher dessus et de lui consacrer une section entière dans cette dernière partie.

_Spoiler alert_ : oui, **Nix peut vraiment vous aider à améliorer vos builds, mais la courbe d’apprentissage est raide**. Peut-être pas autant que pour Bazel, mais pas loin. Vous allez devoir apprendre Nix, ses concepts, son [langage d’expression](https://nixos.wiki/wiki/Nix_Expression_Language) dédié, puis bien sûr comment l’utiliser pour packager votre code dans votre langage et votre framework préférés (jetez un œil au [manuel des nixpkgs](https://nixos.org/nixpkgs/manual/#chap-language-support) pour voir à quoi ça ressemble).

Cela étant dit, j’ai quand même envie de vous parler de Nix en détail, pour deux raisons : **ses concepts centraux sont très puissants** (et peuvent nous donner de bonnes idées sur le packaging de nos applications de façon plus générale), et il existe un **projet particulier appelé [Nixery](https://nixery.dev/)** qui sera un ajout utile à notre arsenal (qu’on décide d’utiliser Nix ou pas).

### Nix, de quoi s’agit-il ?

J’ai entendu parler de Nix pour la première fois il y a en gros 10 ans, alors que j’assistais à un talk aux RMLL “NixOS-The-Only-Functional-GNU-Linux-Distribution”. À l’époque, c’était déjà très solide. Autrement dit, on n’est pas du tout sur un truc de hipster avec rien derrière.

Commençons par un peu de terminologie :

- Nix est un _package manager_ à installer sur n’importe quelle machine Linux ou macOS ;
- NixOS est une _distribution Linux_ basée sur Nix ;
- `nixpkgs` est une collection de packages pour Nix ;
- Une “dérivation” est une recette de build avec Nix.

Nix est un _package manager_ fonctionnel. **“Fonctionnel”** signifie que chaque package est défini par ses *inputs” (code source, dépendances…) et par sa _dérivation_ (recette de build). Rien d’autre. Si on utilise les mêmes inputs avec la même dérivation, on aura toujours le même output. À l’inverse, à chaque fois qu’on modifie quelque chose sur un des inputs (mise à jour du code source, modification des dépendances), ou alors sur la dérivation, l’output est différent. Ca parait logique, non ? Et si ça vous rappelle le cache de build Docker, c’est normal… c’est exactement la même idée !

Sur un système traditionnel, quand un package dépend d’un autre, la dépendance est généralement décrite de manière souple, non stricte. Par exemple, sur Debian, [python3.8](https://packages.debian.org/bullseye/python3.8) dépend de `python3.8-minimal (= 3.8.2-1)`, qui dépend lui même de `libc6 (>= 2.29)`. En parallèle, [ruby2.5](https://packages.debian.org/bullseye/ruby2.5) dépend de `libc6 (>= 2.17)`. Du coup, on installe une seule version de `libc6` et généralement ça fonctionne.

Sur Nix, les package dépendent des **versions _exactes_ de ces bibliothèques** et un mécanisme très malin permet à **chaque programme d’utiliser son propre ensemble de bibliothèques sans conflit avec les autres**. (Si vous vous demandez comment ça marche : les programmes s’appuient sur un _linker_ configuré pour utiliser les bibliothèques à des chemins spécifiques. Conceptuellement, c’est comme lorsqu’on spécifie `#!/usr/local/bin/my-custom-python-3.8` pour lancer un script Python avec une version particulière de l’interpréteur.)

Par exemple, quand un programme utilise la bibliothèque C, sur un système classique le chemin serait `/usr/lib/libc.so.6`, alors qu’avec Nix, ça pourrait être `/nix/store/6yaj...drnn-glibc-2.27/lib/libc.so.6`.

Vous voyez ce chemin `/nix/store` ? C’est le _Nix store_ (étonnant !). Nix y stocke des fichiers et répertoires immuables, chacun identifié par son hash. Ça se rapproche des layers utilisés par Docker, avec une grosse différence : les layers Docker s’appliquent les uns par dessus les autres, alors que les fichiers et répertoires de Nix sont totalement disjoints, il n’y a jamais aucun conflit entre eux (puisque chaque objet est stocké dans un répertoire séparé). Ce détail aura son importance plus tard.

Sous Nix, lorsqu’on veut installer un package, on télécharge en fait un certain nombre de fichiers et de répertoires qu’on dépose dans le _Nix store_ puis on configure un [profil](https://nixos.org/nix/manual/#sec-profiles) (en gros, c’est un paquet de liens symboliques pour que les programmes qu’on installe soient accessibles depuis notre variable `$PATH`).

### Passons à l’expérimentation de Nix !

Jusqu’à présent, on était dans la théorie. Voyons maintenant **Nix en action** !

On peut lancer Nix dans un conteneur avec `docker run -ti nixos/nix`. Puis on peut lister les packages installés avec les commandes `nix-env --query` ou `nix-env -q`. À ce stade, on ne voit que `nix` et `nss-cacert`. Ça peut paraître bizarre : où sont passés le shell et les outils habituels comme `ls` ou autres ? Et bien dans cette image de conteneur, ils sont dans un exécutable statique, appelé `busybox`.

Bon, on va peut-être installer quelque chose ?

On exécute `nix-env --install redis` ou `niv-env -i redis`. La sortie de cette commande nous montre qu’on récupère de nouveaux “paths” et qu’ils sont stockés dans le store Nix. On va avoir au moins un “path” pour redis lui-même et très probablement un autre pour `glibc`. Nix utilise lui aussi glibc (pour le binaire `nix-env` et quelques autres), mais dans une version différente de celle utilisée par redis. Si on lance `ls -ld /nix/store/*glibc*/` on aura deux répertoires qui correspondent aux deux versions de glibc. Pendant que j’écris ces lignes, j’ai deux versions de `glibc-2.27` :

```
ef5936ea667f:/# ls -ld /nix/store/*glibc*/
dr-xr-xr-x    ... /nix/store/681354n3k44r8z90m35hm8945vsp95h1-glibc-2.27/
dr-xr-xr-x    ... /nix/store/6yaj6n8l925xxfbcd65gzqx3dz7idrnn-glibc-2.27/
```

Vous allez me dire : “Attends, mais ce sont pas les mêmes versions ?”. Et bien oui et non ! C’est bien le même numéro, mais ces versions ont sûrement été construites avec des options ou des patchs légèrement différents. **Du point de vue de Nix, ce sont donc deux objets distincts**. C’est exactement comme quand on build son programme avec le même Dockerfile alors qu’on a modifié une ligne de code quelque part : le builder Docker identifie ces petites différences et nous donne des images différentes.

On peut aussi **demander à Nix de nous donner les dépendances de n’importe quel fichier situé dans le store Nix**. Pour ça, on utilise les commandes `nix-store --query --references` ou `nix-store -qR`. Par exemple, pour voir les dépendances des binaires redis qu’on vient d’installer juste avant, on peut faire un `nix-store -qR $(which redis-server)`.

Dans mon conteneur, l’output ressemble à ça :

```
/nix/store/6yaj6n8l925xxfbcd65gzqx3dz7idrnn-glibc-2.27
/nix/store/mzqjf58zasr7237g8x9hcs44p6nvmdv7-redis-5.0.5
```

Et là, ça va être magique. Si on veut lancer Redis, on a besoin seulement de ces répertoires. Rien d’autre. Ça veut dire qu’on peut déposer ces répertoires dans `scratch`, et ça _marche_. On n’a besoin d’aucune librairie supplémentaire.

On peut généraliser cette procédure en utilisant un _profil_ Nix. Un profil contient le répertoire `bin` à ajouter à notre `$PATH` (et quelques autres trucs, mais je simplifie pour expliquer plus facilement). Si on fait `nix-env --profile myprof -i redis memcached`, le répertoire `myprof/bin` va donc contenir les exécutables pour Redis et pour Memcached.

Encore mieux, les profils sont eux aussi stockés dans le store. Du coup on peut lancer la commande `nix-store -qR` pour lister les dépendances d’un profil.

### Créer des images minimales avec Nix

Si on met bout à bout tout ce que j’ai expliqué dans la section précédente, on peut écrire le Dockerfile suivant :

```
FROM nixos/nix
RUN mkdir -p /output/store
RUN nix-env --profile /output/profile -i redis
RUN cp -va $(nix-store -qR /output/profile) /output/store
FROM scratch
COPY --from=0 /output/store /nix/store
COPY --from=0 /output/profile/ /usr/local/
```

La première phase utilise Nix pour installer Redis dans un nouveau _profil_. Ensuite on lui demande de lister toutes les dépendances de ce profil (la commande `nix-store -qR`) et on copie ces dépendances dans `/output/store`.

La seconde phase copie ces dépendances dans l’image finale, dans `/nix/store` (c.a.d. leur place d’origine dans Nix), ainsi que le profil lui même. (Copier le profil nous permet de récupérer un répertoire `bin` qui contient tous les binaires qu’on veut dans notre `$PATH`. Pratique.)

Le résultat est une image de 35Mo avec Redis et _rien d’autre_. Si on veut aussi un shell, on peut simplement mettre à jour le Dockerfile avec un `-i redis bash`, et voilà !

Si vous êtes tenté de ré-écrire tous vos Dockerfiles de cette façon là, attendez un peu. Déjà, cette image ne contient pas certaines metadata cruciales comme `VOLUME`, `EXPOSE`, ou encore `ENTRYPOINT` et son wrapper associé. Et surtout j’ai encore mieux pour vous dans la section suivante …

### Nixery

Tous les gestionnaires de package fonctionnent de la même façon : ils téléchargent (ou génèrent) des fichiers et les installent sur notre système. Mais avec Nix, il y a une différence importante : les fichiers installés sont immuable. Quand on installe des packages avec Nix, ça n’a aucun impact sur ce qu’on avait fait juste avant. Les couches Docker peuvent s’affecter mutuellement (parce qu’une couche peut changer ou supprimer un fichier de la couche précédente), mais ce n’est pas le cas des objets stockés dans le Nix store.

Si on regarde le conteneur Nix qu’on a lancé un peu plus tôt (ou qu’on en lance un nouveau avec `docker run -ti nixos/nix`), dans le répertoire `/nix/store`, on va trouver une liste de répertoires comme ceux-là :

```
b7x2qjfs6k1xk4p74zzs9kyznv29zap6-bzip2-1.0.6.0.1-bin/
cinw572b38aln37glr0zb8lxwrgaffl4-bash-4.4-p23/
d9s1kq1bnwqgxwcvv4zrc36ysnxg8gv7-coreutils-8.30/
```

Si on utilisait Nix pour construire une image (comme on l’a fait avec le Dockerfile plus haut), on aurait juste besoin de ces répertoires dans `/nix/store` et d’un paquet de liens symboliques.

Maintenant, imaginez qu’on stocke chaque répertoire du Nix store comme un _layer_ dans une _registry_ Docker.

Ensuite, quand on a besoin d’une image avec les packages X, Y et Z, il suffit de :

- générer un petit layer avec les liens symboliques évoqués, permettant d’invoquer facilement les programmes contenus dans ces packages X, Y et Z (ce qui correspond à la dernière ligne `COPY` dans le Dockerfile précédent) ;
- demander à Nix quels sont les objets du store correspondant à X, Y et Z ainsi qu’à leurs dépendances ;
- générer un manifest d’image Docker qui référence tous ces layers.

C’est exactement ce que [Nixery](https://nixery.dev/) fait. Nixery, c’est un registre de conteneurs “magique” : il **génère à la volée des manifests d’images de conteneurs qui référencent des layers qui correspondent aux objets du Nix store**.

Concrètement, si on fait un `docker run -ti nixery.dev/redis/memcached/bash bash`, on obtient un shell dans un conteneur avec Redis, Memcached et Bash. Ce qui est beau, c’est que l’image de ce conteneur est générée à la volée, _sans avoir besoin de la construire_. (Notez qu’on devrait plutôt faire un `docker run -ti nixery.dev/shell/redis/memcached sh`, car quand une image commence par `shell`, Nixery nous donne quelques packages essentiels en plus du shell, par exemple `coreutils`).

Nixery implémente quelques optimisations en prime (notamment pour gérer la limite maximale de nombre de _layers_ dans Docker). Si vous êtes intéressé vous pouvez lire [ce blog post](https://grahamc.com/blog/nix-and-layered-docker-images) ou [ce talk de NixConf](https://www.youtube.com/watch?v=pOI9H4oeXqA).

### D’autres manières d’utiliser Nix

On peut aussi utiliser Nix directement pour générer des images de conteneurs. Vous trouverez un bon exemple dans ce [blog post](https://lethalman.blogspot.com/2016/04/cheap-docker-images-with-nix_15.html).

Notez néanmoins que la technique évoquée ici [nécessite kvm](https://twitter.com/jpetazzo/status/1241741547751845888), ce qui veut dire qu’elle ne fonctionnera pas dans des instances Cloud ou dans des conteneurs (sauf en utilisant de la virtualisation imbriquée, mais c’est encore très rare). Apparemment, il faut adapter les exemples et [utiliser buildLayeredImage](https://twitter.com/tazjin/status/1241743569888649218) mais n’ayant pas investigué plus loin, je ne peux pas préciser la quantité de travail et la complexité.

### To Nix or not to Nix ?

Ce blog post ayant vocation à rester court (on me dit dans l’oreillette que c’est raté😅), je n’ai pas la place d’expliquer en détail comment utiliser Nix pour générer l’image parfaite. Mais j’espère que cette petite démo de Nix vous permet de vous faire une idée. À vous d’approfondir le sujet si ça vous a ouvert l’appétit !

Pour ma part, je compte bien utiliser Nixery quand j’ai besoin d’images de conteneurs pour un usage précis, en particulier avec Kubernetes. Prenons un exemple, si j’ai besoin d’une image avec `curl`, `tar` et la CLI de AWS. Mon approche traditionnelle aurait été d’utiliser `alpine` puis d’exécuter `pip install awscli`. Mais avec Nixery, je vais simplement pouvoir utiliser l’image `nixery.dev/shell/curl/gnutar/awscli` !

## Avant de conclure, quelques détails importants…

Lorsqu’on utilise des images vraiment très réduites (comme `scratch`, ou d’une certaine manière `alpine`, ou même certaines images générées avec distroless, Bazel ou Nix), on peut être confronté à des **erreurs inattendues**. Il y a des fichiers auxquels on ne pense pas en général, mais qui sont supposés se trouver sur tout système UNIX bien élevé, y compris dans un conteneur.

De quels fichiers parle-t-on exactement ? Et bien, voici une petite liste non exhaustive :

- les certificats TLS ;
- les fichiers de timezone ;
- la base d’utilisateurs et groupes (UID/GID).

Voyons de quoi il s’agit plus précisément, de quand nous pouvons en avoir besoin, et comment on peut les ajouter à nos images.

### Les certificats TLS

Quand on établit une connection TLS vers un serveur distant (par exemple, une requête vers un service web ou vers une API en HTTPS), en général il nous présente son certificat. En général, ce certificat a été signé par une _autorité de certification_ reconnue. En général, aussi, on veut vérifier que ce certificat est valide et qu’on connait l’autorité qui l’a signé.

(Je dis “en général” car il y a de rares scénarios où on ça nous est égal, ou bien où l’on valide la connection différemment. Mais si vous êtes dans un de ces cas-là, vous devez être au courant. Sinon, par défaut, on part du principe qu’on _doit_ valider les certificats ! Et oui, la sécurité avant tout !).

La clé (sans jeu de mot) de la vérification des certificats réside dans ces fameuses autorités de certification. Pour valider les certificats des serveurs auxquels on se connecte, on a besoin des certificats de ces autorités. Ils sont typiquement installés dans `/etc/ssl`.

Si on utilise `scratch` ou une autre image minimale, et si on se connecte à un serveur TLS, on peut obtenir des erreurs de validation des certificats. En Go, ça ressemble à ce message d’erreur là `x509: certificate signed by unknown authority`. Si ça nous arrive, il suffit de rajouter les certificats dans notre image. Pour ça, on peut les récupérer de quasiment toutes les images classiques comme `ubuntu` ou `alpine`. Peu importe laquelle, les systèmes ont quasiment tous le même groupe de certificats.

La ligne suivante va s’en occuper :

```
COPY --from=alpine /etc/ssl /etc/ssl
```

On voit aussi que si on veut juste copier des fichiers depuis une image, on peut utiliser `--from` pour se référer à cette image, sans référer à un _build stage_ particulier !

### Les fichiers de timezone

Si notre code manipule la date et l’heure, en particulier l’heure _locale_ (par exemple, si on affiche l’heure dans un fuseau horaire particulier, par opposition à une horloge interne), on a besoin des _fichiers de timezone_. Vous pourriez vous dire : “Attends, mais comment ça Jérôme ? Si je veux manipuler les fuseaux horaires, j’ai seulement besoin d’un offset par rapport à UTC ou GMT !”. Oui, sauf que … il y a les changements d’heure d’été et d’heure d’hiver. Là, les choses se compliquent très vite, car différents pays ou régions n’ont pas tous un passage à l’heure d’été, ou bien pas au même moment. Par exemple, au sein même de l’Utah aux États-Unis, les règles changent selon qu’on est en territoire Navajo ou pas ! Et puis par ailleurs, au fil des années, ces règles évoluent. En Europe par exemple, le changement d’heure devrait être supprimé par tous les pays en 2021.

Revenons-en à notre problématique technique. Si on veut pouvoir afficher l’heure locale, on va avoir besoin des fichiers qui décrivent ces informations. Sur UNIX, ce sont les fichiers `tzinfo` ou `zoneinfo`, on les trouve généralement dans le répertoire `/usr/share/zoneinfo`.

Certaines images (par exemple `centos` ou `debian`) incluent nativement ces fichiers de timezone. D’autres non, comme `alpine` ou `ubuntu`, et le package qui inclut ces fichiers s’appelle généralement `tzdata`.

Pour installer les fichiers de timezone dans notre image, on peut par exemple lancer :

```
COPY --from=debian /usr/share/zoneinfo /usr/share/zoneinfo
```

Ou si on utilise déjà `alpine`, on peut simplement faire un `apk add tzdata`.

Pour vérifier que les fichiers de timezone sont bien installés, on peut faire cette commande dans notre conteneur :

Si on obtient quelque chose comme `Fri Mar 13 21:03:17 CET 2020`, on est bon. Si on obtient `UTC`, ça signifie que les fichiers de timezone n’ont pas été trouvés.

### Les fichiers de mapping UID/GID

Une autre chose dont notre programme pourrait avoir besoin : rechercher les identifiants système des utilisateurs (User ID / UID) et de groupes d’utilisateurs (Group ID / GID). Généralement, ça se passe dans `/etc/passwd` et `/etc/group`.

Le seul cas où j’ai été obligé de fournir ces fichiers explicitement, c’était en exécutant des applications de bureau dans des conteneurs (en utilisant des outils comme [clink](https://github.com/soulshake/clink) ou les [dockerfiles](https://github.com/jessfraz/dockerfiles) de [Jessica Frazelle](https://twitter.com/jessfraz).

Si vous avez besoin d’installer ces fichiers dans un conteneur, vous pouvez les générer localement, ou dans un conteneur (single-stage ou multistage), ou alors vous pouvez faire un _bind-mount_ depuis l’hôte (tout dépend de ce que vous êtes en train de faire).

[Ce blog post](https://medium.com/@chemidy/create-the-smallest-and-secured-golang-docker-image-based-on-scratch-4752223b7324) nous montre comment ajouter un utilisateur à un conteneur de build, et ensuite copier les fichiers `/etc/passwd` et `/etc/group` au conteneur de run.

## Conclusions

Comme on a pu le voir, il existe plein de méthodes pour réduire la taille de nos images. Vous vous demandez probablement laquelle est la meilleure de façon absolue ? Et bien j’ai une mauvaise nouvelle : il n’y en a pas. Comme toujours, la réponse est _ça dépend_.

Les **multi-stage builds basés sur Alpine** vont donner d’excellents résultats dans de nombreux scénarios. Mais certaines bibliothèques ne vont pas être disponibles sur Alpine, et les compiler à la main peut prendre du temps. Dans ce cas un multi-stage build avec une distribution classique va faire le job plus efficacement.

Les mécanismes comme **Distroless** ou **Bazel** peuvent apporter des résultats encore meilleurs, mais ils vont souvent nécessiter un investissement significatif en amont qui n’est pertinent que pour certaines organisations.

Les **binaires statiques et l’image `scratch`** peuvent être bien utiles lorsqu’on déploie dans des environnements de très petite taille, comme des systèmes embarqués par exemple.

Un dernier point important pour finir … Si on build et maintient de nombreuses images (des centaines, voire plus), on peut vouloir s’en tenir à une seule technique, même si elle n’est pas optimale dans tous les cas. C’est en effet souvent plus confortable de maintenir des **centaines d’images avec une structure similaire** plutôt que d’avoir des pléthores de variations avec des builds exotiques ou des Dockerfiles de niche.

On approche de la fin de ce tour d’horizon sur l’optimisation des images Docker. Avant de vous laisser, sachez que **si vous utilisez des techniques que je n’ai pas mentionnées** dans cette série d’article, je serai ravi de continuer à en discuter avec vous, alors [contactez-moi](mailto:jerome.petazzoni@gmail.com?subject=Cherie+jai+retreci+docker+post) et réparons ces oublis !

### Mot de fin et remerciements

L’idée d’écrire cette série d’articles m’est venue en voyant ce [tweet](https://twitter.com/ellenkorbes/status/1216458929636630533) de [@ellenkorbes](https://twitter.com/ellenkorbes). Pendant mes formations sur les conteneurs, je passe toujours un peu de temps à expliquer comment réduire la taille des images. Pour évoquer le linking statique vs dynamique par exemple, je suis obligé de prendre des raccourcis pour ne pas y passer la journée. Ca laisse un peu un goût d’inachevé et m’a fait me demander s’il était finalement si nécessaire d’évoquer tous ces détails. Puis quand j’ai vu le **tweet d’Ellen** et certaines de vos réponses, je me suis dit : “Wow, mais en fait ça pourrait aider pas mal de monde si j’écrivais tout ce que j’ai pu expérimenter ces dernières années sur ces sujets !”.

… Je ne sais plus trop ce qui s’est passé ensuite, mais je me suis **réveillé à côté d’une caisse de Club Mate vide et de trois blog posts** ! 🤷🏻

Si vous cherchez des ressources géniales pour faire tourner du Go sur Kubernetes (et autres sujets connexes), je vous recommande vivement de consulter la [liste des talks](http://ellenkorbes.com/#talks) proposée par Ellen. La plupart de ces talks sont [disponibles sur Youtube](https://www.youtube.com/results?search_query=ellen+korbes), et je vous promets que c’est un très bon investissement de votre temps. En particulier, si vous avez aimé cette série sur la taille des images Docker, vous pouvez regarder son futur talk [The Quest for the Fastest Deployment Time!](https://www.youtube.com/watch?v=WgliN_9j91g)

Un **grand merci aussi aux nombreuses personnes qui m’ont fait des retours et des suggestions d’amélioration** ! Je pense en particulier à :

- [David Delabassée](https://twitter.com/delabassee) pour ses conseils sur Java et `jlink` ;
- [Sylvain Rabot](https://twitter.com/sylr) pour les certificats, les timezone et les fichiers UID et GID ;
- [Gleb Peregud](https://twitter.com/gleber) et [Vincent Ambo](https://twitter.com/tazjin) pour le partage de ressources très utiles à propos de Nix.

Ces articles ont été écrits originalement en anglais. La **version anglaise a été relue** par [AJ Bowen](https://twitter.com/s0ulshake). Elle a corrigé de nombreuses fautes de frappe et pas mal d’erreurs, contribuant à améliorer significativement ce que j’avais écrit. Toutes les erreurs restantes sont les miennes. AJ travaille en ce moment sur un projet de préservation de cartes postales anciennes et historiques. Si ça vous intéresse, je vous invite vivement à vous inscrire [ici](https://www.ephemerasearch.com/) pour en savoir plus.

La version française de cette série a été **traduite** par [Aurélien Violet](https://twitter.com/brimstone75) et par [Romain Degez](https://twitter.com/rdegez). Si vous avez aimé cette version française, vous pouvez leur témoigner un _grand merci_, ça a représenté bien plus de travail qu’il n’y paraît !

---

La version anglaise de cet article est disponible [ici](https://www.ardanlabs.com/blog/2020/04/docker-images-part3-going-farther-reduce-image-size.html).

---

Ne ratez pas nos prochains articles DevOps et Cloud Native! Suivez Enix sur [Twitter](https://twitter.com/enixsas)!