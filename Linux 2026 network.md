---
tags:
  - linux
  - network
---
La maîtrise du réseau Linux est essentielle pour tout administrateur système. Ce guide complet couvre les commandes ip, ss, netstat, traceroute et les techniques de diagnostic réseau.

#### Comprendre le Réseau Linux

##### Stack Réseau Linux

```
Application (HTTP, SSH, DNS)
    ↓
Transport (TCP, UDP)
    ↓
Network (IP, ICMP)
    ↓
Link (Ethernet, WiFi)
    ↓
Physical (Câbles, ondes)
```

##### Interfaces Réseau

```bash
# Lister interfaces
ip link show

# Noms courants :
# lo       - Loopback (127.0.0.1)
# eth0     - Ethernet physique
# ens33    - Ethernet (nouveau naming)
# wlan0    - WiFi
# docker0  - Bridge Docker
# tun0     - VPN tunnel
```

---

#### Commande ip : Outil Moderne

##### ip - Remplace ifconfig

```bash
# Voir toutes interfaces
ip addr show
# Raccourci
ip a

# Interface spécifique
ip addr show eth0

# Seulement IPv4
ip -4 addr

# Seulement IPv6
ip -6 addr

# Format coloré
ip -c addr
```

##### Configurer IP

```bash
# Ajouter IP
sudo ip addr add 192.168.1.100/24 dev eth0

# Supprimer IP
sudo ip addr del 192.168.1.100/24 dev eth0

# Activer interface
sudo ip link set eth0 up

# Désactiver interface
sudo ip link set eth0 down

# Changer MAC address
sudo ip link set eth0 down
sudo ip link set eth0 address 00:11:22:33:44:55
sudo ip link set eth0 up
```

##### Routes

```bash
# Voir table routage
ip route show
# Ou
ip r

# Route par défaut (gateway)
ip route show default

# Ajouter route
sudo ip route add 10.0.0.0/24 via 192.168.1.1

# Route par défaut
sudo ip route add default via 192.168.1.1

# Supprimer route
sudo ip route del 10.0.0.0/24

# Route vers interface spécifique
sudo ip route add 10.0.0.0/24 dev eth0

# Voir route vers IP
ip route get 8.8.8.8
```

##### Statistiques Interfaces

```bash
# Stats détaillées
ip -s link

# Stats étendues
ip -s -s link

# Watch temps réel
watch -n 1 'ip -s link show eth0'
```

---

#### ss : Socket Statistics

##### ss - Remplace netstat

```bash
# Toutes connexions
ss

# Sockets TCP
ss -t

# Sockets UDP
ss -u

# Listening sockets
ss -l

# TCP listening
ss -tl

# Avec numéros ports (pas résolution noms)
ss -tln

# Processus utilisant socket
ss -tlnp

# Tout (TCP + UDP, listening + established)
ss -tunap
```

##### Filtrer Connexions

```bash
# Port spécifique
ss -tln sport = :80
ss -tln dport = :443

# Plage ports
ss -tln sport \>= :8000 and sport \<= :9000

# État connexion
ss state established
ss state time-wait

# Par IP
ss dst 192.168.1.100
ss src 10.0.0.0/8

# Combinaisons
ss -tn state established '( dport = :80 or dport = :443 )'
```

##### Statistiques Réseau

```bash
# Résumé connexions
ss -s

# Mémoire utilisée sockets
ss -m

# Timer info
ss -o

# Extended info
ss -e

# Avec processus
ss -p
```

---

#### netstat : Outil Classique

##### Commandes netstat

```bash
# Installer si absent
sudo apt install net-tools

# Toutes connexions
netstat -a

# TCP listening
netstat -tln

# UDP listening
netstat -uln

# Avec processus
netstat -tlnp

# Stats interfaces
netstat -i

# Table routage
netstat -r

# Stats protocoles
netstat -s
```

##### Monitoring Connexions

```bash
# Connexions actives
netstat -tn

# Compter états TCP
netstat -an | grep ESTABLISHED | wc -l

# Top IPs connectées
netstat -tn | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -rn | head

# Watch connexions temps réel
watch -n 1 'netstat -tn'
```

---

#### ping : Test Connectivité

##### Utilisation ping

