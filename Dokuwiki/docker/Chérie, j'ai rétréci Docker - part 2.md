---
link: https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part2/
byline: Enix
excerpt: Cette série de trois articles présente des solutions pour optimiser la taille des images Docker. On parlera dans ce second article de l'application aux différents langages populaires comme Go, Python, Java, Node, Rust... et nous reviendront plus en détails sur la solution efficace basée sur Alpine Linux.
slurped: 2024-05-13T13:40:01.844Z
title: Chérie, j'ai rétréci Docker - part 2/3
tags:
  - docker
  - image
  - size
---

On se retrouve aujourd’hui pour la **deuxième partie** de notre série consacrée à l’**optimisation de la taille des images Docker** :

- [“Chérie, j’ai rétréci Docker !”, Part 1/3](https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part1/)
    
- _“Chérie, j’ai rétréci Docker !”, Part 2/3 : nous y sommes !_
    
- [“Chérie, j’ai rétréci Docker !”, Part 3/3](https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part3/)
    

## Introduction

Dans la première partie, on avait déjà couvert pas mal de méthodes intéressantes et les concepts Linux qui allaient avec : le **multi-stage build**, l’impact du **static** et du **dynamic linking** sur le build des images, et on avait aussi commencé à parler de la fameuse distribution Alpine Linux et son intérêt pour les containers.

Dans cette deuxième partie, on va d’abord se plonger dans quelques détails spécifiques au **langage Go**. Puis on va revenir de façon bien plus approfondie sur **Alpine Linux** ainsi que musl, la bibliothèque système utilisée par Alpine. Enfin, on va passer en revue des méthodes utilisables pour d’autres **langages populaires** comme **Java**, **Node**, **Python**, **Ruby** et **Rust**.

## Golangdorak Go !

_(Désolés.)_

Vous avez peut-être déjà entendu ici ou là que les programmes écrits en Go étaient très faciles à déployer, car lors de la compilation, Go inclut directement toutes dépendances nécessaires dans le binaire. Du coup, il suffit de copier le binaire (pas besoin d’autres fichiers ou bibliothèques). **Pratique**.

Cela voudrait-il dire que Go génère des binaires statiques, qui vont par conséquent être faciles à Dockeriser ? Presque. (Si vous ne savez plus ce qu’est un binaire statique, je vous donne rendez-vous sur la première partie de cette série.)

En fait, il y a certains _packages_ Go qui s’appuient sur des bibliothèques système. Par exemple, tout ce qui concerne la résolution DNS est (par défaut) délégué à des bibliothèques système, qui peuvent être configurées de diverses manières. (Vous avez peut-être l’habitude des fichiers `/etc/resolv.conf` et `/etc/hosts`, mais il y en a d’autres.) **Quand on écrit du code en Go qui utilise ces bibliothèques, on n’a pas le choix** : le compilateur doit générer un binaire dynamique. En pratique, cela passe par un mécanisme appelé **cgo**, qui permet à Go d’utiliser du code C et des bibliothèques dynamiques.

Ca signifie qu’un programme en Go qui utilise par exemple le package `net` va générer un binaire dynamique avec les mêmes contraintes qu’un programme en C. D’un point de vue image Docker, on va donc à nouveau devoir copier les bibliothèques manuellement ou utiliser une image comme `busybox: glibc`.

Cela étant dit, il est possible de complètement **désactiver cgo**. En ce cas, au lieu d’utiliser les bibliothèques système, Go va alors utiliser ses propres réimplémentations de ces bibliothèques. Par exemple, au lieu d’utiliser le _resolver_ DNS du système, il va utiliser son propre _resolver_. On va alors pouvoir obtenir un binaire statique.

Pour désactiver cgo, il suffit de paramétrer la variable d’environnement `CGO_ENABLED=0`.

Ça donne par exemple :

```
FROM golang
COPY whatsmyip.go .
ENV CGO_ENABLED=0
RUN go build whatsmyip.go

FROM scratch
COPY --from=0 /go/whatsmyip .
CMD ["./whatsmyip"]
```

Comme on a désactivé cgo, Go n’utilise aucune bibliothèque système.

Comme Go n’utilise aucune bibliothèque système, il peut générer un binaire statique.

Comme ce binaire est statique, il fonctionnera très bien dans l’image `scratch` !

### Tags et netgo

On peut aussi **dire à Go quelle implémentation utiliser** (bibliothèque système ou version “pur Go”) de manière individuelle, _package_ par _package_. Pour ça, on utilise des _tags_ de compilation Go, qui permettent d’indiquer spécifiquement quels fichiers doivent être utilisés ou ignorés lors du _build_. Par exemple, en activant le tag “netgo”, on va demander à Go d’utiliser sa bibliothèque `net` native plutôt que de s’appuyer sur les bibliothèques systèmes.

