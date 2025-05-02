---
link: https://www.loxodata.com/post/connexions1/
excerpt: Loxodata vous présente les différentes méthodes de connexion à une instance PostgreSQL
slurped: 2025-02-22T09:10
title: Connexion à une instance PostgreSQL
tags:
  - postgresql
  - connexion
---

## Connexion à une instance PostgreSQL

La connexion à une instance PostgreSQL utilise le protocole client-serveur implanté par la libpq, fournie avec PostgreSQL.

Cette bibliothèque offre plusieurs méthodes pour indiquer les coordonnées de l’instance PostgreSQL.

Cet article propose de revoir les différentes possibilités. C’est peut-être l’opportunité de mettre en place des configurations plus souples et stables que celles couramment utilisées.

## Options de connexion

Pour se connecter à une instance PostgreSQL, il faut fournir plusieurs informations servant à trouver l’instance, puis à s’y authentifier :

- le nom d’hôte ou l’adresse IP du système où fonctionne l’instance ;
- le port TCP sur lequel écoute l’instance, par défaut 5432 ;
- le nom de la base de données de l’instance ;
- le nom du rôle ;
- et, très souvent, un mot de passe.

Ces différentes informations sont utilisées pour contacter une instance PostgreSQL, et tenter d’y ouvrir une connexion. La plupart des outils « client » utilisent les mêmes options :

- `-h` : Nom d’hôte ou adresse IP, voire chemin vers la « socket Unix » ;
- `-p` : Numéro de port TCP, par défaut 5432 ;
- `-U` : Nom du rôle, par défaut le même nom que l’utilisateur système exécutant le client ;
- `-d` : Nom de la base de données, dans l’instance, par défaut le même nom que le rôle.

Ces options ne sont pas obligatoires, ayant toutes une valeur par défaut.

Selon la configuration de l’instance PostgreSQL, et la méthode d’appel de l’application cliente, il est possible de se connecter à PostgreSQL sans préciser aucune valeur.

Par exemple, dans un système Debian ou Redhat, en utilisant le même utilisateur système que celui exécutant l’instance (par défaut `postgres`), il suffit de lancer la commande psql pour ouvrir une session interactive dans l’instance :

```
postgres@pg10:~$ psql
psql (10beta3)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" via socket in "/var/run/postgresql" at port "5432".
```

La commande `\conninfo` de `psql` permet de voir quels sont les paramètres réels de la connexion.

Il est également possible de préciser les options de connexions dans la ligne de commande :

```
postgres@pg10:~$ psql -h pg10 -Upostgres
psql (10beta3)
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg10" at port "5432".
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
```

On remarque ici que la connexion utilise TLS, c’est-à-dire que les données sont chiffrées entre le client et le serveur ; ceci est négocié au moment de l’ouverture de la connexion.

### Mots de passe

Lorsque la connexion exige un mot de passe, en suivant la méthode présentée, il doit être saisi interactivement ; il n’existe pas d’option permettant de passer le mot de passe en paramètre de la commande :

```
postgres@pg10:~$ psql -W -h pg10 -Upostgres
Password for user postgres:
psql (10beta3)
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg10" at port "5432".
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
```

Dans l’exemple, l’option `-W` permet de forcer l’invite du mot de passe.

## Variables d’environnement

Il est possible de se connecter de la même façon en utilisant des variables d’environnement, qui sont définies dans l’interpréteur de commande de l’utilisateur. Les variables d’environnements disponibles sont :

- `PGHOST` pour le nom d’hôte ou `PGHOSTADDR` pour une adresse IP ;
- `PGPORT` pour le numéro de port TCP ;
- `PGDATABASE` pour le nom de la base de données ;
- `PGUSER` pour le nom du rôle ;
- `PGPASSWORD` pour le mot de passe ;
- `PGAPPNAME` pour le nom de l’application, correspondant à `application_name` côté serveur.

L’exemple suivant permet de se connecter en utilisant ces variables :

```
postgres@pg10:~$ export PGHOST=pg10
postgres@pg10:~$ export PGPASSWORD=mybadpasswd
postgres@pg10:~$ psql
psql (10beta3)
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg10" at port "5432".
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
```

Idéalement, ces variables seront enregistrées dans les fichiers d’environnement de l’interpréteur de commandes.

Attention, le stockage du mot de passe en clair dans un tel fichier est loin d’être idéal.

## Chaînes de connexion

Autre technique pour se connecter, l’utilisation de chaînes de connexion, technique largement connue des développeurs.

Cette méthode fonctionne aussi pour les outils en ligne de commande.

Il existe deux formats de chaînes. La première sous la forme d’une URI, et la seconde sous la forme d’un ensemble de paires clé-valeur.

