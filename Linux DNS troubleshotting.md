---
tags:
  - linux
  - dns
---
Le DNS est un pilier de l'infrastructure réseau. Quand il dysfonctionne, tout s'arrête. Savoir diagnostiquer rapidement avec dig, comprendre les enregistrements et valider DNSSEC fait la différence entre 5 minutes et 2 heures de downtime.

#### Plan

- Outils de diagnostic DNS : dig, host, nslookup
- Analyser les réponses DNS et les codes de retour
- Timeouts et résolution lente : diagnostic
- DNSSEC : validation et troubleshooting
- Split-horizon DNS et vues
- Conclusion

---

#### Outils de diagnostic DNS

##### dig : l'outil de référence

```bash
# Requête basique
dig example.com

# Spécifier le type d'enregistrement
dig example.com A
dig example.com MX
dig example.com TXT
dig example.com NS

# Interroger un serveur DNS spécifique
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com A

# Réponse courte
dig +short example.com
dig +short example.com MX

# Trace complète (récursion)
dig +trace example.com
```

##### host : simple et rapide

```bash
host example.com
host -t MX example.com
host 93.184.216.34  # Reverse DNS
```

##### nslookup : legacy mais présent

```bash
nslookup example.com
nslookup example.com 8.8.8.8
```

---

#### Analyser les réponses DNS

##### Anatomie d'une réponse dig

```bash
dig example.com A

; <<>> DiG 9.18.24 <<>> example.com A
;; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;example.com.                   IN      A

;; ANSWER SECTION:
example.com.            3600    IN      A       93.184.216.34

;; Query time: 12 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Tue Jan 14 11:30:00 CET 2026
;; MSG SIZE  rcvd: 56
```

**Sections importantes** :

- **QUERY** : nombre de questions
- **ANSWER** : réponses obtenues
- **AUTHORITY** : serveurs autoritaires
- **ADDITIONAL** : infos supplémentaires
- **Query time** : latence
- **SERVER** : serveur DNS utilisé

##### Codes de retour (RCODE)

```bash
# NOERROR (0) : succès
# SERVFAIL (2) : échec serveur DNS
# NXDOMAIN (3) : domaine inexistant
# REFUSED (5) : requête refusée

dig nonexistent.example.com
# status: NXDOMAIN
```

##### TTL : Time To Live

```bash
dig example.com

# Réponse :
example.com.    3600    IN      A       93.184.216.34
#               ↑ TTL en secondes (1h ici)
```

Le TTL indique combien de temps la réponse peut être cachée.

---

#### Timeouts et résolution lente

##### Identifier un timeout

```bash
dig example.com

# Si timeout :
;; connection timed out; no servers could be reached
```

**Causes fréquentes** :

- Firewall bloquant le port 53 (UDP/TCP)
- Serveur DNS down ou injoignable
- Problème réseau

##### Tester la connectivité DNS

```bash
# Ping du serveur DNS
ping 8.8.8.8

# Test port 53 UDP
nc -u -z -v 8.8.8.8 53

# Test port 53 TCP (pour grosses réponses)
nc -z -v 8.8.8.8 53
```

##### Mesurer la latence

```bash
# Comparer plusieurs serveurs
for dns in 8.8.8.8 1.1.1.1 9.9.9.9; do
    echo "DNS: $dns"
    dig @$dns example.com | grep "Query time"
done
```

##### Vérifier /etc/resolv.conf

```bash
cat /etc/resolv.conf

# Contenu typique
nameserver 8.8.8.8
nameserver 1.1.1.1
options timeout:2 attempts:3
```

---

#### DNSSEC : validation et troubleshooting

##### Qu'est-ce que DNSSEC ?

DNSSEC signe cryptographiquement les enregistrements DNS pour garantir leur authenticité et intégrité.

##### Vérifier DNSSEC

```bash
# Vérifier si DNSSEC est actif
dig example.com +dnssec

# Réponse avec flag AD (Authenticated Data)
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 2
```

##### Valider DNSSEC manuellement

```bash
# Obtenir la clé DNSKEY
dig example.com DNSKEY +short

# Valider la chaîne de confiance
dig +dnssec +multiline example.com
```

##### Débugger DNSSEC

```bash
# Si SERVFAIL avec DNSSEC
dig example.com +dnssec +cd
# +cd = checking disabled (ignore DNSSEC)

# Si ça fonctionne avec +cd, problème DNSSEC
```

**Erreurs courantes** :

- Signature expirée
- Clés non synchronisées
- Horloge système incorrecte

---

#### Split-horizon DNS

##### Principe

Retourner des réponses différentes selon la source de la requête (interne vs externe).

##### Exemple avec BIND

```bash
# /etc/bind/named.conf.local
view "internal" {
    match-clients { 192.168.1.0/24; };
    zone "example.com" {
        type master;
        file "/etc/bind/zones/internal/example.com.zone";
    };
};

view "external" {
    match-clients { any; };
    zone "example.com" {
        type master;
        file "/etc/bind/zones/external/example.com.zone";
    };
};
```

##### Tester depuis différents réseaux

```bash
# Depuis le réseau interne
dig @dns-server.local app.example.com
# Réponse : 192.168.1.10 (IP privée)

# Depuis Internet
dig @dns-server.public app.example.com
# Réponse : 203.0.113.50 (IP publique)
```

---

#### Ressources connexes

- Pour déployer votre serveur DNS, consultez [DNS avec Bind9](https://www.shpv.fr/blog/dns-bind9/)
- Vous cherchez une alternative légère ? Découvrez [Unbound DNS](https://www.shpv.fr/blog/unbound-dns/)
- Pour une approche réseau globale, voir [réseau Linux](https://www.shpv.fr/blog/reseau-linux-2026/)

#### Conclusion

Maîtriser dig et les outils de diagnostic DNS permet de résoudre rapidement les problèmes de résolution. Comprendre les TTL, valider DNSSEC et identifier les timeouts sont des compétences essentielles pour tout sysadmin.