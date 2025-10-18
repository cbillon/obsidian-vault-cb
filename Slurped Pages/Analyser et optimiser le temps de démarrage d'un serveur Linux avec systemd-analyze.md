---
link: https://www.shpv.fr/blog/systemd-analyze-boot-performance/
byline: SHPV
site: SHPV
date: 2025-07-21T12:00
excerpt: Guide détaillé pour mesurer, comprendre et accélérer le démarrage d’un serveur Linux grâce à systemd-analyze et à l’optimisation des services systemd.
twitter: https://twitter.com/@SHPVFR
tags:
  - linux
  - optimisation
slurped: 2025-10-12T18:32
title: Analyser et optimiser le temps de démarrage d'un serveur Linux avec systemd-analyze
---

L’optimisation du temps de démarrage d’un serveur Linux est essentielle pour garantir la disponibilité et la réactivité de vos services. Grâce à l’outil intégré `systemd-analyze`, il est possible de mesurer et d’améliorer chaque étape du boot, en identifiant précisément les services responsables des lenteurs.

## Prérequis

Avant de commencer, assurez-vous d’avoir :

- Un serveur Linux moderne utilisant systemd (Debian, Ubuntu, Rocky, AlmaLinux…)
- Accès root ou sudo

## Analyse du temps de démarrage global

Utilisez la commande suivante pour obtenir le temps total de boot :

```
systemd-analyze
```

Vous obtenez :

```
Startup finished in 2.210s (kernel) + 3.055s (userspace) = 5.265s
```

## Détecter les services lents

Listez les services qui ralentissent le démarrage :

```
systemd-analyze blame
```

Résultat typique :

```
6.301s NetworkManager.service
3.009s mariadb.service
1.187s systemd-logind.service
```

## Visualiser la chaîne critique du démarrage

Pour afficher l’ordre exact des services critiques :

```
systemd-analyze critical-chain
```

Exemple :

```
graphical.target @6.308s
└─multi-user.target @6.307s
  └─network.target @6.306s
    └─NetworkManager.service @1.113s +5.193s
```

## Générer un graphe visuel du boot

Créez un diagramme SVG pour l’analyse visuelle :

```
systemd-analyze plot > boot.svg
```

Ouvrez ce fichier dans votre navigateur pour une vue graphique complète.

## Optimisation du temps de boot

- **Désactivez les services inutiles** :
    
    ```
    systemctl disable bluetooth.service
    ```
    
- **Réorganisez les dépendances** pour paralléliser le démarrage
- **Surveillez les logs de boot** :
    
    ```
    journalctl -b
    ```
    

## Cas concret : accélérer le démarrage d’un serveur

- Passez en mode multi-user pour éviter l’interface graphique :
    
    ```
    systemctl set-default multi-user.target
    ```
    
- Désactivez les services non critiques (cups, ModemManager, plymouth, bluetooth…)
- Répétez l’analyse après chaque modification pour suivre l’amélioration

## Conclusion

Avec `systemd-analyze` et une gestion rigoureuse des services systemd, vous disposez de tout le nécessaire pour identifier, diagnostiquer et optimiser le temps de démarrage de vos serveurs Linux. Un boot rapide améliore la disponibilité, la sécurité et la satisfaction de vos utilisateurs.