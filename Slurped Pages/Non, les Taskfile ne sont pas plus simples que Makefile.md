---
link: https://www.metal3d.org/blog/2025/makefile-est-bien-plus-simple-que-taskfile/
site: Metal3d
date: 2025-09-03T08:57
excerpt: Débat houleux ? YAML contre un fichier simple ? Trop compliqué ? Makefile est bien plus simple que Taskfile. Je vais vous expliquer mon point de vue, puis vous montrer qu'en 10 minutes vous allez savoir faire un Makefile.
tags:
slurped: 2026-03-28T10:34
title: Non, les Taskfile ne sont pas plus simples que Makefile
---

Débat houleux ? YAML contre un fichier simple ? Trop compliqué ? Makefile est bien plus simple que Taskfile. Je vais vous expliquer mon point de vue, puis vous montrer qu'en 10 minutes vous allez savoir faire un Makefile.

Voilà des mois que les “Taskfiles”, fichiers utilisés par “GoTask”, censé être le remplaçant de Makefile, font débat dans la communauté des développeurs. Les Makefile sont jugés trop compliqués, illisibles, vieux, pas adaptés… j’entends tout et n’importe quoi. Alors que la promesse des Taskfile est de rendre tout magiquement simple, lisible, etc.

Donc, avant de vous lancer dans GoTask parce que c’est la mode, et de répondre à l’appel des sirènes du “c’est moderne”, on va simplement faire un TL;DR pour savoir faire un Makefile. Et je voue le dis, ce qui suit est **tout ce que vous devez savoir pour faire un Makefile**. Alors que GoTask, vous allez avoir besoin d’une doc (très belle, je ne dis pas le contraire) assez conséquente.

Mais, d’abord, avant de passer à la pratique, parlons un peu de ce qui m’énerve avec les fans de GoTask.

Prêt ?

## Avant de lire la suite, c’est IMPORTANT

> Détendez-vous, je vais être sarcastique et volontairement jouer le nerveux. C’est pour rire. Je suis un gars gentil et je respecte vos préférences. Ce n’est qu’un billet de blog ! 😄 💋 ❤️

J’aime GoTask, je l’utilise parfois. Je suis développeur Go, j’aime Go, j’aime le YAML. Mais je suis pragmatique. Je ne sombre pas dans le fanatisme crasse et l’argumentation fallacieuse sous prétexte qu’un truc me vante ses mérites.

Je sais que Makefile peut devenir un bordel sans nom, qu’il peut devenir illisible, compliqué, etc. Mais je sais aussi qu’on peut faire les choses proprement. J’ai appris à faire un Makefile à peine quelques jours après avoir fait mes premiers pas en C. Je vous le dis, j’avais 20 ans, je n’avais jamais touché un PC de ma vie, et pourtant j’ai trouvé Makefile **ultra-simple à comprendre et à utiliser**.

GoTask, il a plein d’avantages. C’est indéniable. Mais comme énormément de nouvelles choses qui sortent, je vois passer des posts LinkedIn, Mastodon, ou je ne sais quoi, avec un objectif clair : faire passer GoTask / Taskfile comme “le héros de tous les temps”, et chier sur Makefile avec des arguments fallacieux.

Je suis toujours prêt à discuter, échanger, débattre. Les désaccords sont normaux. Mais là, je lis des trucs qui me font bondir.

Je vous échauffe un peu, mais au final, je vais vous demander juste une chose :

```
build:
	commande qui builde un truc
	par exemple:
	go build -o ./dist/app .

clean:
	rm -rf ./dist
```

Est-ce que ça, ça vous dépasse ?

- Si oui, partez de cet article. Ça ne sert à rien, vous allez refuser d’écouter mes arguments.
- Sinon, alors vous allez pouvoir accepter un échange d’idées.

Maintenant, clairement, je vais **volontairement casser un peu de sucre sur le dos de GoTask**. C’est normal, c’est une exagération pour faire réagir, pour que vous compreniez ce que moi, utilisateur averti des deux outils, je ressens.

> Si vous acceptez ce ton, le sarcasme, et que vous savez sourire à tout ça, vous êtes les bienvenus pour lire la suite.

## Les arguments qui ne passent pas

J’entends et je lis énormément d’arguments de “pro Taskfile / GoTask” et beaucoup de trucs contre Makefile. Et quand je les lis, je reste pantois, bouche bée, ébahi, estomaqué… choisissez dans la liste.

C’est parti…

