---
link: https://www.shpv.fr/blog/nginx-varnish-cache/
byline: SHPV
site: SHPV
date: 2025-07-15T12:00
excerpt: Guide complet pour configurer Nginx en front-end et Varnish comme reverse proxy cache afin d’accélérer la distribution de contenus web et réduire la charge serveur.
twitter: https://twitter.com/@SHPVFR
tags:
  - nginx
  - varnish
  - cache
slurped: 2025-10-12T18:42
title: Mettre en place un cache HTTP inverse avec Varnish et Nginx
---

Publié le 15 juillet 2025

Pour améliorer la performance de vos sites web, **Varnish Cache** agit comme un cache HTTP inverse, diminuant la charge applicative. En combinant **Nginx** en front-end et Varnish en back-end, vous obtenez une architecture robuste et évolutive.

## Prérequis

- Serveur Linux (Debian, Ubuntu, CentOS)
- Nginx installé
- Varnish installé (v6+)

## 1. Configuration de Nginx en front-end

Dans `/etc/nginx/sites-available/default` :

```
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:6081;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Redémarrez Nginx :

```
sudo systemctl reload nginx
```

## 2. Configuration de Varnish

Éditez `/etc/varnish/default.vcl` :

```
vcl 4.0;

backend default {
    .host = "127.0.0.1";
    .port = "8080";
}

sub vcl_recv {
    if (req.method == "PURGE") {
        if (client.ip != "127.0.0.1") {
            return (synth(403, "Forbidden"));
        }
        return (purge);
    }
}

sub vcl_backend_response {
    set beresp.ttl = 5m;
}

sub vcl_deliver {
    if (obj.hits > 0) {
        set resp.http.X-Cache = "HIT";
    } else {
        set resp.http.X-Cache = "MISS";
    }
}
```

Démarrez Varnish en écoutant sur le port 6081 et Nginx sur 8080 :

```
# Modifier /etc/default/varnish :
DAEMON_OPTS="-a :6081              -T localhost:6082              -p http_req_hdr_len=8192              -p http_req_size=16384              -b 127.0.0.1:8080"
sudo systemctl reload varnish
```

## 3. Tester le cache

```
curl -I http://example.com/
```

Vérifiez l’en-tête `X-Cache: HIT` ou `MISS`.

## 4. Vidange du cache

Autorisez les PURGE depuis localhost :

```
curl -X PURGE http://example.com/
```

## 5. Optimisation avancée

- Ajustez `beresp.ttl` pour différents chemins.
- Excluez les requêtes dynamiques (`req.url ~ "^/api"`).
- Surveillez Varnish avec **varnishstat** et **varnishncsa**.

## Conclusion

Cette architecture Nginx + Varnish permet de réduire significativement les temps de réponse et la charge serveur, idéale pour les sites à fort trafic.