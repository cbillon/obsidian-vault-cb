---
tags:
  - linux
  - ufw
---
Le firewall est la première ligne de défense d'un serveur Linux. Ce guide complet explique UFW (simple) et iptables (avancé) pour sécuriser efficacement vos serveurs.

#### Comprendre les Firewalls Linux

##### Qu'est-ce qu'un Firewall ?

Un **firewall** (pare-feu) filtre le trafic réseau entrant et sortant selon des règles définies.

**Analogie :** Le firewall est comme un videur de boîte de nuit :

- Liste blanche : qui peut entrer (IPs autorisées)
- Liste noire : qui est banni (IPs bloquées)
- Règles : port SSH ouvert seulement pour admins

##### Architecture Firewall Linux

```
┌────────────────────────────────────────┐
│         Applications                    │
│     (nginx, ssh, mysql, etc.)          │
└────────────────────────────────────────┘
              ↕
┌────────────────────────────────────────┐
│          Userspace Tools                │
│  ┌─────────┐         ┌──────────┐      │
│  │   UFW   │         │ firewalld│      │
│  │ (simple)│         │ (RedHat) │      │
│  └─────────┘         └──────────┘      │
│       ↕                    ↕            │
│  ┌─────────────────────────────┐       │
│  │       iptables              │       │
│  │     (legacy CLI)            │       │
│  └─────────────────────────────┘       │
└────────────────────────────────────────┘
              ↕
┌────────────────────────────────────────┐
│         Kernel Space                    │
│  ┌─────────────────────────────┐       │
│  │    Netfilter (nftables)     │       │
│  │  Framework de filtrage      │       │
│  └─────────────────────────────┘       │
└────────────────────────────────────────┘
              ↕
        [Réseau Physique]
```

##### Trois Niveaux d'Outils

**1. UFW (Uncomplicated Firewall)**

- Interface simplifiée
- Idéal débutants
- Suffisant pour 90 % des cas

**2. iptables**

- Contrôle granulaire
- Syntaxe complexe
- Pour administrateurs avancés

**3. nftables**

- Remplaçant moderne d'iptables
- Syntaxe améliorée
- Kernel 3.13+ (2014)

---

#### UFW : Firewall Simplifié

##### Installation

```bash
# Debian/Ubuntu (souvent préinstallé)
sudo apt update
sudo apt install ufw

# Vérifier statut
sudo ufw status

# Activer au démarrage
sudo systemctl enable ufw
```

##### Commandes de Base

```bash
# Voir statut
sudo ufw status

# Activer firewall
sudo ufw enable

# Désactiver firewall
sudo ufw disable

# Recharger règles
sudo ufw reload

# Reset (supprimer toutes règles)
sudo ufw reset
```

##### Politique par Défaut

**Concept :** Que faire du trafic non explicitement autorisé/bloqué ?

```bash
# Bloquer tout entrant, autoriser tout sortant (RECOMMANDÉ)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Bloquer tout (serveur très sécurisé)
sudo ufw default deny incoming
sudo ufw default deny outgoing

# Voir politique actuelle
sudo ufw status verbose
```

##### Autoriser Services Courants

```bash
# SSH (port 22)
sudo ufw allow ssh
# ou
sudo ufw allow 22

# HTTP (port 80)
sudo ufw allow http
# ou
sudo ufw allow 80

# HTTPS (port 443)
sudo ufw allow https
# ou
sudo ufw allow 443

# Combiner plusieurs
sudo ufw allow 80,443/tcp

# MySQL (port 3306)
sudo ufw allow 3306

# PostgreSQL (port 5432)
sudo ufw allow 5432
```

##### Règles Avancées UFW

**Spécifier protocole :**

```bash
# TCP uniquement
sudo ufw allow 22/tcp

# UDP uniquement
sudo ufw allow 53/udp

# Les deux
sudo ufw allow 80/tcp
sudo ufw allow 80/udp
```

**Plages de ports :**

```bash
# Ports 1000 à 2000
sudo ufw allow 1000:2000/tcp

# Utile pour FTP passif
sudo ufw allow 30000:31000/tcp
```

**Depuis une IP spécifique :**

```bash
# Autoriser SSH depuis IP admin
sudo ufw allow from 203.0.113.50 to any port 22

# Autoriser tout depuis IP de confiance
sudo ufw allow from 203.0.113.100

# Autoriser depuis subnet
sudo ufw allow from 192.168.1.0/24 to any port 22
```

**Vers une interface spécifique :**

```bash
# Autoriser SSH seulement sur eth0
sudo ufw allow in on eth0 to any port 22

# Bloquer trafic externe sur interface interne
sudo ufw deny in on eth1
```

**Applications prédéfinies :**