1. **“GoTask est plus moderne que Makefile”**.  
    **Non…** utiliser YAML ne rend pas un truc “moderne”. Par contre, ça le rend sujet à tout un merdier que YAML apporte. Je vous donne une énigme à résoudre juste après, juste pour vous rendre compte que YAML, ce n’était pas l’idée du siècle pour “coder des tâches”.
2. **“La syntaxe de Makefile est compliquée”**.  
    **Non…** la syntaxe de Makefile est simple. Simple parce qu’il n’y a **aucun** mot clef à connaitre pour démarrer. Vous donnez un nom de tâche, et les commande à effectuer pour cette tâche. Voilà, point.  
    Par contre, avec Taskfile, vous allez devoir apprendre **deux syntaxes** : YAML (et les noms des directives autorisées…) et GoTmpl. Je ne vois vraiment pas comment on peut estimer qu’utiliser GoTmpl est “sympa”. Pourtant, je développe en Go hein, et je fais des Helm Chart. Mais je vous le dis, c’est loin d’être idéal. Taskfile, il vous impose de connaitre sa sémantique, ses noms d’attributs…
3. **“C’est clair, le fichier se décrit lui-même”**.  
    **Non…** Pourquoi “run: once” ? pourquoi la règle c’est pas ce que ça génère ? pourquoi on doit dire “generates:”? Ça demande à connaitre les directives de Taskfile. La moindre erreur, le moindre truc qu’on n’a pas vu dans la doc, le Taskfile ne fait pas ce qu’on veut. Makefile **n’a pas de langage de directive**. Il applique les tâches / cibles / ce que vous voulez, et c’est tout. La cible, c’est ce que vous voulez faire. Si c’est un fichier, la cible est un fichier, si c’est une tâche, alors ça effectue une tâche. Makefile = pas la peine de lui expliquer l’évidence.

**Je vous parlais d’une énigme à résoudre. Amusons-nous. 😉**

Voici un Taskfile :

```
version: "3"
silent: true

vars:
  REFUSED: no

tasks:
  ask:
    cmds:
      - |
        read -p "Would you really delete your entire hardirve ? (yes/no): " answer
        if [ "$answer" == ${REFUSED} ]; then
          exit 0
        else
          echo "ok, let's delete all your data"
        fi
```

Vous lancez `task ask` et vous répondez “no”. Bonne chance…

```
$ task ask
Would you really delete your entire hardirve ? (yes/no) no
ok, let's delete all your data
```

What ? Mais j’ai répondu “no” ! Pourquoi il a pas compris ?

Vous voulez la solution de cette énigme ?

Eh oui, en YAML `no` est un booléen. C'est l'équivalent de `false`. Donc, dans le script, on compare `"$answer"` à `false`, ce qui n'est jamais vrai.

Il faut écrire `REFUSED: "no"` pour que ça marche, avec des doubles quotes. Et ça, c'est une des conneries que le YAML apporte... Parce que oui, YAML c'est bien, mais y'a un paquet de trucs qui foutent le zbeul.

Effectivement mon exemple est volontairement dangereux, mais ça peut être un vrai souci à déboguer.

Ce que je veux dire, dans cet exemple, c’est qu’utiliser YAML qui, de base, n’est pas fait pour ce genre de chose, c’est introduire des risques.

**On a déjà bien mangé nos nerfs sur Ansible à cause des conneries de ce genre, GoTask ne va pas magiquement passer à travers.** YAML, c’est un point de friction, que vous le vouliez ou non, et cette petite énigme en est une preuve, et j’en ai plein sous la main !

## Alors pourquoi les gens pensent que Makefile est compliqué ?

Parce que le souci, c’est que quand vous le maitrisez, je veux dire quand vous allez loin, très (trop) loin, alors vous risquez de rendre le contenu imbuvable. Je le sais, c’est un de mes défauts… Parfois, je pousse trop.

Sauf que là, on parle d’une utilisation très spécifique et surtout technique des Makefile.

Sur un projet web, pour un truc qui lance des conteneurs, build du NodeJS, installe des dépendances PHP… Vous n’aurez absolument pas à utiliser des options poussives.

Prenons un exemple simple, un truc que je fais presque à chaque fois :

```

# docker ou podman
CONTAINER=docker

up:
  $(CONTAINER) compose up -d

down:
  $(CONTAINER) compose down

# alias vers bin/myapp
build: bin/myapp

bin/myapp: *.go
  go build -o app/bin .

```

