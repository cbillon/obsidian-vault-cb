---
tags:
  - nginx
---
[](https://www.shpv.fr/blog/nginx-rate-limiting/)
# Nginx rate limiting : protéger vos APIs et applications web

Le rate limiting protège vos APIs et applications web contre les abus, les pics de charge et les comportements malveillants. Nginx fournit des directives natives qui posent cette protection directement au niveau du reverse proxy, sans surcharger la logique applicative.

Voici comment configurer le rate limiting Nginx en production, des cas simples aux stratégies avancées.

---

#### Plan de l'article

1. **Pourquoi limiter le débit ?**, Les enjeux pratiques et réglementaires
2. **limit_req : limiter les requêtes par seconde**, Algorithme leaky bucket et zones mémoire
3. **limit_conn : limiter les connexions simultanées**, Protection contre les hoarding
4. **Burst et nodelay**, Gérer les pics légitimes
5. **Stratégies par endpoint**, Des taux différents pour login, API, webhooks
6. **Whitelisting et exceptions**, Exclure les services de confiance
7. **Réponses personnalisées**, Pages d'erreur 429 adaptées
8. **Logging et monitoring**, Suivre les rejets en production
9. **Rate limiting avancé**, Two-stage limiting et stratégies dynamiques
10. **Perspectives complémentaires**, Autres couches de sécurité Nginx

---

#### Pourquoi limiter le débit ?

Le rate limiting répond à plusieurs problématiques :

- **Prévention d'abus**, Empêcher les clients malveillants de submerger votre service
- **Fair usage**, Garantir une utilisation équitable des ressources pour tous les utilisateurs
- **Protection serveur**, Réduire la charge même face à un pic légitime
- **Conformité**, Respecter les SLA (99.9 %+ pour la plupart des infrastructures modernes) en stabilisant le débit

Chez SHPV, par exemple, nous recommandons le rate limiting comme première ligne de défense pour les [hébergement web](https://www.shpv.fr/services/hebergement/) et les APIs en production. C'est un investissement léger en CPU pour une résilience considérable.

---

#### limit_req : limiter les requêtes par seconde

La directive `limit_req` fonctionne selon l'**algorithme leaky bucket** : un seau qui se vide au débit configuré.

##### Concept de zone mémoire

Avant d'utiliser `limit_req`, vous devez définir une **zone mémoire** dans le contexte `http` :

```nginx
http {
    # Zone mémoire de 10 MB pour stocker les états des clients
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

    # Zone pour l'upload de fichiers (taux réduit)
    limit_req_zone $binary_remote_addr zone=upload_limit:10m rate=2r/s;

    # ...
}
```

- **$binary_remote_addr**, Clé unique par adresse IP (format binaire, plus efficace mémoire)
- **zone=api_limit:10m**, Nom et taille maximale de la zone
- **rate=10r/s**, 10 requêtes par seconde

##### Appliquer la limite

```nginx
server {
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        proxy_pass http://backend;
    }
}
```

**Sans burst/nodelay**, les requêtes au-delà de 10 r/s attendent en queue, puis le serveur retourne **429 Too Many Requests** dès que la queue est pleine.

---

#### limit_conn : limiter les connexions simultanées

Si `limit_req` contrôle le **débit**, `limit_conn` contrôle le **nombre de connexions ouvertes** par client.

```nginx
http {
    # Max 10 connexions par IP
    limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

    # Max 50 connexions par serveur (utile pour les uploads longs)
    limit_conn_zone $server_name zone=per_server:10m;
}

server {
    location /upload/ {
        limit_conn conn_limit 5;       # Max 5 connexions par IP
        limit_conn per_server 100;     # Max 100 au total
        client_body_timeout 1h;        # Upload long
        proxy_pass http://backend;
    }
}
```

Cas d'usage : un client lance 5 uploads en parallèle. Au-delà, les requêtes reçoivent **503 Service Unavailable** (ou code configuré via `limit_conn_status`).

---

#### Burst et nodelay

##### Burst : tolérer les pics

```nginx
limit_req zone=api_limit burst=20;
```

Sans `burst`, le taux est strict : dès la 11e requête (au-delà de 10 r/s), rejet immédiat.

**Avec burst=20**, les 20 premières requêtes au-delà du taux entrent en queue et sont traitées aux places libres. Les 21e+ sont rejetées.

##### nodelay : traitement sans délai

```nginx
limit_req zone=api_limit burst=20 nodelay;
```

Par défaut, Nginx laisse les requêtes en queue **s'exécuter au débit autorisé** (10 r/s), ce qui crée du délai.

Avec `nodelay`, les requêtes en burst s'exécutent **immédiatement**, mais consomment rapidement le burst. C'est idéal pour les pics légitimes.

##### Exemple réaliste

```nginx
# API public : 100 r/s, burst de 50
limit_req_zone $binary_remote_addr zone=public_api:10m rate=100r/s;

location /api/v1/ {
    limit_req zone=public_api burst=50 nodelay;
    proxy_pass http://api_backend;
}
```

---

#### Stratégies par endpoint

Les endpoints n'ont pas la même importance : login, paiements, webhooks nécessitent des stratégies différentes.

```nginx
http {
    # Strict pour login (force brute)
    limit_req_zone $binary_remote_addr zone=login_limit:10m rate=5r/m;

    # Modéré pour checkout
    limit_req_zone $binary_remote_addr zone=checkout_limit:10m rate=10r/m;

    # Agressif pour API public
    limit_req_zone $binary_remote_addr zone=api_public_limit:10m rate=1000r/m;

    # Permissif pour webhooks (clés API authentifiées)
    limit_req_zone $http_x_api_key zone=webhook_limit:10m rate=10000r/m;
}

server {
    # Protection login : 5 tentatives/minute
    location = /auth/login {
        limit_req zone=login_limit burst=2;
        proxy_pass http://auth_backend;
    }

    # Checkout : 10 commandes/minute par IP
    location /checkout/ {
        limit_req zone=checkout_limit burst=5 nodelay;
        proxy_pass http://payment_backend;
    }

    # API public : tarif élevé
    location /api/public/ {
        limit_req zone=api_public_limit burst=100 nodelay;
        proxy_pass http://public_api;
    }

    # Webhooks : clé API en header
    location /webhooks/ {
        limit_req zone=webhook_limit;
        proxy_pass http://webhook_processor;
    }
}
```

---

#### Whitelisting et exceptions

##### Exclure les services de confiance

Créer une variable `map` pour marquer les IPs confiées :

```nginx
http {
    # Map pour whitelisting
    map $binary_remote_addr $bypass_limit {
        default 0;
        ~^192\.168\.1\. 1;  # Réseau interne
        10.0.0.1 1;         # Load balancer partenaire
    }

    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/s;
}

server {
    location /api/ {
        if ($bypass_limit = 1) {
            set $bypass_rate_limit 1;  # Skip limit_req
        }

        limit_req zone=api_limit burst=50;
        proxy_pass http://api_backend;
    }
}
```

**Note** : Il n'existe pas de directive "skip limit_req", mais vous pouvez router les requêtes confiées via un endpoint différent sans limite.

##### Utiliser le module geo

```nginx
geo $country {
    default ZZ;
    10.0.0.0/8 US;
    192.168.0.0/16 FR;
}

limit_req_zone $country zone=geo_limit:10m rate=1000r/s;

location /api/ {
    limit_req zone=geo_limit burst=200;
    proxy_pass http://api_backend;
}
```

---

#### Réponses personnalisées

##### Code d'erreur personnalisé

Par défaut, `limit_req` retourne **429 Too Many Requests**. Changer le code :

```nginx
location /api/ {
    limit_req zone=api_limit burst=50;
    limit_req_status 503;  # Retourner 503 au lieu de 429
    proxy_pass http://api_backend;
}
```

##### Page d'erreur personnalisée

```nginx
error_page 429 /errors/rate-limit.html;

location = /errors/rate-limit.html {
    root /var/www/html;
    internal;
}
```

##### JSON pour les APIs

Pour une API JSON, créer une location séparée :

```nginx
location /api/ {
    limit_req zone=api_limit burst=50;
    limit_req_status 429;

    # Si limit_req rejette, afficher JSON
    if ($status = 429) {
        add_header Content-Type application/json;
        return 429 '{"error": "rate_limit_exceeded", "retry_after": 60}';
    }

    proxy_pass http://api_backend;
}
```

---

#### Logging et monitoring

##### Format de log personnalisé

```nginx
log_format main '$remote_addr - $remote_user [$time_local] '
                '"$request" $status $body_bytes_sent '
                '"$http_referer" "$http_user_agent" '
                'limit_req=$limit_req_status limit_conn=$limit_conn_status';

access_log /var/log/nginx/access.log main;
```

##### Suivre les rejets

```bash
# Compter les 429 en temps réel
tail -f /var/log/nginx/access.log | grep 'limit_req=429' | wc -l

# Identify clients being rate-limited
grep 'limit_req=429' /var/log/nginx/access.log | awk '{print $1}' | sort | uniq -c | sort -rn
```

##### Métriques Prometheus

Si vous utilisez `nginx-prometheus-exporter`, ajouter une métrique personnalisée :

```bash
# Dans votre collecteur Prometheus
rate_limit_rejections_total{zone="api_limit"} 1234
```

Les requêtes rejetées par `limit_req` ne passent pas au backend, elles sont comptées à `limit_req_status=429`.

---

#### Rate limiting avancé

##### Two-stage limiting

Combiner une limite globale + une limite par utilisateur :

```nginx
http {
    # Limite globale : 100 000 r/s au total
    limit_req_zone $server_name zone=global_limit:100m rate=100000r/s;

    # Limite par IP : 1000 r/s
    limit_req_zone $binary_remote_addr zone=per_ip_limit:10m rate=1000r/s;
}

server {
    location /api/ {
        limit_req zone=global_limit burst=10000;
        limit_req zone=per_ip_limit burst=200;
        proxy_pass http://api_backend;
    }
}
```

Les deux limites s'appliquent : si l'une rejette, la requête reçoit 429.

##### Dynamique avec map + variables

Adapter le taux en fonction de l'utilisateur :

```nginx
map $http_x_user_tier $user_rate_limit {
    "premium" "per_user_premium";
    "standard" "per_user_standard";
    default "per_user_free";
}

limit_req_zone $http_x_user_id zone=per_user_premium:10m rate=10000r/s;
limit_req_zone $http_x_user_id zone=per_user_standard:10m rate=1000r/s;
limit_req_zone $http_x_user_id zone=per_user_free:10m rate=100r/s;

location /api/v2/ {
    limit_req zone=$user_rate_limit burst=100;
    proxy_pass http://api_backend;
}
```

---

#### Perspectives complémentaires

Le rate limiting est une première ligne de défense. Pour une protection complète :

- **[Reverse proxy avancé](https://www.shpv.fr/blog/nginx-reverse-proxy-advanced/)**, Combinaison de rate limiting + authentification + caching
- **[Cache Nginx](https://www.shpv.fr/blog/nginx-cache/)**, Réduire la charge en cachant les réponses, moins besoin de rate limiting strict
- **[HTTP/2 et compression](https://www.shpv.fr/blog/nginx-http2-brotli/)**, Optimiser la bande passante limitée par client
- **WAF et sécurité applicative**, Pour détecter les patterns malveillants au-delà du simple débit