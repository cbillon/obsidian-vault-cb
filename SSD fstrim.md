---
tags:
  - linux
  - ssd
---
Les SSD nécessitent un **trimming** régulier pour maintenir leurs performances et prolonger leur durée de vie. Grâce à la commande `fstrim` et aux **timers systemd**, vous pouvez automatiser cette tâche simplement.

#### Prérequis

- Serveur Linux (Debian, Ubuntu, RHEL, CentOS)
- Disque SSD détecté par le système

#### 1. Tester manuellement le trim

```bash
sudo fstrim -v /
```

Vous verrez la quantité de données libérées, par exemple :

```
/ trimmed at 57.5 GiB
```

#### 2. Activer le timer systemd

La plupart des distributions modernes installent déjà `fstrim.timer`. Vérifiez son statut :

```bash
systemctl status fstrim.timer
```

Pour l'activer :

```bash
sudo systemctl enable --now fstrim.timer
```

Ce timer exécute `fstrim.service` chaque semaine par défaut. Vous pouvez ajuster la fréquence :

```bash
sudo systemctl edit fstrim.timer --full
```

Modifiez `OnCalendar` pour un intervalle différent (quotidien, mensuel, etc.).

#### 3. Vérifier le bon fonctionnement

Consultez les logs :

```bash
journalctl -u fstrim.service
```

##### Exemple de log

```
Jul 10 03:00:01 server systemd[1]: Started Discard unused blocks.
Jul 10 03:00:01 server fstrim[1234]: /: 57.5 GiB trimmed
```

#### 4. Bonnes pratiques

- Exécutez `fstrim` pendant les heures creuses pour minimiser l'impact sur les performances.
- Surveillez le log pour détecter toute erreur de disque.
- Combinez avec des outils de monitoring (Prometheus, Nagios) pour alerter en cas d'échec.

#### Lectures complémentaires

Découvrez [optimisation NVMe complète](https://www.shpv.fr/blog/nvme-optimization/) pour une optimisation plus approfondie. Pour les options de montage détaillées, consultez [montage avec discard](https://www.shpv.fr/blog/montage-filesystems-linux-2026/).

#### Conclusion

Automatiser le trimming des SSD avec `fstrim` et `systemd-timer` est une méthode simple et efficace pour maintenir les performances de vos serveurs Linux.