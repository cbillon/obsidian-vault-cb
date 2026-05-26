---
tags:
  - postgresql
  - connexion
---
# PgBouncer : optimiser les connexions PostgreSQL en production

Les connexions PostgreSQL consomment des ressources précieuses : chaque nouvelle connexion réserve environ **10 MB de mémoire**, crée un processus dédié, et introduit une latence de négociation SSL/TLS. À l'échelle d'une application serveur avec des centaines de requêtes concurrentes, ce surcoût devient rapidement **problématique**. Le `max_connections` par défaut (100) se remplit rapidement. PgBouncer résout ce problème en implémentant un **lightweight connection pooling** qui maintient un petit nombre de connexions physiques vers PostgreSQL et en multiplexe les milliers de connexions clientes.

Cet article vous guide à travers l'installation, la configuration et l'optimisation de PgBouncer en environnement production.

---

#### Plan de l'article

1. **Le problème des connexions PostgreSQL**, contraintes de ressources
2. **PgBouncer : principes et architecture**, proxy léger et event-driven
3. **Les trois modes de pooling**, session, transaction, statement
4. **Installation et configuration**, Debian/Ubuntu, fichiers de config
5. **Configuration avancée**, tuning des paramètres critiques
6. **Monitoring PgBouncer**, SHOW commands, Prometheus, Grafana
7. **PgBouncer vs alternatives**, pgpool-II, Odyssey, solutions intégrées
8. **Bonnes pratiques production**, HA, sécurité, logging
9. **Perspectives complémentaires**, articles connexes et ressources
10. **Conclusion**, déploiement recommandé

---

#### Le problème des connexions PostgreSQL

PostgreSQL crée un **processus OS indépendant** pour chaque client connecté. Voici les coûts directs et indirects :

|Métrique|Impact|
|---|---|
|Mémoire par connexion|~10 MB (shared_buffers, work_mem, context)|
|Temps d'établissement|10–50 ms (handshake TCP + auth)|
|Context switching|O(n²) avec n connexions simultanées|
|`max_connections` par défaut|100 (inadapté aux app modernes)|

**Exemple concret :** Une application NodeJS avec 200 workers, chacun maintenant une connexion, consomme déjà **2 GB de RAM côté PostgreSQL** avant même de traiter la première requête.

---

#### PgBouncer : principes et architecture

PgBouncer est un **lightweight connection pooler** écrit en C. Au lieu d'une nouvelle connexion PostgreSQL par client, PgBouncer :

1. Accepte les connexions clientes
2. Maintient un **petit pool de vraies connexions** vers PostgreSQL
3. Réutilise ces connexions selon le mode de pooling configuré

Architecture typique :

```
Clients (100s)
    ↓
PgBouncer (lightweight, single-threaded, event-based)
    ↓
PostgreSQL Pool (10-20 connexions réelles)
    ↓
PostgreSQL Server
```

PgBouncer est :

- **Single-threaded** : une seule boucle event (libevent)
- **Stateless** : pas de session persistante en mémoire
- **Très léger** : ~2 MB d'empreinte
- **Compatible** : clients PostgreSQL standard, pas de modification du code

---

#### Les trois modes de pooling

|Mode|Réutilisation|Cas d'usage|Limites|
|---|---|---|---|
|**Session**|Connexion attribuée au client jusqu'au disconnect|Clients persistants (connections longues)|Moins efficient en ressources|
|**Transaction**|Connexion réutilisée entre chaque transaction|Applications web, requêtes courtes (recommandé)|`SET` n'affecte que la session client, pas le backend|
|**Statement**|Connexion réutilisée après chaque requête (v1.3+)|Workloads très élevés, auto-commit|Les transactions multi-statements ne fonctionnent pas|

**Configuration par défaut :** mode **session**. **Recommandation production :** mode **transaction** (meilleur compromis).

---

#### Installation et configuration

##### Installation sur Debian/Ubuntu

```bash
sudo apt-get update
sudo apt-get install pgbouncer
```

##### Fichier de configuration : `/etc/pgbouncer/pgbouncer.ini`

