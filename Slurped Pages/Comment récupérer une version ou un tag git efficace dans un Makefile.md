---
link: https://www.metal3d.org/blog/2021/version-git-dans-un-makefile/
site: Metal3d
date: 2021-12-06T09:26
excerpt: J’adore “versionner” mes paquets ou un executable avec le tag git ou la version “sha” du dépôt en cours, alors comment on peut rendre ça vraiment automatique ?
tags:
  - git
  - tag
slurped: 2026-03-28T10:26
title: Comment récupérer une version ou un tag git efficace dans un Makefile
---

J’adore “versionner” mes paquets ou un executable avec le tag git ou la version “sha” du dépôt en cours, alors comment on peut rendre ça **vraiment** automatique ?

Ce billet sera court, c’est juste une astuce que j’ai dans un coin qui va vous permettre de faire une opération qui est généralement bien lourde à gérer. Cette petite technique, elle marche dans un Makefile, mais libre à vous de l’adapter pour d’autres systèmes de build.

Je vous expose le problème. Imaginons que je décide que mon Makefile doive faire un _tarball_ de mon application. Et bien entendu je veux avoir la version dans le nom. En général on fait comme ça:

```
# ! non ne faites PAS ça
VERSION=1.1.O

exemple:
    tar caf  foo-$(VERSION).tgz ./src

```

Et vous allez bien entendu vouloir utiliser un outil CI/CD pour lancer la commande qui va déposer le paquet quelque part. Bravo, c’est pas idiot, mais il y a un hic. Et un sérieux hic en fait: vous allez forcément oublier de changer la variable `VERSION` dans le Makefile.

> Non mais… je vous connais… et puis je fais pareil…

Alors vous allez me dire:

> oui mais moi mon CI/CD il va créer le tag, puis passer la version en argument etc…

Oui… pourquoi pas, et puis c’est vrai que tout le monde utilise un outil CI/CD 🙄, et puis il va falloir aussi penser à “puller” le dépôt sur votre poste, sans compter que parfois “ça foire” sur l’outil CI/CD… Allons allons 😌

Il existe une façon de faire bien plus effcicace, un poil plus simple (si si) et surtout qui est **sûre et contrôlable**. J’entends par ces termes que vous allez simplement taguer (ou pas) votre version, et le Makefile trouvera tout seul l’état, et donc la version, de votre dépôt.

L’idée est simple:

- si j’ai un tag et que mon dépôt est bien à ce tag **précisément** alors je veux que la version soit égale à ce tag
- sinon je veux le nom de la branche + le “SHA” du commit actuel.

Trois commandes suffisent:

- `git branch --show-current` pour avoir le nom de la branche en cours, dans le cas où le dépôt n’est pas positionné sur un tag
- `git log --pretty="%h"` va donner l’empreinte courte du commit en cours
- `git describe --exact-match --tags SHA` (avec SHA qui est le résultat de la commande du dessus), va me donner le tag si et seulement si le SHA correspond au commit du tag. Sinon il ne retourne rien.

Et donc, dans votre Makefile, faites juste ça:

```
CUR_SHA=$(shell git log -n1 --pretty='%h')
CUR_BRANCH=$(shell git branch --show-current)
VERSION=$(shell git describe --exact-match --tags $(CUR_SHA) 2>/dev/null || echo $(CUR_BRANCH)-$(CUR_SHA))

exemple:
    tar caf foo-$(VERSION).tgz ./src
```

Donc:

- si je viens de taguer, et que je n’ai rien touché, `VERSION` contient le nom du tag
- sinon, `VERSION` vaut “nom-de-la-branche-SHA” (par exemple `master-34ge12f`)

Un exemple avec Go, pour builder avec la version du binaire en variable:

```
CUR_SHA=$(shell git log -n1 --pretty='%h')
CUR_BRANCH=$(shell git branch --show-current)
VERSION=$(shell git describe --exact-match --tags $(CUR_SHA) 2>/dev/null || echo $(CUR_BRANCH)-$(CUR_SHA))

build:
    go build -ldflags="-X 'main.Version=$(VERSION)'" .
```

Voilà, c’est tout bête mais je voulais vous le partager 😗