```bash
# Lister applications disponibles
sudo ufw app list

# Nginx Full (HTTP + HTTPS)
sudo ufw allow 'Nginx Full'

# OpenSSH
sudo ufw allow 'OpenSSH'

# Apache Full
sudo ufw allow 'Apache Full'

# Voir ports d'une app
sudo ufw app info 'Nginx Full'
```

##### Bloquer et Supprimer

```bash
# Bloquer port
sudo ufw deny 3306

# Bloquer IP
sudo ufw deny from 203.0.113.99

# Supprimer règle par numéro
sudo ufw status numbered
sudo ufw delete 2

# Supprimer règle par description
sudo ufw delete allow 80/tcp

# Supprimer règle complexe
sudo ufw delete allow from 203.0.113.50 to any port 22
```

##### Logging

```bash
# Activer logs
sudo ufw logging on

# Niveaux : off, low, medium, high, full
sudo ufw logging medium

# Voir logs
sudo tail -f /var/log/ufw.log

# Exemple log bloqué
[UFW BLOCK] IN=eth0 OUT= MAC=... SRC=203.0.113.99 DST=203.0.113.1 PROTO=TCP DPT=22
```

---

#### Configuration Serveur Web avec UFW

##### Exemple : Serveur LEMP

```bash
# Reset pour commencer propre
sudo ufw reset

# Politique par défaut
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH (CRITIQUE : autoriser avant d'activer!)
sudo ufw allow 22/tcp

# Web
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Activer
sudo ufw enable

# Vérifier
sudo ufw status numbered
```

##### Exemple : Serveur SSH Sécurisé

```bash
# Autoriser SSH seulement depuis IPs admins
sudo ufw allow from 203.0.113.50 to any port 22
sudo ufw allow from 203.0.113.51 to any port 22

# Limiter tentatives SSH (rate limiting)
sudo ufw limit 22/tcp
# Bloque IP après 6 tentatives en 30 secondes

# Bloquer tout autre SSH
sudo ufw deny 22/tcp
```

##### Exemple : Serveur Base de Données

```bash
# MySQL accessible seulement depuis serveur web
sudo ufw allow from 192.168.1.10 to any port 3306

# PostgreSQL depuis subnet application
sudo ufw allow from 10.0.1.0/24 to any port 5432
```

---

#### iptables : Contrôle Avancé

##### Concepts Fondamentaux

**Tables :**

- **filter** : Filtrage paquets (défaut)
- **nat** : Translation d'adresses (NAT)
- **mangle** : Modification paquets
- **raw** : Exceptions au suivi de connexion

**Chains (chaînes) :**

- **INPUT** : Trafic entrant
- **OUTPUT** : Trafic sortant
- **FORWARD** : Trafic traversant (routeur)

**Actions (targets) :**

- **ACCEPT** : Accepter
- **DROP** : Rejeter silencieusement
- **REJECT** : Rejeter avec message
- **LOG** : Logger

##### Syntaxe iptables de Base

```bash
iptables -A CHAIN -p PROTOCOLE --dport PORT -j ACTION

# -A : Append (ajouter)
# -I : Insert (insérer en début)
# -D : Delete (supprimer)
# -p : Protocol (tcp, udp, icmp)
# --dport : Destination port
# --sport : Source port
# -s : Source IP
# -d : Destination IP
# -i : Interface entrante
# -o : Interface sortante
# -j : Jump to target (action)
```

##### Commandes iptables Essentielles

```bash
# Lister toutes règles
sudo iptables -L -v -n

# Lister avec numéros
sudo iptables -L --line-numbers

# Lister table nat
sudo iptables -t nat -L -v -n

# Supprimer toutes règles (DANGER)
sudo iptables -F

# Supprimer règle spécifique
sudo iptables -D INPUT 3

# Politique par défaut
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
```

##### Exemples iptables Basiques

```bash
# Autoriser SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Autoriser HTTP et HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Autoriser depuis IP spécifique
sudo iptables -A INPUT -s 203.0.113.50 -j ACCEPT

# Bloquer IP
sudo iptables -A INPUT -s 203.0.113.99 -j DROP

# Autoriser loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Autoriser connexions établies
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

##### Configuration iptables Complète Serveur Web

```bash
#!/bin/bash
# firewall-web.sh

# Flush règles existantes
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X

# Politique par défaut
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Loopback
iptables -A INPUT -i lo -j ACCEPT

# Connexions établies
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Ping (ICMP)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# DNS (si serveur DNS local)
# iptables -A INPUT -p udp --dport 53 -j ACCEPT

# Log paquets rejetés
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables-dropped: "

# Sauvegarder règles
iptables-save > /etc/iptables/rules.v4
```

##### Sauvegarder et Restaurer Règles

```bash
# Sauvegarder
sudo iptables-save > /etc/iptables/rules.v4
sudo ip6tables-save > /etc/iptables/rules.v6

