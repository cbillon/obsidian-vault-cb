---
tags:
  - mysql
  - remote
---

====== Access remote ======

Pour un accés distant 

===== Configuration du serveur =====

on doit avoir le paramètre bind-address avec ip du site distant ou 0.0.0.0 

  bind-address = 0.0.0.0


mettre  jour dans :
  * /etc/mysql/mariadb.cnf
  * /etc/mysql/mariadb-conf.d/50-server.conf

=== Création d'un utilisateur acces distant ===

se connecter en local sur mysql

  CREATE USER 'root'@'%' IDENTIFIED BY 'sesame';
  GRANT ALL PRIVILEGES ON *.* to 'root'@'%';
  FLUSH PRIVILIGES;

Redémarrer mysql

  sudo sytemctl restart mysql
 

==== Firewall ====

Pour UFW 

  sudo ufw allow 3306
  
Test de l'access

pour vérifier l'ouverture du port sur le serveur, 
installer telnet
  
  sudo apt install telnetd
  
Vérifier l'ouvertire du port standard 3306

  telnet <aédresse ip> 3306
  

=== Navicat ===

On peut accéder diretement au serveur base de données

  Adresse : 192.168.122.12
          : root
          : sesame

 