Je vais tout détailler après, mais là, **ne me faites par croire que ce Makefile est compliqué**.

> Vraiment, quand vous me dites que ça, vos collègues ne savent pas comment l’utiliser, **je ne vous crois pas**.

Non mais vraiment, qu’est-ce qui vous rebute là ???

`make up` démarre les conteneurs, `make down` les arrête, `make build` appelle `make app/myapp` qui construit le binaire si un fichier `.go` a changé. Y’a rien à étudier, pas une doc à ouvrir, c’est évident.

En Taskfile, ça donne ça :

```
version: "3"

vars:
  container: "docker"

tasks:
  up:
    cmds:
      - "{{.container}} compose up -d"

  down:
    cmds:
      - "{{.container}} compose down"

  build:
    deps:
      - "bin/myapp"

  bin/myapp:
    cmds:
      - "go build -o bin/myapp ."
    sources:
      - "*.go"
```

Non mais soyez honnête… En quoi c’est plus lisible ? En quoi c’est plus simple ? En quoi c’est plus court ? En quoi c’est plus moderne ? C’est le bordel !

C’est une série de directives dont il faut se souvenir (`deps`, `cmds`, et d’autre dont j’oublis toujours le nom…, `sources`, etc.) et une arborescence longue comme un roman de Ken Follett.

**Et bon sang… faire des listes de 1 seul élément, c’est cool ?**

On double le nombre de lignes, ça va gueuler si je me trompe dans la syntaxe GoTmpl..

> Je suis désolé, mais l’argument de la modernité et de la lisibilité ne passent pas. Mais alors pas **du tout**

Je veux bien entendre plein d’arguments qui sont pour GoTask, mais pas ceux-là.

## Créer un Makefile, tout ce que vous devez savoir

Donc, on va se mettre d’accord. Vous lisez cette partie, ça va durer 10 minutes en gros. Ensuite, si vous continuez de me dire que Makefile, c’est compliqué, je pense qu’on n’est pas de la même planète et je vous demanderai de garer votre soucoupe sur un espace réservé.

Un Makefile, c’est un fichier nommé “`Makefile`” (ou “`GNUmakefile`”) qui contient des **cibles**, des dépendances en options et des commandes. Voilà…

> Avant de vous prendre la tête, gardez à l’esprit ce crédo valable pour tout le document qui suit :
> 
> **Un Makefile est une description de trucs à faire. C’est pas une structure, pas un objet et encore moins un programme à coder, c’est une des-crip-tion**

Une cible, c’est arbitraire. C’est soit un simple nom de tâche (`build`, `test`, `clean`, `deploy`, etc.), soit un nom de fichier à générer. Si la cible est un nom de fichier, `make` vérifiera s’il est à jour en fonction des dépendances.

Et les dépendances, ce sont aussi des cibles. Donc soit une tâche à effectuer, soit un fichier existant ou à générer.

> Donc, arrêtez de vous prendre la tête, une cible, c’est tout ce que vous voulez. Y’a pas de règle à connaitre. C’est la même chose dans GoTask, vous n’avez pas à vous faire saigner du nez pour comprendre ça.

Et, **la première cible est la cible par défaut**. Si vous faites `make` sans argument, c’est la première cible qui sera exécutée.

La syntaxe est simplement :

```
cible1: dépendance1 dépendance2
	commande1
	commande2
```

Les dépendances sont optionnelles, bien entendu. Si une dépendance est listée, elle sera exécutée avant la cible.

🏁 **STOP !!!** Arrêtez-vous ici.

> Là, vous avez déjà **tout ce qu’il faut pour faire un Makefile**. Vous pouvez arrêter de lire et utiliser ce que vous avez appris. Ça suffit pour faire le travail demandé. Alors oui, la suite vous apprendra d’autres trucs, et je vous conseille d’aller au bout. Mais, ce bout de code là, **c’est suffisant pour faire ce que vous voulez !**

Donc, me dire que c’est trop compliqué, c’est de la mauvaise foi.

Important:

> Dans les exemples qui suivent, je mets “`@`” pour ne pas que `make` n’affiche les commandes. Vous pouvez aussi utiliser `.SILENT:` au début du Makefile pour rendre toutes les commandes silencieuses. Ou, l’option `make -s` à l’appel de `make`. Je vous rappelle, une fois de plus, que c’est **pareil sur GoTask**.

Ensuite, les variables. Vous pouvez en créer autant que vous voulez :

```
# des variables
FOO = bar
BAR = baz

# testons:
test:
	echo $(FOO)
	echo $(BAZ)
```

