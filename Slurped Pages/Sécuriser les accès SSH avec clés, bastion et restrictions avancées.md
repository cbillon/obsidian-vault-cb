---
link: https://www.shpv.fr/blog/ssh-securisation/
byline: SHPV
site: SHPV
date: 2025-09-11T12:30
excerpt: Renforcez la sécurité de vos serveurs Linux en configurant l’authentification par clés SSH, en déployant un serveur bastion et en appliquant des restrictions avancées.
twitter: https://twitter.com/@SHPVFR
tags:
  - ssh
  - bastion
slurped: 2025-10-12T18:23
title: Sécuriser les accès SSH avec clés, bastion et restrictions avancées
---

L’accès SSH est l’une des principales portes d’entrée d’un serveur Linux. Mal configuré, il peut devenir une cible privilégiée pour les attaques par force brute ou par exploitation de failles.  
Dans cet article, nous allons voir comment sécuriser vos connexions SSH à l’aide de **clés asymétriques**, d’un **serveur bastion** et de **restrictions avancées**.

## Plan de l’article

- Pourquoi bannir l’authentification par mot de passe
- Génération et gestion des clés SSH
- Configuration sécurisée de `sshd_config`
- Mise en place d’un serveur bastion SSH
- Restrictions avancées : `Match`, `ForceCommand`, `ChrootDirectory`
- Journalisation et monitoring des connexions
- Bonnes pratiques et ressources

---

## Bannir l’authentification par mot de passe

L’authentification par mot de passe est vulnérable aux attaques par dictionnaire ou brute force.  
La première étape consiste à désactiver les mots de passe au profit des clés SSH :

Dans `/etc/ssh/sshd_config` :

```
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

Puis redémarrer le service :

```
systemctl restart sshd
```

---

## Génération et gestion des clés SSH

Sur le poste client :

```
ssh-keygen -t ed25519 -C "admin@exemple.com"
```

- `ed25519` : algorithme moderne, rapide et sécurisé.
- La clé publique est copiée sur le serveur dans `~/.ssh/authorized_keys` via :

```
ssh-copy-id user@serveur
```

> ✅ Bonnes pratiques : protéger la clé privée par un mot de passe et utiliser un agent SSH (`ssh-agent`) pour éviter de le retaper.

---

## Configuration sécurisée de SSHD

Exemples de directives à renforcer dans `/etc/ssh/sshd_config` :

```
PermitEmptyPasswords no
AllowTcpForwarding no
X11Forwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
```

---

## Mise en place d’un serveur bastion

Le **serveur bastion** agit comme point d’entrée unique :

- Les administrateurs se connectent d’abord au bastion.
- Le bastion contrôle les accès et relaie vers les serveurs internes.
- Il permet de centraliser les audits et les journaux.

Avantages :

- Réduction de la surface d’attaque.
- Tracabilité renforcée.
- Possibilité d’imposer une MFA (ex. Duo, Google Authenticator).

---

## Restrictions avancées avec `Match` et `ForceCommand`

Dans `/etc/ssh/sshd_config`, il est possible de restreindre les accès selon l’utilisateur, l’adresse IP ou le groupe.

Exemple : restreindre un utilisateur à une seule commande :

```
Match User sauvegarde
    ForceCommand /usr/local/bin/backup.sh
    AllowTcpForwarding no
    X11Forwarding no
```

Exemple : limiter un groupe à un sous-réseau :

```
Match Group devs Address 192.168.10.0/24
    PermitTTY yes
```

---

## ChrootDirectory pour isoler les utilisateurs

SSH permet d’enfermer un utilisateur dans un répertoire (`chroot`) afin de limiter sa visibilité sur le système.

Exemple minimal :

```
Match User sftpuser
    ChrootDirectory /srv/sftp
    ForceCommand internal-sftp
    AllowTcpForwarding no
```

---

## Journalisation et monitoring

Surveiller les tentatives de connexion dans :

```
/var/log/auth.log
```

Mettre en place :

- **Fail2ban** : bannir automatiquement les IP suspectes.
- **Auditd** : journaliser les actions des utilisateurs.
- **Centralisation** : exporter les logs vers un SIEM (ELK, Loki, Splunk).

---

## Bonnes pratiques

- Interdire `root` en SSH, préférer `sudo`.
- Restreindre l’accès par IP via firewall (`nftables`, `iptables`, `pf`).
- Déployer un bastion pour tout accès administratif.
- Forcer l’usage de clés fortes (RSA ≥ 4096, Ed25519 recommandé).
- Mettre en place une authentification à deux facteurs (MFA).
- Auditer régulièrement les accès et supprimer les clés obsolètes.

---

## Ressources

- `man sshd_config`
- OpenSSH Hardening Guide (Mozilla)
- Documentation sur Fail2ban et Auditd

---

## Conclusion

La sécurisation de l’accès SSH ne se limite pas à la mise en place de clés.  
Elle repose sur une combinaison de **bonnes pratiques**, d’**outils adaptés** (bastion, fail2ban, audit) et d’une **politique stricte de gestion des utilisateurs**.  
En suivant ces recommandations, vous réduisez considérablement les risques d’intrusion et garantissez une administration sereine de vos serveurs Linux.