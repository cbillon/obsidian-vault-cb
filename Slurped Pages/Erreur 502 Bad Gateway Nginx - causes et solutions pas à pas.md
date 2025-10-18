---
link: https://www.shpv.fr/blog/nginx-502-bad-gateway/
byline: SHPV
site: SHPV
date: 2025-09-11T13:30
excerpt: Découvrez les causes courantes de l’erreur 502 Bad Gateway sous Nginx et les étapes concrètes pour la diagnostiquer et la corriger efficacement.
twitter: https://twitter.com/@SHPVFR
tags:
  - nginx
slurped: 2025-10-12T18:21
title: "Erreur 502 Bad Gateway Nginx : causes et solutions pas à pas"
---

L’erreur **502 Bad Gateway** est l’un des messages les plus frustrants rencontrés lors de l’administration de serveurs web Nginx.  
Elle indique que le serveur Nginx, en tant que proxy inverse, n’a pas réussi à obtenir une réponse valide du serveur applicatif (backend).

Dans cet article, nous allons analyser les **causes principales** de cette erreur et fournir des **solutions pas à pas**.

## Plan de l’article

- Qu’est-ce qu’une erreur 502 Bad Gateway ?
- Causes fréquentes côté serveur applicatif
- Problèmes de configuration Nginx
- Limites système et ressources
- Étapes de diagnostic pas à pas
- Solutions pratiques et correctives
- Bonnes pratiques pour prévenir l’erreur 502

---

## Qu’est-ce qu’une erreur 502 Bad Gateway ?

Un code HTTP **502** signifie que le proxy (ici Nginx) a reçu une réponse invalide ou aucune réponse du serveur en amont (PHP-FPM, Node.js, Python, etc.).  
En clair : Nginx fonctionne, mais l’application derrière lui est injoignable ou plante.

---

## Causes fréquentes

### 1. Problème côté application

- Service backend (ex. PHP-FPM, Node.js, Gunicorn) arrêté ou planté.
- Temps de réponse trop long → dépassement du timeout.
- Crash applicatif lié à une erreur de code.

### 2. Mauvaise configuration Nginx

- Mauvais `fastcgi_pass` ou `proxy_pass`.
- Sockets Unix inexistants ou avec de mauvais droits.
- Timeout trop courts (`proxy_read_timeout`, `fastcgi_read_timeout`).

### 3. Ressources système saturées

- CPU ou RAM insuffisants.
- Trop de connexions simultanées → limites `worker_connections`.
- Limite `open files` atteinte.

---

## Diagnostic pas à pas

1. **Vérifier les logs Nginx** :

```
tail -f /var/log/nginx/error.log
```

2. **Vérifier l’état du service backend** :

Exemple avec PHP-FPM :

```
systemctl status php8.2-fpm
```

3. **Tester la connectivité depuis Nginx** :

```
curl -I http://127.0.0.1:9000
```

4. **Augmenter temporairement les timeouts** :

```
proxy_read_timeout 120;
fastcgi_read_timeout 120;
```

---

## Solutions pratiques

- Redémarrer le service applicatif (PHP-FPM, Node.js, etc.).
- Vérifier les permissions sur les sockets (`/run/php/php8.2-fpm.sock`).
- Corriger la directive `proxy_pass` ou `fastcgi_pass`.
- Augmenter les timeouts dans la conf Nginx.
- Optimiser les ressources serveur (RAM, CPU, limites `ulimit`).

Exemple Nginx (PHP-FPM) :

```
location ~ \.php$ {
    include fastcgi_params;
    fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_read_timeout 120;
}
```

---

## Bonnes pratiques de prévention

- **Monitoring** des services backend (Prometheus, Grafana, Icinga).
- **Supervision automatique** avec `systemd` (`Restart=always`).
- Mise en place d’un **load balancer** avec plusieurs backends.
- Optimiser le code applicatif pour éviter les lenteurs.
- Surveiller les ressources système et ajuster les limites.

---

## Conclusion

L’erreur **502 Bad Gateway Nginx** est généralement liée à un problème entre Nginx et son backend.  
En suivant une méthodologie claire de diagnostic (logs, services, configuration, ressources), il est possible d’identifier rapidement la cause et de corriger l’erreur.

Avec de bonnes pratiques (monitoring, redondance, optimisation), vous réduirez fortement le risque de rencontrer ce problème en production.