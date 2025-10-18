---
link: https://www.shpv.fr/blog/ssd-trim/
byline: SHPV
site: SHPV
date: 2025-07-10T12:00
excerpt: Guide rapide pour automatiser le trimming des SSD sous Linux à l'aide de fstrim et d'un timer systemd, garantissant des performances optimales et une longévité accrue.
twitter: https://twitter.com/@SHPVFR
tags:
  - ssd
slurped: 2025-10-12T18:53
title: Optimiser les performances des SSD avec fstrim et systemd-timer
---

Publié le 10 juillet 2025

Maintenance

Performance

Stockage

Les SSD nécessitent un **trimming** régulier pour maintenir leurs performances et prolonger leur durée de vie. Grâce à la commande `fstrim` et aux **timers systemd**, vous pouvez automatiser cette tâche simplement.

## Prérequis

- Serveur Linux (Debian, Ubuntu, RHEL, CentOS)
- Disque SSD détecté par le système

## 1. Tester manuellement le trim

```
sudo fstrim -v /
```

Vous verrez la quantité de données libérées, par exemple :

```
/ trimmed at 57.5 GiB
```

## 2. Activer le timer systemd

La plupart des distributions modernes installent déjà `fstrim.timer`. Vérifiez son statut :

```
systemctl status fstrim.timer
```

Pour l’activer :

```
sudo systemctl enable --now fstrim.timer
```

Ce timer exécute `fstrim.service` chaque semaine par défaut. Vous pouvez ajuster la fréquence :

```
sudo systemctl edit fstrim.timer --full
```

Modifiez `OnCalendar` pour un intervalle différent (quotidien, mensuel, etc.).

## 3. Vérifier le bon fonctionnement

Consultez les logs :

```
journalctl -u fstrim.service
```

### Exemple de log

```
Jul 10 03:00:01 server systemd[1]: Started Discard unused blocks.
Jul 10 03:00:01 server fstrim[1234]: /: 57.5 GiB trimmed
```

## 4. Bonnes pratiques

- Exécutez `fstrim` pendant les heures creuses pour minimiser l’impact sur les performances.
- Surveillez le log pour détecter toute erreur de disque.
- Combinez avec des outils de monitoring (Prometheus, Nagios) pour alerter en cas d’échec.

## Conclusion

Automatiser le trimming des SSD avec `fstrim` et `systemd-timer` est une méthode simple et efficace pour maintenir les performances de vos serveurs Linux.

###### Besoin d'aide sur ce sujet ?

Notre équipe d'experts est là pour vous accompagner dans vos projets.

[Contactez-nous](https://www.shpv.fr/demander-un-devis)

---

##### Articles similaires qui pourraient vous intéresser