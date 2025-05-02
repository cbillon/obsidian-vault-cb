---
tags:
  - docker
  - yaml
---

## PlantUml 

Le logiciel permet de visualiser différents diagrammes.
ici utilisé pour visualiser une structure yaml 

[[https://plantuml.com/fr/|site officiel]]

===== Installation du logiciel =====

Le logiciel va ete installe sur une machine avec une image docker

Le serveur est une VM installé sur le serveur : 194.132.161.115
Adresse de la VM : 192.168.122.20

L'outil va être accessible depuis un navigateur : http://yaml.cbillon.ovh:8080 

==== Mise à jour du DNS ====

Créer une entrée:

  yaml.cbillon.ovh -> 164.132.161.115
  
==== Mise à jour Nginx ===


Créer une configuration reverse proxy Nginx dans /etc/nginx/sites-avaiable/yaml.conf

  server {
    listen 8080;
    server_name yaml.cbillon.ovh;
    access_log /var/log/nginx/reverse-access.log;
    error_log /var/log/nginx/reverse-error.log;
    location / {
       proxy_pass http://192.168.122.20:8080;     
    }
  }   
  
 ## Activer la configuration


  sudo ln -s /etc/nginx/sites-available/yaml.conf /etc/nginx/sites-enabled/yaml.conf

    
 - Vérification de la configuration : sudo nginx -t
 - Relancer nginx : suso systemctl restart nginx

## Test fonctionnement 

Avant le premier accès faire un un nettoyage de l’historique de navigation.

dans le navigateur :    http://yaml.cbillon.ovh