```
go build -tags netgo whatsmyip.go
```

Si le code n’inclut aucun autre package nécessitant des bibliothèques systèmes, on obtiendra bien un binaire statique. Le problème, c’est que si un autre package s’appuie aussi sur cgo, on va revenir au point de départ. Si le but final est d’obtenir est binaire ststique, il vaut mieux **positionner la variable d’environnement** `CGO_ENABLED=0` : c’est plus sûr !

Ces tags peuvent aussi servir à indiquer **quel code on souhaite compiler** en fonction de l’architecture du CPU ou de l’OS. Par exemple, si on veut utiliser du code différent selon que le programme doit tourner sous Linux ou Windows, ou sur un processeur Intel ou ARM, on peut utiliser les tags pour indiquer au compilateur “utilise ce fichier uniquement lorsqu’on compile pour Linux, Windows, Intel, ARM, etc.”

## Alpine

On avait rapidement évoqué Alpine dans le premier article, je vous avais dit qu’on verrait plus tard les détails et **son efficacité pour construire nos images**. Et bien … c’est maintenant !

**Alpine est une distribution Linux** que la plupart des gens considéraient encore il y a quelques années comme “exotique”. Ses principales caractéristiques sont sa petite taille (en termes d’espace disque utilisé) et sa sécurité. Elle utilise son propre package manager, `apk`.

**À la différence des grosses distributions bien connues** comme CentOS ou Ubuntu, Alpine n’est pas maintenue par une armée de gens payés par des grandes entreprises comme Red Hat ou Canonical. Alpine a beaucoup moins de package que ces distributions-là : par défaut, elle doit en avoir dans les 10 000, alors que Debian, Fedora et Ubuntu en ont chacune plus de 50 000.

**Avant l’arrivée des conteneurs**, Alpine n’était pas très répandue, probablement parce que pour la majorité des gens, la taille du système Linux installé sur leur ordinateur a peu d’importance. La taille des programmes, bibliothèques et fichiers systèmes est en effet négligeable comparée à celle des autres documents et fichiers médias qu’on stocke sur nos ordinateurs (i.e. les photos et vidéo pour les ordinateurs personnels, ou alors les bases de données pour les serveurs d’entreprise).

**Alpine s’est démocratisée lorsque les gens se sont rendus compte que ce serait un excellent choix pour les conteneurs**. Je vous ai dit plus haut qu’Alpine était de petite taille. Petit comment, exactement ? Quand les conteneurs sont devenus populaires, tout le monde s’est rendu compte que les images avaient tendance à être lourdes. Elles prennent de la place sur les disques et leur téléchargement prend des plombes (vous, là, ce serait pas un peu pour cette raison que vous lisez cet article ?).

Les premières images de base de conteneurs utilisaient des **“cloud images”**, qui avaient fait leurs preuves notamment sur des machines virtuelles en mode cloud. Ces images avaient des tailles allant de quelques centaines de Mo à quelques Go. C’est une taille acceptable quand on travaille avec des serveurs en datacenter ou dans le cloud, car les images transitent uniquement en local entre un stockage et une machine virtuelle, généralement reliés entre eux par réseau local très rapide. Par contre, quand il s’agit de récupérer les mêmes images avec une connexion ADSL ou câble, ce n’est plus la même histoire : on voit le téléchargement passer !

Du coup, les mainteneurs des différentes distributions se sont attelés **à la tâche pour créer des images plus petites**, spécialement pour les conteneurs. Mais en dépit d’efforts louables, le résultat est sans appel : alors que des distributions comme Debian, Ubuntu et Fedora ont beaucoup de mal à passer sous la barre des 100 Mo (et parfois en enlevant des bibliothèques ou des outils bien utiles comme `ifconfig` ou `netstat`), **Alpine a mis tout le monde d’accord avec une image de 5 Mo seulement**, sans sacrifier aucun de ces outils.

Pour moi, il y a un autre avantage à Alpine Linux : **son _package manager_ est ultra rapide**. La vitesse d’un _package manager_ n’est pas fondamentale pour un système Linux normal, car généralement on installe chaque soft ou chaque outil qu’une seule fois. C’est très différent avec les conteneurs, car on build des images sans arrêt ; et très souvent, on lance un conteneur avec une image de base, dans laquelle on installe quelques outils spécifiques pour faire des tests ponctuels.

Juste pour le fun, je me suis penché sur quelques images de base populaires et j’ai **comparé le temps nécessaire pour installer `tcpdump` dessus**. Voilà le résultat :

|Image de base|Taille|Temps pour installer tcpdump|
|---|---|---|
|alpine:3.11|5.6 Mo|1-2s|
|archlinux:20200106|409 Mo|7-9s|
|centos:8|237 Mo|5-6s|
|debian:10|114 Mo|5-7s|
|fedora:31|194 Mo|35-60s|
|ubuntu:18.04|64 Mo|6-8s|
|.|||

