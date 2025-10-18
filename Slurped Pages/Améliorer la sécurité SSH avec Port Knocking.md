---
link: https://www.shpv.fr/blog/ssh-port-knocking/
byline: SHPV
site: SHPV
date: 2025-10-12T09:00
excerpt: Découvrez comment protéger vos accès SSH grâce au port knocking, une technique qui masque le port d'administration et ne l'ouvre qu'après une séquence d'accès spécifique.
twitter: https://twitter.com/@SHPVFR
tags:
  - ssh
  - port
slurped: 2025-10-12T18:12
title: Améliorer la sécurité SSH avec Port Knocking
---

La sécurité des accès SSH est cruciale pour éviter les tentatives d’intrusion.  
En complément des clés SSH, du filtrage IP et de la limitation de connexions, il est possible d’utiliser une technique appelée **Port Knocking**.  
Cette méthode consiste à garder le port SSH fermé et à ne l’ouvrir qu’après qu’un client ait frappé une séquence spécifique de ports.

## Plan de l’article

- Principe du port knocking
- Installation des outils nécessaires
- Configuration avec `iptables`
- Automatisation du client
- Bonnes pratiques de sécurité
- Conclusion

---

## Principe du port knocking

Le serveur garde le port SSH (par ex. 22) **fermé par défaut**.  
Pour y accéder, le client doit envoyer une série de paquets sur des ports définis (ex. 7000, 8000, 9000).  
Lorsque la séquence est correcte, une règle du pare-feu ouvre temporairement le port SSH.

---

## Installation des outils nécessaires

Sous Debian/Ubuntu :

```
sudo apt install knockd
```

Sous CentOS/RHEL :

```
sudo yum install knock-server
```

---

## Configuration avec iptables

Exemple `/etc/knockd.conf` :

```
[options]
    logfile = /var/log/knockd.log

[openSSH]
    sequence    = 7000,8000,9000
    seq_timeout = 5
    command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn

[closeSSH]
    sequence    = 9000,8000,7000
    seq_timeout = 5
    command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn
```

Redémarrer knockd :

```
sudo systemctl enable knockd
sudo systemctl start knockd
```

---

## Automatisation du client

Pour envoyer la séquence depuis un poste client :

```
knock serveur.exemple.com 7000 8000 9000
ssh utilisateur@serveur.exemple.com
```

---

## Bonnes pratiques de sécurité

- Combiner le port knocking avec des **clés SSH**.
- Utiliser des séquences longues et non triviales.
- Surveiller les logs du serveur knockd.
- Coupler avec Fail2ban pour plus de sécurité.

---

## Conclusion

Le **Port Knocking** renforce la sécurité SSH en masquant le port d’accès et en réduisant drastiquement les tentatives de brute-force.  
C’est une méthode simple à mettre en place, qui complète efficacement les autres bonnes pratiques de sécurisation SSH.