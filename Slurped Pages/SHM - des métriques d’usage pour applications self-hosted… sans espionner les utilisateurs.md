---
link: https://linuxfr.org/news/shm-des-metriques-d-usage-pour-applications-self-hosted-sans-espionner-les-utilisateurs
byline: Benjy33
excerpt: L’actualité du logiciel libre et des sujets voisins (DIY, Open
  Hardware, Open Data, les Communs, etc.), sur un site francophone contributif
  géré par une équipe bénévole par et pour des libristes enthousiastes
tags:
  - slurp/Linux
  - slurp/Logiciel
  - slurp/Libre
  - slurp/GNU
  - slurp/Free
  - slurp/Software
  - slurp/Actualit
  - slurp/Forum
  - slurp/Communaut
slurped: 2026-01-14T14:47
title: "SHM : des métriques d’usage pour applications self-hosted… sans
  espionner les utilisateurs"
---

Quand on développe et distribue des applications **open-source auto-hébergées**, il y a une question très simple à laquelle il est presque impossible de répondre :

> _Combien d’instances actives de mon application sont réellement utilisées ?_

![SHM](app://img.linuxfr.org/img/68747470733a2f2f69696c692e696f2f666c6a415439522e706e67/fljAT9R.png "Source : https://iili.io/fljAT9R.png")

C’est exactement le problème que j’ai rencontré avec **Ackify**, une application open-source de preuve de lecture de documents (politiques internes, procédures, formations, etc.), déployée **en self-hosted** par ses utilisateurs - sans que j'ai le moindre contrôle dessus.

Pas de SaaS, pas de compte centralisé, pas de tracking utilisateur.  
Résultat : zéro visibilité.

👉 **Combien d’instances Ackify tournent vraiment ?**  
👉 **Quelles versions sont encore actives ?**  
👉 **Quelles fonctionnalités sont utilisées (ou pas) ?**

C’est pour répondre à ce besoin très concret que j’ai créé **SHM – Self-Hosted Metrics**.

### SHM, c’est quoi ?

**SHM** est un serveur de télémétrie **privacy-first**, conçu spécifiquement pour les applications **self-hosted open-source**.

L’idée est simple :

- chaque instance auto-hébergée envoie périodiquement **un snapshot de métriques agrégées**
- aucune donnée utilisateur
- aucun événement individuel
- aucun tracking comportemental

Juste ce qu’il faut pour comprendre l’usage réel d’un logiciel déployé “dans la nature”.

---

### Un point important : SHM est agnostique

Contrairement à beaucoup d’outils existants, SHM **n’impose aucun schéma**.

Tu envoies :

```
{
  "documents_created": 123,
  "active_users": 42,
  "webhooks_sent": 9
}
```

➡️ le **dashboard s’adapte automatiquement** :

- nouvelles cartes KPI
- nouvelles colonnes
- graphiques générés dynamiquement

Aucun frontend à recompiler, aucune migration à écrire.

![Dashboard Graph](app://img.linuxfr.org/img/68747470733a2f2f69696c692e696f2f666c6a375352662e706e67/flj7SRf.png "Source : https://iili.io/flj7SRf.png")  
![Dashboard Détail](app://img.linuxfr.org/img/68747470733a2f2f69696c692e696f2f666c6a37554e342e706e67/flj7UN4.png "Source : https://iili.io/flj7UN4.png")

---

### Un petit mot sur Ackify

Ackify est l’application qui a déclenché tout ça :

- open-source
- self-hosted
- preuve de lecture avec signature cryptographique
- alternative légère à DocuSign pour des usages internes

SHM est désormais utilisé pour répondre à des questions très simples :

- combien d’instances actives ?
- combien de documents créés ?
- combien de signatures générées ?

---

### Projet open-source

- Code source : [https://github.com/btouchard/shm](https://github.com/btouchard/shm)
- Déploiement simple en `docker compose`
- SDK Go (et Node.js) pour intégrer la collecte facilement

Le projet est encore très jeune (MVP), mais fonctionnel et déjà utilisé en conditions réelles.

Les retours, critiques et idées sont évidemment bienvenus 🙂

---

### Stack technique (sobre et assumée)

- **Backend : Go** (binaire unique, léger)
- **Stockage : PostgreSQL (JSONB)**
- **Déploiement : Docker**
- **Licence : AGPLv3** (SDK en MIT)
- **Auth des instances : Ed25519** (clé générée localement, signature des snapshots)

Chaque instance :

- génère une identité cryptographique locale
- s’enregistre une seule fois
- signe chaque envoi de métriques ➡️ impossible de spoof une instance existante.

---

### Et côté vie privée ?

C’était non négociable.

SHM :

- ne collecte **aucune donnée personnelle**
- ne collecte pas les IP (hors reverse-proxy)
- ne collecte ni hostname, ni username
- fonctionne sur des **compteurs agrégés uniquement**

C’est au mainteneur du logiciel de décider **quelles métriques exposer**, et à l’utilisateur final de pouvoir désactiver la télémétrie.

## Aller plus loin

- [Self Hosted Metrics (SHM)](https://github.com/btouchard/shm "https://github.com/btouchard/shm") (91 clics)
- [Ackify](https://github.com/btouchard/ackify-ce "https://github.com/btouchard/ackify-ce") (67 clics)