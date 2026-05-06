---
tags:
  - moodle
  - composer
  - install
---

# Support Moodle installation using composer

## What are we looking at and why?

To make the installation, maintenance, support, and upgrade of Moodle easier, we’re looking at adding the ability to install Moodle using Composer.

Other large FOSS projects have successfully made this kind of migration, most notably Drupal.

In our case we want to make it possible to install a range of things using Composer, including:

- Moodle;
    
- Moodle plugins; and
    
- Moodle dependencies.
    

We also need to investigate how this impacts:

- Development of core;
    
- Development of plugins; and
    
- Testing.
    

## Main areas of work

The main areas that have been identified for this body of work are:

- the creation of a template project.
    
- a new composer plugin to help install Moodle packages into the correct location.
    
- some ‘scaffolding' tooling to make it easier to install Moodle
    

### Template project

This has been initially called `moodle/seed` (to grow a new site from). This is essentially a GitHub repository whose zip is fetched, unpacked, and then a first `composer install` run.

The main features are:

- a `composer.json` file
    
- a `README.md`
    
- an initial `.gitignore`
    

A new repository has been created (temporarily) at [GitHub - moodle/seed: The seed to grow a Moodle site from](https://github.com/andrewnicols/seed)

When we’re happy with this approach it will be transferred to the moodle organisation, and created in Packagist.

### Composer Installer plugin

This is currently supported using the `composer/installer` package but is not in our control and we have no control as to when any pull requests may (or may not) land. Therefore I feel that we need our own plugin which we can manage.

A new plugin has therefore been created to support this:

- GitHub Repository: [GitHub - moodle/composer-installer](https://github.com/moodle/composer-installer)
    
- Packagist entry: [moodle/composer-installer - Packagist.org](https://packagist.org/packages/moodle/composer-installer)
    

### Scaffolding

Because the `moodle` directory is removed and replaced at every composer `install` or `upgrade` operation we cannot put files like `config.php` into the Moodle root. Therefore we need to put it into the Composer package root.

The scaffold tooling creates a ‘shim' in the `moodle` directory so that Moodle continues to operate as normal.

Similar shims are also create for the`vendor/autoload.php` so that it can be correctly loaded from the correct location.

A new repository has been created (temporarily) at [GitHub - moodle/moodle-composer-scaffold](https://github.com/andrewnicols/moodle-composer-scaffold)

When we’re happy with this approach it will be transferred to the moodle organisation, and created in Packagist.

## Changes to Moodle

Most of the changes required for this project actually exist _outside_ of Moodle. The main Moodle-specific changes are:

- correcting metadata in Moodle’s `composer.json` file;
    
- adding a new `provide` item to the `composer.json` schema; and
    
- supporting a shimmed copy of the `vendor/autoload.php` directory.
    

## Plugin installation

For plugins to support their own installation they must create a `composer.json` file which:

- declares their correctly-formatted name
    
- sets a package `type` of `moodle-[plugintype]` - for example `moodle-block`
    
- declares a dependency on `moodle/composer-installer`
    

Ideally plugins should also:

- declare a depenency on `moodle/lms`, which is the Composer Virtual Package which is provied by `moodle/moodle` and would be provided by `moodle/workplace` in the future.
    

## Backwards compatibility

This change is intended to be **optional** in the first instance.

A current Moodle install will be able to continue as normal, but will not be able to install plugins using Composer.

Any third-party dependencies of Moodle (for example Guzzle, Slim, PSRs, AWS-SDK) should be installable in both the current system, _and_ a Composer-based installation (obviously not at the same time).

## Testing Instructions

Aucun

## Automated test results

Aucun

## Pre-check results

Aucun

## Workaround

Aucun

Tickets enfant

Progression : 100 %

|Tickets|
|---|
|![Amélioration](https://moodle.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10310?size=small)<br><br>[MDL-87950](https://moodle.atlassian.net/browse/MDL-87950)<br><br>[Provide third-party libraries via Composer](https://moodle.atlassian.net/browse/MDL-87950)|![](https://moodle.atlassian.net/images/icons/priorities/minor.svg)Mineure|3|![](https://secure.gravatar.com/avatar/ca561418de62ca92a215a3b3fb65e041?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FAL-4.png)<br><br>Andrew Lyons|Fermée|
|![Amélioration](https://moodle.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10310?size=small)<br><br>[MDL-87733](https://moodle.atlassian.net/browse/MDL-87733)<br><br>[Add GitHub Action test for composer-installed Moodle](https://moodle.atlassian.net/browse/MDL-87733)|![](https://moodle.atlassian.net/images/icons/priorities/minor.svg)Mineure|3|![](https://secure.gravatar.com/avatar/ca561418de62ca92a215a3b3fb65e041?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FAL-4.png)<br><br>Andrew Lyons|Fermée|
|![Amélioration](https://moodle.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10310?size=small)<br><br>[MDL-87716](https://moodle.atlassian.net/browse/MDL-87716)<br><br>[Update PHPUnit Configuration to support an optional directory prefix](https://moodle.atlassian.net/browse/MDL-87716)|![](https://moodle.atlassian.net/images/icons/priorities/minor.svg)Mineure|2|![](https://secure.gravatar.com/avatar/ca561418de62ca92a215a3b3fb65e041?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FAL-4.png)<br><br>Andrew Lyons|Fermée|
|![Amélioration](https://moodle.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10310?size=small)<br><br>[MDL-87713](https://moodle.atlassian.net/browse/MDL-87713)<br><br>[Switch from individually specified testing dependencies to `moodle/moodle-testing` composer dependency](https://moodle.atlassian.net/browse/MDL-87713)|![](https://moodle.atlassian.net/images/icons/priorities/minor.svg)Mineure|2|![](https://secure.gravatar.com/avatar/ca561418de62ca92a215a3b3fb65e041?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FAL-4.png)<br><br>Andrew Lyons|Fermée|
|![Amélioration](https://moodle.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10310?size=small)<br><br>[MDL-87461](https://moodle.atlassian.net/browse/MDL-87461)<br><br>[Change Moodle Composer project type](https://moodle.atlassian.net/browse/MDL-87461)|![](https://moodle.atlassian.net/images/icons/priorities/minor.svg)Mineure|2|![](https://secure.gravatar.com/avatar/ca561418de62ca92a215a3b3fb65e041?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FAL-4.png)<br><br>Andrew Lyons|Fermée|
|![Amélioration](https://moodle.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10310?size=small)<br><br>[MDL-87526](https://moodle.atlassian.net/browse/MDL-87526)<br><br>[Set composer.json provide field](https://moodle.atlassian.net/browse/MDL-87526)|![](https://moodle.atlassian.net/images/icons/priorities/minor.svg)Mineure|3|![](https://secure.gravatar.com/avatar/ca561418de62ca92a215a3b3fb65e041?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FAL-4.png)<br><br>Andrew Lyons|Fermée|
|![Amélioration](https://moodle.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10310?size=small)<br><br>[MDL-87717](https://moodle.atlassian.net/browse/MDL-87717)<br><br>[Update environment.xml tests to understand that composer may be installed in the parent directory](https://moodle.atlassian.net/browse/MDL-87717)|![](https://moodle.atlassian.net/images/icons/priorities/minor.svg)Mineure|3|![](https://avatar-management--avatars.us-west-2.prod.public.atl-paas.net/initials/MA-0.png)<br><br>Meirza Arson|Fermée|

Tickets associés

### blocks

![Type de ticket : Amélioration](https://moodle.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10310?size=medium)

[MDL-85847](https://moodle.atlassian.net/browse/MDL-85847)

Ouvert

![Priorité : Mineure](https://moodle.atlassian.net/images/icons/priorities/minor.svg)

![Type de ticket : Epic](https://moodle.atlassian.net/images/icons/issuetypes/epic.svg)

[MDL-87215](https://moodle.atlassian.net/browse/MDL-87215)

Development in progress

![](https://secure.gravatar.com/avatar/ca561418de62ca92a215a3b3fb65e041?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FAL-4.png)

![Priorité : Mineure](https://moodle.atlassian.net/images/icons/priorities/minor.svg)

### is child of

![Type de ticket : Tech Transformation](https://moodle.atlassian.net/rest/api/2/universal_avatar/view/type/issuetype/avatar/10300?size=medium)

[IDEA-330](https://moodle.atlassian.net/browse/IDEA-330)

In development

![Priorité : Medium](https://moodle.atlassian.net/images/icons/priorities/medium_new.svg)

## Activité

CommentairesPlus

![](https://secure.gravatar.com/avatar/7d0c9ec7c8645506fcf32f2ee2554796?d=https%3A%2F%2Favatar-management--avatars.us-west-2.prod.public.atl-paas.net%2Finitials%2FCB-4.png)