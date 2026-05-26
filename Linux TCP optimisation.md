---
tags:
  - linux
  - network
---
Les paramètres réseau Linux par défaut ne sont pas optimisés pour les serveurs haute performance. Cet article détaille comment tuner TCP/IP : activation de BBR, ajustement des buffers, optimisation des paramètres kernel pour améliorer débit et latence.

#### Plan

- Comprendre les goulots d'étranglement TCP
- Algorithme de congestion BBR vs CUBIC
- Optimisation des buffers TCP
- Paramètres sysctl essentiels (voir [tuning kernel complet](https://www.shpv.fr/blog/kernel-linux-2026/))
- Tuning spécifique par cas d'usage
- Benchmarking et validation (voir [diagnostic réseau Linux](https://www.shpv.fr/blog/reseau-linux-2026/))
- Conclusion (voir [tuning performance général](https://www.shpv.fr/blog/linux-performance/))

---

#### Comprendre les goulots d'étranglement TCP

**Limitations courantes :**

1. **Algorithme de congestion obsolète** : CUBIC (défaut) n'est pas taillé pour les réseaux modernes
2. **Buffers sous-dimensionnés** : Limite le débit sur liens haute latence
3. **Queue disciplines inadaptées** : FIFO simple crée du bufferbloat
4. **Paramètres kernel conservateurs** : Conçus pour compatibilité, pas performance

**Impact mesurable :**

- Débit limité à 10-20 % de la capacité réelle
- Latence élevée sous charge (bufferbloat)
- Pertes de paquets inutiles
- Utilisation CPU excessive

---

#### Algorithme de congestion BBR vs CUBIC

##### Qu'est-ce que BBR ?

**BBR (Bottleneck Bandwidth and Round-trip propagation time)** est un algorithme de congestion TCP développé par Google qui :

- Mesure la bande passante disponible réelle
- Minimise la latence (pas de bufferbloat)
- Récupère rapidement après perte de paquets
- Fonctionne sur tous les types de réseaux (WiFi, cellulaire, satellite)

**CUBIC** (défaut Linux) :

- Basé sur les pertes de paquets comme signal de congestion
- Remplit les buffers → bufferbloat
- Lent à récupérer après perte

##### Activer BBR

```bash
# Vérifier la disponibilité (kernel 4.9+)
modprobe tcp_bbr
lsmod | grep bbr

# Activer temporairement
sysctl -w net.ipv4.tcp_congestion_control=bbr
sysctl -w net.core.default_qdisc=fq

# Activer de façon permanente
cat >> /etc/sysctl.d/99-bbr.conf << EOF
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
EOF

sysctl -p /etc/sysctl.d/99-bbr.conf

# Vérifier l'activation
sysctl net.ipv4.tcp_congestion_control
# net.ipv4.tcp_congestion_control = bbr

ss -ti | grep bbr
# bbr wscale:7,7 rto:204 rtt:3.5/1.75 ato:40 mss:1448 pmtu:1500 rcvmss:1448 advmss:1448 cwnd:10 bytes_sent:123 bytes_acked:123 bytes_received:456 segs_out:3 segs_in:4 data_segs_out:1 data_segs_in:1 send 33.1Mbps lastsnd:1234 lastrcv:1234 lastack:1234 pacing_rate 66.3Mbps delivery_rate 33.1Mbps delivered:1 app_limited busy:12ms rcv_space:14600 rcv_ssthresh:64088 minrtt:3.5 snd_wnd:14600
```

##### Comparaison de performance

**Test avec iperf3 :**

```bash
# Serveur
iperf3 -s

# Client CUBIC
sysctl -w net.ipv4.tcp_congestion_control=cubic
iperf3 -c server_ip -t 60 -P 4

# Client BBR
sysctl -w net.ipv4.tcp_congestion_control=bbr
iperf3 -c server_ip -t 60 -P 4
```

**Résultats typiques :**

- **Liens haute latence (>50ms)** : BBR +40-60 % débit
- **Liens avec pertes** : BBR +200-300 % débit
- **Latence sous charge** : BBR -50-70 % (réduction bufferbloat)

---

#### Optimisation des buffers TCP

##### Calcul de la taille optimale

**Formule :** `Buffer optimal = Bande passante × RTT`

Exemple : Lien 10 Gbps avec RTT de 20ms

```
10 Gbps = 1.25 GB/s
Buffer = 1.25 GB/s × 0.020s = 25 MB
```

##### Configuration des buffers

```bash
# /etc/sysctl.d/99-tcp-buffers.conf

# Buffers TCP de lecture (min, default, max en bytes)
net.ipv4.tcp_rmem = 4096 87380 33554432
# 4KB min, 85KB default, 32MB max

# Buffers TCP d'écriture
net.ipv4.tcp_wmem = 4096 65536 33554432
# 4KB min, 64KB default, 32MB max

# Buffer global max (pour tous les sockets)
net.core.rmem_max = 33554432  # 32 MB
net.core.wmem_max = 33554432  # 32 MB

# Buffer par défaut
net.core.rmem_default = 262144   # 256 KB
net.core.wmem_default = 262144   # 256 KB

# Activer l'auto-tuning TCP (important !)
net.ipv4.tcp_moderate_rcvbuf = 1
```

**Pour serveurs 10G+ :**

```bash
# Buffers encore plus larges
net.ipv4.tcp_rmem = 4096 87380 134217728  # 128 MB max
net.ipv4.tcp_wmem = 4096 65536 134217728  # 128 MB max
net.core.rmem_max = 134217728
net.core.wmem_max = 134217728
```

##### Optimisation des queues réseau

```bash
# Augmenter la queue du backlog (connexions en attente)
net.core.netdev_max_backlog = 5000

# SYN backlog (protection SYN flood)
net.ipv4.tcp_max_syn_backlog = 8192

# Socket listen backlog
net.core.somaxconn = 4096
```

---

#### Paramètres sysctl essentiels

##### Configuration complète haute performance

```bash
# /etc/sysctl.d/99-network-performance.conf

# === Algorithme de congestion ===
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

# === Buffers TCP ===
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.rmem_default = 262144
net.core.wmem_default = 262144

# === Queues ===
net.core.netdev_max_backlog = 10000
net.ipv4.tcp_max_syn_backlog = 8192
net.core.somaxconn = 4096

# === TCP Keepalive ===
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 60
net.ipv4.tcp_keepalive_probes = 3

# === Fast Open (TFO) ===
net.ipv4.tcp_fastopen = 3
# 1 = client, 2 = server, 3 = both

# === Réutilisation des sockets TIME_WAIT ===
net.ipv4.tcp_tw_reuse = 1

# === Window Scaling ===
net.ipv4.tcp_window_scaling = 1

# === SACK (Selective Acknowledgment) ===
net.ipv4.tcp_sack = 1

# === Timestamps TCP ===
net.ipv4.tcp_timestamps = 1

# === MTU Probing (évite black holes) ===
net.ipv4.tcp_mtu_probing = 1

# === Augmenter le nombre de connexions en tracking ===
net.netfilter.nf_conntrack_max = 262144

# === Timeouts connection tracking ===
net.netfilter.nf_conntrack_tcp_timeout_established = 1200

# === IPv6 si utilisé ===
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.default.forwarding = 0
```

Appliquer :

```bash
sysctl -p /etc/sysctl.d/99-network-performance.conf

# Vérifier
sysctl -a | grep -E 'tcp_congestion|tcp_rmem|tcp_wmem|bbr'
```

---

#### Tuning spécifique par cas d'usage

##### Serveur Web (Nginx/Apache)

```bash
# Optimisé pour nombreuses connexions courtes

# Réutiliser TIME_WAIT rapidement
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15

# Fast Open
net.ipv4.tcp_fastopen = 3

# Limite de connexions
net.core.somaxconn = 4096
net.ipv4.tcp_max_syn_backlog = 8192

# Keepalive court
net.ipv4.tcp_keepalive_time = 300
```

**Nginx :**

```nginx
# /etc/nginx/nginx.conf
events {
    worker_connections 4096;
    use epoll;
    multi_accept on;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 30;
    keepalive_requests 100;
}
```

##### Proxy/Load Balancer (HAProxy)

```bash
# Très nombreuses connexions simultanées

# Augmenter limites fichiers
fs.file-max = 2097152

# Connexions tracking
net.netfilter.nf_conntrack_max = 1048576

# Buffers larges
net.ipv4.tcp_rmem = 4096 87380 134217728
net.ipv4.tcp_wmem = 4096 65536 134217728

# TIME_WAIT agressif
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 10
```

**HAProxy limits :**

```bash
# /etc/security/limits.conf
haproxy soft nofile 1000000
haproxy hard nofile 1000000
```

##### Transfert de fichiers haute performance

```bash
# Optimisé pour gros transferts longue durée

# Buffers très larges
net.ipv4.tcp_rmem = 4096 262144 268435456  # 256 MB
net.ipv4.tcp_wmem = 4096 262144 268435456
net.core.rmem_max = 268435456
net.core.wmem_max = 268435456

# BBR obligatoire
net.ipv4.tcp_congestion_control = bbr

# Désactiver slow start après idle
net.ipv4.tcp_slow_start_after_idle = 0
```

##### VPN/Tunnel (Wireguard, OpenVPN)

```bash
# Optimisé pour overhead encapsulation

# MTU discovery
net.ipv4.tcp_mtu_probing = 1
net.ipv4.ip_no_pmtu_disc = 0

# Buffers moyens
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# BBR pour absorber la latence ajoutée
net.ipv4.tcp_congestion_control = bbr
```

---

#### Benchmarking et validation

##### Test de débit avec iperf3

```bash
# Serveur
iperf3 -s

# Client : test simple
iperf3 -c server_ip -t 60

# Test avec plusieurs flux parallèles
iperf3 -c server_ip -t 60 -P 10

# Test reverse (download)
iperf3 -c server_ip -t 60 -R

# Test avec fenêtre spécifique
iperf3 -c server_ip -t 60 -w 4M
```

##### Test de latence sous charge

```bash
# Terminal 1 : Générer du trafic
iperf3 -c server_ip -t 300

# Terminal 2 : Mesurer latence
ping -i 0.2 server_ip

# Résultats attendus avec BBR :
# - CUBIC : latence passe de 5ms à 200ms sous charge
# - BBR : latence reste 5-15ms sous charge
```

##### Monitoring en production

```bash
# Script de monitoring TCP
#!/bin/bash
# tcp-monitor.sh

echo "=== TCP Stats ==="
ss -s

echo -e "\n=== Connexions par état ==="
ss -tan | awk '{print $1}' | sort | uniq -c | sort -rn

echo -e "\n=== Algorithme de congestion ==="
sysctl net.ipv4.tcp_congestion_control

echo -e "\n=== Retransmissions ==="
netstat -s | grep -i retrans

echo -e "\n=== Drops ==="
netstat -s | grep -i drop
```

##### Outils de diagnostic avancés

```bash
# Installation
apt install ethtool nicstat

# Statistiques carte réseau
ethtool -S eth0

# Drops hardware
ethtool -S eth0 | grep -i drop

# Débit temps réel
nicstat 1

# TCP windows et RTT
ss -ti
```

---

#### Troubleshooting

##### Débit limité malgré BBR

**Vérifier :**

```bash
# 1. BBR vraiment actif ?
ss -ti | grep bbr

# 2. Buffers suffisants ?
sysctl net.ipv4.tcp_rmem
sysctl net.ipv4.tcp_wmem

# 3. MTU optimal ?
ip link show eth0 | grep mtu
# Tester jumbo frames si réseau le supporte
ip link set eth0 mtu 9000

# 4. TSO/GSO activés ?
ethtool -k eth0 | grep segmentation
ethtool -K eth0 tso on gso on
```

##### Bufferbloat persistant

```bash
# Test bufferbloat
# Terminal 1
iperf3 -c server_ip -t 300

# Terminal 2
ping server_ip -i 0.2

# Si latence explose, vérifier :
sysctl net.core.default_qdisc
# Doit être "fq" pas "pfifo_fast"

# Forcer FQ queue
tc qdisc replace dev eth0 root fq
```

##### Drops et retransmissions

```bash
# Statistiques détaillées
nstat -az | grep -i tcp

# Si beaucoup de retransmissions :
# 1. Vérifier MTU path
tracepath server_ip

# 2. Augmenter buffers
sysctl -w net.ipv4.tcp_rmem="4096 131072 67108864"

# 3. Vérifier erreurs matérielles
ethtool -S eth0 | grep err
```

---

#### Checklist d'optimisation

✅ **Basique (tous serveurs) :**

- [ ]  BBR activé
- [ ]  FQ queue discipline
- [ ]  Buffers TCP ajustés (min 32MB max)
- [ ]  TCP Fast Open
- [ ]  Window scaling, SACK, timestamps

✅ **Serveurs web :**

- [ ]  tcp_tw_reuse = 1
- [ ]  tcp_fin_timeout = 15
- [ ]  somaxconn = 4096
- [ ]  Nginx : worker_connections 4096

✅ **Haute performance :**

- [ ]  Buffers 128-256 MB
- [ ]  Jumbo frames (MTU 9000)
- [ ]  TSO/GSO activés
- [ ]  IRQ balancing
- [ ]  CPU affinity

✅ **Monitoring :**

- [ ]  iperf3 benchmarks
- [ ]  Latence sous charge
- [ ]  ss -s quotidien
- [ ]  Drops/retransmissions

---

#### Conclusion

L'optimisation TCP/IP sur Linux améliore significativement les performances réseau : BBR réduit la latence de 50 à 70 % et augmente le débit de 40 à 200 % selon les conditions. Les buffers correctement dimensionnés et les paramètres kernel adaptés au cas d'usage sont essentiels.

**Actions prioritaires :**

1. Activer BBR (kernel 4.9+)
2. Ajuster buffers TCP selon bande passante × RTT
3. Configurer FQ queue discipline
4. Activer TCP Fast Open
5. Benchmarker avant/après avec iperf3

**Gains mesurables :**

- Débit : +40 à +200 % selon le réseau
- Latence sous charge : -50 à -70 %
- Connexions simultanées : +300 %
- Retransmissions : -60 %