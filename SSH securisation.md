---
tags:
  - ssh
  - security
---
L'accès SSH est l'une des principales portes d'entrée d'un serveur Linux. Mal configuré, il peut devenir une cible privilégiée pour les attaques par force brute ou par exploitation de failles.  
Voici comment sécuriser vos connexions SSH avec **clés asymétriques**, **serveur bastion** et **restrictions avancées**.

#### Plan de l'article

- Pourquoi bannir l'authentification par mot de passe
- Génération et gestion des clés SSH
- Configuration sécurisée de `sshd_config`
- Mise en place d'un serveur bastion SSH
- Restrictions avancées : `Match`, `ForceCommand`, `ChrootDirectory`
- Journalisation et monitoring des connexions
- Bonnes pratiques et ressources

---

#### Bannir l'authentification par mot de passe

L'authentification par mot de passe est vulnérable aux attaques par dictionnaire ou brute force.  
La première étape consiste à désactiver les mots de passe au profit des clés SSH :

Dans `/etc/ssh/sshd_config` :

```conf
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```

Puis redémarrer le service :

```bash
systemctl restart sshd
```

---

#### Génération et gestion des clés SSH

Sur le poste client :

```bash
ssh-keygen -t ed25519 -C "admin@exemple.com"
```

- `ed25519` : algorithme moderne, rapide et sécurisé.
- La clé publique est copiée sur le serveur dans `~/.ssh/authorized_keys` via :

```bash
ssh-copy-id user@serveur
```

✅ Bonnes pratiques : protéger la clé privée par un mot de passe et utiliser un agent SSH (`ssh-agent`) pour éviter de le retaper.

Pour aller plus loin, découvrez les [techniques avancées de SSH comme les tunnels et ProxyJump](https://www.shpv.fr/blog/ssh-avance-tunnels-proxy/).

---

#### Configuration sécurisée de SSHD

Exemples de directives à renforcer dans `/etc/ssh/sshd_config` :

```conf
PermitEmptyPasswords no
AllowTcpForwarding no
X11Forwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
```

---

#### Mise en place d'un serveur bastion

Le **serveur bastion** agit comme point d'entrée unique :

- Les administrateurs se connectent d'abord au bastion.
- Le bastion contrôle les accès et relaie vers les serveurs internes.
- Il permet de centraliser les audits et les journaux.

Avantages :

- Réduction de la surface d'attaque.
- Traçabilité renforcée.
- Possibilité d'imposer une MFA (ex. Duo, Google Authenticator).

Pour une mise en œuvre complète, consultez le guide [déployer un bastion SSH avec MFA](https://www.shpv.fr/blog/ssh-bastion-mfa/).

---

#### Restrictions avancées avec `Match` et `ForceCommand`

Dans `/etc/ssh/sshd_config`, il est possible de restreindre les accès selon l'utilisateur, l'adresse IP ou le groupe.

Exemple : restreindre un utilisateur à une seule commande :

```conf
Match User sauvegarde
    ForceCommand /usr/local/bin/backup.sh
    AllowTcpForwarding no
    X11Forwarding no
```

Exemple : limiter un groupe à un sous-réseau :

```conf
Match Group devs Address 192.168.10.0/24
    PermitTTY yes
```

---

#### ChrootDirectory pour isoler les utilisateurs

SSH permet d'enfermer un utilisateur dans un répertoire (`chroot`) afin de limiter sa visibilité sur le système.

Exemple minimal :

```conf
Match User sftpuser
    ChrootDirectory /srv/sftp
    ForceCommand internal-sftp
    AllowTcpForwarding no
```

---

#### Journalisation et monitoring

Surveiller les tentatives de connexion dans :

```bash
/var/log/auth.log
```

Mettre en place :

- **[Fail2ban](https://www.shpv.fr/blog/fail2ban/)** : bannir automatiquement les IP suspectes.
- **Auditd** : journaliser les actions des utilisateurs.
- **Centralisation** : exporter les logs vers un SIEM (ELK, Loki, Splunk).

---

#### Bonnes pratiques

- Interdire `root` en SSH, préférer `sudo`.
- Restreindre l'accès par IP via firewall (`nftables`, `iptables`, `pf`).
- Déployer un bastion pour tout accès administratif.
- Forcer l'usage de clés fortes (RSA ≥ 4096, Ed25519 recommandé).
- Mettre en place une authentification à deux facteurs (MFA).
- Auditer régulièrement les accès et supprimer les clés obsolètes.

Pour une gestion centralisée des clés, explorez les [certificats OpenSSH](https://www.shpv.fr/blog/openssh-ca/).

---

#### Ressources

- `man sshd_config`
- OpenSSH Hardening Guide (Mozilla)
- Documentation sur Fail2ban et Auditd

---

#### Conclusion

La sécurisation de l'accès SSH ne se limite pas à la mise en place de clés.  
Elle repose sur une combinaison de **bonnes pratiques**, d'**outils adaptés** (bastion, fail2ban, audit) et d'une **politique stricte de gestion des utilisateurs**.  
En suivant ces recommandations, vous réduisez considérablement les risques d'intrusion et garantissez une administration sereine de vos serveurs Linux.