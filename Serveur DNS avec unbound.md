---
tags:
  - dns
---
Un serveur DNS local améliore la rapidité des résolutions et protège votre vie privée en évitant de dépendre des DNS publics. Ce guide vous explique comment mettre en place **Unbound**, un serveur DNS récursif, léger et sécurisé.

#### Prérequis

- Un serveur Debian, Ubuntu, Rocky ou AlmaLinux
- Accès root ou sudo

#### Installation d'Unbound

##### Debian/Ubuntu

```bash
sudo apt update
sudo apt install unbound -y
```

##### RHEL/AlmaLinux/Rocky

```bash
sudo dnf install epel-release -y
sudo dnf install unbound -y
```

#### Configuration de base

Fichier de configuration : `/etc/unbound/unbound.conf`

Exemple minimal :

```yaml
server:
    verbosity: 1
    interface: 0.0.0.0
    access-control: 192.168.0.0/16 allow
    access-control: 127.0.0.0/8 allow
    do-ip6: no
    hide-identity: yes
    hide-version: yes
    qname-minimisation: yes
    prefetch: yes

forward-zone:
    name: "."
    forward-tls-upstream: yes
    forward-addr: 1.1.1.1@853
    forward-addr: 9.9.9.9@853
```

#### Démarrer le service

```bash
sudo systemctl enable --now unbound
sudo systemctl status unbound
```

#### Tester votre serveur DNS

```bash
dig @127.0.0.1 www.google.com
```

#### Ajouter DNS-over-TLS ou DNS-over-HTTPS (optionnel)

Unbound supporte DNS-over-TLS natif, pour DNS-over-HTTPS vous pouvez ajouter un proxy comme `cloudflared` ou `dnscrypt-proxy`.

#### Sécurisation avancée

- Firewall : autoriser UDP/TCP port 53 pour le LAN uniquement.
- Journaux réduits pour la confidentialité.
- Vérification DNSSEC (activé par défaut sur Unbound).

#### Alternatives et compléments

- Découvrez [DNS avec Bind9](https://www.shpv.fr/blog/dns-bind9/) pour une solution plus complète
- Explorez [Pi-hole](https://www.shpv.fr/blog/pihole-setup/) pour ajouter du filtrage des publicités
- Pour une meilleure compréhension du réseau Linux, consultez [réseau Linux](https://www.shpv.fr/blog/reseau-linux-2026/)

#### Conclusion

Avec Unbound, vous obtenez un DNS rapide, résilient et respectueux de votre vie privée. Parfait pour un usage personnel, un datacenter ou une PME