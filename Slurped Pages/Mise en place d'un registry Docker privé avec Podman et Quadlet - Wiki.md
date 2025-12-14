---
link: https://www.linuxtricks.fr/wiki/mise-en-place-d-un-registry-docker-prive-avec-podman-et-quadlet
site: Linuxtricks.fr
excerpt: |-
  Introduction
  Dans cet article, je vais vous montrer comment mettre en place un registry privé similaire à Quay.io ou Harbor,...
slurped: 2025-12-14T09:51
title: Mise en place d'un registry Docker privé avec Podman et Quadlet - Wiki
tags:
  - docker
  - registry
  - podman
  - quadlet
---

Table des matières

1. [Introduction](https://www.linuxtricks.fr/wiki/mise-en-place-d-un-registry-docker-prive-avec-podman-et-quadlet#paragraph-introduction)
2. [Prérequis](https://www.linuxtricks.fr/wiki/mise-en-place-d-un-registry-docker-prive-avec-podman-et-quadlet#paragraph-prerequis)
3. [Mettre en place le registry](https://www.linuxtricks.fr/wiki/mise-en-place-d-un-registry-docker-prive-avec-podman-et-quadlet#paragraph-mettre-en-place-le-registry)
4. [Quelques finitions](https://www.linuxtricks.fr/wiki/mise-en-place-d-un-registry-docker-prive-avec-podman-et-quadlet#paragraph-quelques-finitions)
5. [Test sur un PC du fonctionnement du registry](https://www.linuxtricks.fr/wiki/mise-en-place-d-un-registry-docker-prive-avec-podman-et-quadlet#paragraph-test-sur-un-pc-du-fonctionnement-du-registry)
## Introduction

Dans cet article, je vais vous montrer comment mettre en place un registry privé similaire à Quay.io ou Harbor, mais hébergé localement sur notre serveur.  
Vu que je travaille dans un environnement Red Hat, on va utiliser des conteneurs avec podman, et le tout fonctionnera en rootless.

## Prérequis

Si ce n'est pas encore fait, on installe podman. On aura aussi besoin de httpd-tools pour générer un mot de passe à notre registry

Code BASH :

dnf install podman httpd-tools

On va créer un utilisateur dédié pour faire tourner notre conteneur :

On va activer le _linger_ qui permetra au conteneur de démarrer automatiquement après un redémarrage du serveur, même si l'utilisateur n'a pas ouvert de session :

Code BASH :

loginctl enable-linger registry

## Mettre en place le registry

On se connecte ensuite avec cet utilisateur :

On va préparer la structure des données.  
On créé l'arborescence nécessaire pour stocker les données, certificats et informations d'authentification :

Code BASH :

mkdir -p ~/podman-registry/{data,auth,certs}
cd ~/podman-registry

On va utiliser un certificat auto-signé, mais si vous avez votre autorité de certification, c'est mieux (dans un cadre pro notamment).  
Adaptez le CN à votre nom de domaine :

Code BASH :

openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/registry.linuxtricks.lan.key -x509 -days 365 -out certs/registry.linuxtricks.lan.crt -subj "/C=FR/ST=Bourgogne/L=Dijon/O=Linuxtricks/CN=registry.linuxtricks.lan"

On créé un couple identifiant/mot de passe pour sécuriser l'accès au registry avec une authentification basique :

Code BASH :

htpasswd -Bbn admin superpass > auth/htpasswd

On va récupérer le conteneur registry de chez Docker pour ça (adaptez le nom de vos fichiers de certificats) :

Code BASH :

podman run -d \
  --name registry \
  --restart always \
  -p 5000:5000 \
  -v /home/registry/podman-registry/data:/var/lib/registry:Z \
  -v /home/registry/podman-registry/certs:/certs:Z \
  -v /home/registry/podman-registry/auth:/auth:Z \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.linuxtricks.lan.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/registry.linuxtricks.lan.key \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM="Identification au registry" \
  -e REGISTRY_STORAGE_DELETE_ENABLED=true \
  docker.io/library/registry:2

Si tout est OK, on créé un service systemd avec Quadlet pour que le conteneur démarre automatiquement au reboot de la machine :

Code BASH :

mkdir -p ~/.config/containers/systemd/
vim ~/.config/containers/systemd/registry.container

Contenu du fichier **registry.container** :

Code TEXT :

[Container]
Image=docker.io/library/registry:2
ContainerName=registry
PublishPort=5000:5000
Volume=/home/registry/podman-registry/data:/var/lib/registry:Z
Volume=/home/registry/podman-registry/certs:/certs:Z
Volume=/home/registry/podman-registry/auth:/auth:Z
Environment=REGISTRY_HTTP_TLS_CERTIFICATE=/certs/registry.linuxtricks.lan.crt
Environment=REGISTRY_HTTP_TLS_KEY=/certs/registry.linuxtricks.lan.key
Environment=REGISTRY_AUTH=htpasswd
Environment=REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
Environment=REGISTRY_AUTH_HTPASSWD_REALM="Identification au registry"
Environment=REGISTRY_STORAGE_DELETE_ENABLED=true
[Service]
Restart=always
[Install]
WantedBy=default.target

## Quelques finitions

Ici, on revient en tant que root.  
Un utilisateur non privilégié ne pouvant ouvrir un port inférieur à 1024, on a utilisé le port par défaut 5000.

Pour faire une redirection du port 443 sur le port 5000, je vais utiliser firewalld. Mais procédez comme vous voulez, si vous avez l'habitude avec un reverse-proxy, faites avec.

Code BASH :

firewall-cmd --permanent --add-forward-port=port=443:proto=tcp:toport=5000
firewall-cmd --permanent --add-service=https
firewall-cmd --reload

Avec un conteneur rootless, les fichiers temporaires peuvent poser problème au redémarrage.  
Je n'ai pas trouvé mieux que de faire un neootyage automatique avec le service systemd associé.  
Je créé ce fichier de conf :

Code BASH :

vim /etc/tmpfiles.d/podman-rootless.conf

Ajoutez et je mets dedans ceci (mon utilisateur reistry à l'UID 1001) :

Code BASH :

R! /tmp/storage-run-1001/containers
R! /tmp/storage-run-1001/libpod/tmp

## Test sur un PC du fonctionnement du registry

Vu qu'on a un certificat autosigné, je vais sur ma machine cliente configuree l'accès en mode insecure :

Code BASH :

vim /etc/containers/registries.conf

Dedans, on met (adapter avec votre nom de serveur pleinement qualifié) :

Code BASH :

 
[[registry]]
prefix = "registry.linuxtricks.lan"
location = "registry.linuxtricks.lan"
insecure = true

On teste maintenant notre registry depuis une machine cliente !

Je fais le test avec le conteneur hello-world que je récupère :

Je lui ajoute un tag :

Code BASH :

podman tag hello-world registry.linuxtricks.lan/linuxtricks/hello-world

On se connecte au registry :

Code BASH :

podman login registry.linuxtricks.lan

On est amené à saisir les identifiants qu'on a paramétré avec htpasswd :

Code :

`Username: admin   Password:    Login Succeeded!`

Et on pousse l'image :

Code BASH :

podman push registry.linuxtricks.lan/linuxtricks/hello-world

L'envoi se fait

Code :

`Getting image source signatures   Copying blob 2114fc8b7058 done   |    Copying config 5dd467fce5 done   |    Writing manifest to image destination`

On pourra lister les images disponibles dans le registry pour vérifier :

Code BASH :

curl -k -u admin:superpass https://registry.linuxtricks.lan/v2/_catalog

Et voilà :

Code :

`{"repositories":["linuxtricks/hello-world"]}`