### URI de connexion

L’URI de connexion est régulièrement utilisée dans des langages tels que Java, mais peut tout à fait être utilisée dans d’autres cadres. La forme de l’URI est la suivante :

```
postgres://user:password@host:port/dbname?params
```

Par exemple :

```
postgres@pg10:~$ psql postgres://postgres:mybadpasswd@pg10/postgres
psql (10beta3)
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg10" at port "5432".
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
```

### Clé/Valeur

L’autre format de chaîne de connexion utilise des clés, correspondant aux paramètres déjà décrits :

- `host` : Nom d’hôte, ou `hostaddr` pour une adresse IP ;
- `port` : Numéro de port ;
- `user` : Nom de rôle ;
- `dbname` : Nom de la base de données ;
- `password` : Mot de passe ;
- `application_name` : Nom de l’application.

Ces paramètres se combinent avec leurs valeurs en une chaîne, comme dans l’exemple suivant :

```
postgres@pg10:~$ psql "host=pg10 user=postgres password=mybadpasswd"
psql (10beta3)
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg10" at port "5432".
SSL connexion (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
```

Cette chaîne, aisément forgée, peut être stockée dans une variable pour être utilisée.

## Fichier de service

En utilisant les mêmes clés que dans l’exemple précédent, il est possible de définir un fichier de service contenant plusieurs connexions distinctes, identifiées par un mot-clé.

Ce mot-clé identifie le service utilisé par le client pour ouvrir la connexion.

Le choix de ce mot-clé est laissé à l’utilisateur.

Le fichier peut être spécifique à un utilisateur, ou bien global à un système.

Le fichier « utilisateur » est `~/.pg_service.conf`, donc un fichier caché dans le répertoire utilisateur ; ce fichier peut être surchargé par la variable d’environnement `PGSERVICEFILE`.

Le fichier global dépend de la façon dont à été compilé PostgreSQL. Dans les systèmes Debian, le fichier est `/etc/postgresql-common/pg_service.conf` et dans les systèmes Redhat `/etc/sysconfig/pgsql/pg_service.conf`.

Un service est défini comme dans l’exemple suivant :

```
[cnxpg10]
host=pg10
user=postgres
dbname=postgres
password=mybadpasswd
```

Ensuite, il est possible d’utiliser ce service, soit avec la clé `service`, soit avec la variable d’environnement `PGSERVICE` :

```
postgres@pg10:~$ psql "service=cnxpg10"
psql (10beta3)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg10" at port "5432".

postgres@pg10:~$ export PGSERVICE=cnxpg10
postgres@pg10:~$ psql
psql (10beta3)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "pg10" at port "5432".
```

Cette technique permet de gérer les connexions depuis un fichier dédié ; ce qui est idéal lorsque les systèmes sont administrés depuis des outils de gestion de configuration tel qu’Ansible, ou équivalent.

## Fichier de mots de passe

Il existe un fichier dédié aux mots de passe, afin d’éviter de stocker les mots de passe dans le fichier de service ou dans une variable d’environnement.

Ce fichier n’est protégé que par la vérification des droits Unix : si les droits sont trop larges, alors le fichier n’est pas utilisé. Dans ce cas, une erreur est levée par PostgreSQL.

Le format du fichier est simple : une ligne par connexion, en suivant le motif : _hostname:port:database:username:password_

Par exemple :

```
echo "pg10:5432:postgres:postgres:mybadpasswd" >> ~/.pgpass
```

Le mot de passe ayant été retiré du fichier de service, on utilise l’option `-w` pour ne pas demander de mot passe interactivement :

```
psql -w "service=cnxpg10"
WARNING: password file "/var/lib/postgresql/.pgpass" has group or world access; permissions should be u=rw (0600) or less
psql: fe_sendauth: no password supplied
```

On obtient une erreur à cause des droits mal positionnés sur le fichier `.pgpass`. Après correction :

```
postgres@pg10:~$ chmod 0600 .pgpass
postgres@pg10:~$ psql -w "service=cnxpg10"
psql (10beta3)

postgres=#
```

La connexion fonctionne !

## Conclusion

La gestion des connexions peut s’avérer difficile. Notamment lors de l’utilisation de solutions de haute disponibilité, où les noms des machines peuvent changer. Les différentes possibilités présentées ici permettent de simplifier cette gestion.

Dans de prochains articles, nous aborderons des notions plus avancées de la connexion aux instances PostgreSQL.

Nous aborderons notamment l’utilisation de certificats TLS pour l’authentification, le nouveau protocole d’échange de mot de passe, ou encore la possibilité d’indiquer plusieurs noms d’hôte pour une connexion.