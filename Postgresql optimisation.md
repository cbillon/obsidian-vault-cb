---
tags:
  - postgresql
  - optimisation
---
**PostgreSQL** est une base relationnelle mature et riche en fonctionnalités.  
Sa configuration par défaut reste conservatrice et bride les performances.

On voit ici comment tuner un serveur PostgreSQL pour gagner en réactivité et traiter de gros volumes de données.

#### Plan de l'article

- Ajuster les paramètres mémoire
- Optimiser les entrées/sorties (I/O)
- Gérer efficacement l'autovacuum
- Utiliser les index à bon escient
- Conclusion

---

#### Ajuster les paramètres mémoire

Le fichier principal est `postgresql.conf`. Quelques paramètres clés :

```conf
shared_buffers = 25% de la RAM
work_mem = 64MB
maintenance_work_mem = 256MB
effective_cache_size = 50% de la RAM
```

- `shared_buffers` : mémoire tampon pour les données.
- `work_mem` : mémoire utilisée pour les tris et les jointures.
- `effective_cache_size` : estimation de la mémoire disponible pour le cache disque.

---

#### Optimiser les entrées/sorties (I/O)

- Utiliser des disques **SSD** pour améliorer les performances.
- Configurer le **WAL (Write-Ahead Logging)** sur un disque rapide.
- Paramètres utiles :

```conf
wal_buffers = 16MB
synchronous_commit = off
checkpoint_timeout = 15min
```

👉 Attention : désactiver `synchronous_commit` peut améliorer la vitesse, mais réduit la sécurité en cas de crash.

---

#### Gérer efficacement l'autovacuum

L'**autovacuum** nettoie les tables et optimise leur structure. Pour une exploration plus approfondie des stratégies avancées de tuning et d'autovacuum, consultez notre guide sur l'[optimisation PostgreSQL avancée](https://www.shpv.fr/blog/postgresql-performance-advanced/). Paramètres recommandés :

```conf
autovacuum = on
autovacuum_max_workers = 5
autovacuum_naptime = 30s
autovacuum_vacuum_scale_factor = 0.1
```

👉 Un autovacuum trop espacé entraîne du bloat, trop fréquent consomme trop de ressources.

---

#### Utiliser les index à bon escient

- Créez des **index** sur les colonnes fréquemment utilisées dans les filtres (`WHERE`).
- Utilisez les **index partiels** pour limiter la taille.
- Exploitez les **index GIN** pour les recherches en texte intégral.
- Vérifiez les requêtes lentes avec :

```sql
EXPLAIN ANALYZE SELECT ...
```

---

#### Conclusion

L'optimisation d'un serveur **PostgreSQL** repose sur :

- Des réglages mémoire adaptés à la machine,
- Une bonne gestion des I/O,
- Un autovacuum calibré,
- Une stratégie d'indexation intelligente.

Avec ces ajustements, vous obtiendrez une base de données **plus rapide, stable et scalable**. Pour les environnements critiques, envisagez une architecture [PostgreSQL HA avec Patroni](https://www.shpv.fr/blog/postgresql-ha-patroni/) pour la haute disponibilité, et complétez avec une stratégie de [caching Redis](https://www.shpv.fr/blog/redis-caching/) pour réduire la charge sur votre base de données.