La taille est celle donnée par `docker images`. Le temps est mesuré en lançant la commande `time docker run <image> <packagemanager> install tcpdump` plusieurs fois de suite sur une instance `t3.medium` sur `eu-north-1`.

**Instant Greta Thunberg** : quand j’ai besoin d’instances EC2 en Europe, j’utilise en priorité la région `eu-north-1` (Stockholm) car l’électricité suédoise est celle qui génère le moins de CO2, et de très loin. C’est un petit geste, mais c’est mieux que rien ; et si vous aussi vous utilisez des serveurs cloud, vous pouvez probablement faire un geste similaire pour la planète si ça vous dit ! (Et ne vous laissez pas enfumer par le label “green” de régions comme `eu-central-1`. Les datacenters de Francfort sont peut-être “green” mais ils tournent principalement au charbon.

![Capture du site electricitymap.org qui montre qu’actuellement 40% de l’électricité d’Allemagne provient de centrales à charbon](https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part2/germany-datacenters-on-coal.webp)

Revenons à nos moutons. Alpine nous gratifie donc d’images très légères. Comment s’en servir pour nos applications ? On peut identifier ces “stratégies” :

- Utiliser `alpine` pour notre phase de “run”,
- Utiliser `alpine` aussi bien pour notre phase de “build” celle de “run”,

Voyons un peu plus en détail comment ça se passe dans les deux cas.

### Utilisation d’Alpine pour notre phase de “run”

Si on lance un build du Dockerfile suivant et qu’on exécute l’image créée :

```
FROM gcc AS mybuildstage
COPY hello.c .
RUN gcc -o hello hello.c

FROM alpine
COPY --from=mybuildstage hello .
CMD ["./hello"]
```

On va obtenir ce message d’erreur là :

```
standard_init_linux.go:211: exec user process caused "no such file or directory"
```

On avait déjà vu ce message d’erreur dans le premier article lorsqu’on avait voulu exécuter un programme C dans l’image de base `scratch`. L’erreur venait de l’absence de bibliothèques dynamiques dans l’image `scratch`. On dirait donc que ces bibliothèques dynamiques manquent aussi à cette image Alpine ?

Et bien pas vraiment. Alpine utilise également des bibliothèques dynamiques. Après tout, un de ses objectifs est d’avoir la plus petite taille possible, et les binaires statiques ne sont pas vraiment adaptés pour ça.

Le point fondamental, c’est qu’**Alpine utilise une bibliothèque C standard _différente_**. À la place de la bibliothèque GNU C, elle utilise _**musl**_ (que je prononce pour ma part “emm-hu-ess-ell” ou “emm-you-ess-ell”, mais la [prononciation officielle](https://www.musl-libc.org/faq.html) est plutôt “mussel” ou alors “muscle” pour ceux qui parlent anglais avec un bon accent).

Cette bibliothèque est **plus petite**, **plus simple** et **plus sécurisée** que la bibliothèque GNU C. Notez aussi que les programmes compilés pour fonctionner avec la bibliothèque GNU C ne vont pas fonctionner avec musl, et vice versa.

Là, vous pourriez vous demander : “Hey mais si musl est plus petite, plus simple et plus sécurisée, pourquoi on ne bascule pas tous comme des gros oufs dessus ?!”.

… et bien c’est parce que la bibliothèque GNU C a beaucoup d’extensions, et un grand nombre de programmes utilisent ces extensions, la plupart du temps sans même se rendre compte qu’elles ne sont pas standard. La documentation de musl nous donne une liste de [différences fonctionnelles avec la bibliothèque GNU C](https://wiki.musl-libc.org/functional-differences-from-glibc.html).

Une autre limitation, musl n’est pas compatible au niveau binaire. Autrement dit, un programme compilé pour la bibliothèque GNU C ne va pas fonctionner avec musl (sauf dans quelques cas très simples). Ce qui veut dire qu’on va devoir recompiler notre code pour qu’il fonctionne avec musl (et parfois avec quelques tweaks nécessaires en plus).

**Résumé :** utiliser Alpine pendant la phase de “run” ne fonctionne que si votre programme a été build pour musl, la librairie C utilisée par Alpine.

Cela étant dit, ce n’est pas difficile de compiler un programme avec musl. Il suffit de le build avec Alpine elle-même.

### Utilisation d’Alpine pour les phases de “build” et de “run”

Cette fois, on va générer un binaire avec musl pour qu’il puisse être exécuté avec l’image de base `alpine`. On a généralement deux cas possibles :

- Certaines images officielles fournissent **des tags `:alpine`** qui sont les plus proches possibles de l’image normale mais qui utilisent Alpine (et musl) à la place.
- Certaines images officielles n’ont pas de tag `:alpine`. On doit alors construire nous-même l’image équivalente, généralement en utilisant `alpine` comme base.

L’image `golang` est dans la première catégorie : l’image `golang:alpine` fournit tout l’outillage nécessaire pour compiler du Go, mais pour Alpine.

On peut compiler notre petit programme Go avec un Dockerfile comme celui-ci :

```
FROM golang:alpine
COPY hello.go .
RUN go build hello.go

FROM alpine
COPY --from=0 /go/hello .
CMD ["./hello"]
```

L’image générée pèse 7,5 Mo. Ça semble beaucoup pour un programme dont la seule fonction est d’imprimer un pauv’ “Hello, world!” sur la sortie standard. Mais si on y regarde de plus près :

- un programme plus complexe ne serait pas beaucoup plus gros,
- cette image contient un grand nombre d’outils utiles,
- puisqu’elle est basée sur Alpine, c’est facile et rapide de lui ajouter de nouveaux outils à la volée si besoin.

Revenons maintenant à notre programme écrit en C. Au jour où j’écris cet article, il n’existe pas d’image `gcc:alpine`. On doit donc partir de l’image `alpine` et installer un compilateur C.

Ca nous donne un Dockerfile comme ça :

```
FROM alpine
RUN apk add build-base
COPY hello.c .
RUN gcc -o hello hello.c

FROM alpine
COPY --from=0 hello .
CMD ["./hello"]
```

C’est **important d’installer `build-base`** (plutôt que juste `gcc`) car le package `gcc` nous installerait seulement le compilateur, sans les bibliothèques dont on a besoin. Le package `build-base` est une sorte d’équivalent des `build-essentials` de Debian et Ubuntu ; il nous donne les compilateurs, les bibliothèques et des outils bien utiles comme `make`.

**Petit résumé** : quand on fait du `multi-stage build`, on peut utiliser l’image `alpine` comme base pour exécuter notre code. Si notre code est écrit dans un langage qui utilise des bibliothèques dynamiques (c’est le cas de la plupart des langages compilés qu’on utilise dans nos conteneurs), on a besoin de générer un binaire lié à la bibliothèque musl C. La façon la plus simple est de baser notre image de build sur `alpine` ou tout autre image qui l’utilise. La plupart des images officielles proposent un tag `:alpine` qui sert justement à ça.

Allez, faisons ensemble un **petit bilan d’étape** pour notre programme “hello world”. Comparons les techniques qu’on a vues jusqu’ici.

- Single-stage build avec l’image `golang` : 805 Mo
- Multi-stage build avec `golang` et `ubuntu` : 66,2 Mo
- Multi-stage build avec `golang` et `alpine` : 7,6 Mo
- Multi-stage build avec `golang` et `scratch` : 2 Mo

Ca nous donne une **réduction de 400x**, soit **99,75%**. C’est assez impressionnant mais c’est sur un programme qui n’est pas très représentatif de la vraie vie. Regardons plutôt les résultats avec un programme qui utilise le package `net`.

- Single-stage build avec l’image `golang` : 810 Mo
- Multi-stage build avec `golang` et `ubuntu` : 71,2 Mo
- Multi-stage build avec `golang:alpine` et `alpine` : 12,6 Mo
- Multi-stage build avec `golang` et `busybox:glibc` : 12,2 Mo
- Multi-stage build avec `golang`, `CGO_ENABLED=0`, et `scratch` : 7 Mo

Ca nous fait **une réduction de taille de 100x**, soit **99%**. Bon en fait c’est toujours aussi classe !

## Fan de Java, kezakwa pour mes belles images en Java ?

Vous le savez probablement déjà, Java est un langage compilé mais qui est exécuté dans la Java Virtual Machine (ou JVM). Voyons ce que ça signifie pour nos builds multi-stage.

### Edition de liens : statique ou dynamique ?

Conceptuellement, Java utilise l’édition de liens dynamique (**dynamic linking**) puisque le code Java va effectuer des appels aux APIs mises à disposition par la JVM. Le code de ces APIs est donc _à l’extérieur_ de notre “exécutable” Java (typiquement un fichier JAR ou WAR).

Pour autant, ces bibliothèques Java ne sont pas complètement indépendantes des bibliothèques systèmes. **Certaines fonctions utilisent des appels systèmes**. Lorsqu’on ouvre un fichier par exemple, la JVM va appeler une fonction comme `open()` ou `fopen()`.

Comme c’est la JVM qui fait cet appel de fonction (et pas notre code), ça veut dire que c’est la JVM elle-même qui doit être liée dynamiquement aux bibliothèques systèmes. Ça signifie qu’_**en théorie**_, on peut utiliser n’importe quelle JVM pour exécuter notre bytecode Java ; le fait qu’elle utilise `musl` ou la bibliothèque GNU libc n’a aucune importance. On devrait donc, _en théorie_, pouvoir compiler notre code avec n’importe quelle image qui contient un compilateur Java, puis l’exécuter avec n’importe quelle image contenant une JVM…

### Le format des fichiers Java Class

Dans la _**pratique**_, ce n’est pas si simple. Le format des fichiers Java Class (le bytecode généré par le compilateur Java) a évolué au fil du temps. Le gros des changements d’une version de Java vers une autre sont situés dans les APIs Java. Certains changements concernent le langage lui-même, comme l’ajout des `generics` dans Java 5. Ces changements peuvent introduire des différences de format des fichiers Java Class, cassant la compatibilité avec d’anciennes versions.

Cela signifie que _par défaut_, les classes compilées avec une version donnée du compilateur Java ne vont pas fonctionner avec des versions plus anciennes de la JVM. Pour gérer ce problème, on peut **demander au compilateur de cibler une ancienne version du format de fichier Java Class** avec le paramètre `--target` (jusqu’à Java 8) ou `--release` (depuis Java 9). Ce dernier paramètre va aussi sélectionner le bon `class path`, pour s’assurer que du code destiné à tourner sur une JVM version 11 ne va pas accidentellement utiliser les APIs et bibliothèques de Java 12 (ce qui l’empêcherait de tourner avec Java 11).

(N’hésitez pas à lire ce [bon blog post à propos des versions de fichiers Java Class](http://webcode.lemme.at/2017/09/27/java-class-file-major-minor-version/) si vous souhaitez en savoir plus à ce sujet.)

### JDK vs JRE

Si vous êtes familier avec la manière dont Java est packagé sur la plupart des plateformes, vous connaissez déjà probablement la différence entre JDK et JRE.

Le JRE est le _Java Runtime Environment_. Il contient ce dont on a besoin pour _exécuter_ des applications Java, c’est à dire, la JVM.

Le JDK est le _Java Development Kit_. Il contient la même chose que le JRE, mais aussi tout ce dont on a besoin pour _developper_ (et compiler) des applications Java, c’est à dire, le compilateur Java.

Dans l’écosystème Docker, la plupart des images Java embarquent le JDK, elle sont donc adaptées pour compiler et exécuter du code Java. On peut aussi **trouver des images avec un tag `:jre`** (ou un tag contenant `jre` quelque part). Ces images contiennent uniquement le JRE, sans le JDK complet, elles sont donc plus légères.

Qu’est-ce que ça signifie pour nos builds multi-stage ? Et bien qu’**on peut utiliser les images standard pour l’étape de build et une image JRE plus légère pour l’étape de run**.

### `java` vs `openjdk`

Vous le savez peut-être déjà si vous utilisez Java avec Docker : il ne faut pas utiliser les images officielles `java` car elles ne sont plus mises à jours. Utilisez à la place les images `openjdk`.

Vous pouvez aussi essayer les images `amazoncorretto` (Corretto étant le fork d’OpenJDK fait par Amazon et qui inclut leurs patchs additionnels).

### Nos images Java réduites

Bon alors, au final, **que faut-il utiliser comme images pour du Java** ?

Si vous voulez des images Java légères, voici quelques bonnes distributions candidates :

- `openjdk:8-jre-alpine` (85 MB seulement!)
- `openjdk:11-jre` (267 MB) ou même `openjdk:11-jre-slim` (204 MB) si vous avez besoin d’une version plus récente de Java
- `openjdk:14-alpine` (338 MB) si vous avez besoin d’une version encore plus récente

Malheureusement, toutes les combinaisons ne sont pas disponibles. Par exemple, `openjdk:14-jre-alpine` n’existe pas (ce qui est dommage car elle serait probablement plus légère que les variantes `-jre` et `-alpine`), mais il y a sûrement une bonne raison à ça. (D’ailleurs si vous savez pourquoi, dites-le moi !)

Retenez que vous devrez compiler votre code pour la version de JRE correspondante. [Ce blog post](https://www.baeldung.com/java-lang-unsupportedclassversion) explique en détail comment le faire dans divers environnements (IDE, Maven, etc.).

C’est très bien tout ça, mais on peut faire encore mieux en construisant un JRE custom avec `jlink`. Voyons ça de plus près …

### jlink

Java 9 (et les versions suivantes) inclut l’outil `jlink` pour construire des JVM custom avec uniquement les composants dont on a besoin. Cela permet de réduire encore plus la taille de nos images. Je trouve cet outil particulièrement utile pour obtenir une petite image, tout en ayant une version récente du JRE. Le JRE tend en effet à grossir au fil du temps (avec l’augmentation du nombre d’APIs), et avec `jlink` on n’est plus obligé de choisir entre “JRE petit mais ancien” et “JRE récent mais lourd”… on peut avoir les deux !

En lançant la commande suivante, on va créer un JRE custom dans le répertoire `/dir`. La JVM sera disponible dans `/dir/bin/java` :

```
jlink --add-modules java.base,java.some.other.module,etc --output /dir
```

D’accord, mais comment fait-on pour trouver la liste des modules à ajouter ? Et bien on peut utiliser un autre outil appelé `jdeps`. En fait, `jdeps --print-module-deps` va directement calculer les dépendances et les afficher dans un format adapté à `jlink` !

Le Dockerfile ci-dessous montre comment utiliser `jlink` en mode multi-stage. La phase de build compile le code, calcule les dépendances avec `jdeps` puis génère notre JRE avec `jlink`. La phase de run copie à la fois le code compilé et le JRE.

```
FROM openjdk:15-alpine
RUN apk add binutils # for objcopy, needed by jlink
COPY hello.java .
RUN javac hello.java
RUN jdeps --print-module-deps hello.class > java.modules
RUN jlink --strip-debug --add-modules $(cat java.modules) --output /java
FROM alpine
COPY --from=0 /java /java
COPY --from=0 hello.class .
CMD exec /java/bin/java -cp . hello
```

Notez que lorsqu’on utilise `Jlink` il faut faire attention à la bibliothèque C utilisée. Ici, je voulais avoir l’image la plus petite possible et j’ai donc utilisé `alpine` en phase de run. Je dois en conséquence utiliser une image basée sur `alpine` dans la phase de build, afin que `jlink` génère un JRE compatible avec `musl`.

(J’aimerais remercier [David Delabassée](https://twitter.com/delabassee) qui m’a parlé de `jlink` et encouragé à le tester. Quand j’ai voulu en savoir plus sur `jlink`, ces ressources m’ont par ailleurs été très utiles : [Ce blog post](https://medium.com/@greut/java11-jlink-and-docker-2fec885fb2d) par Yoan Blanc, [Ce tutoriel](https://blog.codefx.org/tools/jdeps-tutorial-analyze-java-project-dependencies/) par Nicolai Parlog, et la [documentation de jlink](https://docs.oracle.com/en/java/javase/13/docs/specs/man/jlink.html). David m’a aussi conseillé de jeter un œil à `GraalVM`, mais je me le garde en stock pour la prochaine fois !).

### Optimiser en Java : les chiffres !

Vous voulez peut-être **quelques chiffres** ? Et bien, ça tombe bien, j’en ai ! J’ai écrit un programme tout bête “hello world” (encore lui) en Java :

```
class hello {
  public static void main(String [] args) {
    System.out.println("Hello, world!");
  }
}
```

Vous pouvez retrouver tous les Dockerfiles dans le [repo Github minimage](https://github.com/jpetazzo/minimage), et voici la taille des divers builds :

- Single-stage build avec l’image `java` : 643 Mo
- Single-stage build avec l’image `openjdk` : 490 Mo
- Multi-stage build avec `openjdk` et `openjdk:jre` : 479 Mo
- Single-stage build avec l’image `amazoncorretto` : 390 Mo
- Multi-stage build avec `openjdk:11` et `openjdk:11-jre` : 267 Mo
- Multi-stage build avec `openjdk:15`, `jlink` et `ubuntu`: 106 Mo
- Multi-stage build avec `openjdk:8` et `openjdk:8-jre-alpine` : 85 Mo
- Multi-stage build avec `openjdk:15-alpine`, `jlink` et `alpine`: 47 Mo

## Kezakwé pour les langages interprétés ?

Si vous écrivez du code dans des langages interprétés comme **Node**, **Python** ou **Ruby**, vous vous demandez probablement s’il faut vous inquiéter de ces divers points, et s’il y a des moyens pour optimiser la taille de vos images.

Et bien… la réponse à ces deux questions est oui !

### Alpine et les langages interprétés

On peut utiliser `alpine` (ou toute autre image basée sur Alpine) pour exécuter du code dans notre langage de scripting favori. **Ça fonctionnera parfaitement si notre du code n’utilise que la bibliothèque standard, ou que des dépendances “pures”**, c’est-à-dire écrites dans le même langage, sans faire d’appel à du code C ou à d’autres bibliothèques externes.

Ça devient **plus compliqué si notre code dépend de bibliothèques externes**. On va devoir installer ces bibliothèque sur Alpine. Et en fonction des cas, ça peut être :

- _**Simple**_, si la bibliothèque en question est fournie avec des instructions d‘installation pour Alpine. Ces instructions nous diront quel package Alpine installer et comment gérer ses dépendances éventuelles. C’est relativement rare, puisque la distribution Alpine n’est pas aussi populaire que Debian ou Fedora, par exemple.
    
- _**Un peu plus compliqué**_, pour les bibliothèques qui n’ont pas d’instructions d’installation pour Alpine, mais qui disposent de paquets Alpine _et_ que vous arrivez facilement à trouver la correspondance entre les paquets classiques et ceux pour Alpine.
    
- _**Difficile**_, quand nos dépendances utilisent des packages qui n’ont pas d’équivalent dans Alpine. On doit alors les compiler depuis les sources et c’est une toute autre histoire !
    

Ce dernier scénario est précisément le type de circonstance pour lequel Alpine peut _ne pas_ aider, voire être contre-productif. Si on a besoin de compiler depuis les sources, ça signifie installer un compilateur, des bibliothèques, des en-têtes … Tout ça prendra de la place supplémentaire dans l’image finale. (On pourrait bien utiliser des build multi-stage. Mais dans ce contexte précis, en fonction du langage, ça peut devenir compliqué puisqu’on a besoin de déterminer comment produire un package binaire pour nos dépendances). Compiler depuis les sources va aussi nous prendre beaucoup plus de temps.

Il y a une situation particulière où utiliser Alpine va mettre en évidence toutes ces problématiques : la _data science_ avec Python. Des packages courant comme `numpy` ou `pandas` sont disponibles en package Python pré-compilés appelés [wheels](https://pythonwheels.com/), et ces wheels sont liés à une bibliothèque C spécifique. (“Oh non”, vous ois-je, “Tout mais pas la bibliothèque C!"). Ça veut dire que ces packages vont s’installer sans problème dans les images Python “normales”, mais pas sur les variantes basés sur Alpine. Sur Alpine, elles vont nécessiter d’installer des packages systèmes, avec dans certains cas des recompilations longues. _Très_ longues.

Il existe un très bon article qui traite de ce problème et qui explique comment l’utilisation d’[Alpine peut rendre les builds de Python sur Docker 50x plus lents](https://pythonspeed.com/articles/alpine-docker-python/).

Si vous lisez cet article, vous allez peut-être vous dire, **“oula, est-ce que je dois fuir Alpine pour mes images Python alors ?”**. Je ne serais pas aussi catégorique. Pour la _data science_, oui probablement. Mais pour d’autres workloads, si vous voulez réduire la taille d’image, ça vaut la peine d’essayer.

### Images `:slim`

Si on cherche un **compromis entre les images d’origine et leurs variantes Alpine**, on peut regarder du côté des images `:slim`. Les images `slim` sont souvent basées sur Debian (et sur la GNU libc) mais leurs tailles ont été optimisées, en enlevant les paquets qui ne sont pas indispensables. C’est un peu la loterie. Parfois, elles vont contenir exactement ce que vous voulez. Parfois, des choses essentielles vont manquer (comme un compilateur !) et installer ces choses manuellement va alors finalement vous ramener à une taille similaire à celle de l’image originale. En tout cas, c’est pratique de les avoir sous la main afin de pouvoir les essayer et trouver la combinaison la plus adaptée à notre application.

Pour vous donner une idée, voici les tailles des images d’origine, ainsi que leur variantes `:alpine` et `:slim`, pour quelques langages interprétés couramment utilisés.

|Image|Size|
|---|---|
|`node`|939 Mo|
|`node:alpine`|113 Mo|
|`node:slim`|163 Mo|
|`python`|932 Mo|
|`python:alpine`|110 Mo|
|`python:slim`|193 Mo|
|`ruby`|842 Mo|
|`ruby:alpine`|54 Mo|
|`ruby:slim`|149 Mo|
|.||

Pour le cas particulier de Python, voilà les tailles obtenues après installation des packages matplotlib, numpy et pandas (des classiques en data science), sur diverses images Python de base :

|Image et technique|Size|
|---|---|
|`python`|1.26 Go|
|`python:slim`|407 Mo|
|`python:alpine`|523 Mo|
|`python:alpine` multi-stage|517 Mo|
|.||

On voit bien ici que l’utilisation d’Alpine ne nous aide pas du tout, même en utilisant un build “multi-stage”. (Vous pouvez retrouver les Dockerfiles correspondant dans le repository [minimage](https://github.com/jpetazzo/minimage) ; ils sont nommés `Dockerfile.pyds.*`.)

Mais n’en concluez pas trop vite qu’Alpine est un mauvais choix pour du Python ! Voici les tailles pour une application Django qui a un grand nombre de dépendances :

|Image et technique|Size|
|---|---|
|`python`|1.23 Go|
|`python:alpine`|636 Mo|
|`python:alpine` multi-stage|391 Mo|
|.||

(Dans ce cas précis, j’ai abandonné l’utilisation de l’image `:slim` car elle nécessitait d’installer beaucoup trop de packages supplémentaires !).

Comme on peut le voir, ce n’est donc pas toujours très tranché. Parfois, `:alpine` donnera de meilleurs résultats, alors que dans d’autres cas `:slim` sera plus efficace. **Si vous cherchez à optimiser vos images à fond, il faut donc essayer les deux méthodes et comparer le résultat**. Au fil du temps et de l’expérience, on finit par avoir un bon “feeling” de la variante la plus appropriée pour tel ou tel type d’applications.

### Multi-stage avec des langages interprétés

Et qu’en est-il alors des builds multi-stage ?

Il vont être particulièrement utiles quand on génère des assets avec des outils supplémentaires.

Par exemple, si on a une application **Django** (utilisant probablement l’image de base `python`) et qu’on minimise notre Javascript avec [UglifyJS](https://www.npmjs.com/package/uglify-js) et notre CSS avec [Sass](https://sass-lang.com/).

L’approche naïve serait d’inclure tout ce bazar dans notre image, avec un Dockerfile qui deviendrait vite compliqué (puisqu’on installerait Node dans une image Python) et surtout une image finale qui serait bien trop lourde. À la place, **il vaut mieux utiliser un _build_ en plusieurs étapes** : l’une utilisant `node` pour minimiser nos ressources, et une autre utilisant `python` pour l’application elle-même, qui embarque les ressources JS et CSS générées à l’étape précédente.

Autre effet positif de cette approche, elle va aussi **améliorer le temps de build**, car les changements dans le code Python ne vont pas toujours impliquer un rebuild des JS et CSS (et vice-versa). Dans ce cas précis, je recommanderais même d’utiliser deux stages séparés pour la partie JS et pour la partie CSS, pour qu’un changement de l’un ne déclenche pas de rebuild de l’autre.

## Et pour Rust ? Kézakrust ?

Je m’intéresse beaucoup à [Rust](https://www.rust-lang.org/), ce langage de programmation moderne initialement conçu chez Mozilla, et dont la cote de popularité dans le monde du web et de l’infrastructure augmente rapidement. Je me suis demandé à **quel genre de comportement on pouvait s’attendre si on mixait la problématique d’images Docker avec du Rust**.

Allons-y. Rust génère des exécutables qui sont liés dynamiquement avec la bibliothèque C. Les binaires compilés avec l’image `rust` vont donc s’exécuter sans problème avec des images classiques comme `debian`, `ubuntu`, `fedora`, etc. mais ils ne _vont pas_ fonctionner avec `busybox:glibc`. C’est parce que les binaires sont liés avec `libdl` qui n’est pas incluse dans `busybox:glibc` pour l’instant. Par contre, il existe bien une image `rust:alpine`, et les binaires générés marchent parfaitement avec `alpine` comme image de base.

Je me suis demandé si Rust pouvait générer des binaires statiques. Et en fait, pas besoin d’aller chercher loin, [la documentation de Rust](https://doc.rust-lang.org/1.9.0/book/advanced-linking.html#static-linking) nous explique comment faire.

Sous Linux, on génère une version spéciale du compilateur Rust, et cela requiert `musl`. Ah oui tiens, cette même musl utilisée par Alpine dont je vous parlais un peu plus haut. **Si vous voulez obtenir des images minimales avec Rust**, ça devrait être relativement facile en suivant les instructions de la documentation et en déposant les binaires résultants dans l’image `scratch`. (Mais je n’ai pas essayé !)

## Et qu’allons nous apprendre dans le dernier article ?

Dans les **deux premières parties** de cette série, on a passé en revue les méthodes les plus fréquemment utilisées pour optimiser la taille des images Docker. On a regardé précisément comment celles-ci s’appliquent à différents langages, qu’ils soient compilés ou interprétés.

Dans la troisième et **dernière partie**, on complète l’analyse avec les derniers éléments qui me semblent fondamentaux sur l’optimisation d’images Docker. On voit comment le fait de tout **standardiser autour d’une image de base particulière** permet de réduire non seulement sa taille, mais aussi les I/O et l’utilisation mémoire. On donne aussi **quelques nouvelles techniques bien utiles**, même si elles ne sont pas spécifiques aux conteneurs. Et enfin, par souci d’exhaustivité, on parle de techniques plus **exotiques**.

RDV sur [“Chérie, j’ai rétréci Docker !”, Part 3/3](https://enix.io/fr/blog/cherie-j-ai-retreci-docker-part3/)

---

Cet article a également été publié en version anglaise [ici](https://www.ardanlabs.com/blog/2020/02/docker-images-part2-details-specific-to-different-languages.html).

---

Ne ratez pas nos prochains articles DevOps et Cloud Native! Suivez Enix sur [Twitter](https://twitter.com/enixsas)!