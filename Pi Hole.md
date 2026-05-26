---
tags:
  - linux
  - dns
---
Pi-hole agit comme un **serveur DNS** filtrant les requêtes publicitaires et de tracking, améliorant la vitesse de navigation et la confidentialité. Ce guide vous montre comment l'installer sur Linux ou Raspberry Pi.

#### Prérequis

- Serveur Linux (Debian, Ubuntu) ou Raspberry Pi OS
- Accès root ou sudo
- Accès internet

#### Installation rapide

Exécutez le script d'installation officiel :

```bash
curl -sSL https://install.pi-hole.net | bash
```

Suivez les étapes :

1. Choisissez l'interface réseau
2. Sélectionnez un upstream DNS (Cloudflare, Google, OpenDNS…)
3. Activez le blocking IPv4/IPv6
4. Notez le mot de passe de l'interface web

#### Configuration manuelle (optionnel)

##### Installation des dépendances

```bash
sudo apt update
sudo apt install -y dnsutils lighttpd php-common php-cgi php-sqlite3
```

##### Télécharger et configurer Pi-hole

```bash
git clone --depth 1 https://github.com/pi-hole/pi-hole.git /etc/pihole
git clone --depth 1 https://github.com/pi-hole/AdminLTE.git /var/www/html/admin
```

##### Configurer Lighttpd

```bash
sudo tee /etc/lighttpd/conf-available/15-pihole.conf << 'EOF'
server.document-root = "/var/www/html/admin"
EOF
sudo lighttpd-enable-mod pihole
sudo systemctl restart lighttpd
```

#### Utilisation

- Accédez à l'interface web : `http://IP_SERVEUR/admin`
- Connectez-vous avec le mot de passe défini

#### Listes de blocage

- Ajouter ou supprimer des bloc lists dans **Group Management > Adlists**
- Mettre à jour les listes via **Tools > Update Gravity**

#### Surveillance

- Consultez **Query Log** pour voir les requêtes bloquées
- Activez les notifications par email ou webhook

#### Sécurisation

- Modifiez le port de l'interface web (fichier `/etc/lighttpd/lighttpd.conf`)
- Restreignez l'accès IP via pare-feu
- Maintenez le système à jour

#### Approfondissement

- Si vous avez besoin d'un DNS sans Pi-hole, explorez [Unbound DNS](https://www.shpv.fr/blog/unbound-dns/)
- Pour une meilleure compréhension de l'infrastructure réseau, consultez [réseau Linux](https://www.shpv.fr/blog/reseau-linux-2026/)

#### Conclusion

Pi-hole est une solution légère et efficace pour bloquer les publicités et trackers au niveau DNS, améliorant la rapidité et la confidentialité de votre réseau domestique ou d'entreprise.