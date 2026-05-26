---
tags:
  - postgresql
  - performance
---
PostgreSQL avec sa configuration par défaut n'exploite pas pleinement les ressources serveur. Cet article détaille comment optimiser : paramètres mémoire, stratégies d'indexation, autovacuum, et analyse des requêtes lentes pour des performances optimales. Si vous êtes novice en PostgreSQL, commencez par notre guide d'[optimisation PostgreSQL de base](https://www.shpv.fr/blog/postgresql-performance/).

#### Plan

- Diagnostic de performance initial
- Configuration mémoire et cache
- Stratégies d'indexation avancées
- Autovacuum et maintenance
- Analyse et optimisation des requêtes
- Connection pooling
- Monitoring et alerting
- Conclusion

---

#### Diagnostic de performance initial

##### Extensions essentielles

```sql
-- pg_stat_statements : tracking des requêtes
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- pg_trgm : recherche full-text performante
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- pgstattuple : statistiques sur les tables
CREATE EXTENSION IF NOT EXISTS pgstattuple;
```

##### Identifier les goulots d'étranglement

```sql
-- Top 10 requêtes les plus lentes
SELECT
    calls,
    mean_exec_time::numeric(10,2) as avg_ms,
    total_exec_time::numeric(10,2) as total_ms,
    (total_exec_time / sum(total_exec_time) OVER ()) * 100 as pct,
    query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Tables les plus volumineuses
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size,
    n_live_tup as rows
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;

-- Index inutilisés (candidats à suppression)
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as size,
    idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelname !~ '^.*_pkey$'
ORDER BY pg_relation_size(indexrelid) DESC;

-- Bloat (gonflement des tables/index)
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
    round(100 * pg_relation_size(schemaname||'.'||tablename) /
          NULLIF(pg_total_relation_size(schemaname||'.'||tablename), 0), 2) as table_pct,
    n_dead_tup,
    round(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) as dead_pct
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC
LIMIT 20;
```

---

#### Configuration mémoire et cache

##### Calcul des paramètres selon RAM disponible

**Serveur dédié PostgreSQL avec 32 GB RAM :**

```conf
# /etc/postgresql/16/main/postgresql.conf

# === MÉMOIRE ===

# shared_buffers : 25% de la RAM (max 40%)
shared_buffers = 8GB

# effective_cache_size : 50-75% de la RAM
# (indique au planner combien de RAM pour le cache OS)
effective_cache_size = 24GB

# work_mem : RAM pour tris/hash par connexion
# Formule : RAM / (max_connections * 2-4)
# 32GB / (100 * 3) = ~100MB
work_mem = 100MB

# maintenance_work_mem : pour VACUUM, CREATE INDEX
# 5-10% de RAM ou 2GB max
maintenance_work_mem = 2GB

# === WRITE AHEAD LOG (WAL) ===

# wal_buffers : 16MB par défaut, monter si écritures intensives
wal_buffers = 16MB

# checkpoint_timeout : fréquence des checkpoints
checkpoint_timeout = 15min

# max_wal_size : taille max WAL avant checkpoint forcé
max_wal_size = 4GB
min_wal_size = 1GB

# checkpoint_completion_target : étaler l'écriture checkpoint
checkpoint_completion_target = 0.9

# === PARALLÉLISME ===

# max_worker_processes : workers background
max_worker_processes = 8

# max_parallel_workers_per_gather : workers par query
max_parallel_workers_per_gather = 4

# max_parallel_workers : total workers pour parallélisme
max_parallel_workers = 8

# === PLANNER ===

# random_page_cost : coût lecture aléatoire
# 1.1 pour SSD (4.0 par défaut pour HDD)
random_page_cost = 1.1

# effective_io_concurrency : I/O concurrent (SSD)
effective_io_concurrency = 200

# === LOGGING ===

# Log requêtes lentes (> 100ms)
log_min_duration_statement = 100

# Log checkpoints
log_checkpoints = on

# Log connexions
log_connections = on
log_disconnections = on

# Log lock waits
log_lock_waits = on
deadlock_timeout = 1s
```

**Appliquer :**

```bash
# Vérifier la config
sudo -u postgres psql -c "SHOW ALL;"

# Recharger config (sans redémarrage pour la plupart)
sudo systemctl reload postgresql

# Redémarrer si changement shared_buffers
sudo systemctl restart postgresql
```

##### Configuration selon profil d'utilisation

**OLTP (nombreuses transactions courtes) :**

