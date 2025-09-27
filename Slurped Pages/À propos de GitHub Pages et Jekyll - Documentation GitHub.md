---
link: https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll
site: GitHub Docs
excerpt: Jekyll est un générateur de site statique avec une prise en charge intégrée de GitHub Pages.
tags:
  - github
  - jekyll
  - page
slurped: 2025-09-27T17:46
title: À propos de GitHub Pages et Jekyll - Documentation GitHub
---

Jekyll est un générateur de site statique avec une prise en charge intégrée de GitHub Pages.

## Qui peut utiliser cette fonctionnalité ?

GitHub Pages est disponible dans les référentiels publics avec GitHub Free et GitHub Free pour les organisations, et dans les référentiels publics et privés avec GitHub Pro, GitHub Team, GitHub Enterprise Cloud et GitHub Enterprise Server. Pour plus d’informations, consultez [Plans de GitHub](https://docs.github.com/fr/get-started/learning-about-github/githubs-plans).

Remarque

While the `github-pages` gem remains supported for some workflows, GitHub Actions is now the recommended approach for deploying and automating GitHub Pages sites.

## [À propos de Jekyll](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll#about-jekyll)

Jekyll est un générateur de site statique avec une prise en charge intégrée de GitHub Pages et un processus de génération simplifié. Jekyll prend les fichiers Markdown et HTML et crée un site web statique complet en fonction de vos choix de dispositions. Jekyll prend en charge Markdown et Liquid, langage de modèle qui charge du contenu dynamique sur votre site. Pour plus d’informations, consultez [Jekyll](https://jekyllrb.com/).

Jekyll n’est pas officiellement pris en charge pour Windows. Pour plus d’informations, consultez [Jekyll sur Windows](https://jekyllrb.com/docs/windows/#installation) dans la documentation de Jekyll.

Nous recommandons d’utiliser Jekyll avec GitHub Pages. Si vous préférez, vous pouvez utiliser d’autres générateurs de sites statiques ou personnaliser votre propre processus de génération en local ou sur un autre serveur. Pour plus d’informations, consultez « [Création d’un site GitHub Pages](https://docs.github.com/fr/pages/getting-started-with-github-pages/creating-a-github-pages-site#static-site-generators) ».

## [Configuration de Jekyll dans votre site GitHub Pages](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll#configuration-de-jekyll-dans-votre-site-github-pages)

Vous pouvez configurer la plupart des paramètres Jekyll, comme le thème et les plug-ins de votre site, en modifiant votre fichier `_config.yml`. Pour plus d’informations, consultez [Configuration](https://jekyllrb.com/docs/configuration/) dans la documentation de Jekyll.

Certains paramètres de configuration ne peuvent pas être modifiés pour les sites GitHub Pages.

```
lsi: false
safe: true
source: [your repo's top level directory]
incremental: false
highlighter: rouge
gist:
  noscript: false
kramdown:
  math_engine: mathjax
  syntax_highlighter: rouge
```

Par défaut, Jekyll ne génère pas de fichiers ni de dossiers qui :

- Se trouvent dans un dossier appelé `/node_modules` ou `/vendor`
- Commencent par `_`, `.` ou `#`
- Se terminent par `~`
- Sont exclus par le paramètre `exclude` de votre fichier de configuration

Si vous souhaitez que Jekyll traite l’un de ces fichiers, vous pouvez utiliser le paramètre `include` dans votre fichier de configuration.

## [En-tête](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll#front-matter)

Pour définir des variables et des métadonnées, telles qu’un titre et une mise en page, pour une page ou une publication de votre site, vous pouvez ajouter des pages liminaires YAML en haut de n’importe quel fichier Markdown ou HTML. Pour plus d’informations, consultez [Pages liminaires](https://jekyllrb.com/docs/front-matter/) dans la documentation de Jekyll.

Vous pouvez ajouter `site.github` à une publication ou une page pour ajouter des métadonnées de référence de dépôt à votre site. Pour plus d’informations, consultez [Utilisation de `site.github`](https://jekyll.github.io/github-metadata/site.github/) dans la documentation sur les métadonnées de Jekyll.

## [Thèmes](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll#themes)

Vous pouvez ajouter un thème Jekyll à votre site GitHub Pages pour personnaliser son apparence. Pour plus d’informations, consultez [Thèmes](https://jekyllrb.com/docs/themes/) dans la documentation de Jekyll.

Vous pouvez ajouter un thème pris en charge à votre site dans GitHub. Pour plus d’informations, consultez [Thèmes pris en charge](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll#supported-themes) sur le site GitHub Pages et [Ajout d’un thème à votre site GitHub Pages avec Jekyll](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll).

Pour utiliser tout autre thème Jekyll open source hébergé sur GitHub, vous pouvez ajouter le thème manuellement. Pour plus d’informations, consultez [Thèmes hébergés sur GitHub](https://github.com/topics/jekyll-theme) et [Ajout d’un thème à votre site GitHub Pages avec Jekyll](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll).

Vous pouvez remplacer les valeurs par défaut de votre thème en modifiant les fichiers du thème. Pour plus d’informations, consultez la documentation de votre thème et [Remplacement des valeurs par défaut de votre thème](https://jekyllrb.com/docs/themes/#overriding-theme-defaults) dans la documentation de Jekyll.

## [Plug-ins](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll#plugins)

Vous pouvez télécharger ou créer des plug-ins Jekyll pour étendre les fonctionnalités de Jekyll pour votre site. Par exemple, le plug-in [jemoji](https://github.com/jekyll/jemoji) vous permet d’utiliser l’emoji GitHub dans n’importe quelle page de votre site de la même façon que vous le feriez sur GitHub. Pour plus d’informations, consultez [Plug-ins](https://jekyllrb.com/docs/plugins/) dans la documentation de Jekyll.

GitHub Pages utilise des plug-ins qui sont activés par défaut et qui ne peuvent pas être désactivés :

- [`jekyll-coffeescript`](https://github.com/jekyll/jekyll-coffeescript)
- [`jekyll-default-layout`](https://github.com/benbalter/jekyll-default-layout)
- [`jekyll-gist`](https://github.com/jekyll/jekyll-gist)
- [`jekyll-github-metadata`](https://github.com/jekyll/github-metadata)
- [`jekyll-optional-front-matter`](https://github.com/benbalter/jekyll-optional-front-matter)
- [`jekyll-paginate`](https://github.com/jekyll/jekyll-paginate)
- [`jekyll-readme-index`](https://github.com/benbalter/jekyll-readme-index)
- [`jekyll-titles-from-headings`](https://github.com/benbalter/jekyll-titles-from-headings)
- [`jekyll-relative-links`](https://github.com/benbalter/jekyll-relative-links)

Vous pouvez activer des plug-ins supplémentaires en ajoutant le gem du plug-in au paramètre `plugins` de votre fichier `_config.yml`. Pour plus d’informations, consultez [Configuration](https://jekyllrb.com/docs/configuration/) dans la documentation de Jekyll.

Pour obtenir la liste des plug-ins pris en charge, consultez [Versions des dépendances](https://pages.github.com/versions.json) sur le site GitHub Pages. Pour plus d’informations sur l’utilisation d’un plug-in spécifique, consultez la documentation du plug-in.

GitHub Pages ne peut pas générer de sites avec des plug-ins non pris en charge. Si vous souhaitez utiliser des plugins non pris en charge, générez votre site localement, puis envoyez les fichiers statiques de votre site vers GitHub.

## [Mise en évidence de la syntaxe](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll#syntax-highlighting)

Pour faciliter la lecture de votre site, les extraits de code sont mis en évidence sur les sites GitHub Pages de la même manière qu'ils sont mis en évidence sur GitHub. Pour plus d'informations sur la coloration syntaxique, voir [Création et mise en évidence de blocs de code](https://docs.github.com/fr/get-started/writing-on-github/working-with-advanced-formatting/creating-and-highlighting-code-blocks).

Par défaut, les blocs de code de votre site sont colorés par Jekyll. Jekyll utilise le surligneur [Rouge](https://github.com/rouge-ruby/rouge), qui est compatible avec [Pygments](https://pygments.org/)). Si vous spécifiez Pygments dans votre fichier `_config.yml`, Rouge sera utilisé à la place comme solution de repli. Jekyll ne peut pas utiliser d’autres colorateurs de syntaxe et vous obtiendrez un avertissement de génération de page si vous spécifiez un autre colorateur de syntaxe dans votre fichier `_config.yml`. Pour plus d’informations, consultez « [À propos des erreurs de build Jekyll pour les sites GitHub Pages](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/about-jekyll-build-errors-for-github-pages-sites) ».

Remarque

Rouge reconnaît uniquement les identifiants de langue en minuscules pour les blocs de code délimités. Pour obtenir la liste des langues prises en charge, consultez [Langues](https://rouge-ruby.github.io/docs/file.Languages.html).

Si vous voulez utiliser un autre surligneur, par exemple [highlight.js](https://github.com/highlightjs/highlight.js), vous devez désactiver la coloration syntaxique de Jekyll en mettant à jour le fichier `_config.yml` de votre projet.

```
kramdown:
  syntax_highlighter_opts:
    disable : true
```

Si votre thème n’inclut pas de CSS pour la coloration syntaxique, vous pouvez générer le CSS de coloration syntaxique de GitHub et l’ajouter au fichier `style.css` de votre projet.

```
rougify style github > style.css
```

## [Création de votre site en local](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll#building-your-site-locally)

Si vous publiez à partir d'une branche, les modifications apportées à votre site sont publiées automatiquement lorsqu'elles sont fusionnées dans la source de publication de votre site. Si vous publiez à partir d'un flux de travail personnalisé GitHub Actions, les modifications sont publiées chaque fois que votre flux de travail est déclenché (généralement par une poussée vers la branche par défaut). Si vous souhaitez prévisualiser vos modifications, vous pouvez les effectuer localement plutôt que sur GitHub. Ensuite, testez votre site localement. Pour plus d’informations, consultez « [Test de votre site GitHub Pages localement avec Jekyll](https://docs.github.com/fr/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll) ».