# Restaurer
sudo iptables-restore < /etc/iptables/rules.v4

# Automatiser au démarrage (Debian/Ubuntu)
sudo apt install iptables-persistent

# Règles sauvegardées dans
/etc/iptables/rules.v4
/etc/iptables/rules.v6

# Update règles persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

---

#### Cas d'Usage Avancés

##### Rate Limiting (Protection DDoS)

**UFW :**

```bash
# Limiter SSH (6 connexions / 30 sec)
sudo ufw limit ssh
```

**iptables :**

```bash
# Limiter SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP

# Limiter HTTP (20 req/sec par IP)
iptables -A INPUT -p tcp --dport 80 -m limit --limit 20/sec --limit-burst 100 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j DROP
```

##### Port Knocking

Séquence secrète de ports pour ouvrir SSH. Pour une approche dédiée, consultez le guide [port knocking](https://www.shpv.fr/blog/ssh-port-knocking/) :

```bash
# Port knocking avec iptables
# Frapper ports 7000, 8000, 9000 dans l'ordre ouvre SSH pendant 30s

iptables -N KNOCKING
iptables -N GATE1
iptables -N GATE2
iptables -N GATE3

# État initial
iptables -A INPUT -p tcp --dport 7000 -m recent --name knock1 --set -j DROP
iptables -A INPUT -p tcp --dport 8000 -m recent --name knock1 --rcheck -m recent --name knock2 --set -j DROP
iptables -A INPUT -p tcp --dport 9000 -m recent --name knock2 --rcheck -m recent --name knock3 --set -j DROP

# SSH ouvert si séquence complète
iptables -A INPUT -p tcp --dport 22 -m recent --rcheck --seconds 30 --name knock3 -j ACCEPT

# Alternative simple : knockd daemon
sudo apt install knockd
```

##### NAT et Port Forwarding

**Activer forwarding :**

```bash
# Temporaire
sudo sysctl -w net.ipv4.ip_forward=1

# Permanent
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

**NAT (Masquerading) :**

```bash
# Partager connexion Internet (routeur)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

**Port Forwarding :**

```bash
# Rediriger port 8080 externe vers 80 interne
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80

# Forward vers autre machine
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:80
iptables -A FORWARD -p tcp -d 192.168.1.10 --dport 80 -j ACCEPT
```

##### Bloquer Pays (GeoIP)

```bash
# Installer module geoip
sudo apt install xtables-addons-common libtext-csv-xs-perl

# Télécharger base GeoIP
sudo mkdir -p /usr/share/xt_geoip
cd /usr/share/xt_geoip
sudo wget https://www.ipdeny.com/ipblocks/data/countries/all-zones.tar.gz
sudo tar -xzf all-zones.tar.gz

# Bloquer Chine et Russie (exemple)
iptables -A INPUT -m geoip --src-cc CN,RU -j DROP

# Autoriser seulement France
iptables -A INPUT -m geoip ! --src-cc FR -j DROP
```

---

#### UFW vs iptables : Comparaison

|Feature|UFW|iptables|
|---|---|---|
|Facilité|⭐⭐⭐⭐⭐|⭐⭐|
|Puissance|⭐⭐⭐|⭐⭐⭐⭐⭐|
|Syntaxe|Simple|Complexe|
|Learning curve|10 minutes|Plusieurs heures|
|Use case|Serveur web basique|Configurations avancées|
|Recommandé pour|Débutants, VPS simple|Experts, routeurs, NAT|

**Quand utiliser UFW :**

- Serveur web/application simple
- Premier firewall
- Configuration rapide
- Maintenance facile

**Quand utiliser iptables :**

- Règles complexes
- NAT/Routing
- Rate limiting avancé
- Contrôle granulaire

---

#### Debugging et Troubleshooting

##### UFW ne fonctionne pas

```bash
# 1. Vérifier service actif
sudo systemctl status ufw

# 2. Vérifier règles
sudo ufw status verbose

# 3. Voir logs
sudo tail -f /var/log/ufw.log

# 4. Tester port ouvert
sudo netstat -tlnp | grep :80
sudo ss -tlnp | grep :80

# 5. Tester depuis externe
# Sur autre machine
telnet SERVER_IP 80
nmap SERVER_IP
```

##### Bloqué hors SSH

**Prévention :**

```bash
# Toujours tester AVANT d'activer firewall
sudo ufw allow 22
sudo ufw enable

# Ou utiliser at pour désactiver auto
echo "ufw disable" | sudo at now + 5 minutes
# Si connexion perdue, firewall désactivé après 5min
```

**Solution (accès physique/console requis) :**

```bash
# Console serveur ou interface web hébergeur
sudo ufw disable
sudo ufw allow 22
sudo ufw enable
```

##### Ports semblent fermés malgré règles

```bash
# 1. Vérifier service écoute
sudo netstat -tlnp
sudo ss -tlnp

# 2. Vérifier ordre règles (iptables)
sudo iptables -L -n --line-numbers
# Règle DENY avant ALLOW = problème

# 3. Vérifier politique défaut
sudo ufw status verbose
sudo iptables -L -v

# 4. Logs firewall
sudo tail -f /var/log/ufw.log
sudo dmesg | grep -i iptables
```

##### Tester Règles Firewall

```bash
# Depuis serveur (test sortant)
curl -I https://google.com
ping 8.8.8.8

# Depuis externe (test entrant)
nmap -p 22,80,443 SERVER_IP

# Test spécifique
nc -zv SERVER_IP 80
telnet SERVER_IP 443

# Scanner complet
nmap -sV -p- SERVER_IP
```

---

#### Sécurité et Bonnes Pratiques

##### Règles de Base

✓ **Toujours autoriser SSH AVANT d'activer firewall**

```bash
sudo ufw allow 22
sudo ufw enable
```

✓ **Principe du moindre privilège**

```bash
# Bloquer par défaut, ouvrir le strict nécessaire
sudo ufw default deny incoming
```

✓ **Rate limiting sur services publics**

```bash
sudo ufw limit ssh
```

✓ **Logger les connexions suspectes**

```bash
sudo ufw logging medium
```

✓ **Backup règles avant modifications**

```bash
sudo iptables-save > /root/iptables-backup-$(date +%Y%m%d).rules
```

##### Erreurs Courantes

❌ **Activer firewall sans autoriser SSH**

```bash
# MAUVAIS
sudo ufw enable  # Bloqué!

# BON
sudo ufw allow 22
sudo ufw enable
```

❌ **Ordre règles iptables**

```bash
# MAUVAIS (deny avant allow)
iptables -A INPUT -j DROP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT  # Jamais atteint!

# BON
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP
```

❌ **Oublier connexions établies**

```bash
# MAUVAIS (bloque réponses)
iptables -P INPUT DROP

# BON
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

##### Audit Firewall

```bash
#!/bin/bash
# firewall-audit.sh

echo "=== Firewall Audit ==="

# UFW
if command -v ufw > /dev/null; then
    echo "UFW Status:"
    sudo ufw status verbose
fi

# iptables
echo ""
echo "iptables Rules:"
sudo iptables -L -v -n

# Ports ouverts
echo ""
echo "Listening Ports:"
sudo netstat -tlnp

# Services exposés
echo ""
echo "Exposed Services:"
sudo nmap -sV localhost
```

---

#### Scripts Utiles

##### Script UFW Configuration Serveur Web

```bash
#!/bin/bash
# setup-ufw-web.sh

# Couleurs
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'

echo -e "${GREEN}Configuration UFW pour serveur web${NC}"

# Vérifier UFW installé
if ! command -v ufw > /dev/null; then
    echo "Installation UFW..."
    sudo apt update
    sudo apt install -y ufw
fi

# Reset
read -p "Reset UFW? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    sudo ufw --force reset
fi

# Politique
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH
echo -e "${GREEN}Autoriser SSH...${NC}"
sudo ufw allow 22/tcp

# Web
echo -e "${GREEN}Autoriser HTTP/HTTPS...${NC}"
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Rate limiting SSH
sudo ufw limit 22/tcp

# Activer logging
sudo ufw logging medium

# Activer
sudo ufw --force enable

# Status
echo -e "${GREEN}Configuration terminée!${NC}"
sudo ufw status verbose
```

---

#### Conclusion

Le firewall est essentiel pour sécuriser un serveur Linux. Points clés :

**Pour débutants : UFW**

```bash
sudo ufw allow 22
sudo ufw allow 80,443/tcp
sudo ufw enable
```

**Pour experts : iptables**

```bash
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -P INPUT DROP
```

**Bonnes pratiques :**

- ✓ Autoriser SSH avant activation (voir [sécuriser SSH](https://www.shpv.fr/blog/ssh-securisation/))
- ✓ Bloquer par défaut (deny incoming)
- ✓ Rate limiting services publics
- ✓ Logger activité suspecte
- ✓ Tester règles régulièrement

**Compléments recommandés :**

- [nftables avancé](https://www.shpv.fr/blog/nftables-advanced/) pour une granularité maximale
- [Fail2ban](https://www.shpv.fr/blog/fail2ban/) pour le blocage automatique d'IPs malveillantes
- [CrowdSec](https://www.shpv.fr/blog/crowdsec-2026/) pour une protection collaborative

**Ressources :**

- `man ufw`
- `man iptables`
- [https://help.ubuntu.com/community/UFW](https://help.ubuntu.com/community/UFW)

Avec ce guide, vous pouvez sécuriser efficacement n'importe quel serveur Linux !