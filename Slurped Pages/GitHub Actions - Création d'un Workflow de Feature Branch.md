---
link: https://enix.io/fr/blog/github-feature-branch/
byline: David Donchez
site: "@enixsas"
excerpt: Découvrez comment créer pas à pas un workflow de Feature Branch avec la
  solution de CI/CD Github Actions.
twitter: https://twitter.com/@enixsas
slurped: 2024-11-24T12:18
title: "GitHub Actions : Création d'un Workflow de Feature Branch"
---

Dans cet article d'**introduction à [Github Actions](https://github.com/features/actions)**, je vous présente une méthode pour mettre en place un workflow de CI ([Intégration Continue](https://enix.io/fr/blog/integration-continue-ci/)) souvent désigné sous le terme de **“_feature branch_”**.

Nous allons l’illustrer sur un projet concret que nous avons récemment forké pour notre usage interne chez Enix. Même si le projet en lui-même n’est pas le sujet principal de l’article, il est bon de préciser qu’il permet de générer des images d’extensions système pour l’OS [Flatcar](https://www.flatcar.org/).

## Pourquoi un Workflow de Feature Branch ?

Nous sommes plusieurs collègues à collaborer sur ce projet, l’intégration d’un workflow de CI est donc essentielle pour nous permettre de **participer simultanément au développement** et **tester efficacement nos modifications** sans impacter la base de code ni le travail des autres contributeurs.

Chaque développeur doit avoir la capacité de **créer des artéfacts** (dans notre cas d’usage, il s’agit d’une image d’extension système) et de les **tester individuellement** avant de faire sa _merge request_ finale. Ces artéfacts seront publiés sous forme de release publique (disponible pour tous) ou privée (lors de la phase de développement).

Il est pour nous essentiel :

- D'**éviter la création d’une release publique accessible à tous** les utilisateurs pendant la phase de développement.
- D’**automatiser le processus de création d’une release** selon nos critères préalablement établis.

Partant de ces prérequis, **on peut imaginer ces étapes dans notre workflow** :

- Récupération des sources du projet ;
- Création de notre artéfact ;
- Création d’une release, qui sera privée (=_draft release_) ou publique en fonction du besoin du développeur.

Voici la représentation visuelle du workflow :

![Diagramme du workflow de Feature Branch](https://enix.io/fr/blog/github-feature-branch/diagramme.webp "Diagramme du workflow de Feature Branch")

## Pourquoi Utiliser Github Actions ?

Au cœur de l’écosystème de Github se trouve **Github Actions**, un **outil d’automatisation** conçu pour **améliorer et rationaliser les workflows des développeurs**.

D’un point de vue technique, plusieurs aspects de cet outil se démarquent. Attention tout de même, ma liste n’est pas exhaustive et n’est donc pas une analyse complète de l’outil… :-)

### Avantages

- **Visibilité des workflows** : Avec Github Actions, les workflows sont nativement intégrés dans l’interface utilisateur. Ceci permet d’avoir une visibilité claire de leur exécution tout en gardant un historique ;
- **Github Marketplace** : C’est une des forces majeures de Github Actions, elle permet aux développeurs d’accéder à une vaste gamme d'_actions_ préconstruites ;
- **Intégration avec Github** : Le système d'_actions_ de Github est nativement intégré avec le dépôt de code Github, rendant les automatisations faciles à mettre en place sur ce type d’environnement de développement ;
- **Configuration simple** : La gestion des workflows se fait facilement via des fichiers de configuration YAML.

### Inconvénients

- **Visibilité publique** : toute erreur, en particulier lors d’une release publique, est visible publiquement. Cela nécessite une diligence accrue et une vérification soignée avant les déclenchements du workflow ;
- **Performance des _runners_** : Bien que les exécuteurs (nommés **_runners_** en langage Github) offrent une durée d’utilisation substantielle, ils peuvent parfois être considérés comme manquant de puissance, en particulier lors de l’exécution de tâches intensives comme de la compilation. Cela peut entraîner des temps d’exécution parfois longs ;
- **Absence de validation des workflows** : Il est regrettable de ne pas avoir d’outil natif aidant à analyser ou debugger les workflows ;
- **Cas d’usages complexe** : Certaines configurations peuvent s’avérer complexes à mettre en place comme par exemple les dépendances entre plusieurs dépôts de code.

L’utilisation de Github Action est aisée dans notre cas car notre code est hébergé sur un dépôt Github avec lequel l’intégration est native. Il est cependant possible d’implémenter un **workflow de Feature Branch similaire avec d’autres solutions de CI/CD**, nous apprécions notamment Gitlab CI ou encore Tekton.

## Implémentation du Workflow de Feature Branch avec Github Actions

Pour rappel, d’un point de vue Github Actions, un **_job_** représente une succession d’étapes (ou **_steps_**) exécutées sur le même **_runner_** (exécuteur).

### Déclenchement du Workflow

Notre workflow étant relativement simple, nous allons écrire **un seul _job_** que nous allons découper en plusieurs _steps_. Tout d’abord, nous allons déclarer la **méthode de déclenchement de notre workflow** :

```
name: Build and release Systemd sysext images
on:
  workflow_dispatch:
```

Ici, la méthode de déclenchement est très simple, cela consiste à dire “Nous allons lancer notre workflow en utilisant l’interface web de Github”. Mais notez qu’il est tout à fait possible d’utiliser d’autres méthodes en fonction des besoins : déclenchement lors de la création d’une _pull request_, lors du push du code sur une branche particulière, etc.

### Les Etapes du Workflow

La première étape du workflow (probablement la même sur 99% des workflows de CI) est de récupérer le code du projet :

```
    steps:
      # récupération des sources du projet
      - uses: actions/checkout@v3
```

Puis vient une _step_ de _build_ largement dépendante du projet sur lequel vous travaillez. Dans certains cas, elle pourrait consister en la construction d’une image de conteneur, ou encore la compilation d’un binaire. Dans notre cas, nous il s’agit de **construire notre artéfact en exécutant un script shell**.

Cette _step_ va **produire un certain nombre de fichiers** : nos images d’extension systèmes ainsi que les checksums associés. Nous en aurons besoin un peu plus tard dans le workflow. Voici le YAML associé :

```
      - name: Build
        run: |
          set -euo pipefail

          images=(
              "teleport-9.3.26"
              "teleport-10.3.16"
              "teleport-11.3.22"
          )

          for image in ${images[@]}; do
              component="${image%-*}"
              version="${image#*-}"
              "./create_${component}_sysext.sh" "${version}" \
                "${component}"
              mv "${component}.raw" "${image}.raw"
          done

          sha256sum *.raw > SHA256SUMS          
```

La **Marketplace de Github Actions** est vraiment complète et permet de facilement trouver des _actions_ qui couvrent un besoin spécifique. Le système de notation est également particulièrement intéressant pour s’assurer d’utiliser une _action_ fonctionnelle.

![Screenshot de la Marketplace Github Actions](https://enix.io/fr/blog/github-feature-branch/screenshot-marketplace-github-actions.webp "Screenshot de la Marketplace Github Actions")

Comme nous souhaitons **créer une release publique sur Github** référençant le résultat de notre _build_, nous allons utiliser une _action_ populaire :[https://github.com/softprops/action-gh-release](https://github.com/softprops/action-gh-release).

Pour fonctionner correctement, cette _action_ nécessite qu’on lui désigne un tag git. Nous lançons donc notre workflow non pas sur une branche mais sur un tag (qui correspond au dernier commit de notre branche).

Cette _step_ est commune pour nos deux types de release, privée ou publique. Il est donc essentiel de déterminer dans quel contexte nous nous trouvons. Pour ce faire, cette distinction se fera en fonction du tag git.

Dans notre cas, par souci de simplicité, nous choisissons la **nomenclature suivante** :

- Les tags numériques correspondront à une release publique.
- Les tags alphanumériques seront utilisés pour le développement.

Ajoutons deux _steps_ à notre workflow : une pour **récupérer le tag**, et une autre pour **déterminer le type de release**.

```
      - name: Get git tag
        id: tag
        uses: devops-actions/action-get-tag@v1.0.2

      - name: (Pre)Release check
        run: |
          if [[ ${{ steps.tag.outputs.tag }} =~ ^[0-9]+$ ]]; then
             echo "TAG_TYPE=numeric" >> $GITHUB_ENV
          else
             echo "TAG_TYPE=alphanumeric" >> $GITHUB_ENV
          fi          
```

Ici, nous utilisons des variables d’environnements qui seront utilisées dans la prochaine _step_ pour déterminer le type de release que nous souhaitons créer.

Enfin, nous pouvons créer notre release en utilisant les artéfacts qui ont été produits lors de l’étape précédente de _build_ :

```
      - name: Release
        uses: softprops/action-gh-release@v1
        if: env.TAG_TYPE == 'numeric'
        with:
          files: |
            SHA256SUMS
            *.raw            
      - name: Draft Release
        uses: softprops/action-gh-release@v1
        if: env.TAG_TYPE == 'alphanumeric'
        with:
          draft: true
          files: |
            SHA256SUMS
            *.raw            
```

Notez que dans le cas d’une branche de développement, nous créons une release privée qui n’est pas exposée publiquement. Elle nous permet de récupérer nos artéfacts pour pouvoir les tester.

### Affichage des Steps Github Actions

Nous parlions précédemment de la visibilité des workflows qui est un atout majeur de Github Actions. C’est vrai lorsque l’on construit un workflow avec des _jobs_ interdépendants, mais également pour les _jobs_ avec plusieurs _steps_ (comme pour ce workflow de Feature Branch présenté dans cet article) :

![Steps Github Actions](https://enix.io/fr/blog/github-feature-branch/steps.webp "Steps Github Actions")

Github Actions propose également un affichage détaillé de chacune des _steps_ de notre workflow, notamment les **logs d’exécution**. C’est idéal pour tester le résultat de la construction de notre branche.

## Conclusion

Nous avons pu le voir à travers la création de ce workflow de “feature branch”, **Github Actions est une solution de CI/CD riche, flexible et relativement facile à utiliser**.

Parmi ses atouts, sa Marketplace contient de nombreuses _actions_ prêtes à l’emploi. Largement utilisée, la solution est également **facile à intégrer avec de nombreux autres outils pour enrichir les workflow** d’intégration et de tests avant la mise en production des applications.

Suivant les besoins et préférences des développeurs, nous pouvons **déployer et gérer des chaînes CI/CD complètes** sur nos infrastructures Enix, qu’elles soient basées sur Github Actions ou sur Gitlab CI. N’hésitez pas à nous contacter si vous avez besoin d’aide ! :-)
