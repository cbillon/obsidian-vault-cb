---
tags:
  - livebox
  - Orange
  - configuration
---
## Pre requis
On doit avoir déjà paramétrer le zones du domaine **cbillon.ovh**

- créer une zone *\*.cbillon.ovh**

## Paramétrage

Se connecter à la Livebox
- 192.168.1.1   admin Se$ame!1

dans l'onglet Réseau
- personnaliser TTL : 60 pour un rafraîchissement automatique dyndns
- cible: indiquer adresse actuelle de la frebox : 109.222.121.67
- DHCP
-Attribuer une adresse IP fixe à l'équipement
- Onglet DNS
Attribuer un nom au serveur asus 192.168.1.102
- DynDNS 
- parametrer le mise à jour automatique 
-   OVH-dyndns cbillon.ovh bc1678707-ovh qoznof-xesxu6-kutZuc

la livebox doit se connecter pour mettre à jour les entrées DNS qui ont un TTL de 60
L'adresse IP n'est pas fixe , est dite préférentielle
n'est changée si arrete > 7 jours, ou nécessité de service d'Orange
sinon on peut forcer la resistance dans l'onglet CGN
pour obtenir l'adresse ip de la Livebox sur Internet: dans le navigateur whatismyip.com