```conf
shared_buffers = 8GB
effective_cache_size = 24GB
work_mem = 50MB              # Plus faible, nombreuses connexions
maintenance_work_mem = 1GB
checkpoint_timeout = 5min    # Checkpoints fréquents
```

**OLAP (analytics, requêtes complexes) :**

```conf
shared_buffers = 16GB
effective_cache_size = 48GB
work_mem = 500MB             # Plus élevé pour tris/joins
maintenance_work_mem = 4GB
checkpoint_timeout = 30min   # Checkpoints moins fréquents
max_parallel_workers_per_gather = 8
```

---

#### Stratégies d'indexation avancées

##### Types d'index et cas d'usage

```sql
-- B-tree : défaut, égalité et range
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_date ON orders(created_at);

-- Partial index : seulement certaines lignes
CREATE INDEX idx_orders_pending ON orders(status)
WHERE status = 'pending';

-- Composite index : plusieurs colonnes
-- Ordre important : colonnes les plus sélectives en premier
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Expression index : sur fonction
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- GIN : full-text search, arrays, jsonb
CREATE INDEX idx_articles_content ON articles
USING GIN(to_tsvector('english', content));

CREATE INDEX idx_tags ON posts USING GIN(tags);

-- GiST : géospatial, full-text
CREATE INDEX idx_locations ON stores
USING GIST(location);

-- BRIN : très grandes tables avec données séquentielles
-- 1000x plus petit qu'un B-tree, mais moins précis
CREATE INDEX idx_logs_timestamp ON logs
USING BRIN(timestamp);

-- Hash : seulement égalité (rarement utile)
CREATE INDEX idx_hash ON table USING HASH(column);
```

##### Optimisation d'index existants

```sql
-- Identifier les index manquants (suggestions)
SELECT
    schemaname,
    tablename,
    attname,
    n_distinct,
    correlation
FROM pg_stats
WHERE schemaname = 'public'
  AND n_distinct > 100
  AND correlation < 0.1;  -- Faible corrélation = bon candidat index

-- Rebuilder les index gonflés
REINDEX INDEX CONCURRENTLY idx_name;
REINDEX TABLE CONCURRENTLY table_name;

-- Supprimer index inutilisé
DROP INDEX CONCURRENTLY idx_unused;

-- Analyser l'utilisation d'un index spécifique
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM users WHERE email = 'test@example.com';
```

##### Index covering (INCLUDE)

```sql
-- Index covering : colonnes supplémentaires dans l'index
-- Évite l'accès à la table (index-only scan)
CREATE INDEX idx_orders_covering ON orders(user_id)
INCLUDE (total, created_at);

-- Query devient index-only scan
EXPLAIN (ANALYZE, BUFFERS)
SELECT user_id, total, created_at
FROM orders
WHERE user_id = 123;
```

---

#### Autovacuum et maintenance

##### Configuration autovacuum

```conf
# postgresql.conf

# Activer autovacuum (déjà on par défaut)
autovacuum = on

# Nombre de workers autovacuum
autovacuum_max_workers = 4

# Seuil de déclenchement
# vacuum si : dead_tuples > threshold + scale_factor * tuples
autovacuum_vacuum_threshold = 50
autovacuum_vacuum_scale_factor = 0.1  # 10% de la table

# Coût I/O (limite impact performance)
autovacuum_vacuum_cost_delay = 2ms
autovacuum_vacuum_cost_limit = 200

# Pour tables volumineuses, augmenter :
autovacuum_work_mem = 1GB
```

##### Tuning par table

```sql
-- Table à forte écriture : autovacuum plus fréquent
ALTER TABLE high_writes SET (
    autovacuum_vacuum_scale_factor = 0.02,  -- 2%
    autovacuum_vacuum_threshold = 1000,
    autovacuum_analyze_scale_factor = 0.01
);

-- Table volumineuse : augmenter work_mem
ALTER TABLE large_table SET (
    autovacuum_work_mem = 2048  -- 2GB en KB
);

-- Désactiver autovacuum (à éviter, mais parfois nécessaire)
ALTER TABLE bulk_load SET (
    autovacuum_enabled = false
);
-- N'oubliez pas de VACUUM ANALYZE après !
```

##### VACUUM manuel