```bash
# Ping basique
ping google.com

# Limiter nombre paquets
ping -c 4 google.com

# Intervalle (secondes)
ping -i 0.5 google.com

# Taille paquet
ping -s 1000 google.com

# Flood ping (root, tests performance)
sudo ping -f google.com

# IPv6
ping6 google.com

# Interface spécifique
ping -I eth0 192.168.1.1

# Timeout
ping -W 2 192.168.1.100
```

##### Diagnostic avec ping

```bash
# Test connectivité locale
ping 127.0.0.1        # Loopback OK ?
ping 192.168.1.1      # Gateway OK ?
ping 8.8.8.8          # Internet OK ?
ping google.com       # DNS OK ?

# Latence
ping -c 100 server.com | tail -1
# rtt min/avg/max = X/Y/Z ms

# Packet loss
ping -c 100 -i 0.2 server.com | grep loss
# 5% packet loss
```

---

#### traceroute : Tracer Route

##### Utilisation traceroute

```bash
# Installer
sudo apt install traceroute

# Tracer route
traceroute google.com

# Avec numéros (pas résolution DNS)
traceroute -n google.com

# TCP traceroute (port 80)
sudo traceroute -T -p 80 google.com

# Limiter hops
traceroute -m 15 google.com

# Interface spécifique
traceroute -i eth0 192.168.1.1

# IPv6
traceroute6 google.com
```

##### Alternatives

```bash
# mtr (meilleur que traceroute)
sudo apt install mtr
mtr google.com

# Mode rapport
mtr -r -c 10 google.com

# tcptraceroute
sudo apt install tcptraceroute
sudo tcptraceroute google.com 443
```

---

#### nmap : Scanner Réseau

##### Installer nmap

```bash
sudo apt install nmap
```

##### Scans de Base

```bash
# Scanner hôte
nmap 192.168.1.1

# Scanner réseau
nmap 192.168.1.0/24

# Ports spécifiques
nmap -p 80,443 192.168.1.1

# Plage ports
nmap -p 1-1000 192.168.1.1

# Tous ports
nmap -p- 192.168.1.1

# Scan rapide (100 ports courants)
nmap -F 192.168.1.1

# Détecter OS
sudo nmap -O 192.168.1.1

# Détecter services/versions
nmap -sV 192.168.1.1

# Scan agressif (OS + version + scripts)
sudo nmap -A 192.168.1.1
```

##### Types Scans

```bash
# TCP SYN scan (défaut, nécessite root)
sudo nmap -sS 192.168.1.1

# TCP connect scan (pas root)
nmap -sT 192.168.1.1

# UDP scan
sudo nmap -sU 192.168.1.1

# Ping scan (découvrir hôtes actifs)
nmap -sn 192.168.1.0/24

# Pas de ping (scan même si hôte semble down)
nmap -Pn 192.168.1.1
```

##### Exemples Pratiques

```bash
# Trouver serveurs web réseau
nmap -p 80,443 --open 192.168.1.0/24

# Scan vulnérabilités
nmap --script vuln 192.168.1.1

# Scan stealthy (lent, discret)
sudo nmap -sS -T2 192.168.1.1

# Scan rapide
nmap -T4 -F 192.168.1.1
```

---

#### tcpdump : Capture Paquets

