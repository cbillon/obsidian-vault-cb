---
tags:
  - nginx
  - configuration
  - reverse-proxy
---

---
Vous avez une application qui tourne sur le port 3000. Vous voulez la servir sur le port 443 avec du SSL, gérer plusieurs sites sur le même serveur, bloquer les requêtes abusives, et peut-être faire du load balancing entre plusieurs instances. Nginx fait tout ça. Mais entre installer Nginx et le configurer correctement pour de la production, il y a un fossé que beaucoup de tutos ne franchissent pas. Ce guide vous emmène là où ça compte vraiment.

#### Prérequis

- Linux : Debian, Ubuntu, RHEL, Rocky
- Un domaine pointant vers votre IP publique
- Accès root
- Une application backend qui écoute sur un port local (Node.js, Python, PHP-FPM, etc.)
- `certbot` pour la gestion SSL avec Let's Encrypt

#### Installation

```bash
# Debian / Ubuntu
sudo apt update && sudo apt install -y nginx certbot python3-certbot-nginx

# RHEL / Rocky
sudo yum install -y nginx certbot python3-certbot-nginx
sudo systemctl enable --now nginx
```

```bash
# Vérifier que Nginx tourne
sudo systemctl status nginx
nginx -t  # teste la syntaxe de la config sans redémarrer
```

#### La structure de configuration

Tout est dans `/etc/nginx/`. Ne touchez pas à `nginx.conf` directement, utilisez des fichiers séparés dans `sites-enabled/` (Debian) ou `conf.d/` (RHEL).

```
/etc/nginx/
├── nginx.conf              ← config globale (ne pas toucher en dehors de http {}
├── sites-available/        ← vos configs de sites (inactifs)
├── sites-enabled/          ← liens symboliques vers sites-available (actifs)
├── conf.d/                 ← alternative à sites-enabled (RHEL style)
└── mime.types
```

#### Config de base : reverse proxy vers une application

```nginx
# /etc/nginx/sites-available/myapp.conf

server {
    listen 80;
    server_name myapp.exemple.com;

    # Rediriger tout vers HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name myapp.exemple.com;

    # SSL — sera complété par certbot
    # ssl_certificate /etc/letsencrypt/live/myapp.exemple.com/fullchain.pem;
    # ssl_certificate_key /etc/letsencrypt/live/myapp.exemple.com/privkey.pem;

    # Logs
    access_log /var/log/nginx/myapp-access.log;
    error_log  /var/log/nginx/myapp-error.log;

    location / {
        proxy_pass http://127.0.0.1:3000;

        # Headers essentiels pour le backend
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeouts
        proxy_connect_timeout 10s;
        proxy_read_timeout    30s;
        proxy_send_timeout    30s;
    }
}
```

```bash
# Activer le site
sudo ln -s /etc/nginx/sites-available/myapp.conf /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

#### SSL avec Let's Encrypt

```bash
# Obtenir et installer le certificat automatiquement
sudo certbot --nginx -d myapp.exemple.com -d www.myapp.exemple.com