Les variables sont référencées avec `$(NOM_DE_LA_VARIABLE)`.

Un truc cool, vous pouvez utiliser d’autres symboles d’affectation :

- `=` veut dire “la valeur est évaluée une fois, au **moment de son expansion, quand elle apparait dans une cible**”
- `:=` “la valeur est évaluée une fois, au **moment de la déclaration, quand on lance `make`**”
- `+=` concaténation
- `?=` la valeur est affectée seulement si la variable n’existe pas déjà (utile pour les valeurs à passer en ligne de commande)

On va juste parler de la subtilité entre `=` et `:=`, parce que je sais que c’est un point de friction pour beaucoup.

Si le contenu de la variable est “simple”, pas long à assigner, donc comme dans 99% des cas, utilisez “`=`”. Si par contre, vous préférez que la variable soit évaluée une fois pour toutes, genre son contenu est long à générer, utilisez “`:=`”.

> Mais franchement, **on s’en tape** dans notre cas d’utilisation, utilisez “`=`” partout, ça ira très bien.

Et enfin, pour `?=` :

```
FOO ?= la valeur par défaut

test:
	echo $(FOO)
```

C’est simple :

```
$ make -s
la valeur par défaut

$ make -s FOO=couou
coucou
```

Holala c’était dur… 🥱

## Plus loin

Comme je vous le disais, les “cibles” sont tout et n’importe quoi. Si un fichier porte le nom de la cible, alors `make` va évaluer son existence et sa date de modification par rapport aux dépendances. Si le fichier n’existe pas ou s’il est plus vieux, alors la cible sera exécutée. Si le fichier existe et est plus récent que toutes les dépendances, alors la cible ne sera pas exécutée (sauf si on appelle `make` avec l’option `-B`).

Surprise, c’est pareil dans GoTask… 😒

```
example.txt: dépendance1 dépendance2
	echo "Génération de example.txt"
	touch example.txt
```

```
$ make -s
Génération de example.txt

$ make -s
make: « example.txt » est à jour.
```

Sérieux, qu’est-ce qui vous rebute là ?

Avec les dépendances :

```
example.txt: *.foo
	echo "Génération de example.txt"
	touch example.txt
```

Quel que soit le fichier qui se termine par `.foo` dans le répertoire, si l’un d’eux est plus récent que `example.txt`, alors la cible sera exécutée.

```
$ touch a.foo
$ make -s
Génération de example.txt

$ make -s
make: « example.txt » est à jour.

$ touch a.foo
$ make -s
Génération de example.txt
```

Et, si on utilisait `make` pour générer “`b.foo`” ?

```
example.txt: *.foo b.foo
	echo "Génération de example.txt"
	touch example.txt

b.foo:
	touch b.foo
```

En appelant `make` ou `make example.txt`, si `b.foo` n’existe pas, il sera généré avant, puis `example.txt` sera généré.

Et, c’est ~~différent~~ ha non zut, c’est pareil dans GoTask… sauf que c’est implicite avec Makefile. J’ai pas eu à lui expliquer un truc bateau, normal, logique. En gros, Makefile est plus intelligent.

## Les variables spéciales à connaitre pour aller plus loin

Voilà, là oui, je l’admets, c’est la partie qui est le vrai sujet de discorde. Parce que oui, il y a des variables “magiques” dans Makefile et ça peut vraiment poser un problème à ceux qui ne les connaissent pas.

Si c’est ce point qui bloque, je peux le comprendre. Il n’y en a pas beaucoup, mais elles ne sont pas hyper intuitives.

> Mais, dans ce cas, ne les utilisez pas. Voilà… ça règle le problème !

Il y a des variables spéciales utilisables n’import où :

- `$@`, c’est “le nom de la cible”.
- `$<`, le premier élément dans la liste des dépendances.
- `$^`, tous les éléments dans la liste des dépendances.
- `$?`, tous les éléments dans la liste des dépendances qui sont plus récentes que la cible.
- `$*`, le nom de la cible sans son suffixe (utile pour les règles génériques).

Et aussi :

**`%` dans une cible ou une dépendance, c’est un joker qui représente n’importe quelle chaîne de caractères. Sa valeur est reportée partout, vous allez voir c’est génial.**

En gros une cible `%.html: %.md` est générique, je peux demande `foo.html`, automatiquement `make` va vérifier `foo.md` (et le générer si une règle existe pour ce fichier). Dans l’exemple ci-dessous, je m’en sers pour générer des fichiers HTML avec `pandoc` :