```ini
[databases]
myapp = host=localhost port=5432 dbname=mydb_prod

[pgbouncer]
; Écoute sur localhost:6432
listen_addr = 127.0.0.1
listen_port = 6432

; Pool size = nombre de connexions physiques par base
pool_size = 12
min_pool_size = 5
reserve_pool_size = 3
reserve_pool_timeout = 3

; Mode de pooling : session, transaction, statement
pool_mode = transaction

; Timeouts (secondes)
server_lifetime = 3600
server_idle_in_transaction_session_timeout = 60
idle_in_transaction_session_timeout = 600

; Log
logfile = /var/log/pgbouncer/pgbouncer.log
log_connections = 1
log_disconnections = 1
log_pooler_stats = 1

; Stats
stats_users = pgbouncer_stats
admin_users = pgbouncer_admin

; TCP keepalive
tcp_keepalives_idle = 30
tcp_keepalives_interval = 30
tcp_keepalives_count = 5
```

##### Fichier d'authentification : `/etc/pgbouncer/userlist.txt`

```
"appuser" "mot_de_passe_hash"
"pgbouncer_admin" "hash_admin"
"pgbouncer_stats" "hash_stats"
```

Les hashes sont générés avec :

```bash
echo -n "password" | md5sum | sed 's/-//'
# Pour md5 simple, ou utiliser pg_password:
python3 -c "import hashlib; print('md5' + hashlib.md5(b'passworduser').hexdigest())"
```

---

#### Configuration avancée

##### Tuning du pool_size

Règle empirique :

```
pool_size = (core_count * 2) + effective_spindle_count

Exemples :
- Serveur 4 cœurs, SSD : pool_size ≈ 10–12
- Serveur 16 cœurs, HDD : pool_size ≈ 20–25
```

Vérifier avec benchmark :

```bash
# Test de charge
pgbench -h localhost -p 6432 -U appuser -d mydb -c 50 -j 10 -T 60
```

##### server_lifetime et recyclage

```ini
; Recyclage des connexions après 1h
server_lifetime = 3600
; Timeout inactivité (évite les idle-in-transaction)
server_idle_in_transaction_session_timeout = 60
```

##### Authentification et sécurité

```ini
; md5, scram-sha-256, plain (déconseillé)
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt
```

---

#### Monitoring PgBouncer

##### Commandes SHOW intégrées

Accès via admin :

```bash
psql -h localhost -p 6432 -U pgbouncer_admin -d pgbouncer
```

Requêtes utiles :

```sql
-- Vue d'ensemble
SHOW POOLS;

-- Statistiques clients
SHOW CLIENTS;

-- Statistiques serveurs
SHOW SERVERS;

-- État du pooler
SHOW STATS;
```

##### Export Prometheus

