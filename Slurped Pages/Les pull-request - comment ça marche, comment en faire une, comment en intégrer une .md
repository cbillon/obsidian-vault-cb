---
link: https://blog.zenika.com/2017/01/24/pull-request-demystifie/
byline: Alexandre Garnier
site: Blog Zenika
date: 2017-01-24T09:54
excerpt: "GitHub a popularisé le principe de pull-request et tous les autres outils de gestion de dépôt Git s’y sont mis : Bitbucket Cloud, Bitbucket Server (anciennement Stash), GitLab (sous le terme …"
slurped: 2025-01-23T09:41
title: "Les pull-request : comment ça marche, comment en faire une, comment en intégrer une ?"
tags:
  - git
  - pull-request
---

[GitHub](https://github.com/) a popularisé le principe de pull-request et tous les autres outils de gestion de dépôt Git s’y sont mis : [Bitbucket Cloud](https://bitbucket.org/), [Bitbucket Server](https://bitbucket.org/product/server) (anciennement Stash), [GitLab](https://gitlab.com/) (sous le terme de merge-request).  
Dans le principe c’est simple : pour contribuer à un projet sur l’une de ces plateformes :

1. Forker le projet
2. Créer une branche et travailler dessus
3. Publier la branche sur son fork
4. Créer la pull-request

Mais dans les faits, ça peut être un peu plus compliqué…  
Nous allons voir étape par étape comment cela fonctionne et comment s’en servir au mieux.  
  
Nous allons nous concentrer sur la situation de workflow triangulaire, c’est à dire de travail avec 3 dépôts :

- **Un dépôt de référence, conventionnellement appelé _upstream_**  
    C’est le dépôt du projet auquel nous voulons contribuer.  
    Nous n’avons que les droits en lecture dessus.
- **Un dépôt de fork, conventionnellement référencé _origin_**  
    C’est une copie du dépôt de référence.  
    Nous avons tous les droits dessus.
- **Un dépôt local**  
    C’est notre dépôt de travail.

[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_00_triangular-1.png?resize=800%2C700&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_00_triangular-1.png?ssl=1)  
Depuis la version 2.5, Git simplifie le travail avec ce type de workflow avec l’introduction de la référence [<branch>@{push}](https://git-scm.com/docs/git-rev-parse#Documentation/git-rev-parse.txt-emltbranchnamegtpushemegemmasterpushemempushem).  
Il existe déjà la référence `[<branch>@{upstream}](https://git-scm.com/docs/git-rev-parse#Documentation/git-rev-parse.txt-emltbranchnamegtupstreamemegemmasterupstreamememuem)` qui permet de déterminer la branche distante traquée par une branche. Celle-ci est automatiquement définie lorsque vous faites un `git checkout -b branch upstream_branch` ou peut être explicitement définie par un `git branch --set-upstream-to upstream_branch branch`. Cette référence permet de déterminer la branche avec laquelle fusionner/rebaser lors d’un `git pull`. Elle permet aussi, si votre configuration [`push.default`](https://git-scm.com/docs/git-config#git-config-pushdefault) est à `upstream`, de déterminer la branche vers laquelle publier par défaut lors d’un `git push`.  
Avec l’apparition de la référence `<branch>@{push}`, il est maintenant possible de mieux contrôler la branche vers laquelle publier par défaut. Celle-ci est configurée par les options de configuration :

- [`remote.pushDefault`](https://git-scm.com/docs/git-config#git-config-remotepushDefault) (ou `branch.<name>.pushRemote` pour une branche spécifique) : pour indiquer le dépôt par défaut sur lequel publier
- [`push.default`](https://git-scm.com/docs/git-config#git-config-pushdefault) : une valeur à `current` va provoquer une publication par défaut sur une branche portant le même nom que la branche courante dans le dépôt de publication

Il est possible de vérifier la valeur de ces références via les commandes suivantes (ici des valeurs dans un cas de workflow triangulaire) :

$ git rev-parse --symbolic-full-name --abbrev-ref @{upstream}
upstream/master
$ git rev-parse --symbolic-full-name --abbrev-ref @{push}
origin/current_branch

Dans notre exemple, nous allons apporter une contribution au logiciel ‘example’ publié sur _[http://git.example.com/org/example](http://git.example.com/org/example)_.  
Ce dépôt contient 2 commits `C1` et `C2` et une branche par défaut `master` positionnée sur `C2` :  
[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_01_upstream-1.png?resize=292%2C198&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_01_upstream-1.png?ssl=1)

## Côté contributeur

### Faire une contribution

Tout d’abord, il faut forker le projet.  
Cette étape est simple : il suffit d’aller sur l’interface web du projet auquel nous voulons contribuer, en l’occurrence _[http://git.example.com/org/example](http://git.example.com/org/example)_, et de cliquer sur le bouton ‘Fork’ (c’est a priori le même sur GitHub, GitLab ou Bitbucket).  
Cela va avoir pour effet de créer un clone du dépôt mais côté serveur.  
Ce dépôt va alors être accessible à une URL du genre _[http://git.example.com/contributor/example](http://git.example.com/contributor/example)_.  
Il nous appartient et nous avons tous les droits dessus.  
[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_02_remotes-1.png?resize=699%2C198&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_02_remotes-1.png?ssl=1)  
Ensuite nous clonons localement notre dépôt :

git clone http://git.example.com/contributor/example

Comme indiqué en introduction, nous allons configurer un peu pour publier par défaut sur notre fork _origin_ :

git config remote.pushdefault origin # Publier par défaut sur le dépôt 'origin'
git config push.default current # Publier par défaut sur une branche portant le même nom que la branche courante dans le dépôt de publication

Enfin, nous allons ajouter en remote le dépôt de référence avec le nom conventionnel _upstream_ :

git remote add upstream http://git.example.com/org/example
git fetch upstream

[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_03_cloned-1.png?resize=526%2C508&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_03_cloned-1.png?ssl=1)  
Voilà, nous pouvons enfin vraiment travailler ! Enfin presque…  
Une bonne pratique est de toujours travailler sur une branche spécifique, jamais sur la branche cible de notre développement.

git checkout -b contribution upstream/master

Ainsi nous allons travailler sur la branche `contribution` qui va automatiquement traquer la branche `upstream/master`.  
[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_04_branched-1.png?resize=526%2C508&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_04_branched-1.png?ssl=1)  
Nous pouvons maintenant faire les modifications de code et commiter :

# hack, hack, hack
git add X Y Z
git commit
# hack again
git add A B C
git commit

[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_05_committed-1.png?resize=551%2C508&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_05_committed-1.png?ssl=1)

### Publier et proposer une contribution

Avant publication, nous pouvons vérifier les commits que nous allons publier et où :

$ git rev-parse --symbolic-full-name --abbrev-ref @{push}
origin/contribution
$ git log --graph --oneline --decorate --date-order --full-history @{push}..HEAD
* C4 (HEAD -> contribution) Commit C4
* C3 Commit C3

Pour publier ces modifications, il nous suffit alors simplement de faire un :

git push

Ainsi, git va publier sur la branche configurée en publication ou `@{push}`, or, puisque nous avons défini les options `remote.pushdefault=origin` et `push.default=current`, c’est `origin/contribution`.  
[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_06_pushed-1.png?resize=663%2C508&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_06_pushed-1.png?ssl=1)  
Enfin, il faut passer par l’interface web du projet pour créer la pull-request depuis votre branche `contribution` de votre fork vers la branche `master` du dépôt de référence.

### Corriger une pull-request

Si vous avez des retours sur votre pull-request et des corrections à faire, il suffit de réitérer l’étape précédente.  
Simplement corriger et commiter :

# fix, fix, fix
git add --update
git commit

[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_07_fixed-1.png?resize=702%2C508&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_07_fixed-1.png?ssl=1)  
Puis publier :

$ git log --graph --oneline --decorate --date-order --full-history @{push}..
* C5 (HEAD -> contribution) Commit C5
$ git push

[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_08_fix_pushed-1.png?resize=790%2C508&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_08_fix_pushed-1.png?ssl=1)  
La pull-request sera automatiquement mise à jour avec le(s) nouveau(x) commit(s).

### Mettre à jour une pull-request

Il peut arriver que le dépôt de référence évolue pendant le temps de validation de votre pull-request, il vous faut alors la mettre à jour.  
[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_09_upstream_changed-1.png?resize=800%2C421&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_09_upstream_changed-1.png?ssl=1)  
Se pose ici la question du merge vs. rebase mais je vais laisser d’autres répondre à ce [débat](https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa), et opter pour ma recommandation dans ce cas : le rebase.  
Vérifions ce que l’on va récupérer :

$ git log --graph --oneline --date-order --full-history ..@{upstream}
* C7 Commit C7
* C6 Commit C6

Et intégrons-le :

git pull --rebase

Cela va faire un `rebase` sur la dernière version de la branche traquée par votre branche locale ou `@{upstream}`, qui est `upstream/master` comme expliqué plus haut.  
[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_10_rebased-1.png?resize=800%2C429&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_10_rebased-1.png?ssl=1)  
Et maintenant republions tout simplement.

git push

Mais ça coince… Car la publication n’est pas **fast-forward**.  
En effet, `origin/contribution` n’est pas un ancêtre de `contribution`.  
Mais ce n’est pas grave, nous voulons justement écraser avec notre nouvelle version, donc nous allons forcer un peu, mais en s’assurant quand même que nous n’allons pas écraser quelque chose que nous ne connaîtrions pas (ce qui est peu probable dans le cas de notre fork personnel) :

git push --force-with-lease

[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_11_rebase_pushed-1.png?resize=800%2C342&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_11_rebase_pushed-1.png?ssl=1)  
La pull-request sera encore une fois automatiquement mise à jour.

## Côté mainteneur

Voyons maintenant comment gérer les pull-requests qui nous sont soumises.

### Vérifier une pull-request

Si la pull-request n’est pas en conflit et que nous pouvons nous contenter de lire la pull-request en ligne et que nous avons des tests automatisés, tant mieux.  
Par contre, si vous avez besoin de la retravailler en local, les différents outils proposent des références spéciales pour récupérer les pull-requests :

# GitHub
git fetch upstream refs/pull/{PR_NUMBER}/from:pull/{PR_NUMBER}
# GitLab
git fetch upstream refs/merge-requests/{PR_NUMBER}/head:pull/{PR_NUMBER}
# BitBucket Server
git fetch upstream refs/pull-requests/{PR_NUMBER}/from:pull/{PR_NUMBER}
# Bitbucket Cloud (obligé ici de taper directement sur le fork du contributeur)
git fetch http://git.example.com/{PR_CONTRIBUTOR}/example {PR_SOURCE_BRANCH}:pull/{PR_NUMBER}

[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_12_pulled-1.png?resize=800%2C489&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_12_pulled-1.png?ssl=1)  
Nous nous retrouvons alors avec une branche locale `pull/{PR_NUMBER}` sur laquelle nous pouvons travailler classiquement.

### Intégrer une pull-request

Si tout se passe bien et que la pull-request peut être intégrée via l’interface web, parfait : simplement cliquer sur ‘fusionner’.  
Mais parfois, si la branche cible a évolué de manière conflictuelle avec la pull-request (et que le contributeur ne peut la mettre à jour comme montré précédemment) ou que vous préférez simplement faire les choses vous-même, voici comment procéder.  
Tout d’abord, récupérer la branche de la pull-request comme présenté dans le point précédent, ensuite, selon vos préférences, fusionner avec ou sans _fast-forward_, avec ou sans _squashing_ :

git merge --no-ff pull/{PR_NUMBER}

[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_13_merged-1.png?resize=800%2C494&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_13_merged-1.png?ssl=1)  
Il ne reste plus qu’à publier :

git push upstream

[![](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_14_merge_pushed-1.png?resize=800%2C471&ssl=1)](https://i0.wp.com/blogzenika.wpcomstaging.com/wp-content/uploads/2016/12/pr_14_merge_pushed-1.png?ssl=1)  
La pull-request sera alors automatiquement fermée.

## Conclusion

J’espère que vous comprenez un peu mieux comment fonctionne une pull-request et comment la manipuler de manière plus fine.

---

## En apprendre plus :

- [Formation Git : savoir mettre en place et configurer Git](https://training.zenika.com/fr/training/git/description)
- [Formation Clean Code : Concevoir et écrire un code propre, améliorer un code existant](https://training.zenika.com/fr/training/clean-code/description)
- [Formation Crafting Code : Apprendre les secrets du Clean Code et du TDD avec Sandro Mancuso](https://training.zenika.com/fr/training/crafting-code/description)

- ![Alexandre Garnier](https://secure.gravatar.com/avatar/333cc4610b5aeba3603a4efda95dc866?s=80&d=identicon&r=g)