```sql
-- VACUUM simple : nettoie dead tuples
VACUUM table_name;

-- VACUUM FULL : reconstruit la table (bloque la table !)
-- Libère vraiment l'espace disque
VACUUM FULL table_name;

-- VACUUM ANALYZE : vacuum + mise à jour stats
VACUUM ANALYZE table_name;

-- VACUUM pour toute la DB
VACUUM ANALYZE;

-- REINDEX pour réduire bloat des index
REINDEX TABLE CONCURRENTLY table_name;
```

##### Monitoring autovacuum

```sql
-- Derniers vacuum/analyze
SELECT
    schemaname,
    tablename,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze,
    n_dead_tup,
    n_mod_since_analyze
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- Autovacuum en cours
SELECT
    pid,
    usename,
    application_name,
    client_addr,
    backend_start,
    state,
    query
FROM pg_stat_activity
WHERE query LIKE '%autovacuum%'
  AND state != 'idle';

-- Bloat estimation
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size,
    round(100 * pg_relation_size(schemaname||'.'||tablename)::numeric /
          pg_total_relation_size(schemaname||'.'||tablename), 2) as data_pct
FROM pg_stat_user_tables
WHERE pg_total_relation_size(schemaname||'.'||tablename) > 100000000  -- > 100MB
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

---

#### Analyse et optimisation des requêtes

##### EXPLAIN ANALYZE

```sql
-- EXPLAIN ANALYZE : exécute la requête et donne stats réelles
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT u.name, COUNT(o.id)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id;

-- Interpréter les résultats :
-- - Seq Scan = mauvais (scan complet), ajouter index
-- - Index Scan = bon
-- - Bitmap Heap Scan = OK pour résultats moyens
-- - Nested Loop = bon si petits datasets
-- - Hash Join = bon pour gros datasets
-- - Shared Buffers Hit = % cache hit (viser >95%)
```

##### Optimisations courantes

**Problème : Seq Scan au lieu d'Index Scan**

```sql
-- Vérifier stats à jour
ANALYZE users;

-- Si toujours Seq Scan, forcer index
SET enable_seqscan = off;
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
SET enable_seqscan = on;

-- Ou créer l'index manquant
CREATE INDEX idx_users_email ON users(email);
```

**Problème : N+1 queries**

```sql
-- Mauvais : N+1
SELECT * FROM users;
-- Puis pour chaque user :
SELECT * FROM orders WHERE user_id = ?;

-- Bon : JOIN ou subquery
SELECT
    u.*,
    (SELECT COUNT(*) FROM orders WHERE user_id = u.id) as order_count
FROM users u;

-- Ou mieux : agrégation
SELECT
    u.*,
    COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
```

**Problème : COUNT(*) lent sur grandes tables**

```sql
-- Lent : COUNT(*) fait un scan complet
SELECT COUNT(*) FROM large_table;

-- Alternative : estimation rapide
SELECT reltuples::bigint AS estimate
FROM pg_class
WHERE relname = 'large_table';

-- Ou : garder un compteur incrémental
CREATE TABLE table_counters (
    table_name TEXT PRIMARY KEY,
    count BIGINT DEFAULT 0
);

-- Mettre à jour via trigger
CREATE OR REPLACE FUNCTION update_counter()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        UPDATE table_counters SET count = count + 1 WHERE table_name = TG_TABLE_NAME;
    ELSIF TG_OP = 'DELETE' THEN
        UPDATE table_counters SET count = count - 1 WHERE table_name = TG_TABLE_NAME;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

---

#### Connection pooling

##### PgBouncer

**Installation :**

```bash
apt install pgbouncer

# Configuration
cat > /etc/pgbouncer/pgbouncer.ini << EOF
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
listen_addr = 127.0.0.1
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 25
reserve_pool_size = 5
reserve_pool_timeout = 3
EOF

# Fichier users
echo '"username" "md5password"' > /etc/pgbouncer/userlist.txt

systemctl enable --now pgbouncer
```

**Pool modes :**

- `session` : 1 connexion = 1 session (défaut PostgreSQL)
- `transaction` : pool après chaque transaction (recommandé)
- `statement` : pool après chaque statement (risqué)

**Connection string avec PgBouncer :**

```
postgresql://user:pass@localhost:6432/mydb
```

---

#### Monitoring et alerting

##### Métriques essentielles

