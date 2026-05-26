---
tags:
  - linux
  - optimisation
---
L'optimisation du temps de démarrage d'un serveur Linux est essentielle pour garantir la disponibilité et la réactivité de vos services. Grâce à l'outil intégré `systemd-analyze`, il est possible de mesurer et d'améliorer chaque étape du boot en identifiant précisément les services responsables des lenteurs. Pour les bases de systemd, consultez notre [guide complet sur les services systemd](https://www.shpv.fr/blog/systemd-services-2026/).

#### Prérequis

Avant de commencer, assurez-vous d'avoir :

- Un serveur Linux moderne utilisant systemd (Debian, Ubuntu, Rocky, AlmaLinux…)
- Accès root ou sudo

#### Analyse du temps de démarrage global

Utilisez la commande suivante pour obtenir le temps total de boot :

```bash
systemd-analyze
```

Vous obtenez :

```text
Startup finished in 2.210s (kernel) + 3.055s (userspace) = 5.265s
```

#### Détecter les services lents

Listez les services qui ralentissent le démarrage :

```bash
systemd-analyze blame
```

Résultat typique :

```text
6.301s NetworkManager.service
3.009s mariadb.service
1.187s systemd-logind.service
```

#### Visualiser la chaîne critique du démarrage

Pour afficher l'ordre exact des services critiques :

```bash
systemd-analyze critical-chain
```

Exemple :

```text
graphical.target @6.308s
└─multi-user.target @6.307s
  └─network.target @6.306s
    └─NetworkManager.service @1.113s +5.193s
```

#### Générer un graphe visuel du boot

Créez un diagramme SVG pour l'analyse visuelle :

```bash
systemd-analyze plot > boot.svg
```

Ouvrez ce fichier dans votre navigateur pour une vue graphique complète.

#### Optimisation du temps de boot

- **Désactivez les services inutiles** :
    
    ```bash
    systemctl disable bluetooth.service
    ```
    
- **Réorganisez les dépendances** pour paralléliser le démarrage
- **Surveillez les logs de boot** :
    
    ```bash
    journalctl -b
    ```
    

Pour une optimisation complète du système, consultez notre guide sur le [tuning des performances Linux](https://www.shpv.fr/blog/linux-performance/).

#### Cas concret : accélérer le démarrage d'un serveur

- Passez en mode multi-user pour éviter l'interface graphique :
    
    ```bash
    systemctl set-default multi-user.target
    ```
    
- Désactivez les services non critiques (cups, ModemManager, plymouth, bluetooth…)
- Répétez l'analyse après chaque modification pour suivre l'amélioration

#### Conclusion

Avec `systemd-analyze` et une gestion rigoureuse des services systemd, vous disposez de tout le nécessaire pour identifier, diagnostiquer et optimiser le temps de démarrage de vos serveurs Linux. Un boot rapide améliore la disponibilité, la sécurité et la satisfaction de vos utilisateurs.