Je vous donne un exemple :

```
# quelque soit le fichier HTML qu'on veut générer, on prend son
# équivalent Markdown dans le répertoire src
doc/%.html: src/%.md
	pandoc -o $@ $^

# pour générer plein de docs
docs: doc/index.html doc/about.html doc/contact.html

# ou, on utilise des fonctions spéciales
docs: $(wildcard doc/*.html)
```

Vous pouvez appeler `make docs` pour générer toute la doc, ou générer un seul fichier avec `make doc/index.html`, ou `make doc/about.html`…

Comprenez que :

```
pandoc -o $@ $^
# équivaut à prendre la cible dans $@ et les dépendances dans $^, donc
# make doc/index.html produit:
pandoc -o doc/index.html src/index.md
```

> Astuce mnémotechnique :
> 
> - `$<`, c’est une flèche qui pointe vers la première dépendance,
> - `$^`, c’est une flèche vers le haut qui pointe sur les dépendances…

Donc, maintenant :

- `make doc/index.html` va générer `doc/index.html` à partir de `doc/index.md`
- `make docs` va générer `doc/index.html`, `doc/about.html` et `doc/contact.html` à partir de `doc/index.md`, `doc/about.md` et `doc/contact.md`

C’est vrai, ça je vous l’accorde, ces variables, ça peut rebuter… Mais au point de partir sur GoTask qui va vous demander plus de boulot, et ne pas vous proposer ce genre d’options ? Et donc, de vous taper du GoTmpl qui rend fou ? J’ai du mal à le comprendre.

## Les cerises sur le gâteau

Makefile a plein de petites options qui rendent l’outil d’une puissance rare.

- `.PHONY` est une cible spéciale qui indique que les cibles listées ne sont pas des fichiers. Comme ça on évite les “machin est à jour” alors qu’on ne veut pas…
- `.ONE_SHELL:` à partir de GNU Make 4.0, cette directive indique que toutes les commandes d’une même cible doivent être exécutées dans le même shell. Par défaut, chaque ligne est exécutée dans un shell séparé, ce qui peut poser des soucis sur les changements de répertoire, les variables d’environnement, etc.
- `$(MAKE)` permet d’appeler `make`, ça évite des problèmes sur certains OS donc la commande n’est pas focément `make`.
- `-include fichier` permet d’inclure un autre Makefile. Le `-` indique que l’inclusion est optionnelle (si le fichier n’existe pas, pas d’erreur)
- `$(shell commande)` exécute une commande shell et retourne son résultat. Utile pour affecter des variables.
- `@` au début d’une commande empêche l’affichage de la commande avant son exécution. Utile pour éviter le bruit dans la sortie. On peut utiliser `.SILENT:` pour rendre toutes les commandes silencieuses.
- `-` au début d’une commande indique que les erreurs de cette commande doivent être ignorées.
- `$(wildcard motif)` retourne une liste de fichiers correspondant au motif. Utile pour les dépendances.
- `$(patsubst motif, remplacement, texte)` remplace les occurrences du motif dans le texte par le remplacement.

Et en dernier lieu, on peut faire des tests conditionnels :

```
LOCAL_INSTALL=1

install:
ifeq ($(LOCAL_INSTALL),1)
	echo "Installation locale"
	cp machin ~/.local/bin/machin
else
	echo "Installation globale"
	sudo cp machin /usr/local/bin/machin
endif
```

## Comparé à GoTask / Taskfile

Make sait faire du parallélisme avec l’option `-j` (nombre de jobs en parallèle). On me sort cet argument pour GoTask, j’ai l’impression que les détracteurs de `make` n’on pas ouvert la doc 😉

La gestion de dépendances est automatique avec `make`, il n’y a pas besoin de les lister partout.

Et pareillement, la gestion de construction de fichier est implicite, pas besoin de définir une tâche de 8 lignes pour dire “alors si ça, c’est pas à jour, il faut que tu regardes ces fichiers”. Dans un Makefile, **c’est automatique**.

On me sort souvent que le YAML c’est pratique et moderne. Pratique oui, moderne… ça se discute. YAML c’est avant tout un format de sérialisation de données. Pas un langage de script. Donc, on se retrouve avec un format qui n’est pas directement prévu pour cela. Et de ce fait, on va faire des routines tordues pour effectuer des tâches sous conditions, alors que le langage de Makefile est prévu pour cela.

