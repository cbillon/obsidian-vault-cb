---
link: https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part1/
byline: Enix
excerpt: Cette série de trois articles présente des solutions pour optimiser la taille des images Docker. Dans cette première partie, on parle surtout de *multi-stage build*, parce que dans la démarche de réduction de taille des images, ça devrait presque toujours être notre première étape. On va également expliquer les différences entre les bibliothèques statiques et dynamiques, et on expliquera pourquoi c'est important. Ce sera l'occasion de présenter la fameuse distribution Alpine Linux.
slurped: 2024-05-13T13:38:38.264Z
title: Chérie, j'ai rétréci Docker - part 1/3
tags:
  - docker
  - image
  - size
---

On se retrouve aujourd’hui pour la **première partie** de notre série consacrée à l’**optimisation de la taille des images Docker** :

- _“Chérie, j’ai rétréci Docker !”, Part 1/3 : nous y sommes !_
    
- [“Chérie, j’ai rétréci Docker !”, Part 2/3](https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part2/)
    
- [“Chérie, j’ai rétréci Docker !”, Part 3/3](https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part3/)
    

## Introduction

Lorsqu’on débute avec les conteneurs, on est souvent choqué par la taille des images générées. “Comment est-ce possible ? 1 giga pour un pauv' programme de rien du tout ?” On va voir qu’il est tout à fait possible de **réduire la taille des images**, et ce, sans sacrifier le confort des devs et des ops.

Dans cette **première partie**, on parlera surtout de _multi-stage build_, parce que dans la démarche de réduction de taille des images, ça devrait presque toujours être notre première étape. On va également expliquer les différences entre les bibliothèques statiques et dynamiques, et on expliquera pourquoi c’est important. Ce sera l’occasion de présenter la fameuse distribution Alpine Linux.

Dans la **deuxième partie**, on verra comment ça se passe avec quelques langages populaires comme Go, mais aussi Java, Node, Python, Ruby et Rust. On reviendra aussi sur Alpine et on verra plus en détail comment l’utiliser au mieux dans différentes situations.

Dans la **troisième partie**, on verra quelques patterns (et anti-patterns !) plus généraux, indépendants des langages et frameworks utilisés. Par exemple on parlera de _factorisation_ des images en utilisant des images de base communes ; de la suppression des symboles dans les binaires, ou encore de la réduction du poids des assets.

On terminera avec des méthodes plus exotiques ou plus avancées comme Bazel, Distroless, DockerSlim ou UPX. On verra comment certaines d’entre elles peuvent être contre-productives dans certains scénarios mais très utiles dans d’autres.

Afin de permettre à chacun·e d’expérimenter, tester, bidouiller … Nous avons créé un **dépôt GitHub public contenant tous les exemples** (code et Dockerfiles) mentionnés dans ces trois articles, avec un fichier Compose en bonus, permettant de construire toutes ces images d’un claquement de doigts (ou presque).

