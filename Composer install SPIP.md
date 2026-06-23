---
tags:
  - composer
  - install
---
# Composer : La commande create-project

Depuis la sortie de ecrire/ du dépot SPIP, SPIP est un projet composer. Ce document explique un peu comment utiliser composer pour des installations SPIP. Elle ne s’adresse donc qu’aux gens qui

- dev SPIP (core et plugins-dist)
- gens qui dev AVEC SPIP et veulent utiliser la puissance de composer et la ligne de commande

## [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-avant-propos-laide-de-composer-2)Avant propos : l’aide de composer

Dans tous les cas, en cas de doute, se référer à l’aide de composer

### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-en-ligne-3)En ligne

[getcomposer.org](https://getcomposer.org/doc/03-cli.md#create-project)

### [Command-line interface / Commands - Composer](https://getcomposer.org/doc/03-cli.md#create-project)

A Dependency Manager for PHP

### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-dans-un-terminal-4)Dans un terminal

`composer help create-project`

`Usage:   create-project [options] [--] [<package> [<directory> [<version>]]]`

Les « [options] » seront détaillées plus bas ou ultérieurement.

Les 3 options suivantes s’expliquent ainsi :

- `package` = un vendor/name valide et connu sur les dépôts composer connus (par défaut, un seul = [packagist.org](http://packagist.org/))
- `directory` = le dossier où installer l’application sinon « name ». Ne doit pas exister si mentionné
- `version` = la contrainte de version (4.2.16, ^4.1, ~4.4.0, 5.0.*, …) voir [Versions and constraints - Composer](https://getcomposer.org/doc/articles/versions.md)

## [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-principe-de-base-dun-projet-composer-5)Principe de base d’un projet composer

La commande « composer create-project » cherche un fichier composer.json dans le dossier courant (une erreur se produit si le dossier est vide), exactement comme les autres commandes composer (install, update, require, …).

Exemple : Situation où on a le dernier commit de la branche par défaut

- en ssh et étant authentifié·e `git@git.spip.net:spip/spip.git`
- ou bien en https (anonymement ou pas) `https://git.spip.net/spip/spip.git`

`git clone -b master --no-tags --depth 1 git@git.spip.net:spip/spip.git /my/path/to/spip cd /my/path/to/spip`

`composer create-project` => installe le « project » spip/spip à l’aide des directives du fichier composer.json téléchargé dans le dossier `/my/path/to/spip`

Seul le fichier composer.json est nécessaire pour cette commande. Et aucun autre fichier ne lui est utile

Récupérer une copie du fichier [composer.json · master · spip / spip · GitLab](https://git.spip.net/spip/spip/-/blob/master/composer.json?ref_type=heads) suffirait aussi, s’il n’y avait pas d’autres fichiers versionnés dans ce dépôt et nécessaires au fonctionnement de l’application SPIP.

## [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-dans-le-monde-de-spip-6)Dans le monde de SPIP

Le plus utile est d’installer SPIP sans passer directement par la commande git et **sans fichier présent** sur la machine.

### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-le-dpt-composer-7)Le dépôt composer

Les packages spip sont référencés sur un dépôt composer propre à SPIP, il faut donc qu’il soit connu de l’outil composer:

En ligne de commande, à la volée, l’option `--repository=https://get.spip.net/composer` est suffisante

Alternative : paramétrer ce dépôt au niveau global du compte utilisateur avec la commande

`composer global config repositories.spip composer https://get.spip.net/composer`

Le paramètre « package » à installer en tant que **project** devient nécessaire (puisqu’il n’y a pas de fichier composer.json sur le disque)

### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-le-projet-8)Le projet

Contrairement à une bibilothèque de code (library), qui s’installe usuellement dans un sous-dossier vendor/, en tant que dépendance, un project est installé à la racine du dossier qui contiendra toute l’application (vendor/ y compris), à savoir, le code, les fichiers de configuration, les assets web ou tout autre fichier.

Malgré cette distinction, c’est un package composer ordinaire, si ce n’est qu’on préférera, par convention, lui associer le **type** « project » et un « vendor/name » distinct signifiant dans son propre fichier composer.json comme ci-dessous :

`{     "name": "spip/spip",     "type": "project",     ... }`

d’où, la commande de base désormais

`composer create-project --repository=https://get.spip.net/composer spip/spip`

Sans plus d’information, composer ira chercher, en s’appuyant sur les références connues du dépôt dédié à SPIP :

- la dernière version stable (tagguée au sens git du terme)
- sémantiquement correcte (ex: 4.3.5)
- et compatible avec les contraintes de la « plate-forme » où la commande est lancée, c’est-à-dire la version de PHP et ses extensions.

Dans la configuration actuelle, cette dernière version est téléchargée sous la forme du zip généré par le serveur gitlab du dépôt git associé au package spip/spip à savoir « [spip / spip · GitLab](https://git.spip.net/spip/spip.git) ». Ce zip est décompressé dans le dossier indiqué explicitement par la commande (ou dans un sous-dossier « spip » créé à l’occasion (voir explications plus haut).

### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-la-version-9)La version

On peut choisir une version du projet et l’installer sur une machine en utilisant les contraintes de versions gérées par composer. Encore une fois, voir [Versions and constraints - Composer](https://getcomposer.org/doc/articles/versions.md)

Par défaut, composer choisira parmi les versions sémantiquement « stables » (créées via un tag git).

#### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-situation-au-2-janvier-2025-10)Situation au 2 janvier 2025

Sans précision, ce sera la dernière version stable. La dernière version stable de spip est la 4.3.5 du 3 décembre 2024.

Il y a les dossiers ecrire/ et prive/ dedans, mais pas les dossiers plugins-dist/ et squelettes-dist/, gérés via d’autres outils.

on peut aussi récupérer le code d’une version plus ancienne, exemple la 4.2.16 ou la 4.1.18

- `composer create-project --repository=https://get.spip.net/composer spip/spip spip-4.2 ~4.2`
- `composer create-project --repository=https://get.spip.net/composer spip/spip spip-4.1.18 4.1.18`

De même, il y a les dossiers ecrire/ et prive/ dedans, mais pas plugins-dist/ et squelettes-dist/, gérés via d’autres outils

`composer create-project --repository=https://get.spip.net/composer spip/spip spip-4.4 ^4.4` provoquera une erreur, car il n’y pas de tag stable pour l’instant pour cette branche.

#### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-versions-beta-ou-alpha-ou-rc-11)Versions beta (ou alpha ou RC)

Ajouter l’option « –stability=beta » aux autres options.

`composer create-project --repository=https://get.spip.net/composer --stability=beta spip/spip`

Installe la version 5.0.0-beta du 3 décembre 2024 dans le dossier « spip »

Les dossiers ecrire/, prive/, plugins-dist/, squelettes-dist/ sont présents. Les dossier IMG/, local/ et tmp/ sont créés. Le dossier config/ contient ce qui a été versionné au moment du tag. Évidement, les dépendances « externes » sont installées dans le dossier vendor/

`composer create-project --repository=https://get.spip.net/composer --stability=beta spip/spip spip-4.4.0-beta ^4.4`

Installe la version 4.4.0-beta du 3 décembre 2024 dans le dossier spip-4.4.0-beta sans le dossier plugins-dist/ toujours traité avec d’autres outils.

Une commande spécifique à SPIP est disponible : « composer switch:forward » donne à composer le contrôle de l’installation et des mises à jour des plugins-dist. Voir [docs/plugins-dist.md · main · spip-league / composer-installer · GitLab](https://git.spip.net/spip-league/composer-installer/-/blob/main/docs/plugins-dist.md) pour les effets et les conséquences.

#### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-versions-de-dveloppement-12)Versions de développement

Ajouter l’option `--stability=dev` aux options.

`composer create-project --repository=https://get.spip.net/composer --stability=dev spip/spip spip-dev`

Installe, dans le dossier spip-dev, la branche de développement la plus « haute » possible en version en s’appuyant sur le nom des branches et la compatibilité de PHP et de ses extensions.

### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-mode-dtach-versus-synchronis-13)mode détaché versus synchronisé

#### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-ce-quil-y-a-retenir-principalement-14)Ce qu’il y a retenir principalement

Par défaut, create-project installe SPIP en mode détaché, c’est-à-dire, sans référence git, qui est le mode qui correspond au besoin de base : avoir un SPIP opérationnel, prêt à recevoir la touche finale de sa configuration.

C’est **suffisant** pour développer/personnaliser « son » SPIP, un plugin/squelette personnel ou communautaire, et désormais, intégrer d’autres packages composer, d’où qu’ils viennent. Cela permet aussi de versionner « son » SPIP dans un dépôt git indépendant, sans être pollué par les liens aux dépôts des composants « spip/* »

Les personnes qui **développent**, **testent** ou **maintiennent** SPIP lui-même peuvent avoir besoin d’une installation qui reste liée aux dépôts git de tous ses composants. Ci-dessous, quelques options pour parvenir aux besoins spécifiques de chaque personne, en fonction de ce qu’elle souhaite développer, tester ou maintenir dans la distribution classique de SPIP elle-même.

Au lieu de manipuler des commandes git ou s’en remettre à un outil qui cadre les modes d’installation et contraignent les développements et la capacités de tester des branches spécifiques, il devient possible de n’utiliser que composer, à travers ses commandes les plus usuelles, à savoir _install_, _update_, _require_*.*

Par la suite, seules les dépendances du **project**, pourront être changées ou mises à jour avec composer, qui s’appuiera alors sur le fichier composer.lock du dossier d’installation. Tout ce que contient le project sera figé mais modifiable à la discrétion de la personne qui a fait l’installation et non versionné sur le dépôt d’origine.

#### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-option-pour-le-dveloppement-sans-synchro-git-15)Option pour le développement : sans synchro GIT

`composer create-project --repository=https://get.spip.net/composer --stability=dev spip/spip spip-dev`

Installe donc une version de développement de la branche la plus haute (à savoir le dernier commit de la branche « master »)

Les branches de développement du **project** sont installable aussi. Exemple : `composer create-project --repository=https://get.spip.net/composer --stability=dev spip/spip spip-dev dev-issue_5892`

#### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-options-pour-le-dveloppement-avec-synchro-git-16)Options pour le développement : avec synchro GIT

Pour que le dossier de travail reste synchronisé avec le dépôt git du projet, ajouter l’option `--keep-vcs`, composer utilise git de manière masquée pour faire toutes les opérations.

`composer create-project --repository=https://get.spip.net/composer --stability=dev --keep-vcs spip/spip spip-dev`

installe donc une version de développement de la branche la plus haute et conserve le lien avec le dépôt git « [spip / spip · GitLab](https://git.spip.net/spip/spip.git) »

À partir de ce type d’installation, il faudra utiliser git pour basculer le code du **projet** d’une branche à l’autre et composer update ou composer require pour basculer le code des dépendances.

#### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-options-pour-le-dveloppement-avec-synchro-git-aussi-pour-les-composants-spip-league-plugins-dist-commu-php-17)Options pour le développement : avec synchro GIT aussi pour les composants (spip-league, plugins-dist, commu PHP)

Pour que les dépendances (plugins-dist/, squelettes-dist/, ecrire/, prive/ et vendor/, selon les branches de développement) restent elles aussi synchronisées à leurs dépôts git respectifs, ajouter l’option `--prefer-source`

`composer create-project --repository=https://get.spip.net/composer --stability=dev --keep-vcs --prefer-source spip/spip spip-dev`

donne un dossier de travail où le code du projet spip/spip ainsi que toutes les dépendances sans exception, restent synchronisées via git, dépendances de dev du projet y compris.

Pour enlever les dépendances de dev de cette installation, ajouter l’option « –no-dev »

`composer create-project --repository=https://get.spip.net/composer --stability=dev --keep-vcs --prefer-source --no-dev spip/spip spip-dev`

La commande spécifique à SPIP pour passer les urls git en mode ssh reste valable à partir de ce moment:

`composer local mode-dev --no-interaction`

#### [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-options-pour-le-dveloppement-avec-synchro-git-aussi-pour-les-composants-mais-uniquement-ceux-de-spip-spip-league-plugins-dist-18)Options pour le développement : avec synchro GIT aussi pour les composants mais uniquement ceux de SPIP (spip-league, plugins-dist)

`composer config "preferred-install.spip/*" source composer config "preferred-install.spip-league/*" source composer create-project`

# [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-la-commande-archive-19)La commande archive

## [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-aide-20)Aide

En ligne : [Command-line interface / Commands - Composer](https://getcomposer.org/doc/03-cli.md#archive)

Dans le terminal : composer help archive

## [](https://discuter.spip.net/t/composer-la-commande-create-project/186621#p-487027-pour-spip-21)Pour SPIP

Pour faire **un zip de release**, il est possible, de combiner create-project et archive de la manière suivante.

`# Build rm -Rf spip COMPOSER_ROOT_VERSION=5.0.0-beta composer create-project --repository=https://get.spip.net/composer --no-dev "spip/spip:${COMPOSER_ROOT_VERSION}" composer --working-dir=spip archive --format=zip --dir=.. # Verif ls -lh ./spip-5.0.0-beta.zip unzip -l ./spip-5.0.0-beta.zip | more`

il n’y a qu’un seul paramètre à changer pour générer le zip d’une autre version, la variable d’environnement COMPOSER_ROOT_VERSION

Pour les versions de la branche 4.4, l’étape d’installation des plugins-dist est à traiter entre create-project et archive

Pour les versions antérieures à 4.4 (à savoir, la 4.3 qui sera en security-fix dès la fin janvier, et pour 6 mois) l’ancien mode de release sera toujours à utiliser.

Les opérations de vérification de versions des dépendances, de changelog etc. sont a effectuer avant de créer un tag et ne concernent donc pas composer.

De même pour l’opération pour pousser les artefacts dans une release gitlab à effectuer après et sans composer.

Le changelog est

- soit mis à jour manuellement par l’équipe maintenance au file de l’eau (après merge de PR)
- soit sa génération est automatisée après la création du tag.

Les vérifications de dépendances pour SPIP4.4 dépendent encore du fichier plugins-dist.json sur lequel composer n’a ni impact, ni influence.