Pour une [observabilité réseau avancée avec eBPF](https://www.shpv.fr/blog/ebpf-revolution/), explorez nos techniques modernes.

##### Installer tcpdump

```bash
sudo apt install tcpdump
```

##### Captures de Base

```bash
# Capturer sur interface
sudo tcpdump -i eth0

# Limiter nombre paquets
sudo tcpdump -c 10

# Verbose
sudo tcpdump -v
sudo tcpdump -vv
sudo tcpdump -vvv

# Voir contenu ASCII
sudo tcpdump -A

# Voir contenu hexadécimal
sudo tcpdump -X

# Sans résolution DNS
sudo tcpdump -n

# Sans résolution ports
sudo tcpdump -nn
```

##### Filtres

```bash
# Par hôte
sudo tcpdump host 192.168.1.100

# Par réseau
sudo tcpdump net 192.168.1.0/24

# Par port
sudo tcpdump port 80
sudo tcpdump portrange 8000-9000

# Par protocole
sudo tcpdump tcp
sudo tcpdump udp
sudo tcpdump icmp

# Source/destination
sudo tcpdump src 192.168.1.100
sudo tcpdump dst 8.8.8.8

# Combinaisons (AND, OR, NOT)
sudo tcpdump 'tcp and port 80'
sudo tcpdump 'host 192.168.1.1 and (port 80 or port 443)'
sudo tcpdump 'tcp and not port 22'
```

##### Sauvegarder Captures

```bash
# Sauvegarder dans fichier
sudo tcpdump -w capture.pcap

# Lire fichier
sudo tcpdump -r capture.pcap

# Filtrer lors lecture
sudo tcpdump -r capture.pcap port 80

# Limiter taille fichier (MB)
sudo tcpdump -w capture.pcap -C 100

# Rotation fichiers
sudo tcpdump -w capture.pcap -C 100 -W 5
```

##### Exemples Pratiques

```bash
# Capturer HTTP requests
sudo tcpdump -i eth0 -A 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# Capturer DNS queries
sudo tcpdump -i eth0 udp port 53

# Capturer SYN packets
sudo tcpdump 'tcp[tcpflags] & tcp-syn != 0'

# Monitorer connexions SSH
sudo tcpdump -i eth0 port 22

# Capturer ping
sudo tcpdump icmp
```

---

#### dig et nslookup : DNS

##### dig - DNS Lookup

```bash
# Installer
sudo apt install dnsutils

# Query simple
dig google.com

# Seulement réponse courte
dig +short google.com

# Type record spécifique
dig google.com A      # IPv4
dig google.com AAAA   # IPv6
dig google.com MX     # Mail servers
dig google.com NS     # Name servers
dig google.com TXT    # TXT records

# Serveur DNS spécifique
dig @8.8.8.8 google.com

# Reverse DNS
dig -x 8.8.8.8

# Trace query
dig +trace google.com

# Stats seulement
dig +stats google.com
```

##### nslookup

```bash
# Query simple
nslookup google.com

# Serveur DNS spécifique
nslookup google.com 8.8.8.8

# Mode interactif
nslookup
> server 8.8.8.8
> google.com
> exit
```

##### host

```bash
# Simple lookup
host google.com

# Tous records
host -a google.com

# Reverse lookup
host 8.8.8.8
```

---

#### curl et wget : HTTP

##### curl

```bash
# GET request
curl https://api.github.com

# Sauvegarder output
curl -o output.html https://example.com

# Suivre redirects
curl -L https://bit.ly/short

# Voir headers
curl -I https://example.com

# Verbose (debug)
curl -v https://example.com

# POST request
curl -X POST -d "data=value" https://api.example.com

# JSON POST
curl -X POST -H "Content-Type: application/json" \
     -d '{"key":"value"}' \
     https://api.example.com

# Authentification
curl -u user:password https://api.example.com

# Upload fichier
curl -F "file=@document.pdf" https://upload.example.com

# Timeout
curl --connect-timeout 5 --max-time 10 https://example.com

# Tester temps réponse
curl -w "@-" -o /dev/null -s https://example.com <<'EOF'
    time_total:  %{time_total}s
EOF
```

##### wget

```bash
# Télécharger fichier
wget https://example.com/file.zip

# Reprendre téléchargement interrompu
wget -c https://example.com/bigfile.iso

# Télécharger site complet
wget -r -np -k https://example.com

# Limiter vitesse
wget --limit-rate=500k https://example.com/file.zip

# Background
wget -b https://example.com/file.zip

# User agent custom
wget --user-agent="MyBot" https://example.com
```

---

#### Configuration Réseau Permanente

##### netplan (Ubuntu 18.04+)

```yaml
# /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      dhcp4: true
    eth1:
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
```

```bash
# Appliquer config
sudo netplan apply

# Tester sans appliquer
sudo netplan try

# Debug
sudo netplan --debug apply
```

##### NetworkManager

```bash
# CLI NetworkManager
nmcli

# Voir connexions
nmcli connection show

# Status device
nmcli device status

# Configurer IP statique
nmcli con mod eth0 ipv4.addresses 192.168.1.100/24
nmcli con mod eth0 ipv4.gateway 192.168.1.1
nmcli con mod eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod eth0 ipv4.method manual

# Appliquer
nmcli con up eth0

# DHCP
nmcli con mod eth0 ipv4.method auto
```

---

#### Troubleshooting Réseau

##### Diagnostic Complet

```bash
#!/bin/bash
# network-diag.sh

echo "=== Interfaces ==="
ip addr show

echo -e "\n=== Routes ==="
ip route show

echo -e "\n=== DNS ==="
cat /etc/resolv.conf

echo -e "\n=== Connexions ==="
ss -tunap

echo -e "\n=== Test connectivité ==="
ping -c 3 8.8.8.8

echo -e "\n=== Test DNS ==="
dig google.com +short

echo -e "\n=== Ports listening ==="
ss -tlnp
```

##### Problèmes Courants

**Pas de connexion Internet :**

```bash
# 1. Interface up ?
ip link show eth0
sudo ip link set eth0 up

# 2. IP configurée ?
ip addr show eth0

# 3. Gateway accessible ?
ping 192.168.1.1

# 4. DNS fonctionne ?
dig google.com

# 5. Internet accessible ?
ping 8.8.8.8
```

**DNS ne résout pas :**

```bash
# Vérifier /etc/resolv.conf
cat /etc/resolv.conf

# Tester DNS différent
dig @8.8.8.8 google.com

# Flush DNS cache (si systemd-resolved)
sudo systemd-resolve --flush-caches
```

**Port déjà utilisé :**

```bash
# Trouver qui utilise port
sudo ss -tlnp | grep :80
sudo lsof -i :80

# Tuer processus
sudo kill <PID>
```

---

#### Monitoring Réseau

##### iftop - Bande Passante Temps Réel

```bash
# Installer
sudo apt install iftop

# Monitorer interface
sudo iftop -i eth0

# Voir ports
sudo iftop -P

# Pas résolution DNS
sudo iftop -n
```

##### vnstat - Stats Long Terme

```bash
# Installer
sudo apt install vnstat

# Init database
sudo vnstat -u -i eth0

# Stats
vnstat

# Par heure
vnstat -h

# Par jour
vnstat -d

# Live
vnstat -l
```

##### nethogs - Par Processus

```bash
# Installer
sudo apt install nethogs

# Monitorer
sudo nethogs

# Interface spécifique
sudo nethogs eth0
```

---

#### Sécurité Réseau

##### Scan Ports Ouverts

```bash
# Voir ports listening localement
sudo ss -tlnp

# Scanner depuis externe
nmap -p- your-server.com

# Vérifier firewall
sudo iptables -L -n -v
sudo ufw status
```

##### Bloquer IP

```bash
# Temporaire (iptables)
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# Permanent (ufw)
sudo ufw deny from 192.168.1.100
```

---

#### Bonnes Pratiques

##### Checklist Réseau

```bash
□ Interfaces configurées correctement
□ Routes par défaut définies
□ DNS configurés (8.8.8.8, 1.1.1.1)
□ Firewall actif (ufw/iptables)
□ Seulement ports nécessaires ouverts
□ Monitoring bande passante actif
□ Logs réseau archivés
□ Backup configuration réseau
```

##### Scripts Monitoring

```bash
# Alerter si interface down
#!/bin/bash
if ! ip link show eth0 | grep -q "UP"; then
    echo "Interface eth0 DOWN!" | mail -s "Network Alert" admin@example.com
fi
```

---

#### Conclusion

Les commandes réseau Linux essentielles :

**Diagnostic :**

```bash
ip addr show              # Interfaces
ss -tunap                 # Connexions
ping 8.8.8.8             # Connectivité
traceroute google.com    # Route
dig google.com           # DNS
```

**Configuration :**

```bash
sudo ip addr add IP/24 dev eth0
sudo ip route add default via GATEWAY
sudo netplan apply
```

**Troubleshooting :**

```bash
1. ip link show          # Interface up ?
2. ip addr show          # IP configurée ?
3. ping gateway          # Gateway OK ?
4. ping 8.8.8.8         # Internet OK ?
5. dig google.com        # DNS OK ?
```

**Monitoring :**

```bash
iftop                    # Bande passante
nethogs                  # Par processus
vnstat                   # Long terme
```

Avec ces outils, vous maîtrisez diagnostic, configuration et troubleshooting réseau Linux !