→ [https://github.com/jpetazzo/minimage](https://github.com/jpetazzo/minimage)

Quand on construit une image de conteneur (par exemple avec un Dockerfile et “docker build”), on est souvent désagréablement surpris par la taille des images.

Par exemple, ce petit “Hello World” de rien du tout, écrit en C :

```
/* hello.c */
int main () {
puts("Hello, world!");
return 0;
}
```

Si on construit une image avec ce Dockerfile :

```
FROM gcc
COPY hello.c .
RUN gcc -o hello hello.c
CMD ["./hello"]
```

… La taille de l’image dépasse joyeusement le giga-octet, car en plus du binaire compilé, elle embarque aussi tout le contenu de l’image de base `gcc` !

En utilisant `ubuntu` comme image de base, et en installant nous-mêmes le compilateur C, on obtiendrait une image de 300 Mo. C’est déjà mieux, mais c’est encore _beaucoup trop_ pour un binaire qui fait à peine 20 ko :

```
$ ls -l hello
-rwxr-xr-x 1 root root 16384 Nov 18 14:36 hello
```

Le résultat est le même pour ce programme écrit en Go :

```
package main

import "fmt"

func main () {
fmt.Println("Hello, world!")
}
```

On a une image de 800 Mo alors que le programme `hello` ne fait que 2 Mo :

```
$ ls -l hello
-rwxr-xr-x 1 root root 2008801 Jan 15 16:41 hello
```

Il doit bien y avoir des méthodes plus efficaces pour construire nos images …

On va voir qu’on peut réduire considérablement la taille de nos images grâce à différentes méthodes. Dans certains cas, on atteindra carrément une amélioration de 99,8% (et on verra aussi que ce n’est pas toujours une bonne idée d’aller aussi loin…).

**Conseil de pro** : pour comparer facilement la taille de plusieurs images, on peut utiliser le même nom mais avec des tags différents. Par exemple, en nommant nos images `hello:gcc`, `hello:ubuntu`, `hello:thisweirdtrick`, etc. il suffira d’exécuter la commande `docker images hello` pour lister toutes les variantes de notre image, avec leurs tailles respectives. (Sans lister les autres images stockées par notre Docker Engine.)

## Multi-stage Docker builds

C’est la **première technique à dégainer** car c’est bien souvent la plus efficace pour réduire la taille de nos images. Mais il faut faire attention à quelques petits détails, sans quoi nos images peuvent finir par être plus difficiles à opérer (voire ne pas marcher du tout).

Les multi-stage builds reposent sur cette idée simple: “Je n’ai pas besoin d’inclure le compilateur C ou Go et la toolchain dans mon image finale. Je veux juste le binaire !”

Comment fait-on un multi-stage build ? Il suffit d’ajouter une nouvelle ligne `FROM` dans notre Dockerfile. Par exemple comme ça :

```
FROM gcc AS mybuildstage
COPY hello.c .
RUN gcc -o hello hello.c
FROM ubuntu
COPY --from=mybuildstage hello .
CMD ["./hello"]
```

Dans ce Dockerfile, on a deux étapes, ou deux phases, selon comment vous voudrez traduire “stage” en français. L’étape de “build” utilise l’image `gcc` pour compiler `hello.c`. Puis, la deuxième instruction `FROM` démarre une nouvelle étape, qu’on appellera “l’étape d’exécution” (ou “run stage”). Cette étape utilise l’image `ubuntu`, à laquelle on ajoute le binaire `hello` de l’étape précédente.

L’image finale fait 64 Mo au lieu de 1,1 Go, ce qui permet une **réduction de taille d’environ 95%** :

```
$ docker images minimage
REPOSITORY TAG ... SIZE
minimage hello-c.gcc ... 1.14Go
minimage hello-c.gcc.ubuntu ... 64.2Mo
```

Pas mal, non ? Cela dit, on peut faire encore mieux. Mais avant de voir comment, on va voir quelques astuces et pièges à éviter …

Il n’est pas nécessaire d’utiliser le mot clé `AS` pour déclarer notre build stage. Lors de la copie de fichiers d’une étape précédente, on peut simplement indiquer le numéro de cette étape de build (en commençant à zéro).

Les deux lignes ci-dessous sont donc identiques :

```
COPY --from=mybuildstage hello .
COPY --from=0 hello .
```

Personnellement, j’aime bien utiliser des numéros dans les Dockerfiles très courts (disons, 10 lignes ou moins). Par contre, dès que le Dockerfile s’allonge (et devient plus complexe, par exemple avec _plusieurs_ build stage) il vaut mieux nommer les étapes de façon explicite. Ça facilite la maintenance pour les collègues qui bossent sur le code (et pour soi-même quand on revient sur le Dockerfile plusieurs mois plus tard).

### Warning : utilisez des images classiques

Je recommande fortement de n’utiliser que des images classiques pour votre stage “run”. Par “classique”, je pense à des images comme CentOS, Debian, Fedora, Ubuntu, et autres images officielles. Évitez les trucs obscurs dénichés sur le darkgithub … Si vous avez déjà entendu parler d’Alpine, vous brûlez peut-être de l’utiliser. C’est une bonne idée, mais attendez encore un peu pour ça : on parlera (en détail!) d’Alpine plus tard, et on expliquera pourquoi c’est une très bonne option à condition d’être prudent !

### Warning : `COPY --from` utilise des chemins absolus

Quand on copie des fichiers depuis une étape précédente, les chemins sont calculés par rapport à la racine de l’étape précédente. Le problème apparaît dès lors qu’on utilise une étape de build avec un `WORKDIR`, par exemple l’image `golang`.

Démonstration si on tente de `docker build` ce Dockerfile :

```
FROM golang
COPY hello.go .
RUN go build hello.go
FROM ubuntu
COPY --from=0 hello .
CMD ["./hello"]
```

On obtient cette erreur :

```
COPY failed: stat /var/lib/docker/overlay2/1be...868/merged/hello: no such file or directory
```

La commande `COPY` essaie de trouver le fichier `/hello`, mais vu que le `WORKDIR` dans `golang` est `/go` le chemin à utiliser est en fait `/go/hello`.

Quand on utilise des images officielles (ou très stables) dans notre build, on peut généralement spécifier le chemin absolu et ne plus y penser. Par contre, si nos images de build ou de run peuvent évoluer à l’avenir, il vaut mieux spécifier le `WORKDIR` directement dans l’image de build. Ca permet d’être sûr que les fichiers sont là où on les attend, et ce, même si l’image de base utilisée pour notre étape de build évolue plus tard.

Avec un `WORKDIR`, le Dockerfile pour build notre programme Go ressemblerait à ça :

```
FROM golang
WORKDIR /src
COPY hello.go .
RUN go build hello.go
FROM ubuntu
COPY --from=0 /src/hello .
CMD ["./hello"]
```

Avec tout ça, vous vous demandez peut-être si les builds multi-stage sont efficaces sur du code Go ? Eh bien, pour le petit programme ci-dessus, on passe de 800 Mo … à 66 Mo.

```
$ docker images minimage
REPOSITORY TAG ... SIZE
minimage hello-go.golang ... 805Mo
minimage hello-go.golang.ubuntu-workdir ... 66.2Mo
```

Pas mal !

## FROM scratch

Revenons à notre programme “Hello World” du début. La version en C est de 16 ko, la version en Go est de 2 Mo. Est-ce qu’on peut créer des images aussi petites ? Par exemple, construire une image avec _juste le binaire et rien d’autre ?_

Et bien oui ! Il suffit de faire du multi-stage build en utilisant l’image `scratch` comme image de run (avec des précautions qu’on verra un peu plus bas).

`scratch` est une image virtuelle. On ne peut pas la `docker pull` ni la `docker run` car elle est totalement vide. Donc quand un Dockerfile commence par « FROM scratch », ça veut dire qu’on construit _à partir de zéro_, sans code pré-existant.

Et ça nous donne le Dockerfile suivant :

```
FROM golang
COPY hello.go .
RUN go build hello.go

FROM scratch
COPY --from=0 /go/hello .
CMD ["./hello"]
```

Si on construit cette image, sa taille est exactement celle du binaire (2 Mo), et elle marche !

Par contre, il y a quelques éléments importants à avoir en tête lorsqu’on utilise `scratch` comme image de base.

### Absence de shell

L’image `scratch` n’inclut pas de shell, donc on ne peut pas utiliser de _string syntax_ avec `CMD` (ou `RUN`).

Par exemple, avec ce Dockerfile :

```
...
FROM scratch
COPY --from=0 /go/hello .
CMD ./hello
```

Si on tente un « docker run » sur l’image résultante, on obtient le message d’erreur suivant :

```
docker: Error response from daemon: OCI runtime create failed:
container_linux.go:345: starting container process caused
"exec:\"/bin/sh\": stat /bin/sh: no such file or directory": unknown.
```

Ce n’est pas très explicite, mais la dernière ligne nous dit exactement ce qui se passe, à savoir qu’il n’y a pas de `/bin/sh` dans l’image.

Pourquoi cette erreur ? Parce que lorsqu’on utilise la _string syntax_ avec `CMD` ou `RUN`, Docker va passer la chaîne en argument à `/bin/sh -c`. Concrètement, notre `CMD. /hello` ci-dessus exécute `/bin/sh -c "./hello"`, et comme on n’a pas de `/bin/sh` dans l’image `scratch`, on se fait recaler.

La solution est simple : on utilise la _JSON syntax_ dans le Dockerfile. `CMD /Hello` devient alors `CMD ["./hello"]`. Quand Docker voit la syntaxe JSON, il exécute les arguments directement, sans recourir à un shell. Bingo !

### Aucun outil de debug

On a vu que l’image « scratch » était vide… donc elle n’intègre rien non plus pour aider au débug de notre conteneur. Pas de `ls`, de `ps`, ni de `ping`, etc. Impossible d’entrer dans le conteneur pour le debug (avec `docker exec` ou `kubectl exec` par exemple).

(En fait, il existe quand même quelques méthodes de troubleshooting. On peut lancer un `docker cp` pour extraire les fichiers du conteneur ; faire un `docker run --net container:` pour interagir avec la stack réseau ; ou utiliser un outil bas niveau comme « nsenter » qui est encore plus flexible. Les versions récentes de Kubernetes intègrent aussi le concept de [conteneur éphémère](https://kubernetes.io/docs/concepts/workloads/pods/ephemeral-containers/) même s’il n’est qu’en version alpha. Mais bon, tout ça nous complique pas mal la vie, surtout quand on a autre chose à faire !)

On peut s’en sortir en utilisant des images comme `busybox` ou `alpine` à la place de `scratch`. Elles sont plus lourdes (1,2 Mo et 5,5 Mo respectivement), mais c’est un petit prix à payer comparé aux centaines de mégaoctets, voire gigaoctets de notre image d’origine.

### Pas de libc

On va maintenant s’aventurer en terrain un peu plus complexe. Notre “Hello World” en Go fonctionne bien dans l’image `scratch`, mais si on tente le coup avec un programme en C (ou avec un programme en Go plus complexe, par exemple utilisant des fonctions réseau), on obtient ce message d’erreur  :

```
standard_init_linux.go:211: exec user process caused "no such file or directory"
```

C’est clair comme de l’eau de roche. Il manque un fichier ! Oui, mais lequel ?

Il s’agit en fait d’une _bibliothèque dynamique_ nécessaire à notre programme.

Qu’est-ce qu’une _bibliothèque dynamique_ et pourquoi on en a besoin ?

Après compilation, un programme est _linké_ aux bibliothèques qu’il utilise. Même s’il est très simple, notre programme “Hello World” utilise la bibliothèque qui contient la fonction `puts`. Il y a longtemps, fort longtemps (avant les années 90), on faisait principalement du _static linking_ : toutes les bibliothèques utilisées par un programme étaient embarquées dans le binaire. C’est parfait pour un logiciel qu’on lance depuis une disquette, ou lorsqu’on n’utilise pas de bibliothèque standard. Mais dans un système multitâche comme Linux, on a plein de programmes qui tournent simultanément, et qui sont (généralement) stockés sur un disque dur ; et ces programmes utilisent presque toujours la bibliothèque C standard.

Dans ce scénario, il est plus avantageux d’utiliser du _dynamic linking_. Avec le dynamic linking, le binaire final ne contient pas le code de ses bibliothèques. A la place, il contient des références à ces bibliothèques, et ça peut donner quelque chose comme “ce programme nécessite les fonctions `cos`, `sin` et `tan` de `libtrigonometry.so`”. C’est seulement lorsque le programme est exécuté que le système recherche ce « libtrigonometry.so » et le charge pour que le programme puisse appeler les fonctions.

Le dynamic linking présente de multiples avantages :

1. Il économise de l’espace disque, car les bibliothèques communes n’ont pas à être dupliquées.
2. Il économise de la mémoire, car ces bibliothèques sont chargées une fois depuis le disque, puis ensuite partagées entre les programmes.
3. Il facilite la maintenance, car lorsqu’une bibliothèque est mise à jour on n’a pas besoin de recompiler tous les programmes qui utilisent cette bibliothèque.

(Pour être précis, les économies de mémoire ne sont pas réalisées grâce aux _bibliothèques dynamiques_ mais plutôt grâce aux _bibliothèques partagées_. Cela étant dit, les deux vont généralement de pair. Savez-vous aussi que sous Linux, les fichiers de bibliothèques dynamiques ont normalement l’extension `.so` qui signifie _shared object_ ? Sous Windows, ce sont les fameux `.DLL`, pour [Dynamic-link library](https://en.wikipedia.org/wiki/Dynamic-link_library).)

Revenons à nos moutons : par défaut, les programmes en C sont _liés dynamiquement_. (C’est aussi le cas de programmes en Go qui utilisent certains packages, comme on le verra dans la deuxième partie de cette série d’articles.) Notre programme utilise la bibliothèque C standard. Sur les systèmes Linux récents, cette bibliothèque se trouve dans le fichier `libc.so.6`. On a besoin de ce fichier dans l’image du conteneur pour lancer notre programme. Et avec l’image `scratch`, ce fichier est évidemment absent.

(Même chose si on utilise `busybox` ou `alpine`, car `busybox` ne contient pas de bibliothèques standard et `alpine` en utilise une autre qui est incompatible. On y reviendra plus tard.)

Alors, quelles sont nos solutions ?

On a au moins 3 options. 

### Construire un binaire statique

On peut dire à notre toolchain (le compilateur et les outils qui vont avec) de créer un binaire statique. Il y a pas mal de méthodes permettant de faire ça (selon ce qu’on utilise pour compiler), mais ici, avec l’image `gcc`, il suffit d’ajouter `-static` à la ligne de commande :

```
gcc -o hello hello.c -static
```

Le binaire créé fait maintenant 760 ko (sur mon système) au lieu de 16 ko. Vu que la bibliothèque est intégrée dans le binaire, l’image est forcément beaucoup plus grande, mais ce binaire va pouvoir fonctionner avec l’image `scratch`. (On pourrait obtenir une image encore plus petite si on construisait une image statique binaire _avec Alpine_. Le résultat serait inférieur à 100 ko ! Mais encore une fois, on reparlera d’Alpine plus en détail plus tard.)

### Ajouter les bibliothèques à notre image

On peut lister les bibliothèques nécessaires à notre programme avec l’outil `ldd` :

```
$ ldd hello
linux-vdso.so.1 (0x00007ffdf8acb000)
libc.so.6 => /usr/lib/libc.so.6 (0x00007ff897ef6000)
/lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007ff8980f7000)
```

Ça nous donne le chemin exact des fameuses bibliothèques sur le système.

Dans l’exemple au-dessus, la seule “vraie” bibliothèque est `libc.so.6`. La bibliothèque `linux-vdso.so.1` est en fait liée à un mécanisme appelé [VDSO (virtual dynamic shared object)](https://en.wikipedia.org/wiki/VDSO) qui sert à accélèrer certains appels système. Faisons comme si elle n’était pas là. Même chose pour `ld-linux-x86-64.so.2` : c’est le _linker_ qui s’occupe de résoudre les bibliothèques avant de lancer le programme.

(Techniquement, notre binaire `hello` contient des infos qui disent : “Hé, je suis un programme dynamique, et l’élément qui permet d’assembler mes parties c’est `ld-linux-x86-64.so.2`”.)

_Si on est vraiment motivé·e_, on peut ajouter manuellement à notre image tous les fichiers répertoriés par `ldd`. On voit bien que ce serait assez fastidieux et difficile à maintenir, surtout pour les programmes avec de nombreuses dépendances.

Pour notre “Hello World”, ça fonctionnerait. Mais pour un programme plus complexe (par exemple n’importe quel programme faisant des résolutions DNS), on rencontrerait un nouveau problème. La bibliothèque GNU C (utilisée sur la majorité des systèmes Linux) implémente DNS (et quelques autres éléments) via un mécanisme complexe appelé le _Name Service Switch_ (NSS). Ce mécanisme nécessite un fichier de configuration, `/etc/nsswitch.conf`, et des bibliothèques supplémentaires. Ces bibliothèques n’apparaissent pas avec `ldd` car elles sont chargées à la volée après le lancement du programme. Pour qu’une résolution DNS fonctionne correctement, on devrait donc également les inclure dans notre image ! (Elles se trouvent généralement dans `/lib64/libnss_ *`.)

Pour ma part, je ne recommande donc pas cette solution. Elle est complexe à mettre en oeuvre, difficile à maintenir et elle peut facilement devenir inopérante avec le temps.

### busybox:glibc

Une image a été conçue spécifiquement pour résoudre tous ces problèmes: `busybox:glibc`. C’est une petite image (5 Mo) utilisant `busybox` (avec de nombreux outils utiles pour le debug et l’opération) ainsi que la bibliothèque GNU C (ou « glibc »). Elle contient exactement tous les fichiers qui nous manquaient juste avant. C’est la solution la plus efficace pour exécuter un binaire dynamique avec une petite image.

Mais n’oubliez pas que si votre programme utilise d’autres bibliothèques, il faudra les copier elles aussi.

## Résumé et conclusion (partielle)

Bon, c’est l’heure du bilan ! Voyons ce que donnent nos différents efforts avec notre programme “Hello World” écrit en C …

_Attention, spoiler :_ cette liste comprend les résultats obtenus en utilisant Alpine, avec une méthode qui sera décrite dans la prochaine partie de cette série d’articles.

- Image originale construite avec `gcc` : 1,14 Go
- _Multi-stage build_ avec `gcc` et `ubuntu` : 64,2 Mo
- Binaire glibc statique dans `alpine` : 6,5 Mo
- Binaire dynamique dans `alpine` : 5,6 Mo
- Binaire statique dans `scratch` : 940 ko
- Binaire musl statique dans `scratch` : 94 ko

Ca nous fait quand même une réduction de taille de 12 000x, soit 99,99% d’espace disque (et de transfert réseau) en moins.

_Pas mal._

À titre personnel, je n’utilise pas les images `scratch` (car ça devient galère à déboguer et opérer) mais si vous pensez que c’est ce qu’il vous faut, faites vous plaisir !

Dans le prochain article de cette série, on mentionnera des points plus spécifiques au langage Go, comme cgo et tags. On s’attaquera également à d’autres langages populaires. Et puis on évoquera de façon plus approfondie ce qu’on peut faire avec Alpine, et on verra que pour obtenir des images légères, ça envoie du lourd. (Eh oui!)

RDV sur [“Chérie, j’ai rétréci Docker !”, Part 2/3](https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part2/)

---

Cet article a également été publié en version anglaise [ici](https://www.ardanlabs.com/blog/2020/02/docker-images-part1-reducing-image-size.html)

---

Ne ratez pas nos prochains articles DevOps et Cloud Native! Suivez Enix sur [Twitter](https://twitter.com/enixsas)!