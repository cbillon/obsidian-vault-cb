---
tags:
  - mysql
---

====== Création User Base de données avec Ansible ======

https://stackoverflow.com/questions/44614863/ansible-unable-to-find-my-cnf-cant-connect-to-local-mysql-server

Le module Ansible pour créer un utilisateur demande 3 paramètres:
  - login_host
  - login_user
  - login_password


pour la création d'un base de données le paramètre login_host n'est pas indispensable
le user doit être accessible depuis l’extérieur (erreor 2003 can connect)

si ces paramètres ne sont pas présent ou ne sont pas corrects, Ansible recherche dans  /root/.my.cnf

le format du fichier :

[client]\\
user=root\\
password=sesame\\