# Certbot modifie votre config nginx automatiquement avec les chemins SSL
# Vérifier que le renouvellement automatique est actif
sudo certbot renew --dry-run
```

Certbot ajoute automatiquement les directives `ssl_certificate` et `ssl_certificate_key` à votre config. Il installe aussi un timer systemd qui renouvelle les certificats avant qu'ils expirent.

#### Multi-sites : plusieurs applications sur le même serveur

Chaque application a son propre fichier de config. Nginx gère le routing par `server_name`.

```nginx
# /etc/nginx/sites-available/api.conf
server {
    listen 443 ssl;
    server_name api.exemple.com;

    access_log /var/log/nginx/api-access.log;
    error_log  /var/log/nginx/api-error.log;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```nginx
# /etc/nginx/sites-available/blog.conf
server {
    listen 443 ssl;
    server_name blog.exemple.com;

    access_log /var/log/nginx/blog-access.log;
    error_log  /var/log/nginx/blog-error.log;

    root /var/www/blog;
    index index.html;

    # Site statique — pas de proxy_pass nécessaire
    location / {
        try_files $uri $uri/ =404;
    }
}
```

#### Load Balancing : répartir la charge

Si votre application tourne sur plusieurs instances, Nginx répartit automatiquement les requêtes.

```nginx
# /etc/nginx/conf.d/upstream.conf

upstream app_backend {
    # Round-robin par défaut
    server 127.0.0.1:3001 weight=3;
    server 127.0.0.1:3002 weight=2;
    server 127.0.0.1:3003 weight=1;

    # Keepalive — réutilise les connexions vers le backend
    keepalive 32;
}
```

```nginx
# Dans votre server block
location / {
    proxy_pass http://app_backend;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # Nécessaire avec keepalive
    proxy_http_version 1.1;
    proxy_set_header Connection "";
}
```

##### Stratégies de load balancing disponibles

```nginx
upstream app_backend {
    # Par IP du client — même client = même backend (sessions)
    ip_hash;

    # Par hash de l'URI — même URL = même backend (cache cohérent)
    # hash $request_uri consistent;

    server 127.0.0.1:3001;
    server 127.0.0.1:3002;

    # Marquer un backend comme down sans le supprimer
    # server 127.0.0.1:3003 down;

    # Nombre d'échecs avant de considérer le backend comme down
    # server 127.0.0.1:3001 max_conns=100 fail_timeout=30s max_fails=3;
}
```

#### Rate Limiting : bloquer les requêtes abusives

C'est ici que Nginx devient vraiment utile pour la sécurité. Vous définissez une limite de requêtes par IP, et Nginx fait le reste.

```nginx
# Dans nginx.conf, dans le bloc http {}
http {
    # Créer une zone de rate limiting — 10 Mo en mémoire
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

    # Zone pour les pages HTML — plus permissif
    limit_req_zone $binary_remote_addr zone=web_limit:10m rate=30r/s;
}
```

```nginx
# Dans votre server block
location /api/ {
    limit_req zone=api_limit burst=20 nodelay;
    # burst=20 : permet 20 requêtes en rafale avant de bloquer
    # nodelay : refuse immédiatement au-delà du burst (pas de file d'attente)

    proxy_pass http://app_backend;
    proxy_set_header Host $host;
}

location / {
    limit_req zone=web_limit burst=50 nodelay;
    try_files $uri $uri/ =404;
}
```

```bash
# Vérifier les requêtes bloquées dans les logs
grep "limiting requests" /var/log/nginx/error.log
```

#### Cache statique : ne pas re-servir les mêmes fichiers

Les fichiers statiques (images, CSS, JS) ne changent pas souvent. Dites-le au navigateur.

```nginx
# Cache agressif pour les assets statiques
location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2|svg)$ {
    expires 30d;
    add_header Cache-Control "public, immutable";
    add_header Vary Accept-Encoding;
}

# Cache plus court pour les pages HTML
location ~* \.html$ {
    expires 1h;
    add_header Cache-Control "public, must-revalidate";
}
```

#### Headers de sécurité

Une poignée de headers qui renforcent la sécurité sans configuration complexe.

```nginx
# Dans votre server block
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header X-Content-Type-Options nosniff always;
add_header X-Frame-Options SAMEORIGIN always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;

# Cacher la version de Nginx
server_tokens off;
```

#### Logs structurés : ce qui compte vraiment

Les logs par défaut de Nginx sont utiles mais limités. Customisez le format pour avoir plus d'infos.

```nginx
# Dans nginx.conf, bloc http {}
http {
    log_format structured '$remote_addr - $remote_user [$time_local] '
                          '"$request" $status $body_bytes_sent '
                          '"$http_referer" "$http_user_agent" '
                          'rt=$request_time '
                          'upstream_rt=$upstream_response_time '
                          'upstream_addr=$upstream_addr';

    access_log /var/log/nginx/access.log structured;
}
```

```bash
# Analyser les requêtes les plus lentes
awk '{print $NF}' /var/log/nginx/access.log | sort -n | tail -20

# IPs les plus actives
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10

# Codes d'erreur
grep -oP '"[A-Z]+ [^"]*" \K[0-9]+' /var/log/nginx/access.log | sort | uniq -c | sort -rn
```

#### Tableau récapitulatif, config rapide

|Fonctionnalité|Directive clé|
|---|---|
|Reverse proxy|`proxy_pass http://backend:port`|
|SSL Let's Encrypt|`certbot --nginx -d domaine.com`|
|Rate limiting|`limit_req_zone` + `limit_req`|
|Load balancing|`upstream { server ...; }`|
|Cache statique|`expires` + `Cache-Control`|
|Headers sécurité|`add_header Strict-Transport-Security`|
|Multi-sites|Fichiers séparés dans `sites-enabled/`|

#### Pièges classiques

Le premier piège, c'est de modifier `nginx.conf` directement au lieu de créer des fichiers séparés dans `sites-enabled/`. À la première mise à jour du paquet, votre config part. Le deuxième, c'est d'oublier `proxy_set_header X-Real-IP` - votre application verra toutes les requêtes comme venant de `127.0.0.1`. Et enfin, ne jamais oublier `nginx -t` avant un `systemctl reload` - une erreur de syntaxe avec un `reload` direct, et Nginx ne démarre plus.

#### Conclusion

Nginx est probablement le logiciel le plus utilisé dans votre infrastructure sans que vous ne le réalisiez vraiment. Reverse proxy, SSL, rate limiting, load balancing, cache, tout ça en un seul fichier de config. Maîtrisez ces blocs de base, et vous couvrez 90 % des besoins d'une infrastructure en production.

Complétez votre configuration en apprenant le [caching Nginx](https://www.shpv.fr/blog/nginx-cache/), en explorant [HTTP/2 et Brotli](https://www.shpv.fr/blog/nginx-http2-brotli/), et en comprenant comment gérer les erreurs [502](https://www.shpv.fr/blog/nginx-502-bad-gateway/) et [504](https://www.shpv.fr/blog/nginx-504-gateway-timeout/). La suite logique ? Mettre en place une supervision avec [Prometheus + Grafana](https://www.shpv.fr/blog/deployer-stack-monitoring/) pour visualiser vos métriques.