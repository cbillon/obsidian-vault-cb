---
link: https://nahan.fr/tutoriel-comment-piloter-un-chrome-avec-behat
site: JB Dev Labs | Artisan développeur informatique
date: Invalid date
excerpt: Il est parfois nécessaire d‘exécuter le Javascript présent dans vos pages HTML pendant les tests d‘acceptation. Voici comment piloter Chromium.
twitter: https://twitter.com/@jbnahan69
slurped: 2025-03-11T13:30
title: Comment piloter Chrome avec Behat ?  | JB Dev Labs
tags:
  - behat
  - chrome
---

## Table des matières

1. [Choix du navigateur](https://nahan.fr/tutoriel-comment-piloter-un-chrome-avec-behat#Choix-du-navigateur)
2. [Howto step : Lancement du navigateur](https://nahan.fr/tutoriel-comment-piloter-un-chrome-avec-behat#Howto-step-Lancement-du-navigateur)
3. [Howto step : Configurer le projet](https://nahan.fr/tutoriel-comment-piloter-un-chrome-avec-behat#Howto-step-Configurer-le-projet)
4. [Howto step : Configuration des scénarii](https://nahan.fr/tutoriel-comment-piloter-un-chrome-avec-behat#Howto-step-Configuration-des-scenarii)
5. [Howto step : Exécuter les tests](https://nahan.fr/tutoriel-comment-piloter-un-chrome-avec-behat#Howto-step-Executer-les-tests)
6. [Voici le résultat en vidéo](https://nahan.fr/tutoriel-comment-piloter-un-chrome-avec-behat#Voici-le-resultat-en-video)

Suite au succès de mon [précédent tutoriel sur Behat](https://nahan.fr/tutoriel-test-behat-sur-un-projet-symfony "[Tutoriel] Test Behat sur un projet Symfony"), ce tutoriel est le premier d'une série consacrée au pilotage des navigateurs.

J'utilise le projet [todo list](https://github.com/macintoshplus/todo-list-example/releases/tag/behat-symfony-base "https://github.com/macintoshplus/todo-list-example/releases/tag/behat-symfony-base") disponible sur GitHub pour ce tutoriel.

## Choix du navigateur

Avant de commencer, il est nécessaire de choisir le navigateur à utiliser pour l'exécution des tests. Pour ce premier tutoriel, nous allons utiliser Chromium sous Linux.

Il est possible de piloter un grand nombre de navigateurs pour ordinateur ou smartphone. Ce sera l'objet de prochains tutoriels.

Durée estimée : 1 heure

## Lancement du navigateur

Les navigateurs basés sur Chromium peuvent être pilotés directement par Behat grâce à l'extension Mink et le pilote Chrome "dmore/chrome-mink-driver".

Avant de lancer Behat, il est nécessaire de lancer le navigateur avec les bonnes options pour que Behat puisse l'utiliser.

Chromium via snap

Commandes pour installer Snapd et Chrmium : `apt-get install snapd   snap install core chromium`  
Le binaire à utiliser dans la commande ci dessous est : `/snap/bin/chromium`

Exécuter cette commande dans un terminal pour utiliser Chromium ou Chrome installé sur le système. Il faudra le laisser ouvert pendant toute la durée des tests.

Commande pour Linux

```
chromium --enable-automation --disable-background-networking --no-default-browser-check --no-first-run --disable-popup-blocking --disable-default-apps --disable-translate --disable-extensions --no-sandbox --enable-features=Metal --remote-debugging-port=9222 https://127.0.0.1 
```

Commande pour macOs

```
/Applications/Chromium.app/Contents/MacOS/Chromium --enable-automation --disable-background-networking --no-default-browser-check --no-first-run --disable-popup-blocking --disable-default-apps --disable-translate --disable-extensions --no-sandbox --enable-features=Metal --remote-debugging-port=9222 https://127.0.0.1
```

Commande pour Windows

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" --enable-automation --disable-background-networking --no-default-browser-check --no-first-run --disable-popup-blocking --disable-default-apps --disable-translate --disable-extensions --no-sandbox --enable-features=Metal --remote-debugging-port=9222 --window-size=1280,1024 https://127.0.0.1
```

Pour mettre fin à l'exécution du navigateur, appuyer sur `Ctrl + C`.

Pour exécuter Chromium sans la fenêtre, ajouter l'option `--headless`.

## Configurer le projet

Ajouter l'extension "dmore/behat-chrome-extension" à votre projet et activer l'extension dans la configuration de Behat.

Ce tutoriel part du principe que vous avez déjà installé Behat tel que présenté dans le [précédent tutoriel](https://nahan.fr/tutoriel-test-behat-sur-un-projet-symfony "[Tutoriel] Test Behat sur un projet Symfony"). Si ce n'est pas le cas, réaliser l'installation avant de continuer.

Ajouter l'extension Behat et le driver Mink au projet:

```
composer require --dev dmore/behat-chrome-extension
```

Dans le fichier `behat.yml.dist`, ajouter la configuration suivante:

```
default:
    extensions:
        DMoreChromeExtensionBehatServiceContainerChromeExtension: ~
        BehatMinkExtension:
            base_url: "https://127.0.0.1:8000/"
            javascript_session: chrome_headless
            sessions:
                chrome_headless:
                    chrome:
                        api_url: "http://127.0.0.1:9222"
                        validate_certificate: false
```

## Configuration des scénarii

Pour exécuter des scénarii avec un navigateur piloté, il est nécessaire d'ajouter le tag @javascript au début du fichier de fonctionnalité ou d'un scénario en particulier.

Pour exécuter des scénarii avec un navigateur piloté, il est nécessaire d'ajouter le tag @javascript au début du fichier de fonctionnalité ou d'un scénario en particulier.

```
#language: fr
@javascript
Fonctionnalité: Connexion à l'application

  Contexte:
    Etant donné que l'utilisateur "todo@me.fr" est enregistré avec le mot de passe "MonSuperPassWord"

  Scénario: Connexion
    Etant donné que je suis sur la page de connexion
    Lorsque je me connecte en tant que "todo@me.fr" avec le mot de passe "MonSuperPassWord"
    Alors je dois être sur la page de double authentification
    Lorsque je saisi le code de double authentification
    Alors je dois être sur la liste des todo listes
```

## Exécuter les tests

Le navigateur est déjà lancé, il est nécessaire la lancer le serveur web.

Pour cela j'utilisais Rymfony ([vidéo de présentation](https://www.youtube.com/watch?v=y0BEkzZZCmM "https://www.youtube.com/watch?v=y0BEkzZZCmM")). Pour le lancer, exécuter `APP_ENV=test rymfony serve -d` depuis le dossier racine du projet.

Le projet Rymfony est inactif depuis 3 ans, il est possible d'utiliser le client Symfony officiel avec la commande : `APP_ENV=test symfony serve -d`

Le projet utilise une base de données SQLite. Si votre projet a besoin d'un serveur MariaDb ou Postgres, lancez-le pour que votre projet ait accès aux données.

## Voici le résultat en vidéo

Pour vous montrer le résultat obtenu, voici la vidéo de la réalisation de ce tutoriel sur le projet Todo list

[![](app://obsidian.md/media/cache/resolve/article_640_jpeg/media/7f/51/704ee011f7945bd54aef38533b45.jpg)](https://www.youtube.com/watch?v=VKZXb5yFYMA)

Voir la vidéo : [https://www.youtube.com/watch?v=VKZXb5yFYMA](https://www.youtube.com/watch?v=VKZXb5yFYMA)

![Author avatar](https://nahan.fr/media/cache/resolve/article_640_webp/images/63/17/bdaabfab04.56494539.png)