J’entends que vous aimez YAML, et je ne dis pas que GoTask est mauvais. Je dis simplement que si vous prenez quelques minutes pour lire cet article, ou une doc condensée sur Makefile, vous verrez que **Makefile est simple à utiliser**.

Oui, il y a des variables spéciales, oui, il y a des subtilités, mais franchement :

- y’en a peu, vous les retenez et vous commentez votre Makefile, ça ira tout seul
- **vous n’êtes pas obligé de les utiliser**, vous pouvez faire votre tambouille avec des règles simples, sans gérer les variables spéciales, sans faire de règles génériques, etc. Vous pouvez faire un Makefile textuel bateau.

Je refuse de croire que des ingénieurs, des gens qui codent tous les jours des machins tordus avec des langages compliqués, n’arrivent pas à comprendre un Makefile avec des règles simples. J’ai peut-être trop confiance en les capacités des techniciens, peut-être que je suis trop optimiste… Mais bon sang, j’ai appris à faire un Makefile en 10 minutes, à 20 ans, sans jamais avoir touché un PC de ma vie…

> Qu’est-ce qui a fait que moi, sans compétences, sans aucune expérience, à 20 balais, j’ai absorbé la simplicité de Makefile, et pas vous ?

## Conclusion

Les gens qui me disent que Makefile est compliqué sont les mêmes qui me disent “mais si, la programmation des “generics” en C# / Java / C++ c’est super simple”. Ou encore “comment ça l’injection de dépendance de Symfony est compliquée ? C’est juste du XML”.

Ce sont aussi les mêmes qui me disent que la syntaxe de template Go utilisée dans Taskfile est simple. Je suis développeur Go, j’aime beaucoup Go, je l’adore, mais je ne trouve pas la syntaxe de template évidente. Loin de là…

Donc, là, cette page que j’ai produit, **elle recense 90% de ce que vous devez savoir pour faire un Makefile**.

À moins que vous ayez des étapes de compilation de bourrin (genre des projets en C/C++ avec des tonnes de fichiers), le Makefile que vous ferez sera simple, lisible, efficace et rapide à écrire. **Beaucoup plus lisible et plus simple que du Taskfile**.

Tiens, vous voulez voir le Makefile que j’utilise pour ce site web ? Le voilà :

```
.ONE_SHELL:

# contains KUBECONFIG, S3_ROOT, S3_BUCKET
-include .env


# start the website locally
dev:
	hugo server -s src

# build the website
build:
	mkdir -p public
	hugo -s src -d ../public

# deploy the new version
deploy: build push-on-s3 restart-website clean

# push code to scaleway S3 bucket, using "rclone"
push-on-s3:
	rclone sync -Pv \
		--fast-list --checksum \
		--no-update-modtime \
		./public/ ${S3_ROOT}:${S3_BUCKET}/

restart-website:
	KUBECONFIG=$(KUBECONFIG) kubectl -n metal3d \
		rollout restart deployment blog-metal3d-blog

# remove the generated site
clean:
	rm -rf public

# create a new blog post
# in blog/<yeear>/<name>/index.md
# NAME is the title, replace ' ' by '-'
NAME?=default-title
NAME:=$(shell printf "%s" "$(NAME)"|tr ' ' '-')
YEAR:=$(shell date +"%Y")

blog-post:
	hugo -s src new content "blog/$(YEAR)/$(NAME)/index.md"
```

Alors, je peux comprendre l’intérêt de GoTask, et j’accepte volontiers que vous le préfériez. Mais, **je refuse l’argument de la complexité de Makefile**. `make` est simple du moment où vous décidez de l’utiliser simplement, c’est justement son atout majeur !

Makefile a plusieurs niveaux de complexité, mais c’est vous qui décidez quand vous arrêter de faire de l’optim.

Dans vos projets, tout ce que vous devez faire, c’est :

```
tache: dependance1 dependance2
	commande1
	commande2
```

C’est **tout ce qu’il y a à retenir**. Le reste, c’est du bonus.

Et je conclus : les gars (et les dames), bien entendu que je trouve GoTask sympa, pratique dans certains cas, et que je l’utilise. Ne me prenez pas pour un vieux con archaïque qui ne jure que par la vieille méthode. Mais je refuse de lâcher des trucs qui marche bien, qui sont pratiques et clairement adaptés, sous prétexte que la mode est d’infantiliser les développeurs.

Apprendre, maitriser et utiliser plein d’outils, de langages, c’est quand même plus passionnant.