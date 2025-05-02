---
tags:
  - php
  - sql
  - composer
---

====== Installtion php Composer Drivers sql server ======

===== Installation Visual C++ =====


[[https://davidperonne.com/installation-de-php-7-et-composer-sur-windows-10-en-mode-natif/]]

  * Telecharger Visual C++ redistritrubuable codeà partir de :

  https://support.microsoft.com/fr-fr/help/2977003/the-latest-supported-visual-c-downloads

  * Sélectionner la version vc_redist.x64.exe
  * Executer vc_reredist.x64.exe

===== Installation Php 7.4 =====

Télécharger la version php à partir de :

  https://windows.php.net/download/
  
  

  * Choisir : version php 7.4 VS16 x64 « Non Thread Safe », fichier zip nommé VC16 x64 Non Thread Safe
  * Décompresser php-7.4.16-nts-Win32-vc15-x64.zip dans le répertoire c:\Php7

nota:
  * Thread-Safe(TS) - utilisé pour des serveurs web n'ayant qu'un seul processus, comme Apache avec mod_php
  * Non-Thread-Safe(NTS) - utilisé pour IIS et les autres serveurs web FastCGI (Apache avec mod_fastcgi) et c'est 
    la version recommandée pour les scripts en ligne de commande

  * Installation dans c:\Php7

  * Recopier c:\Php7\php.ini-devlopment en c:\Php7\php.ini
  * Mettre à jour PATH

  Pour mettre à jour PATH : taper path dans la zone recherche puis sélectionner
  Windows Configuration systeme Variables d’environnement
  Modifier PATH  Ajouter c:\Php7

  * Vérifier 

  php --version

php –i pour vérifier le .ini pris en compte

  php –i | findstr Thread 
  
nota : Mai 2023 j'ai une erreur VCRUNTILE140.dll introuvable ou poit d'entrée DllUnregisterServer introuvale
Le probléme a été régle en chargeant les 2 versions  x86 and x64 versions of the multi-installer Visual C++ 2015, 2017 and 2019 redistributables
[[https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads]]

===== Installation drivers sqlsrv =====

Rechercher Télécharger Microsoft Drivers for PHP Sql Server

  https://learn.microsoft.com/fr-fr/sql/connect/php/download-drivers-php-sql-server?view=sql-server-ver18

Sélectionner la version pour php 7.4 -> Version 5.10

nota : la version 5.11 abandonne le support php 7.4

Télécharger  SQLSRV510.ZIP  le decompresser dans SQLSRV510

Recopier copy *74_nts_64.dll dans c:\Php7\ext
Copier dans c:\Php7\ext :  php_pdo_sqlsrv_74_nts_x64.dll et    php_sqlsrv_74_nts_x64.dll

===== Installation driver odbc for sql server =====

Recherche sur Google : msodbcsql.msi

  https://learn.microsoft.com/fr-fr/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver18

Mai 2023 la version est version 18
Choisir version x64  installer msodbcsql.msi 

====== Mise à jour du paramétrage de php ======
 
Paramétrage c:\PHP7\php.ini

pour le systeme windows mettre à jour extension_dir

extension_dir=‘c:\Php7\ext’  indiquer un chemin absolu

Dé commenter

  Extension=pdo_mysql
  Extension=mysqli

Ajouter :

  Extension = php_pdo_sqlsrv_74_nts_x64.dll   
  Extension = php_sqlsrv_74_nts_x64.dll 
  
Redemarrer Windows

Les drivers sont installés

Vérifier les drivers installés
Créer un fichier drivers.php

  <?php
    print_r(PDO::getAvailableDrivers());

On devrait avoir 1 tableau
 [0]  => sqlsrv
 [1]  => mysql


Note Avril 2021 
Php 7.4 est installé dans c:\Php7
Installation driver odbc for sql server

