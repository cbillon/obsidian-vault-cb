---
tags:
  - linux
  - network
  - tcpdump
---
Un problème réseau peut bloquer toute une infrastructure. Savoir utiliser tcpdump, ss et traceroute permet de diagnostiquer rapidement : port fermé, firewall qui bloque, route cassée, ou latence excessive.

#### Plan

- tcpdump : capturer et analyser les paquets
- ss : analyser les connexions actives
- traceroute et mtr : diagnostic de routage
- netstat (legacy) et alternatives
- Troubleshooting courants
- Conclusion

---

#### tcpdump : capturer les paquets

##### Installation

```bash
apt install tcpdump  # Debian/Ubuntu
dnf install tcpdump  # RHEL/Fedora
```

##### Captures basiques

```bash
# Toutes les interfaces
tcpdump -i any

# Interface spécifique
tcpdump -i eth0

# Limiter à 100 paquets
tcpdump -c 100

# Sauvegarder dans un fichier
tcpdump -w capture.pcap

# Lire un fichier
tcpdump -r capture.pcap
```

##### Filtres essentiels

```bash
# Port spécifique
tcpdump port 80
tcpdump port 443

# Hôte spécifique
tcpdump host 192.168.1.10

# Réseau
tcpdump net 192.168.1.0/24

# Protocole
tcpdump tcp
tcpdump udp
tcpdump icmp

# Combinaisons
tcpdump 'tcp port 80 and host 10.0.0.5'
tcpdump 'udp port 53 or tcp port 443'
```

##### Exemples pratiques

```bash
# Capturer trafic HTTP
tcpdump -i eth0 'tcp port 80' -A

# Capturer SYN (connexions TCP)
tcpdump 'tcp[tcpflags] & tcp-syn != 0'

# Capturer DNS
tcpdump -i any port 53 -vv

# Exclure SSH (pour ne pas polluer)
tcpdump -i eth0 not port 22
```

---

#### ss : analyser les connexions

##### Remplaçant moderne de netstat

```bash
# Toutes les connexions
ss -a

# TCP uniquement
ss -t

# UDP uniquement
ss -u

# Listening ports
ss -l

# TCP listening
ss -tl

# Avec numéros de port (pas de résolution DNS)
ss -tn
```

##### Options utiles

```bash
# Afficher PID/programme
ss -tp

# Statistiques
ss -s

# Connexions établies
ss -t state established

# Connexions en attente (TIME_WAIT)
ss -t state time-wait
```

##### Exemples pratiques

```bash
# Qui écoute sur le port 80 ?
ss -tlnp | grep :80

# Connexions vers un hôte
ss -tn dst 192.168.1.10

# Connexions depuis un hôte
ss -tn src 192.168.1.5

# Compter connexions par état
ss -tan | awk '{print $1}' | sort | uniq -c
```

---

#### traceroute et mtr

##### traceroute : tracer la route

```bash
# Installation
apt install traceroute

# Trace vers une destination
traceroute example.com

# TCP mode (plus fiable)
traceroute -T example.com

# Nombre de sauts max
traceroute -m 15 example.com
```

##### mtr : traceroute en continu

```bash
# Installation
apt install mtr

# Mode interactif
mtr example.com

# Mode rapport (10 paquets)
mtr -r -c 10 example.com

# TCP mode
mtr -T example.com
```

**Interpréter les résultats** :

- `???` ou timeout : firewall ou filtrage ICMP
- Perte de paquets élevée : saturation ou mauvaise route
- Latence croissante : goulot d'étranglement

---

#### netstat (legacy)

##### Équivalences ss ↔ netstat

```bash
# Toutes connexions
netstat -a     →  ss -a

# TCP listening
netstat -tl    →  ss -tl

# Avec programmes
netstat -tlnp  →  ss -tlnp

# Statistiques
netstat -s     →  ss -s

# Routes
netstat -r     →  ip route
```

**Recommandation** : Utiliser `ss` plutôt que `netstat` (plus rapide, plus moderne).

---

#### Troubleshooting courants

##### Problème : "Connection refused"

```bash
# Vérifier si le port écoute
ss -tlnp | grep :80

# Si rien : service non démarré
systemctl status nginx

# Si présent : firewall bloque ?
iptables -L -n | grep 80
nft list ruleset | grep 80
```

##### Problème : "No route to host"

```bash
# Vérifier la route
ip route get 192.168.1.10

# Vérifier ARP (même réseau)
ip neigh show

# Ping
ping 192.168.1.10

# Traceroute
traceroute 192.168.1.10
```

##### Problème : Lenteur réseau

```bash
# Mesurer latence
ping -c 100 example.com

# MTR pour identifier le hop lent
mtr -r example.com

# Vérifier la bande passante
iperf3 -c server_ip
```

##### Problème : DNS ne résout pas

```bash
# Tester résolution
dig example.com

# Vérifier /etc/resolv.conf
cat /etc/resolv.conf

# Tester serveur DNS
dig @8.8.8.8 example.com
```

---

#### Ressources complémentaires

- Approfondissez votre compréhension du [réseau Linux](https://www.shpv.fr/blog/reseau-linux-2026/)
- Diagnostiquez des problèmes DNS spécifiques avec [le dépannage DNS](https://www.shpv.fr/blog/dns-troubleshooting/)
- Mettez en place une [stack de monitoring](https://www.shpv.fr/blog/deployer-stack-monitoring/) pour surveiller votre réseau en continu

#### Conclusion

tcpdump capture les paquets pour analyse fine. ss remplace netstat efficacement. traceroute et mtr diagnostiquent les problèmes de routage. Ces outils suffisent pour résoudre 90 % des problèmes réseau.