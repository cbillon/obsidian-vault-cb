---
tags:
  - moodle
  - windows
---

====== Installation environnements des scripts sous Windows ======

===== Installation PHP7.4 =====

Installation de php
Voir davidperonne installation php7 windows10

Les fichiers d’installation se trouvent dans Google drive

=== 1 Télécharger Visual C++ redistritrubuable code ===

https://support.microsoft.com/fr-fr/help/2977003/the-latest-supported-visual-c-downloads
Sélectionner la version vc_redist.x64.exe
Exécuter vc_reredist.x64.exe

=== 2 Télécharger la version php ===

https://windows.php.net/download/
Choisir : version php 7.4 VS16 x64 « Non Thread Safe », 
fichier zip nommé VC16 x64 Non Thread Safe
Décompresser php-7.4.16-nts-Win32-vc15-x64.zip dans le répertoire c:\Php7

Thread-Safe(TS) - utilisé pour des serveurs web n'ayant qu'un seul processus, comme Apache avec mod_php
Non-Thread-Safe(NTS) - utilisé pour IIS et les autres serveurs web FastCGI (Apache avec mod_fastcgi) et c'est la version recommandée pour les scripts en ligne de commande

Installation dans c:\Php7
Recopier c:\Php7\php.ini-devlopment en c:\Php7\php.ini

Pour mettre à jour PATH : taper path dans la zone recherche puis sélectionner
Windows Configuration systeme Variables d’environnement
Modifier PATH  A
Ajouter c:\Php7

Vérifier 

  php --version
  php –ini  pour vérifier le .ini pris en compte
  php –i pour avoir le détail

Pour vérifier : 

  php –i | findstr Thread 

=== 3 Installation drivers sqlsrv ===

Rechercher Télécharger Microsoft Drivers for PHP Sql Server
https://learn.microsoft.com/fr-fr/sql/connect/php/download-drivers-php-sql-server?view=sql-server-ver16

Sélectionner la version pour php 7.4 -> Version 5.10
Télécharger  SQLSRV510.ZIP  le decompresser dans SQLSRV510
Recopier copy *74_nts_64.dll c:\Php7\ext

Installation driver odbc for sql server

Recherche sur Google : msodbcsql.msi
https://learn.microsoft.com/fr-fr/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16
Choisir version x64  installer msodbcsql.msi 

Installation drivers mysql 

Ces drivers sont déjà installés

== 4 Mise à jour Php.ini ===

Paramétrage c:\Php7\php.ini

;on windows
extension_dir=‘c:\php7\ext’  indiquer un chemin absolu

Dé commenter:
Extension=pdo_mysql
Extension=mysqli
Extension=openssl

Ajouter :

extension=pdo_sqlsrv_74_nts_x64   
extension=sqlsrv_74_nts_x64 

Redémarrer Windows

=== Vérification installation drivers ===

Vérifier les drivers installés
Créer un fichier drivers.php

  <?php
      print_r(PDO::getAvailableDrivers());

On devrait avoir 1 tableau
 [0]  => sqlsrv
 [1]  => mysql

