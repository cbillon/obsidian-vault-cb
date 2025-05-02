---
tags:
  - php
  - debug
---

====== Php Install Tideways ======

Insttalation du profiler Xhprof dans l'environnement Moodle

[[https://moodledev.io/docs/guides/profiling]]

===== Tideways Extension ======

Tideways is an alternative to XHProf, and support for this is built into Moodle. The maintainer, Tideways, is a large, paid-for service, which can be used to identify a range of issues with production services.

==== Installation ====


[[https://github.com/tideways/php-xhprof-extension]]

=== Download Pre-Compiled Binaries ===

We pre-compile binaries for Linux AMD64 and for Windows. See the releases page for the downloads for each tagged version.

[[https://github.com/tideways/php-xhprof-extension/releases]]

Choisir la version correspondant à la version de PHP 

Si on veut recopier le fichier .deb sur unsite distant

  

Syntaxe 
  
  scp -o ProhyJump=<bastion > <fichier local>  <site distant>:>fichier destination>
  
  scp -o ProxyJump=cb@164.132.161.115:50635 ~/Téléchargements/tideways-xhprofr_5.0.2_amd64.deb moodleadm@192.168.122.15:/home/moodleadm/tideways-xhprofr_5.0.2_amd64.deb
  
pour installer :
  
  sudo apt install  tideways-xhprofr_5.0.2_amd64.deb

The Debian and RPM packages install the PHP extension to /usr/lib/tideways_xhprof and doesn't automatically put it into your PHP installation extension directory. You should link the package by full path for a simple installation:


creer un fichier tideways-xhprof.ini dans /etc/php/7.4/mods-available
dans le fichier .ini 

  extension=/usr/lib/tideways_xhprof/tideways_xhprof-7.4.so
  
  sudo phpenmod -v 7.4 tideways-xhprof
  sudo systemctl restart php7.4-fpm
  sudo systemctm restart apache2
  
 
=== Installation graphviz ===

Installer graphwiz

  sudo apt install graphviz
  

Mettre à jour le parametrage de moddle

Server > Symtem Paths mettre jour Path to dot: /usr/bin/dot


nota : si o a une erreur function str_contains not found (existe en php > 8)
     remplacer str_contains($key, '==>') par
               strpos($key, '==>') !== false

 
=== Parametrage Moodle ===

Pour faire un test

[[https://tracker.moodle.org/browse/MDL-62280]]

  - Navigate to Site administration -> Development
*Confirm that "Profiling" is listed
  - Open Profiling
Tick "Enable profiling"
  - Set "Profile these" to include /my/*
  - Save changes

View your Dashboard

  - Open Site administration -> Development -> Profiling runs
    Confirm that there is a run listed
  - Open the run
    Confirm that it has a Run ID, URL, Date, Exec time, CPU time, Func calls, and Memory used
  - Click on "View profiling details"
    Confirm that the report it shown

