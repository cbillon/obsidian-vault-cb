---
link: https://nahan.fr/mes-extensions-pour-behat
byline: Jean-Baptiste Nahan
site: JB Dev Labs | Artisan développeur informatique
date: Invalid date
excerpt: Comment intégrer Lambdatest ou XRay dans votre projet de test avec Behat ? La solution est l‘utilisation des extensions spéciales pour ces services.
twitter: https://twitter.com/@jbnahan69
slurped: 2025-03-11T13:36
title: Mes extensions pour Behat | JB Dev Labs
tags:
  - behat
  - extension
---

1. [JB Dev Labs](app://obsidian.md/)
2. [Mes extensions pour Behat](https://nahan.fr/mes-extensions-pour-behat)

Depuis quelque temps, je parle des [tests d'acceptation](https://nahan.fr/tutoriel-test-behat-sur-un-projet-symfony) réalisés avec Behat, des [cas concrets d'un test](https://nahan.fr/behat-et-symfonypage), ou encore [comment piloter Chrome avec Behat](https://nahan.fr/tutoriel-comment-piloter-un-chrome-avec-behat).

L'intéret de Behat par rapport à PHPUnit ou Panther est l'utilisation de la [syntaxe Gerkin](https://cucumber.io/docs/gherkin/) pour décrire les tests.

Cette syntaxe permet d'écrire vos tests en français ou dans une autre langue supportée. Ainsi plus proche d'un langage courant, les tests sont plus facilement compréhensible par les équipes en charge la qualité du produit final.

Cependant, qui n'a jamais été confronté aux difficultés d'accès au résultat des tests pour l'équipe qualité ? Il est souvent nécessaire que le service qualité relise le texte des tests et qu'il vérifie la bonne exécution lors de la dernière session de test.

Qui n'a jamais pesté contre tel ou tel navigateur qui ne fonctionne plus après une mise à jour ?

Je vais vous présenter deux extensions Behat que j'ai écris et qui peuvent vous aider.

## XRay

C'est le but du service [XRay](https://www.getxray.app/) pour Jira. Ce plugin permet de sauvegarder tous les scénarii Gerkin dans des issues Jira relié a l'épic de la fonctionnalité.

Dans ce contexte, il est nécessaire que pour chaque exécution des test Behat, les scénarii soit mis à jour dans XRay et que le résultat de l'exécution soit également envoyé au service.

C'est pour cela que j'ai développé l’extension XRay pour Behat.

## Lambdatest

Pour l'exécution des tests nécessitant l'exécution du code Javascript, il est assez complexe de gérer tous les navigateurs et système d'exploitation nécessaire pour qualifier les nouvelles version.

C'est pour cela que le service [Lambdatest](https://www.lambdatest.com/) met à disposition des serveurs pilotable par Behat via Selenium.

Cependant, ce service a des spécificités qu'il est important de prendre en compte pour le bon déroulement des tests.

Par exemple:

- L'authentification.
- L'utilisation du tunnel.
- La vérification de la possibilité d'exécuter un test automatisé.
- Définir l'état du test à la fin (réussite ou échec).
- L'envoi de fichier pour les tests d'upload sur le site.

Ces intégrations sont disponibles dans une extension Behat.