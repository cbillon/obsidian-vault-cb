---
link: https://medspx.fr/blog/Sysadmin/modern_ssh/
byline: Médéric Ribreux
excerpt: |-
  Posted by Médéric Ribreux
         🗓 2020-05-01
          In 
  blog/
  Sysadmin/
tags:
  - ssh
slurped: 2026-03-19T15:53
title: Une configuration SSH plus moderne
---

## Introduction

[source](https://medspx.fr/blog/Sysadmin/modern_ssh/#l2-1)

En ces temps de confinement généralisé, SSH est votre ami. Du moins, SSH est mon ami. Car sans lui, je serai bien en peine de respecter le télétravail recommandé par le confinement (ainsi qu'avec Wireguard, mon autre ami).

Oui, car c'est avec SSH que j'accède aux machines à distances via plusieurs moyens (en direct via Wireguard ou en utilisant le mode tunnel pour quelques applications).
## ssh-audit

Par hasard, j'ai trouvé un programme d'audit SSH nommé [ssh-audit](https://github.com/arthepsy/ssh-audit). Il est [disponible sous Debian](https://packages.debian.org/search?keywords=ssh-audit&searchon=names&suite=all&section=all&sourceid=mozilla-search). L'objectif de ssh-audit est de tester un serveur SSH et indiquer les éléments de configuration qui posent problème en suivant les différents [CVE](https://en.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures). Attention, ssh-audit date de 2016 il n'est donc plus nécessairement à jour par rapport aux différentes failles de sécurité spécifique de SSH. Mais dans la pratique, ça peut être un bon outil pour commencer à améliorer votre configuration.

D'une manière générale, ssh-audit est trivial à utiliser. Son intérêt essentiel, c'est qu'il vous indiquera en rouge ce qui ne va pas. Chez moi, il s'agissait principalement des modes d'échange de clef (KexAlgorithms dans la configuration), les algorithmes de chiffrements, les fonctions de hash. Après audit, j'ai ajouté les lignes suivantes dans `/etc/ssh/sshd_config`, afin d'utiliser uniquement des protocoles/fonctions éprouvées d'un point de vue sécurité:

KexAlgorithms curve25519-sha256@libssh.org
HostKeyAlgorithms ecdsa-sha2-nistp521-cert-v01@openssh.com,ecdsa-sha2-nistp384-cert-v01@openssh.com,rsa-sha2-512-cert-v01@openssh.com,rsa-sha2-256-cert-v01@openssh.com,ssh-rsa,ssh-ed25519
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com

Vous pouvez vous référer au manuel d'OpenSSH pour avoir plus d'informations sur ces 4 directives. Attention, modifier cette configuration revient souvent à invalider les clefs SSH autorisées (pour cause de changement de hash bien souvent). Il vous faudra faire un peu de ménage dans votre fichier `~/.ssh/authorized_keys`.

## Amélioration de la continuité de connexion

J'avais aussi un autre problème à régler: mon serveur de "bastion" (qui est en fait un vulgaire PC de bureau) se déconnectait relativement souvent, figeant ainsi la connexion SSH. Le problème vient de la configuration du routeur du bureau qui coupe les connexions sortantes de manière un peu trop rapide (on est dans un intervalle de moins d'une minute). De fait, la connexion Wireguard, à défaut de générer du trafic sortant au niveau du bastion, s'interrompt toutes les minutes. J'aurais pu régler ça en utilisant la directive `PersistentKeepalive` de la connexion pour la mettre en phase avec la configuration du routeur mais ça me semblait être un peu trop lourd de générer du trafic même quand la connexion n'est pas utilisée.

Au lieu de ça, j'ai ajouté les lignes suivantes dans mon fichier de configuration client SSH (`~/.ssh/config`)

# Accès au serveur de bastion
Host vpnwork
  User me
  ServerAliveInterval 15
  ServerAliveCountMax 1

La directive ServerAliveInterval teste la connexion distante toutes les 15 secondes. Ça génère assez de trafic pour laisser la connexion disponible et ne pas avoir d'état fantôme dans le shell distant.

## Rebondir vers des machines distantes

Un truc que j'ai redécouvert pendant le confinement, c'est le rebond: accéder à une machine distante via une autre machine. Il y a encore quelques années c'était assez pénible à gérer dans SSH (il fallait indiquer des commandes à exécuter). De nos jours, la directive ProxyJump fait tout le travail.

# Accès à la VM1
Host vm1
  User me
  ProxyJump vpnwork

# Accès à la VM2
Host vm2
  User other
  ProxyJump vpnwork

Dans cette condition, ProxyJump va déclencher une première connexion sur le serveur de bastion (vpnwork ici) puis utiliser cette connexion pour attaquer les autres machines, comme si on était sur le réseau local. C'est très pratique car ça permet de faire des choses aussi simples que:

ssh vm2

permet d'attaquer la machine vm2 en direct, par rebond, sans avoir à se soucier de la configuration du rebond (en plus l'auto-completion Bash fonctionne avec les noms d'hôte, c'est tout simplement magique).

On peut également l'utiliser avec un tunnel pour accéder à du RDP par exemple:

ssh -f -N -L localhost:3389:vm2:3389 vpnwork

Je vous laisse lire le manuel d'OpenSSH pour comprendre les détails de la commande (cet article n'est pas un tutoriel maintenu).

## Conclusions

En 2020, OpenSSH reste plus que jamais d'actualité. D'une manière générale, je crois que ce dernier demeurera, pour de nombreuses années encore, un incontournable de l'administration système distante et sécurisée. Rendez-vous compte qu'OpenSSH est sorti il y a près de 20 ans et que c'est encore une solution de référence. Rares sont les programmes de 2020 qui peuvent se targuer d'avoir le même pedigree, dans un monde où la durée de vie d'un produit informatique n'excède pas quelques mois avant de devenir obsolète.