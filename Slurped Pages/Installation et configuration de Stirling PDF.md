---
link: https://blog.raspot.in/fr/blog/installation-et-configuration-de-stirling-pdf
excerpt: Un blog sur le logiciel libre
slurped: 2025-02-22T09:14
title: Installation et configuration de Stirling PDF
tags:
  - pdf
  - tools
  - stirling
---

En recherchant une boite à outils pour travailler sur les PDF, je suis tombé sur le logiciel libre et gratuit **Stirling PDF**. L'outil est facilement installable et configurable avec Docker et présente les avantages d'être centralisé et auto-hébergeable.

**[Update 07/06/2024]** : mise à jour des variables d'environnement dans docker-compose.yml

**[Update 22/06/2024]** : mise en place du SSO ici : [Stirling-PDF + Authelia + Traefik + LLDAP = ❤️](https://doc.quercylibre.fr/Projets/Cluster%20RPI/Applications/app01/)

Le logiciel **Stirling PDF** est accessible via votre navigateur web et présente les fonctionnalités suivantes (liste non exhaustive) :

- Interface graphique interactive complète pour fusionner/diviser/faire pivoter/déplacer des pages et des PDF complets.
- Possibilité de diviser les PDF en plusieurs fichiers à partir des numéros de page spécifiés ou d’extraire toutes les pages en tant que fichiers individuels.
- Fusionner plusieurs PDF ensemble en un seul fichier.
- Ajouter/Générer des signatures.
- Auto Split PDF (Avec séparateurs de page numérisés physiquement)
- Supprimer les pages vierges.
- Convertir PDF en Word/Powerpoint/Autres (à partir de fichiers non scannés, tout comme les logiciels payant du commerce)
- Reconnaissance optique des caractères avec l’OCR (Optical Character Recognition).
- API

[Liste complète des fonctionnalités](https://github.com/Frooodle/Stirling-PDF#features)

L'application est sous licence [GPL-3.0](https://www.gnu.org/licenses/gpl-3.0.fr.html).

- Un nom de domaine de portée locale.
- Ubuntu Server pour faire tourner la partie serveur / VM sur Proxmox VE.
- Docker
- un reverse-proxy Nginx avec chiffrement

```
mkdir /srv/stirling-pdf
cd /srv/stirling-pdf
vim docker-compose.yml
```

```
services:
  stirling-pdf:
    image: frooodle/s-pdf:latest
    ports:
      - '8181:8080'
    volumes:
      -  /usr/share/tesseract-ocr/4.00/tessdata:/usr/share/tesseract-ocr/4.00/tessdata
    environment:
      SYSTEM_DEFAULTLOCALE: fr-FR
      LANGS: fr-FR
      UI_APPNAME: Boite à outils PDF
      UI_HOMEDESCRIPTION: Votre guichet unique hébergé localement pour travailler sur vos PDF.
      UI_APPNAVBARNAME: Boite à outils PDF
    restart: unless-stopped
```

Ne démarrez pas de suite le conteneur.

Pour la reconnaissance OCR, il nous faut installer un paquet pour la langue française :

```
apt install tesseract-ocr-fra
```

Vous pouvez démarrer le conteneur docker.

```
docker compose up -d
```

N'ayant pas l'utilité d'exposer le service sur Internet, j'utilise un nom de portée locale et ma propre PKI pour les certificats.

```
server {
  listen 443 ssl http2;
  server_name pdf.mondomaine.lan;

  ssl_certificate      /etc/nginx/ssl/pdf.mondomaine.lan/cert-pdf.crt;
  ssl_certificate_key  /etc/nginx/ssl/pdf.mondomaine.lan/cert-pdf.key;  

  # Specify SSL config if using a shared one.
  include conf.d/ssl/ssl.conf;

  # Allow large attachments
  client_max_body_size 128M;

  location / {
    proxy_pass http://127.0.0.1:8181;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
  access_log      /var/log/nginx/pdf.access.log;
  error_log       /var/log/nginx/pdf.error.log;
}
```

Connectez-vous à l'interface web depuis votre navigateur web (Firefox dans mon cas) : https://pdf.mondomaine.lan

![stirlingpdf](https://blog.raspot.in/images/c/8/7/8/9/c8789bd25dd0dfd29e0ff60a713af76600f61e3f-stirlingpdf.png "stirlingpdf")

Je ne vais pas détailler l'ensemble des fonctionnalités mais celles testées fonctionnent parfaitement bien.

Personnellement je trouve l'outil tout simplement génial. Il est complet et permet de rivaliser avec les outils payants. Qui plus est, il est auto-hébergeable et donc évite à vos utilisateurs d'utiliser des sites en ligne avec le risque de fuites de données. Vous pouvez tester la version en démo sur [https://pdf.adminforge.de](https://pdf.adminforge.de/)/

Source : [Dépôt Github Stirling-PDF](https://github.com/Frooodle/Stirling-PDF)