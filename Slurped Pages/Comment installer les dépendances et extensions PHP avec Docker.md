---
link: https://kinsta.com/fr/blog/installer-dependances-php-docker/
byline: Jeremy Holcombe
site: Kinsta®
date: 2024-04-16T16:43
excerpt: Simplifiez votre flux de développement en apprenant à installer les
  dépendances et les extensions PHP au sein des conteneurs Docker.
twitter: https://twitter.com/@kinsta_fr
slurped: 2026-10-21T08:22
title: Comment installer les dépendances et extensions PHP avec Docker
---

[Docker](https://kinsta.com/fr/blog/qu-est-ce-que-docker/) est une plateforme [open-source](https://kinsta.com/fr/blog/logiciels-open-source-vs-proprietaire/) populaire pour le développement, l’expédition et l’exécution d’applications. Elle encapsule les applications et leurs dépendances dans des conteneurs afin de favoriser la cohérence entre les différents environnements. Cette cohérence permet d’éviter les problèmes liés aux conflits de dépendances et aux divergences « ça marche sur ma machine ».

Avec Docker, vous pouvez définir les dépendances et les extensions nécessaires à l’aide de [Dockerfiles](https://kinsta.com/fr/blog/dockerfile-entrypoint/). Cela signifie que vos applications disposent de toutes les bibliothèques et de tous les outils nécessaires dans un environnement reproductible et isolé.

L’[hébergement d’application de Kinsta](https://sevalla.com/application-hosting/) offre un support dynamique pour les [Dockerfiles](https://docs.sevalla.com/applications/build-options/dockerfile) personnalisés, ce qui vous permet de mieux contrôler l’environnement de votre application. Avec un Dockerfile, vous pouvez utiliser presque n’importe quel langage – vous n’êtes pas limité aux langages pris en charge par les [Nixpacks](https://docs.sevalla.com/applications/build-options/nixpacks) ou [Buildpacks](https://docs.sevalla.com/applications/build-options/buildpacks).

Cet article pratique vous explique comment installer et utiliser les dépendances et extensions [PHP](https://kinsta.com/fr/blog/qu-est-ce-que-php/) dans Docker.

## Pourquoi utiliser Docker pour installer des dépendances ?

Utiliser Docker pour installer des dépendances a plusieurs avantages :

- **Cohérence de l’environnement** – Docker vous permet de regrouper votre application, ses dépendances et ses configurations dans un conteneur. Cette approche garantit que l’environnement dans lequel votre application s’exécute reste cohérent entre les différentes configurations et les différents systèmes, y compris les environnements de développement, de staging et de production.
- **Facilité de configuration et de résolution des dépendances** – Docker vous permet de configurer l’environnement de votre application sur n’importe quelle machine qui la prend en charge en spécifiant toutes les dépendances et configurations nécessaires. Vous évitez ainsi les problèmes de compatibilité et les difficultés liées à l’installation manuelle des dépendances.
- **Contrôle de version et isolation** – Les conteneurs Docker encapsulent toutes les dépendances, y compris leurs versions spécifiques, dans le conteneur. Cela signifie que vous pouvez contrôler la version de votre application et de ses dépendances, ce qui favorise la reproductibilité et élimine les conflits entre les différentes versions des dépendances.

En outre, les conteneurs Docker permettent l’isolation, ce qui aide à éliminer les conflits entre les dépendances de différentes applications fonctionnant sur le même système.
## Préparer votre environnement Docker

Un fichier Docker est un fichier texte contenant toutes les commandes, classées dans l’ordre nécessaire, dont vous avez besoin pour construire l’image nécessaire. Un fichier Docker de base pour une application PHP ressemblerait à ceci :

```
FROM php:8.3-cli
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
CMD [ "php", "./your-script.php" ]
```

Ce Dockerfile crée un environnement de conteneur pour exécuter un script PHP à partir d’une interface de ligne de commande (CLI). Il commence par utiliser l’image officielle de PHP avec la variante CLI, en s’assurant que seul le runtime PHP est inclus dans le conteneur.

Il copie ensuite le contenu du répertoire local – qui contient le fichier Docker – dans le répertoire **/usr/src/myapp** à l’intérieur du conteneur. Après avoir défini le répertoire de travail à **/usr/src/myapp**, il spécifie le script PHP à exécuter, **your-script.php**, en utilisant la commande `php`.

L’image contient de nombreuses extensions PHP pré-compilées. Commencez donc par vérifier si l’extension dont vous avez besoin est installée en exécutant la commande `php -m`.

Si l’extension dont vous avez besoin n’est pas disponible dans l’image de base, Docker fournit les scripts d’aide suivants pour vous aider à installer des extensions supplémentaires plus facilement :

- `docker-php-ext-configure`
- `docker-php-ext-install`
- `docker-php-ext-enable`

Par exemple, l’application de démonstration de ce tutoriel utilise l’extension [MySQLi](https://www.php.net/manual/en/book.mysqli.php) pour interagir avec la base de données MariaDB. Cette extension n’est pas compilée dans l’image par défaut, alors installez-la en utilisant la commande suivante dans votre Dockerfile :

```
RUN docker-php-ext-install mysqli
```

Pour écrire un Dockerfile pour l’application de démonstration, créez un nouveau fichier nommé **Dockerfile** dans le dossier racine du projet. Collez le code suivant dans le fichier :

```
FROM php:8.3-apache

# Install MySQL client, server, and other dependencies
RUN apt-get update && \
	apt-get install -y \
	default-mysql-client \
	default-mysql-server \
	git \
	&& apt-get clean \
	&& rm -rf /var/lib/apt/lists/*

# Install mysqli PHP extension for MySQL support
RUN docker-php-ext-install mysqli
```

Ce code spécifie l’[image PHP officielle](https://hub.docker.com/_/php) – en particulier la version 8.3 – avec Apache installé comme image de base pour le conteneur. Cette image fournit un environnement préconfiguré pour l’exécution d’applications PHP avec Apache comme serveur web.

Elle installe ensuite les dépendances supplémentaires dont l’application de démonstration a besoin, notamment `default-mysql-client` et `default-mysql-server`, qui créent et interagissent avec la [base de données MariaDB](https://docs.sevalla.com/databases/import-export/mariadb).

Enfin, l’extrait ci-dessus installe l’extension PHP `mysqli` à l’aide de `docker-php-ext-install`, qui permet à l’application de démonstration d’établir une connexion et d’effectuer des opérations sur la base de données MariaDB.

## Gérer les dépendances avec Docker

La gestion des dépendances PHP avec Docker implique l’utilisation de [Composer](https://getcomposer.org/), un gestionnaire de dépendances populaire pour PHP. En installant Composer dans un conteneur Docker, vous pouvez rationaliser le processus d’installation, de mise à jour et de gestion des dépendances PHP, tout en facilitant la cohérence et la reproductibilité dans différents environnements.

Suivez les étapes ci-dessous pour utiliser Composer pour la gestion des dépendances dans l’application de démonstration. Ajoutez le code de chacune des étapes à votre **fichier Docker**.

1. Installez Composer dans l’image Docker en utilisant la commande ci-dessous :
    
    ```
    # Install Composer
    COPY --from=composer/composer:latest-bin /composer /usr/bin/composer
    ```
    
    Cette commande copie un fichier binaire Composer pré-construit à partir du dépôt Docker Hub dans votre image. `--from=composer/composer:latest-bin` spécifie l’image source à partir de laquelle copier les fichiers.
    
    Le chemin `/composer` de l’image source contient les fichiers à copier, tandis que le chemin `/usr/bin/composer` de votre image Docker est la destination de ces fichiers.
    
2. Ensuite, définissez le répertoire de travail en exécutant ce qui suit :
    
    ```
    # Set working directory
    WORKDIR /var/www/html
    ```
    
3. Copiez les fichiers **composer.json** et **composer.lock** dans l’image.
    
    ```
    # Copy composer.json and composer.lock
    COPY composer.json composer.lock ./
    ```
    
4. Installez les dépendances de Composer listées dans **composer.json**.
    
    ```
    # Install Composer dependencies
    RUN composer install --no-dev --optimize-autoloader
    ```
    
    `--no-dev` demande à Composer d’ignorer les dépendances spécifiques au développement. `--optimize-autoloader` demande à Composer d’optimiser l’autoloader pour de meilleures performances.
    

Vous pouvez mettre à jour et gérer les dépendances au fil du temps en modifiant le fichier **composer.json** si nécessaire, en ajoutant de nouvelles dépendances, en mettant à jour les contraintes de version et en supprimant les dépendances qui ne sont plus nécessaires.

Après avoir effectué les modifications nécessaires, vous exécutez `composer install` ou `composer update` dans le conteneur Docker pour installer ou mettre à jour les dépendances en conséquence.

### Appliquer les configurations finales

Avant que l’application ne fonctionne avec Docker, vous devez créer la base de données et l’utilisateur de la base de données, définir les variables d’environnement et copier les fichiers du projet dans l’image Docker.

Pour cela, suivez les étapes ci-dessous et copiez chaque extrait de code dans votre fichier Docker.

1. Définissez les variables d’environnement que l’application PHP utilisera pour se connecter à la base de données MariaDB. Vous pouvez remplacer les variables `DB_USER`, `DB_PASSWORD`, et `DB_NAME` par vos identifiants préférés.
    
    ```
    # Set environment variables
    ENV DB_HOST=localhost \
    	DB_USER=kinsta_user \
    	DB_PASSWORD=your_password_here \
    	DB_NAME=kinsta_docker_auth \
    	# Path to the Unix socket file used for connecting to the MariaDB server.
    	DB_SOCKET=/var/run/mysqld/mysqld.sock
    ```
    
2. Copiez les fichiers de l’application PHP dans l’image Docker :
    
    ```
    # Copy PHP application files into the image
    COPY . .
    ```
    
3. Copiez le script de démarrage dans l’image :
    
    ```
    # Copy the startup script
    COPY scripts/start.sh /usr/local/bin/start.sh
    ```
    
    Ce script s’exécute au démarrage du conteneur et est responsable du démarrage du serveur de base de données, de la création d’une base de données, d’une table et d’un utilisateur, ainsi que du démarrage du serveur Apache. Vous créerez ce script plus tard.
    
4. Rendez le script exécutable à l’aide de la commande `[chmod](https://www.geeksforgeeks.org/chmod-command-linux/)`
    
    ```
    # Make the script executable
    RUN chmod +x /usr/local/bin/start.sh
    ```
    
5. Enfin, exécutez le script à l’aide de la commande ci-dessous :
    
    ```
    # Execute the startup script
    CMD ["/usr/local/bin/start.sh"]
    ```
    

Maintenant que le fichier Docker est complet, vous créez le script de démarrage dont il a été question précédemment.

Créez un nouveau fichier nommé **scripts/start.sh** dans le dossier racine du projet et ajoutez le code suivant. Veillez à utiliser les informations d’identification de la base de données que vous avez définies dans le fichier Docker.

```
#!/bin/bash

# Start MariaDB server
service mariadb start

# Wait for MariaDB to be fully initialized (retry up to 30 times, waiting 1  second between each attempt)
attempt=0
while [ $attempt -lt 30 ]; do
		if mysqladmin ping &>/dev/null; then
    		echo "MariaDB is up and running."
    		break
		else
    		echo "MariaDB is not yet available. Retrying..."
    		attempt=$((attempt+1))
    		sleep 1
		Fi
done

# If MariaDB failed to start within the specified attempts, exit with an error
if [ $attempt -eq 30 ]; then
		echo "Error: MariaDB failed to start within the specified time."
		exit 1
fi

# Set the database name, username, and password
dbname="kinsta_docker_auth"
dbuser="kinsta_user"
dbpassword="your_password_here"

# Create the database if it does not exist
mysql -u root -e "CREATE DATABASE IF NOT EXISTS $dbname;"

# Create a database user and assign privileges to the database
mysql -u root -e "CREATE USER '$dbuser'@'localhost' IDENTIFIED BY '$dbpassword';"
mysql -u root -e "GRANT ALL PRIVILEGES ON $dbname.* TO '$dbuser'@'localhost';"
mysql -u root -e "FLUSH PRIVILEGES;"

# Create users table if it does not exist
mysql -u root -e "USE $dbname; CREATE TABLE IF NOT EXISTS users (
		id INT AUTO_INCREMENT PRIMARY KEY,
		username VARCHAR(20) UNIQUE NOT NULL,
		password_hash VARCHAR(255) NOT NULL
);"

# Start Apache server
apache2ctl -D FOREGROUND
```

Ce [script Bash](https://kinsta.com/fr/blog/langages-de-script/#8-bash) démarre le serveur MariaDB. Ensuite, lorsque le serveur est entièrement initialisé, il crée une base de données, un utilisateur avec les privilèges appropriés et une table dans la base de données. Enfin, il démarre le serveur Apache.

## Déployer sur Kinsta

Avant de déployer votre application sur Kinsta, vous devez configurer le serveur Apache. Le service d’[hébergement d’applications](https://sevalla.com/application-hosting/) de Kinsta a les ports `80` et `443` ouverts, mais nous recommandons d’utiliser le port `8080` si votre application expose des ports.

Par défaut, le serveur Apache écoute les connexions sur le port `80`, vous devez donc le configurer pour qu’il écoute les connexions sur le port `8080`. Pour cela, procédez comme suit :

1. Créez un dossier nommé **apache-conf** dans le dossier racine du projet.
2. Dans le dossier nouvellement créé, créez un fichier nommé **apache-config.conf.** Ajoutez le code ci-dessous pour configurer Apache afin qu’il écoute les connexions sur le port 8080 :
    
    ```
    <VirtualHost *:8080>
    	ServerAdmin webmaster@localhost
    	DocumentRoot /var/www/html
    
    	ErrorLog ${APACHE_LOG_DIR}/error.log
    	CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
    
3. Dans le dossier **apache-conf**, créez un nouveau fichier nommé **apache-ports.conf** et ajoutez le code ci-dessous.
    
    ```
    # Change the Listen directive to listen on port 8080
    Listen 8080
    
    # Keep the existing configuration for port 443
    <IfModule ssl_module>
    	Listen 443
    </IfModule>
    
    <IfModule mod_gnutls.c>
    	Listen 443
    </IfModule>
    ```
    
    La directive `Listen 8080` indique à Apache d’écouter les connexions entrantes sur le port 8080.
    
4. Vous devez maintenant modifier le fichier Dockerfile pour appliquer cette configuration au serveur Apache. Dans le fichier Docker, ajoutez ces commandes juste après la commande de définition du répertoire de travail (`WORKDIR /var/www/html`). Elles sont chargées de copier les fichiers de configuration personnalisés dans les répertoires appropriés.
    
    ```
    # Set up Apache virtual host
    COPY apache-conf/apache-config.conf /etc/apache2/sites-available/000-default.conf
    
    # Set up Apache ports
    COPY apache-conf/apache-ports.conf /etc/apache2/ports.conf
    ```
    
5. Enfin, créez un fichier **.dockerignore** dans le dossier racine du projet. Ce fichier contient une liste de dossiers et de fichiers qui ne doivent pas être copiés dans l’image Docker. Ajoutez `vendor` pour exclure le dossier **vendor**.

Une fois toutes les configurations effectuées, vous pouvez déployer votre projet sur Kinsta. Téléversez les fichiers de votre projet sur votre fournisseur Git préféré ([Bitbucket](https://docs.sevalla.com/applications/git/bitbucket#grant-access-to-the-kinsta-bitbucket-application), [GitHub](https://docs.sevalla.com/applications/git/github#authenticate-and-authorize) ou [GitLab](https://docs.sevalla.com/applications/git/gitlab#authorize-the-kinsta-gitlab-application)), puis suivez les étapes ci-dessous.

1. Dans votre [tableau de bord MyKinsta](https://my.kinsta.com/?lang=fr), sélectionnez **Applications** dans la colonne latérale et cliquez sur le bouton **Ajouter une application**.
2. Sur la page **Ajouter une application**, sélectionnez votre dépôt Git et votre branche, donnez un nom à l’application, choisissez l’emplacement du centre de données et cliquez sur **Continuer**.
3. Sur la page **Environnement de construction** , sélectionnez **Utiliser Dockerfile pour configurer l’image du conteneur**. Indiquez « Dockerfile » comme **chemin d’accès au fichier Docker** et un point (« . ») comme **contexte**. Cliquez sur **Continuer**.
    
    ![Section Environnement de construction du formulaire Ajouter une application.](https://kinsta.com/wp-content/uploads/2024/04/build-environment.png)
    
    Section Environnement de construction du formulaire Ajouter une application.
    
4. **Remarque** : La base de données perdra des données lorsque vous redémarrez ou redéployez l’application. Si vous avez besoin de résoudre ce problème, consultez le [stockage persistant](https://docs.sevalla.com/applications/storage) de Kinsta et appliquez-le sur cette page. Ce guide ne passe pas en revue la configuration du stockage persistant.
5. Confirmez les détails du paiement sur la page **Résumé** et cliquez sur **Déployer maintenant**.
6. Une fois le déploiement terminé, ouvrez le domaine de votre application pour voir le site en direct.
    
    ![Page de connexion sur le site web en ligne.](https://kinsta.com/wp-content/uploads/2024/04/live-site.png)
    
    Page de connexion sur le site web en ligne.
    
    Créez un compte, connectez-vous à l’aide de vos données et vérifiez que vous pouvez accéder à la page protégée et que tout fonctionne comme prévu.
    
    ![La page d'accueil de l'application une fois connecté.](https://kinsta.com/wp-content/uploads/2024/04/login-homepage.png)
    
    La page d’accueil de l’application une fois connecté.
    

## Surveiller et optimiser votre application

Une fois que votre application est opérationnelle sur Kinsta, il est essentiel de surveiller et d’optimiser ses performances. Cela permet à votre application de fonctionner sans heurts et d’utiliser les ressources de manière efficace.

L’optimisation de la performance de votre application est cruciale pour offrir une expérience utilisateur rapide et réactive.

Kinsta travaille en permanence à l’[optimisation des performances de votre application](https://kinsta.com/fr/blog/performance-wordpress/). Nous offrons des outils de surveillance complets pour vous aider à suivre diverses mesures de performance, y compris la bande passante, le temps total de construction, l’utilisation du CPU, la consommation de mémoire, le trafic réseau et les temps de réponse. Vous pouvez consulter toutes ces informations en vous rendant dans la section **Analyses** de votre application sur le tableau de bord de Kinsta.

Si vous rencontrez un problème ou un comportement inattendu de l’application, vous pouvez vous rendre dans la section **Journaux** du tableau de bord de Kinsta. Vous y trouverez un aperçu détaillé des performances et du comportement de votre application, y compris les journaux d’erreurs et d’accès.

En examinant méticuleusement ces journaux, vous pouvez obtenir des informations précieuses sur les causes possibles du problème, telles que les messages d’erreur, les traces de pile ou les modèles de comportement anormal. Ces informations peuvent vous aider à diagnostiquer la cause première du problème et à prendre les mesures appropriées pour le résoudre.

En outre, vous pouvez exploiter des techniques d’optimisation des performances dans votre environnement Docker, telles que l’optimisation des images Docker et la mise en œuvre des meilleures pratiques pour l’architecture des applications et l’optimisation du code.

## Résumé

Les conteneurs légers et portables de Docker facilitent l’empaquetage et le déploiement d’applications avec toutes les dépendances nécessaires, quelle que soit l’infrastructure sous-jacente. En conteneurisant votre application et ses dépendances, vous favorisez la cohérence entre les différents environnements, rationalisez les processus d’installation et bénéficiez d’un meilleur contrôle des versions.

L’utilisation de Docker avec Kinsta améliore le processus de déploiement de votre application. Sa prise en charge des fichiers Docker personnalisés et d’un large éventail de langages de programmation, y compris PHP, garantit la compatibilité et la flexibilité de vos projets.

Grâce à l’[infrastructure d’hébergement d’application](https://docs.sevalla.com/applications/get-started/infrastructure) et à la technologie de conteneurisation de Docker, vous pouvez obtenir un déploiement rationalisé, une évolutivité et une fiabilité pour vos applications.

_Avez-vous déjà utilisé Docker pour votre projet ? Avez-vous une expérience ou des idées sur Docker ? Partagez-les dans les commentaires ci-dessous !_

Jeremy Holcombe [Kinsta](https://kinsta.com/fr/)

Rédacteur en chef du contenu et du marketing chez Kinsta, développeur web WordPress et rédacteur de contenu. En dehors de WordPress, j'aime la plage, le golf et le cinéma. J'ai aussi des problèmes avec les personnes de grande taille ;).

- [LinkedIn](https://www.linkedin.com/in/jeremyholcombe/)