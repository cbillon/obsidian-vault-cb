---
tags:
  - nginx
  - cache
---
**Nginx** est l'un des serveurs web les plus utilisés pour sa rapidité et sa légèreté.  
Pour améliorer encore ses performances, il est possible d'activer la **mise en cache** afin de réduire le temps de réponse et la charge serveur.

Ce guide détaille comment configurer la mise en cache avec Nginx.

#### Plan de l'article

- Activer le cache dans Nginx
- Configurer un cache disque
- Gérer l'expiration et la validation du cache
- Purger et surveiller le cache
- Conclusion

---

#### Activer le cache dans Nginx

Dans la configuration principale (`/etc/nginx/nginx.conf`) :

```conf
http {
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=STATIC:10m inactive=24h max_size=1g;
    include /etc/nginx/conf.d/*.conf;
}
```

---

#### Configurer un cache disque

Dans un VirtualHost :

```conf
server {
    listen 80;
    server_name exemple.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_cache STATIC;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404 1m;
        add_header X-Cache-Status $upstream_cache_status;
    }
}
```

- `proxy_cache_valid` définit la durée de mise en cache.
- L'en-tête `X-Cache-Status` permet de voir si la réponse vient du cache.

---

#### Gérer l'expiration et la validation du cache

Définir des règles spécifiques :

```conf
location ~* \.(jpg|jpeg|png|gif|css|js|ico|webp)$ {
    expires 30d;
    access_log off;
}
```

👉 Les fichiers statiques seront servis directement depuis le cache disque.

---

#### Purger et surveiller le cache

Pour purger le cache :

```bash
sudo rm -rf /var/cache/nginx/*
sudo systemctl reload nginx
```

Pour surveiller :

```bash
tail -f /var/log/nginx/access.log
```

Vérifier la présence de `X-Cache-Status: HIT`.

---

#### Conclusion

En configurant correctement la **mise en cache de Nginx**, vous :

- Accélérez vos sites web,
- Réduisez la charge sur vos serveurs applicatifs,
- Améliorez l'expérience utilisateur.

Une optimisation simple, à fort impact, pour tout administrateur système.

Pour aller plus loin, consultez [HTTP/2 et Brotli](https://www.shpv.fr/blog/nginx-http2-brotli/) et notre guide sur [Web Vitals](https://www.shpv.fr/blog/core-web-vitals/) pour le SEO.