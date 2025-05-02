---
link: https://zonetuto.fr/astuce/connaitre-la-taille-dune-base-de-donnees-mysql-mariadb/
byline: MrTuto
site: ZoneTuto
date: 2024-08-29T10:37
excerpt: Comment récupérer la taille et mesurer le poids de votre base de données MySQL ou MariaDB avec 2 techniques facile
slurped: 2025-02-22T09:38
title: Connaitre la taille d'une base de données MySQL / MariaDB - ZoneTuto
tags:
  - mariadb
  - mysql
  - size
---

C’est une question que l’on est en droit de se poser lorsque l’on a des bases de données qui commencent à avoir beaucoup de data. En ce moment, je suis sur un projet qui grossit très vite et j’ai des millions de lignes qui s’ajoutent chaque jour dans certaines tables. J’aimerai bien monitorer le poids réel de ma base de données sur le SSD pour savoir où je vais et me rendre compte à quel point sa taille augmente.

Pour obtenir le poids de sa base de données MySQL ou de son fork MariaDB il y a deux techniques bien connues pour récupérer la taille d’une base de données facilement lisible pour un humain.

## Mesurer le poids des fichier mysql

La première technique est finalement assez simple, mais vous devez avoir des [droits](https://zonetuto.fr/linux/ajouter-les-bonnes-permissions-sur-les-fichiers-et-dossiers-de-wordpress/) d’administrateur sur la machine pour que ce soit rapide à faire. Pour informations, les fichiers appartiennent à mysql et au groupe mysql. Après être passé en super admin, je vais donc faire la commande suivante dans mon terminal :

```
du -sh /var/lib/mysql/*
```

Avec cette simple commande, je vais donc obtenir une fois le poids de l’ensemble des bases de données sur ce [serveur Ubuntu](https://zonetuto.fr/linux/faire-migration-ubuntu-lts-vers-version-superieure/). Dans mon cas, elle me donne le résultat suivant :

```
412K	/var/lib/mysql/aria_log.00000001
4.0K	/var/lib/mysql/aria_log_control
16K	/var/lib/mysql/ddl_recovery.log
0	/var/lib/mysql/debian-10.11.flag
20K	/var/lib/mysql/ib_buffer_pool
13M	/var/lib/mysql/ibdata1
96M	/var/lib/mysql/ib_logfile0
12M	/var/lib/mysql/ibtmp1
0	/var/lib/mysql/multi-master.info
1.2G	/var/lib/mysql/mabddtuto
3.8M	/var/lib/mysql/mysql
4.0K	/var/lib/mysql/mysql_upgrade_info
8.0K	/var/lib/mysql/performance_schema
600K	/var/lib/mysql/sys
```

Ici je vois que m’a base de donnée mabddtuto pèse 1,2 Go sur l’espace de stockage de mon serveur. Simple et efficace !

## Estimer la taille de la base de données avec une requête SQL

Si vous n’avez pas assez de droit pour faire ce que vous voulez dans le [système de fichier de votre serveur Linux](https://zonetuto.fr/linux/comprendre-arborescence-des-dossiers-du-systeme-gnu-unix/), mais que vous avez accès à la base de données il y a une autre technique possible. Il est possible de passer par une requête SQL pour obtenir la taille de votre base de données. Pour récupérer la taille de ma base de données, je vais donc utiliser la requête SQL suivante :

```
SELECT table_schema AS "Database", ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS "Size (Mo)" FROM information_schema.tables GROUP BY table_schema;
```

Elle me donne alors les informations suivantes sur le poids de ma base de données :

```
+--------------------+-----------+
| Database           | Size (Mo) |
+--------------------+-----------+
| information_schema    |      0.20  |
| mabddtuto                   |   989.67 |
| mysql                           |      3.38  |
| sys                               |      0.03  |
+--------------------+-----------+
5 rows in set (0.129 sec)
```

## Pourquoi il y a une différence entre les 2 techniques pour mesurer la taille d’une base de données ?

Les différences observées entre la commande SQL qui calcule la taille des bases de données et la commande `du -sh` sous Linux sont dues à la manière dont chacune mesure et rapporte l’espace disque utilisé. Ces différences sont courantes et peuvent prêter à confusion, mais elles s’expliquent par la nature des données mesurées et par les éléments inclus dans chaque calcul.

Il est important de comprendre que la commande SQL que l’on a utilisé (`SELECT table_schema AS "Database", ROUND(SUM(data_length + index_length) / 1024 / 1024, 2) AS "Size (Mo)" FROM information_schema.tables GROUP BY table_schema;`) se concentre uniquement sur la taille des tables de la base de données en termes de données brutes et d’index. Cette commande extrait les informations de la table `information_schema.tables`, qui contient la taille des données et des index pour chaque table de chaque base de données MySQL. Cependant, cette commande ne prend en compte que les données effectivement stockées dans les tables, sans inclure d’autres types de fichiers ou d’éléments de stockage qui peuvent exister dans le répertoire de la base de données.

La commande `du -sh /var/lib/mysql/*` sous Linux mesure l’espace disque total utilisé par tous les fichiers présents dans le répertoire `/var/lib/mysql/`, qui est généralement le répertoire par défaut où MySQL stocke les bases de données. Cette commande inclut non seulement les fichiers de données et d’index des tables, mais également tous les autres fichiers que MySQL utilise, tels que les fichiers de log (`ib_logfile0`, `ib_logfile1`), les fichiers de tablespace partagés (`ibdata1` pour InnoDB), les fichiers temporaires (`ibtmp1`), et potentiellement d’autres fichiers de configuration ou de métadonnées.

Il y une différence notable liée à la gestion des espaces de stockage par MySQL. Par exemple, si vous utilisez le moteur de stockage InnoDB, certaines informations cruciales, comme les métadonnées des transactions et les informations d’état des tables, sont stockées dans un fichier tablespace partagé (`ibdata1`). Ce fichier est comptabilisé par `du`, mais pas directement par la commande SQL qui ne s’intéresse qu’à la taille des données dans les tables. De plus, MySQL peut allouer de l’espace disque à l’avance pour optimiser les performances, ce qui peut créer une différence entre l’espace disque réellement utilisé (rapporté par `du`) et l’espace réellement occupé par les données (rapporté par la commande SQL).

Il ne faut pas négliger l’overhead lié au système de fichiers. Chaque fichier sur le disque occupe non seulement l’espace pour son contenu, mais également un certain overhead lié à la gestion du système de fichiers (comme les blocs de disque alloués, les inodes, etc.). La commande `du` prend en compte cet overhead, ce qui peut également contribuer à la différence observée entre les deux mesures.

Pour conclure cet article, la commande SQL mesure uniquement la taille des données et des index au sein des tables, tandis que `du -sh` mesure l’ensemble de l’espace disque utilisé par le répertoire MySQL, incluant de nombreux autres fichiers nécessaires au bon fonctionnement de MySQL. C’est pourquoi la taille rapportée par `du` est généralement plus grande que celle rapportée par la commande SQL.