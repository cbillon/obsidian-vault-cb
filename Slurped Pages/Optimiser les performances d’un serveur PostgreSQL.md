---
link: https://www.shpv.fr/blog/postgresql-performance/
byline: SHPV
site: SHPV
date: 2025-10-01T09:00
excerpt: "Découvrez les réglages essentiels pour améliorer les performances d’un serveur PostgreSQL : mémoire, I/O, indexation et autovacuum."
twitter: https://twitter.com/@SHPVFR
tags:
  - postgresql
  - optimisation
slurped: 2025-10-12T18:18
title: Optimiser les performances d’un serveur PostgreSQL
---

Publié le 1 octobre 2025

Base de données

PostgreSQL

Administration

**PostgreSQL** est une base de données relationnelle puissante et riche en fonctionnalités.  
Par défaut, sa configuration est assez conservatrice, ce qui peut limiter ses performances.

Dans cet article, nous allons voir comment optimiser un serveur PostgreSQL pour améliorer sa réactivité et sa capacité à traiter de gros volumes de données.

## Plan de l’article

- Ajuster les paramètres mémoire
- Optimiser les entrées/sorties (I/O)
- Gérer efficacement l’autovacuum
- Utiliser les index à bon escient
- Conclusion

---

## Ajuster les paramètres mémoire

Le fichier principal est `postgresql.conf`. Quelques paramètres clés :

```
shared_buffers = 25% de la RAM
work_mem = 64MB
maintenance_work_mem = 256MB
effective_cache_size = 50% de la RAM
```

- `shared_buffers` : mémoire tampon pour les données.
- `work_mem` : mémoire utilisée pour les tris et les jointures.
- `effective_cache_size` : estimation de la mémoire disponible pour le cache disque.

---

## Optimiser les entrées/sorties (I/O)

- Utiliser des disques **SSD** pour améliorer les performances.
- Configurer le **WAL (Write-Ahead Logging)** sur un disque rapide.
- Paramètres utiles :

```
wal_buffers = 16MB
synchronous_commit = off
checkpoint_timeout = 15min
```

👉 Attention : désactiver `synchronous_commit` peut améliorer la vitesse, mais réduit la sécurité en cas de crash.

---

## Gérer efficacement l’autovacuum

L’**autovacuum** nettoie les tables et optimise leur structure.  
Paramètres recommandés :

```
autovacuum = on
autovacuum_max_workers = 5
autovacuum_naptime = 30s
autovacuum_vacuum_scale_factor = 0.1
```

👉 Un autovacuum trop espacé entraîne du bloat, trop fréquent consomme trop de ressources.

---

## Utiliser les index à bon escient

- Créez des **index** sur les colonnes fréquemment utilisées dans les filtres (`WHERE`).
- Utilisez les **index partiels** pour limiter la taille.
- Exploitez les **index GIN** pour les recherches en texte intégral.
- Vérifiez les requêtes lentes avec :

```
EXPLAIN ANALYZE SELECT ...
```

---

## Conclusion

L’optimisation d’un serveur **PostgreSQL** repose sur :

- Des réglages mémoire adaptés à la machine,
- Une bonne gestion des I/O,
- Un autovacuum calibré,
- Une stratégie d’indexation intelligente.

Avec ces ajustements, vous obtiendrez une base de données **plus rapide, stable et scalable**.