Utiliser [pgbouncer_exporter](https://github.com/pgbouncer/pgbouncer_exporter) :

```bash
wget https://github.com/pgbouncer/pgbouncer_exporter/releases/download/v0.5.1/pgbouncer_exporter-0.5.1.linux-amd64.tar.gz
tar xzf pgbouncer_exporter-0.5.1.linux-amd64.tar.gz
sudo mv pgbouncer_exporter /usr/local/bin/
```

Configuration systemd `/etc/systemd/system/pgbouncer_exporter.service` :

```ini
[Unit]
Description=PgBouncer Prometheus Exporter
After=network.target

[Service]
Type=simple
User=postgres
ExecStart=/usr/local/bin/pgbouncer_exporter --pgbouncer.address=localhost:6432
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Scrape Prometheus (`prometheus.yml`) :

```yaml
scrape_configs:
  - job_name: pgbouncer
    static_configs:
      - targets: ['localhost:9127']
```

Métriques clés :

- `pgbouncer_pools_client_connections_active`
- `pgbouncer_pools_server_connections_active`
- `pgbouncer_pools_server_connections_idle`
- `pgbouncer_stat_queries_total`

---

#### PgBouncer vs alternatives

|Solution|Overhead|Langage|HA natif|Cas d'usage|
|---|---|---|---|---|
|**PgBouncer**|Ultra-léger (~2 MB)|C|Non (Keepalived)|Pooling simple, haute charge|
|**pgpool-II**|Moyen (~20 MB)|C|Oui (watchdog)|Failover, Load-balancing, Replication|
|**Odyssey**|Léger (~5 MB)|C|Partiellement|Très haute charge (250k+ conn)|
|**PgBouncer (builtin)**|N/A|PL/pgSQL|N/A|Ancienne approche, déprécié|

**Recommandation :** PgBouncer pour pooling pur + Keepalived pour HA. pgpool-II si failover automatique requis.

---

#### Bonnes pratiques production

##### Haute disponibilité avec Keepalived

Déploiement recommandé : **deux instances PgBouncer** avec VIP failover.

Configuration Keepalived `/etc/keepalived/keepalived.conf` :

```
vrrp_script check_pgbouncer {
  script "/usr/local/bin/check_pgbouncer.sh"
  interval 2
  weight -20
  fall 2
}

vrrp_instance PGBOUNCER {
  state MASTER
  interface eth0
  virtual_router_id 51
  priority 100
  advert_int 1
  virtual_ipaddress {
    10.0.0.100/24
  }
  track_script {
    check_pgbouncer
  }
}
```

Script de vérification :

```bash
#!/bin/bash
# /usr/local/bin/check_pgbouncer.sh
psql -h localhost -p 6432 -U pgbouncer_admin -d pgbouncer \
  -c "SHOW STATS" > /dev/null 2>&1
exit $?
```

##### Rotation des logs

Fichier `/etc/logrotate.d/pgbouncer` :

```
/var/log/pgbouncer/*.log {
  daily
  rotate 30
  compress
  delaycompress
  notifempty
  create 0640 postgres postgres
  postrotate
    systemctl reload pgbouncer > /dev/null 2>&1 || true
  endscript
}
```

##### Hardening sécurité

```ini
# pgbouncer.ini
; Désactiver les commandes dangereuses
allow_unsecured_connections = 0
; Authentification SCRAM-SHA-256
auth_type = scram-sha-256
; Restreindre admin_users
admin_users = pgbouncer_admin
; Limiter stats_users
stats_users = monitoring_user
; Unix socket (localhost uniquement)
unix_socket_dir = /var/run/pgbouncer
unix_socket_mode = 0660
```

---

#### Perspectives complémentaires

Pour approfondir les sujets connexes :

- **[PostgreSQL : haute disponibilité avec Patroni](https://www.shpv.fr/blog/postgresql-ha-patroni/)**, orchestration des replicas et failover automatique
- **[Optimiser les performances PostgreSQL](https://www.shpv.fr/blog/postgresql-performance/)**, indexation, EXPLAIN ANALYZE, vacuum
- **[PostgreSQL avancé : tuning production](https://www.shpv.fr/blog/postgresql-performance-advanced/)**, shared_buffers, effective_cache_size, work_mem
- **[Redis comme cache applicatif](https://www.shpv.fr/blog/redis-caching/)**, réduire la charge sur PostgreSQL

---

#### Sources

- **Documentation officielle PgBouncer** : [pgbouncer.github.io](https://pgbouncer.github.io/)
- **PostgreSQL Server Documentation** : [postgresql.org/docs](https://www.postgresql.org/docs/)
- **PgBouncer Exporter for Prometheus** : [github.com/pgbouncer/pgbouncer_exporter](https://github.com/pgbouncer/pgbouncer_exporter)
- **Connection Pooling Patterns** : [Digital Ocean - Connection Pooling with PgBouncer](https://www.digitalocean.com/)
- **Keepalived Documentation** : [keepalived.org](https://www.keepalived.org/)

---

#### Conclusion

PgBouncer est un outil **essentiel pour toute infrastructure PostgreSQL** en production. Sa légèreté, sa stabilité et sa configuration simple en font le choix préféré des équipes DevOps et infrastructure.

**Checklist de déploiement :**

✅ Installation et configuration du pool_size optimal ✅ Choix du mode de pooling (transaction recommandé) ✅ Mise en place du monitoring Prometheus ✅ Configuration HA avec Keepalived et health-checks ✅ Logs rotatés et alertes Grafana ✅ Tests de charge et validation en staging