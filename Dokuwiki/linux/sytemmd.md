---
tags:
  - linux
  - tips
---

====== Configuration des services avec systemd ======

La commande à connaître est maintenant systemctl. C'est elle qui permet de gérer la grande majorité des interactions au niveau administrateur.

La syntaxe est la suivante :

  systemctl [OPTIONS} COMMANDE [SERVICE]

Activation / Désactivation d'un service au démarrage
On utilise les commandes enable ou disable, selon le cas

systemctl enable/disable SERVICE
Exemple :

  systemctl disable mysql.service

Opérations de démarrage / arrêt / vérification
On utilise les commandes start / stop / status, pour une interaction directe avec le service :

  systemctl start / stop / restart / status SERVICE

Exemple :

   sudo systemctl start mysql

La commande ne donne généralement pas d'information en cas de succès.

On peut vérifier le statut du service :


   sudo systemctl status mysql
   
   
===== Lister les services déclarés =====

On utilise la commande list-unit-files. On peut filtrer sur l'état 

  systemctl list-unit-files [--state=disabled|enabled]

Après avoir modifié un fichier de déclaration de service, dans (/usr)/lib/systemd/system ou /etc/ystemd/system (pour les services "perso", il est recommandé de recharger la configuration, via la commande daemon-reload

  systemctl daemon-reload
  
==== Fichiers de définition des services ====

A la différence des fichiers de démarrage sysV, qui reposaient sur un seul fichier dans /etc/init.d, et des liens symboliques vers /etc/rc*.d, on distingue maintenant la définition du service et les scripts de démarrage, arrêt, etc.

Les fichiers de définiition se trouvent dans /lib/system/system (debian/ubuntu) ou /usr/lib/system (rhel, sles) pour les paquets fournis par la distribution.

Pour des services personnalisés, il est recommandé de placer les fichiers dans le répertoire /etc/systemd/system.

Lorsqu'on active (enable) un service, on crée un lien symbolique dans le répertoire précisé par la section [Install] du fichier de configuration.
Exemple pour le service mysql :

  [Install]
  WantedBy=multi-user.target

Par défaut losqu'on installe mysql, le service est  activé. On peut le vérifier avec la commande status, dans les informations vendor preset  (enabled) :

  systemctl status mysql.service
    mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: inactive (dead)

On peut vérifier la présence d'un lien vers le répertoire cible ;

  ls -l /etc/systemd/system/multi-user.target.wants/mysql.service
  
  lrwxrwxrwx 1 root root 33 déc.   7 16:46 /etc/systemd/system/multi-user.target.wants/mysql.service -> 
  /lib/systemd/system/mysql.service

Au reboot de la machine, mysql sera donc relancé automatiquement.

=== Format d'un fichier de configuration ===

Le fichier de configuration a un format de fichier "ini" comprenant plusieurs sections :

Unit    : définiition du nom du service et de ses dépendances (Before / After)
Install : niveau à utiliser lors de l'activation du service
Service : définition des paramètres : scripts de lancement, arrêt, pre/post start, variables, etc
Exemple pour le service MySQL :

  # MySQL systemd service file

  [Unit]
  Description=MySQL Community Server
  After=network.target

  [Install]
  WantedBy=multi-user.target

  [Service]
  User=mysql
  Group=mysql
  PermissionsStartOnly=true
  ExecStartPre=/usr/share/mysql/mysql-systemd-start pre
  ExecStart=/usr/sbin/mysqld
  ExecStartPost=/usr/share/mysql/mysql-systemd-start post
  TimeoutSec=600
  Restart=on-failure
  RuntimeDirectory=mysqld
  RuntimeDirectoryMode=755

On distingue bien ici les scripts de démarrage et arrêt, ainsi que les paramètres de lancement (utilisateur, groupe, etc).

Fichier de configuration pour OpenDJ
En bonus, je fournis un exemple de fichier de configuration pour OpenDJ, un annuaire LDAP écrit en Java, qui offre la possibilité d'avoir plusieurs installations et donc plusieurs instances sur une machine, de manière simple.

Par défaut, le démarrage et l'arrêt de l'annuaire se font via les commandes bin/start-ds et bin/stop-ds du répertoire d'installation de OpenDJ. De ce fait, la configuration systemd est assez simple. En prenant l'hypothèse où l'annuaire est installé dans le répertoire /opt/opendj-idm :

  #
  # Service definition for OpenDJ
  #
  [Unit]
  Description=A Java Directory Server
  After=network.target

  [Service]
  Type=forking
  PIDFile=/opt/opendj-idm/logs/server.pid
  ExecStart=/opt/opendj-idm/bin/start-ds
  ExecStop=/opt/opendj-idm/bin/stop-ds
  TimeoutStopSec=5
  KillMode=mixed

  [Install]
  WantedBy=multi-user.target
 