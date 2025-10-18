---
link: https://www.shpv.fr/blog/sysctl-optimisation/
byline: SHPV
site: SHPV
date: 2025-07-20T12:00
excerpt: Guide complet sur l'optimisation des performances Linux en ajustant les paramètres kernel via sysctl pour améliorer les performances réseau, mémoire et système.
twitter: https://twitter.com/@SHPVFR
tags:
  - linux
  - optimisation
slurped: 2025-10-12T18:39
title: "Optimiser Linux avec sysctl : Ajuster les paramètres kernel pour la performance"
---

Le réglage des paramètres du noyau Linux via `sysctl` est essentiel pour optimiser les performances et la stabilité du système. Ce guide vous présente les paramètres les plus couramment utilisés pour ajuster les performances réseau, mémoire et générales.

## Prérequis

- Accès root ou sudo sur un serveur Linux (Debian, Ubuntu, CentOS, Rocky Linux)

## 1. Modifier les paramètres kernel temporairement

Vous pouvez tester les réglages immédiatement (temporaire jusqu'au reboot):

```
sudo sysctl -w net.ipv4.ip_forward=1
```

## 2. Rendre les modifications persistantes

Éditez le fichier `/etc/sysctl.conf` ou ajoutez un fichier personnalisé dans `/etc/sysctl.d/`:

```
sudo nano /etc/sysctl.d/99-custom.conf
```

Exemples de réglages courants:

```
# Réseau
net.ipv4.ip_forward = 1
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 65535

# Mémoire
vm.swappiness = 10
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5

# Système
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288
```

Chargez les nouveaux paramètres immédiatement:

```
sudo sysctl -p /etc/sysctl.d/99-custom.conf
```

## 3. Paramètres réseau avancés

Optimisez votre stack TCP/IP pour améliorer les performances:

```
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_rmem = 4096 87380 6291456
net.ipv4.tcp_wmem = 4096 65536 6291456
```

## 4. Optimisation des performances mémoire

Réduisez l'utilisation du swap et ajustez la gestion de la mémoire:

```
vm.swappiness = 10
vm.vfs_cache_pressure = 50
```

## 5. Optimiser le nombre de fichiers ouverts

Ajustez le nombre maximal de fichiers ouverts:

```
fs.file-max = 2097152
```

Vérifiez avec:

```
cat /proc/sys/fs/file-max
```

## 6. Vérification et surveillance

Consultez les valeurs actuelles avec:

```
sudo sysctl -a
```

Surveillez avec des outils comme `htop`, `free -m` ou Prometheus + Grafana.

## Conclusion

L'ajustement régulier des paramètres kernel via sysctl est essentiel pour maintenir et améliorer les performances de votre serveur Linux. Personnalisez les réglages selon votre cas d'usage pour tirer le meilleur parti de votre infrastructure.