```sql
-- Cache hit ratio (viser >95%)
SELECT
    sum(heap_blks_read) as heap_read,
    sum(heap_blks_hit) as heap_hit,
    round(sum(heap_blks_hit) / NULLIF(sum(heap_blks_hit) + sum(heap_blks_read), 0) * 100, 2) as cache_hit_ratio
FROM pg_stattuple_approx('table_name');

-- Connexions actives
SELECT
    count(*) FILTER (WHERE state = 'active') as active,
    count(*) FILTER (WHERE state = 'idle') as idle,
    count(*) FILTER (WHERE state = 'idle in transaction') as idle_in_tx,
    count(*) as total
FROM pg_stat_activity;

-- Locks
SELECT
    locktype,
    relation::regclass,
    mode,
    granted,
    pid
FROM pg_locks
WHERE NOT granted;

-- Taille DB
SELECT
    pg_database.datname,
    pg_size_pretty(pg_database_size(pg_database.datname)) as size
FROM pg_database
ORDER BY pg_database_size(pg_database.datname) DESC;

-- Réplication lag (si réplicas)
SELECT
    client_addr,
    state,
    pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) as lag_bytes,
    replay_lag
FROM pg_stat_replication;
```

##### Prometheus exporter

```bash
# Installation postgres_exporter
docker run -d \
  -p 9187:9187 \
  -e DATA_SOURCE_NAME="postgresql://user:pass@localhost:5432/postgres?sslmode=disable" \
  prometheuscommunity/postgres-exporter

# Vérifier métriques
curl localhost:9187/metrics | grep pg_
```

##### Alertes critiques

```yaml
# prometheus alerts
groups:
  - name: postgresql
    rules:
      - alert: PostgreSQLDown
        expr: pg_up == 0
        for: 1m

      - alert: PostgreSQLCacheHitRatioLow
        expr: pg_stat_database_blks_hit / (pg_stat_database_blks_hit + pg_stat_database_blks_read) < 0.95
        for: 5m

      - alert: PostgreSQLTooManyConnections
        expr: sum(pg_stat_activity_count) > 0.8 * pg_settings_max_connections
        for: 5m

      - alert: PostgreSQLReplicationLag
        expr: pg_replication_lag_bytes > 10000000 # 10MB
        for: 2m
```

---

#### Checklist d'optimisation

✅ **Configuration de base :**

- [ ]  shared_buffers = 25 % RAM
- [ ]  effective_cache_size = 75 % RAM
- [ ]  work_mem adapté au workload
- [ ]  random_page_cost = 1.1 (SSD)

✅ **Indexation :**

- [ ]  Index sur foreign keys
- [ ]  Index sur colonnes WHERE fréquentes
- [ ]  Partial index pour filtres communs
- [ ]  Supprimer index inutilisés
- [ ]  REINDEX sur bloat >30 %

✅ **Maintenance :**

- [ ]  Autovacuum activé et tuné
- [ ]  VACUUM ANALYZE régulier
- [ ]  Monitoring bloat
- [ ]  REINDEX CONCURRENTLY mensuel

✅ **Requêtes :**

- [ ]  pg_stat_statements activé
- [ ]  Log slow queries (>100ms)
- [ ]  EXPLAIN ANALYZE sur requêtes lentes
- [ ]  Éviter N+1 queries
- [ ]  Connection pooling (PgBouncer)

✅ **Monitoring :**

- [ ]  Cache hit ratio >95 %
- [ ]  Connexions inférieures à 80 % max
- [ ]  Replication lag inférieur à 10MB
- [ ]  Dead tuples inférieurs à 10 %
- [ ]  Alertes Prometheus

---

#### Conclusion

L'optimisation PostgreSQL repose sur 4 piliers : configuration mémoire adaptée, indexation intelligente, autovacuum bien tuné, et analyse continue des requêtes lentes. Ces optimisations peuvent améliorer les performances de 10x à 100x.

**Actions prioritaires :**

1. Configurer shared_buffers et effective_cache_size
2. Activer pg_stat_statements
3. Créer index sur foreign keys et colonnes WHERE
4. Tuner autovacuum pour tables volumineuses
5. Implémenter PgBouncer si >100 connexions

**Gains typiques :**

- Configuration mémoire : +200-500 % throughput
- Index optimisés : +1000-10000 % sur queries ciblées
- Autovacuum tunné : +50 % write performance
- Connection pooling : +300 % connexions simultanées

Pour les environnements de production, complétez ces optimisations par une architecture [PostgreSQL HA avec Patroni](https://www.shpv.fr/blog/postgresql-ha-patroni/) et une [stack monitoring Prometheus](https://www.shpv.fr/blog/deployer-stack-monitoring/) pour superviser vos métriques critiques.