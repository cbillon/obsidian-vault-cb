---
tags:
---
Vous avez un firewall nftables qui fonctionne. Une règle par port, un `ct state established,related accept` en haut, quelques `drop` à la fin. Ça marche. Mais vous passez à côté de la vraie puissance du framework.

nftables n'est pas juste un remplacement d'iptables avec une syntaxe plus propre. C'est un moteur avec des structures de données intégrées, des sets qui font du lookup en O(1), des maps qui font du verdict routing, des meters qui comptent par IP au niveau du noyau. Tout ça sans log parsing, sans script externe, sans redémarrage.

Ce guide vous emmène directement sur les features qui changent la façon dont vous gérez un firewall en production. Si vous débutez, commencez par [UFW et iptables](https://www.shpv.fr/blog/ufw-iptables-2026/).

#### Prérequis

- Linux kernel >= 5.10 (idéalement 6.x+)
- nftables >= 1.0.0, vérifier avec `nft --version`
- Debian 12, Ubuntu 22.04+, RHEL 9, Rocky 9
- Accès root

#### Sets : des listes massives, des lookups O(1)

Le problème classique : vous avez 200 IPs à bloquer. Avec des règles individuelles, chaque paquet parcourt les 200 règles. Avec un set, nftables utilise une table de hachage en arrière-plan. Un seul lookup, peu importe le nombre d'éléments.

##### Set statique, liste noire simple

```bash
nft add table inet firewall
nft add set inet firewall blocked_ips { type ipv4_addr \; }
nft add element inet firewall blocked_ips { 203.0.113.1, 198.51.100.5, 192.0.2.10 }

nft add chain inet firewall input { type filter hook input priority 0 \; policy drop \; }
nft add rule inet firewall input ip saddr @blocked_ips counter drop
```

Une règle. Même si vous ajoutez 10 000 IPs dans le set, la performance ne change pas.

##### Set dynamique avec timeout, liste noire temporaire

C'est où nftables prend un avantage massif sur iptables. Un set dynamique permet d'ajouter des éléments depuis les règles elles-mêmes, avec une expiration automatique gérée par le noyau.

```bash
# Créer le set : dynamique, avec timeout de 10 minutes
nft add set inet firewall temp_ban {
    type ipv4_addr \;
    flags dynamic, timeout \;
    timeout 10m \;
}

# Règle qui ajoute une IP dans temp_ban si elle dépasse le quota SSH
nft add rule inet firewall input \
    tcp dport 22 \
    ct state new \
    meter ssh_rate { ip saddr limit rate over 5/minute burst 3 packets } \
    add @temp_ban { ip saddr timeout 10m } \
    counter log prefix "SSH-BANNED: " drop

# Règle qui bloque tout ce qui est dans temp_ban — À METTRE AVANT les autres règles
nft add rule inet firewall input \
    ip saddr @temp_ban \
    counter drop
```

L'IP entre dans `temp_ban`, reste bloquée 10 minutes, puis disparaît automatiquement. Vérifier en temps réel :

```bash
nft list set inet firewall temp_ban
# table inet firewall {
#     set temp_ban {
#         type ipv4_addr
#         flags dynamic,timeout
#         timeout 10m
#         elements = { 203.0.113.42 expires 7m23s,
#                      198.51.100.7 expires 2m51s }
#     }
# }
```

##### Set d'intervalle, bloquer des plages CIDR

Pour bloquer ou autoriser des plages entières (un AS, un datacenter, votre réseau interne) :

```bash
nft add set inet firewall admin_ranges {
    type ipv4_addr \;
    flags interval \;
    elements = { 45.80.0.0/22, 185.212.224.0/22 } \;
}

nft add rule inet firewall input \
    ip saddr @admin_ranges \
    tcp dport 22 \
    ct state new \
    counter accept
```

#### Verdict Maps : du routing intelligent

Un set vous dit oui ou non. Une verdict map vous dit : pour cette valeur d'entrée, faire cette action. C'est la différence entre un filtre et un cerveau.

##### Verdict map par IP, politiques différentiées

Vous voulez que certaines IPs puissent tout faire, d'autres n'ont droit qu'au web, et le reste est bloqué :

```bash
# Créer une chain pour le trafic "web seulement"
nft add chain inet firewall web_only { \; }
nft add rule inet firewall web_only tcp dport { 80, 443 } accept
nft add rule inet firewall web_only drop

# La verdict map
nft add map inet firewall ip_policy {
    type ipv4_addr : verdict \;
    elements = {
        10.0.0.1 : accept,
        10.0.0.2 : accept,
        203.0.113.50 : jump web_only,
        198.51.100.1 : drop
    } \;
}

# Une seule règle qui applique la map
nft add rule inet firewall input ip saddr @ip_policy
```

Le noyau fait le lookup dans la map et exécute le verdict directement. Zéro chaîne intermédiaire pour les cas courants.

##### Map pour du DNAT multi-service

Vous gérez plusieurs services sur des ports internes, exposés sur des ports extérieurs configurables :

```bash
nft add map inet firewall dnat_map {
    type inet_service : inet_service \;
    elements = {
        2222 : 22,
        8443 : 443,
        9090 : 3000
    } \;
}

nft add chain inet firewall prerouting { type nat hook prerouting priority dstnat \; }
nft add rule inet firewall prerouting \
    tcp dport @dnat_map \
    dnat to 127.0.0.1:tcp dport map @dnat_map
```

Ajouter un nouveau port forwarding sans toucher aux règles :

```bash
nft add element inet firewall dnat_map { 8080 : 80 }
```

Atomique. Pas de rechargement.

#### Conntrack avancé : au-delà de `ct state established`

##### Conntrack zones, isoler les flux par interface

Par défaut, toutes les interfaces partagent la même table conntrack. Sur un serveur avec une DMZ, un LAN interne, et un VPN, les connexions peuvent se croiser et créer des failles. Les zones isolent le tracking par interface.

```bash
# Chains avec priorité raw (-300) — AVANT que conntrack ne prenne la main (-200)
nft add table inet firewall
nft add chain inet firewall zone_pre { type filter hook prerouting priority raw \; }
nft add chain inet firewall zone_out { type filter hook output priority raw \; }

# Interface DMZ (eth1) → zone 1
nft add rule inet firewall zone_pre iifname eth1 ct zone set 1
nft add rule inet firewall zone_out oifname eth1 ct zone set 1
```

Une connexion initiée depuis le LAN ne sera jamais considérée comme `established` dans la DMZ. Les zones sont séparées au niveau du noyau.

##### Conntrack marks, tagger des connexions

Les marks permettent de marquer une connexion et de la traiter différemment plus loin dans la chaîne, sans avoir à recalculer les conditions :

```bash
# Marquer les nouvelles connexions vers le port 443 avec le mark 0x01
nft add rule inet firewall input \
    tcp dport 443 \
    ct state new \
    ct mark set 0x01

# Plus loin dans la chaîne, traiter les connexions marquées différemment
nft add rule inet firewall input \
    ct mark 0x01 \
    counter accept
```

##### notrack, court-circuiter le conntrack

Le conntrack consomme de la mémoire et du CPU pour chaque connexion. Pour du trafic que vous connaissez parfaitement et qui ne nécessite jamais de filtrage stateful (un DNS sur un réseau fermé, par exemple) vous pouvez le désactiver :

```bash
# Chain avec priorité raw, avant le hook conntrack
nft add chain inet firewall notrack_chain { type filter hook prerouting priority raw \; }

# Ne pas tracker le DNS vers notre résolveur interne
nft add rule inet firewall notrack_chain \
    ip daddr 10.0.0.1 \
    udp dport 53 \
    notrack
```

**Attention** : la priorité doit être strictement inférieure à -200 pour être évaluée avant conntrack. `raw` (-300) est le bon choix.

#### Rate Limiting avec Meters : par IP, dans le noyau

Le `limit rate` classique est global. 10 requêtes par seconde pour tout le monde, y compris les attaquants qui flood. Les **meters** font du rate limiting **par IP**, entièrement géré par le noyau, sans log, sans script.

##### SSH : rate limit par IP avec bannissement automatique

```bash
# Set pour les IPs bannies
nft add set inet firewall ssh_banned {
    type ipv4_addr \;
    flags dynamic, timeout \;
    timeout 15m \;
}

# Bloquer les IPs déjà bannies (en premier dans la chaîne)
nft add rule inet firewall input \
    ip saddr @ssh_banned \
    tcp dport 22 \
    counter log prefix "SSH-BANNED: " drop

# Rate limit : max 5 nouvelles connexions par minute par IP
# Si dépassé → bannissement de 15 minutes
nft add rule inet firewall input \
    tcp dport 22 \
    ct state new \
    meter ssh_meter { ip saddr limit rate over 5/minute burst 3 packets } \
    add @ssh_banned { ip saddr timeout 15m } \
    counter log prefix "SSH-RATELIMIT: " drop

# Autoriser le SSH dans les limites
nft add rule inet firewall input \
    tcp dport 22 \
    ct state new \
    counter accept
```

##### Web : rate limit avec burst configurable

Pour du trafic HTTP vous acceptez des pics courts mais vous limitez le débit soutenu :

```bash
nft add set inet firewall web_banned {
    type ipv4_addr \;
    flags dynamic, timeout \;
    timeout 5m \;
}

# Bloquer les web banned
nft add rule inet firewall input \
    ip saddr @web_banned \
    tcp dport { 80, 443 } \
    counter drop

# Rate limit web : 100 req/s, burst de 500
nft add rule inet firewall input \
    tcp dport { 80, 443 } \
    ct state new \
    meter web_meter { ip saddr limit rate over 100/second burst 500 packets } \
    add @web_banned { ip saddr timeout 5m } \
    counter log prefix "WEB-FLOOD: " drop

# Autoriser le web dans les limites
nft add rule inet firewall input \
    tcp dport { 80, 443 } \
    ct state new \
    counter accept
```

##### Consulter les compteurs des meters

```bash
nft list meter inet firewall ssh_meter
# table inet firewall {
#     meter ssh_meter {
#         type ipv4_addr
#         size 65535
#         elements = { 203.0.113.42 counter packets 47 bytes 3128,
#                      198.51.100.7 counter packets 12 bytes 892 }
#     }
# }
```

Visibilité complète sur qui fait quoi, sans toucher à vos logs.

#### Ruleset complet : production-ready

```bash
#!/usr/bin/env nft -f
# /etc/nftables.conf
# Rechargement atomique : nft -f /etc/nftables.conf

flush ruleset

# ─── Table principale ────────────────────────────────────────
table inet firewall {

    # ── Sets ───────────────────────────────────────────────────
    set admin_ranges {
        type ipv4_addr
        flags interval
        elements = { 45.80.0.0/22, 185.212.224.0/22 }
    }

    set ssh_banned {
        type ipv4_addr
        flags dynamic, timeout
        timeout 30m
    }

    set web_banned {
        type ipv4_addr
        flags dynamic, timeout
        timeout 10m
    }

    set tcp_open {
        type inet_service
        elements = { 80, 443 }
    }

    # ── Zones conntrack (priorité raw) ─────────────────────────
    chain zone_pre {
        type filter hook prerouting priority raw
        iifname "eth1" ct zone set 1
    }
    chain zone_out {
        type filter hook output priority raw
        oifname "eth1" ct zone set 1
    }

    # ── Input ──────────────────────────────────────────────────
    chain input {
        type filter hook input priority 0
        policy drop

        # Loopback
        iifname lo accept

        # Bannies en premier
        ip saddr @ssh_banned tcp dport 22 counter log prefix "SSH-BAN: " drop
        ip saddr @web_banned tcp dport @tcp_open counter log prefix "WEB-BAN: " drop

        # Conntrack
        ct state established,related accept
        ct state invalid drop

        # ICMP rate limité
        icmp type echo-request limit rate 5/second accept
        icmpv6 type echo-request limit rate 5/second accept

        # SSH — admins sans limite
        tcp dport 22 ip saddr @admin_ranges ct state new counter accept

        # SSH — rate limit strict pour le reste
        tcp dport 22 ct state new meter ssh_meter { ip saddr limit rate over 5/minute burst 3 packets } add @ssh_banned { ip saddr timeout 30m } counter log prefix "SSH-RATE: " drop
        tcp dport 22 ct state new counter accept

        # Web — rate limit par IP
        tcp dport @tcp_open ct state new meter web_meter { ip saddr limit rate over 100/second burst 500 packets } add @web_banned { ip saddr timeout 10m } counter log prefix "WEB-RATE: " drop
        tcp dport @tcp_open ct state new counter accept

        # Tout le reste
        counter log prefix "DROP-INPUT: " drop
    }

    # ── Forward ────────────────────────────────────────────────
    chain forward {
        type filter hook forward priority 0
        policy drop

        ct state established,related accept
        counter log prefix "DROP-FORWARD: " drop
    }

    # ── Output ─────────────────────────────────────────────────
    chain output {
        type filter hook output priority 0
        policy accept
    }
}
```

##### Charger et vérifier

```bash
# Vérifier la syntaxe sans appliquer
nft -c -f /etc/nftables.conf

# Charger — atomique : soit tout, soit rien
nft -f /etc/nftables.conf

# Lister l'état complet
nft list ruleset
nft list sets inet firewall
nft list meters inet firewall
```

#### Pièges classiques

Le premier piège : `limit rate` sans `over` **accepte** dans les limites et ignore silencieusement les paquets au-delà, ils continuent dans les règles suivantes. Pour bloquer au-delà du seuil, utilisez `limit rate over` suivi d'un `drop`.

Le deuxième : les priorités des chains. Le conntrack se situe à -200. Si vous utilisez `ct zone set` ou `notrack`, votre chain doit être en priorité `raw` (-300). Sinon, conntrack a déjà traité le paquet avant que votre règle ne soit évaluée.

Le troisième : les sets dynamiques sans `size` explicite. La valeur par défaut est 65535 éléments. Sur un serveur exposé avec beaucoup d'IPs, vérifiez régulièrement la taille de vos sets.

En production, chaque `drop` important doit avoir un `log prefix` avant lui. Et consultez vos meters régulièrement, c'est votre tableau de bord réseau, gratuit et intégré.

Pour l'IPv6, consultez le guide [nftables IPv6](https://www.shpv.fr/blog/nftables-ipv6-firewall/). Et pour la détection d'intrusions avancée, complétez avec [CrowdSec](https://www.shpv.fr/blog/crowdsec-2026/) ou [Suricata](https://www.shpv.fr/blog/suricata-ids/).