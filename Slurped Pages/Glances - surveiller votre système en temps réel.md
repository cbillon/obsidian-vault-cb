---
link: https://blog.stephane-robert.info/docs/outils/systeme/glances/
byline: Stéphane Robert
site: Stéphane ROBERT - DevSecOps Website
date: 2025-04-11T02:00
excerpt: "Guide complet Glances 4.4 : surveillez CPU, RAM, réseau, disques et
  conteneurs en une seule interface. Installation, modes web/client-serveur,
  export Prometheus et bonnes pratiques."
twitter: https://twitter.com/@stephane_robert
slurped: 2026-03-03T18:45
title: "Glances : surveiller votre système en temps réel"
---

**Glances affiche en temps réel l’état complet de votre système : CPU, mémoire, disques, réseau, processus et conteneurs [Docker](https://blog.stephane-robert.info/docs/conteneurs/moteurs-conteneurs/docker/) — le tout dans une seule interface.** Contrairement à `top` ou `htop` qui se limitent aux processus, Glances centralise toutes les métriques système et peut les exporter vers [Prometheus](https://blog.stephane-robert.info/docs/observer/metriques/prometheus/) ou InfluxDB. Ce guide vous apprend à l’installer, le configurer et l’utiliser pour surveiller vos serveurs efficacement.

- **Installer Glances** sur [Linux](https://blog.stephane-robert.info/docs/admin-serveurs/linux/), macOS ou Windows
- **Utiliser les 4 modes** : local, web, client/serveur et fetch
- **Configurer les alertes** et personnaliser l’affichage
- **Exporter les métriques** vers Prometheus ou InfluxDB
- **Surveiller vos conteneurs** Docker et [Podman](https://blog.stephane-robert.info/docs/conteneurs/moteurs-conteneurs/podman/)

Imaginez que vous devez diagnostiquer un serveur lent. Avec `top`, vous voyez les processus. Avec `htop`, vous avez une vue plus jolie des mêmes processus. Mais pour voir l’état du réseau, vous lancez `iftop`. Pour les disques, `iotop`. Pour les températures, `sensors`. Glances rassemble **tout cela dans une seule interface**.

|Outil|CPU|RAM|Processus|Réseau|Disques|Conteneurs|Alertes|
|---|---|---|---|---|---|---|---|
|`top`|✅|✅|✅|❌|❌|❌|❌|
|`htop`|✅|✅|✅|❌|❌|❌|❌|
|**Glances**|✅|✅|✅|✅|✅|✅|✅|

**Analogie** : Si `top` est un thermomètre, Glances est un tableau de bord complet de voiture avec compteur de vitesse, jauge d’essence, température moteur et voyants d’alerte.

Glances collecte les données depuis plusieurs sources (psutil, Docker, capteurs matériels) et les expose via différentes interfaces :

![Architecture Glances : sources de données, core et interfaces de sortie](https://blog.stephane-robert.info/_astro/glances-architecture.BJmzsMHv_Z2cf7Lt.svg)

Glances propose quatre façons de surveiller vos systèmes selon vos besoins :

![Les 4 modes de Glances : local, web, client/serveur et fetch](https://blog.stephane-robert.info/_astro/glances-modes.CT2871dB_Z1l0e3p.svg)

|Mode|Commande|Usage|
|---|---|---|
|**Local**|`glances`|Surveillance rapide en terminal|
|**Web**|`glances -w`|Accès navigateur, surveillance à distance|
|**Client/Serveur**|`glances -s` / `glances -c <ip>`|Centraliser plusieurs machines|
|**Fetch**|`glances --fetch`|Snapshot instantané style neofetch|

Glances est écrit en Python. La méthode recommandée est d’utiliser **[pipx](https://blog.stephane-robert.info/docs/developper/programmation/python/pipx/)** ou **mise** pour l’isoler des dépendances système.

pipx installe les applications Python dans des environnements isolés :

```
# Installer pipx si nécessairepython3 -m pip install --user pipxpython3 -m pipx ensurepath# Installer Glances avec toutes les fonctionnalitéspipx install 'glances[all]'
```

L’option `[all]` active le support réseau avancé, les capteurs matériels et les intégrations (Prometheus, InfluxDB, etc.).

Le mode par défaut affiche une interface TUI (Text User Interface) complète :

L’interface affiche en temps réel :

- **En haut** : informations système, uptime, charge CPU
- **Au centre** : mémoire, swap, réseau, disques I/O
- **En bas** : liste des processus triés par consommation

|Touche|Action|
|---|---|
|`h`|Afficher l’aide|
|`q`|Quitter|
|`c`|Trier par CPU|
|`m`|Trier par mémoire|
|`i`|Trier par I/O|
|`p`|Trier par nom|
|`a`|Mode automatique (tri dynamique)|
|`d`|Afficher/masquer les disques|
|`n`|Afficher/masquer le réseau|
|`s`|Afficher/masquer les capteurs|
|`2`|Désactiver la sidebar gauche|
|`1`|Afficher CPU par cœur|
|`/`|Filtrer les processus|

Le mode `--fetch` (nouveauté v4.4) affiche un résumé style neofetch, idéal pour un diagnostic rapide ou l’intégration dans des scripts :

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━✨ master1 | Uptime: 3 days, 2:57:34⚙️  Ubuntu 24.04 64bit / Linux 6.8.0-90-generic💡 LOAD     0.18/min1 | 0.28/min5 | 0.43/min15⚡ CPU      ■□□□□□□□□□□□□□□□□□ 6.3% of 16 cores🧠 MEM      ■■□□□□□□□□□□□□□□□□ 12.8% (6.01G / 46.8G)💾 DISK     ■■■■■■■■■■□□□□□□□□ 56.1% (158G / 295G) for /📡 NET      ↓ 16Kb/s ↑ 13Kb/s for enp2s0🔥 TOP PROCESS by CPU1️⃣ node                    ⚡ 0.0% CPU           🧠 1.67GB MEM2️⃣ node                    ⚡ 0.0% CPU           🧠 766MB MEM3️⃣ node                    ⚡ 0.0% CPU           🧠 400MB MEM━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Le mode web lance un serveur HTTP avec une interface accessible depuis n’importe quel navigateur :

```
Glances Web Server started on http://0.0.0.0:61208/
```

Ouvrez ensuite [http://localhost:61208](http://localhost:61208/) dans votre navigateur.

Pour protéger l’accès :

```
glances -w --username --password
```

Glances vous demandera de créer un mot de passe au premier lancement.

Pour surveiller plusieurs machines depuis un poste central :

1. **Sur chaque serveur à surveiller**, lancez Glances en mode serveur :
    
    ```
    Glances XML-RPC server is running on 0.0.0.0:61209
    ```
    
2. **Depuis votre poste**, connectez-vous en mode client :
    
    L’interface affiche les métriques du serveur distant.
    
3. **Pour découvrir les serveurs** automatiquement sur le réseau :
    
    Une liste des instances Glances détectées s’affiche.
    

Glances peut exporter ses données vers différents backends pour du monitoring long terme.

Exportez les métriques dans un fichier CSV :

```
glances --export csv --export-csv-file /tmp/metrics.csv --stop-after 10
```

Le fichier contient toutes les métriques avec un timestamp :

```
timestamp,cpu.total,cpu.user,cpu.system,mem.percent,mem.used,...2026-01-26 10:01:54,2.0,1.2,0.8,12.8,6437355520,...
```

Pour récupérer des métriques spécifiques :

```
glances --stdout cpu,mem --stop-after 2
```

```
cpu: {'total': 3.0, 'user': 4.3, 'system': 0.9, 'idle': 93.9, ...}mem: {'total': 50282524672, 'percent': 12.7, 'used': 6405283840, ...}
```

Format CSV pour un traitement simplifié :

```
glances --stdout-csv cpu.total,mem.percent --stop-after 3
```

```
cpu.total,mem.percent2.0,12.81.6,12.8
```

Glances expose un endpoint compatible Prometheus :

1. **Créez le fichier de configuration** `~/.config/glances/glances.conf` :
    
    ```
    [prometheus]host=0.0.0.0port=9091prefix=glanceslabels=instance:myserver
    ```
    
2. **Lancez Glances avec l’export Prometheus** :
    
    ```
    glances --export prometheus
    ```
    
3. **Configurez Prometheus** pour scraper les métriques :
    
    ```
    scrape_configs:  - job_name: 'glances'    scrape_interval: 15s    static_configs:      - targets: ['localhost:9091']
    ```
    

Les métriques sont disponibles sur `http://localhost:9091/metrics`.

Glances supporte InfluxDB 1.x, 2.x et 3.x :

```
[influxdb2]host=localhostport=8086protocol=httporg=myorgbucket=glancestoken=my-token
```

```
glances --export influxdb2
```

Glances est modulaire. Voici les principaux plugins :

```
Plugins list: alert, amps, cloud, connections, containers, core, cpu, diskio,folders, fs, gpu, help, ip, irq, load, mem, memswap, network, now, percpu,ports, processcount, processlist, programlist, psutilversion, quicklook, raid,sensors, smart, system, uptime, version, vms, wifiExporters list: cassandra, couchdb, csv, duckdb, elasticsearch, graph, graphite,influxdb, influxdb2, influxdb3, json, kafka, mongodb, mqtt, opentsdb, prometheus,rabbitmq, restful, riemann, statsd, timescaledb, zeromq
```

Glances détecte automatiquement les conteneurs Docker et Podman :

```
glances# Appuyez sur 'D' pour afficher/masquer les conteneurs
```

Pour chaque conteneur, vous voyez :

- Nom et statut
- Utilisation CPU et mémoire
- Ports exposés (nouveauté v4.3)

Depuis la version 4.3.2, Glances peut surveiller les machines virtuelles [KVM](https://blog.stephane-robert.info/docs/virtualiser/type1/kvm/) :

```
glances --enable-plugin vms
```

Le fichier de configuration permet de personnaliser le comportement de Glances.

- **Linux** : `~/.config/glances/glances.conf`
- **macOS** : `~/Library/Application Support/glances/glances.conf`
- **Windows** : `%APPDATA%\glances\glances.conf`

```
[global]# Rafraîchissement toutes les 3 secondesrefresh=3# Vérifier les mises à jourcheck_update=true[outputs]# Nombre maximum de processus affichésmax_processes_display=30[cpu]# Seuils d'alerte CPUuser_warning=70user_critical=90[mem]# Seuils d'alerte mémoirewarning=70critical=90[fs]# Masquer certains systèmes de fichiershide=/boot.*,/snap.*[network]# Masquer certaines interfaceshide=lo,docker.*,veth.*[containers]# Activer la surveillance des conteneursdisable=false
```

Glances utilise un code couleur pour signaler les problèmes :

|Couleur|Signification|Action|
|---|---|---|
|🟢 Vert|Normal|Rien à faire|
|🟡 Jaune|Attention (warning)|Surveiller|
|🔴 Rouge|Critique|Investiguer immédiatement|

Les seuils sont configurables dans `glances.conf` pour chaque métrique.

|Symptôme|Cause probable|Solution|
|---|---|---|
|`ModuleNotFoundError: psutil`|Dépendance manquante|Réinstaller avec `pipx install glances[all]`|
|Interface web inaccessible|Pare-feu|Ouvrir le port 61208|
|Pas de données conteneurs|Docker socket|Vérifier les permissions sur `/var/run/docker.sock`|
|Capteurs non détectés|Dépendance manquante|Installer `lm-sensors` sur Linux|
|Erreur API Prometheus|Configuration|Vérifier `[prometheus]` dans glances.conf|

```
# Vérifier la version et les dépendancesglances --version# Mode debug pour voir les erreursglances -d# Lister les modules disponiblesglances --modules-list# Tester l'export sans interfaceglances --stdout system --stop-after 1
```

- **Ne pas exposer** le serveur web sur Internet sans authentification
- **Utiliser un reverse proxy** ([nginx](https://blog.stephane-robert.info/docs/services/web/nginx/), [Traefik](https://blog.stephane-robert.info/docs/services/reseau/traefik/)) pour le HTTPS
- **Limiter les interfaces** avec `--bind-address 127.0.0.1`

- **Ajuster le refresh** : 3-5 secondes suffisent généralement
- **Désactiver les plugins inutiles** : `--disable-plugin sensors,gpu`
- **Filtrer les interfaces réseau** : masquer les veth Docker

- **Standardiser la configuration** sur tous vos serveurs
- **Utiliser le mode client/serveur** pour centraliser la surveillance
- **Exporter vers Prometheus/[Grafana](https://blog.stephane-robert.info/docs/observer/grafana/)** pour l’historique et les alertes

1. **Glances centralise** CPU, RAM, disques, réseau, processus et conteneurs dans une seule interface
2. **4 modes d’utilisation** : local (`glances`), web (`-w`), client/serveur (`-s`/`-c`), fetch (`--fetch`)
3. **Mode fetch** (v4.4+) : snapshot instantané style neofetch
4. **Export natif** vers Prometheus, InfluxDB, CSV et 15+ autres backends
5. **Alertes colorées** : vert (OK), jaune (warning), rouge (critique)
6. **Configuration centralisée** dans `~/.config/glances/glances.conf`
7. **Support conteneurs** : Docker et Podman détectés automatiquement
8. **Installation recommandée** : via mise ou pipx pour l’isolation

[Prometheus](https://blog.stephane-robert.info/docs/outils/observabilite/metriques/prometheus/) Configurez la collecte de métriques à grande échelle

[mise](https://blog.stephane-robert.info/docs/outils/systeme/mise/) Gérez vos outils de développement avec mise

- **Site officiel** : [nicolargo.github.io/glances](https://nicolargo.github.io/glances/)
- **Documentation** : [glances.readthedocs.io](https://glances.readthedocs.io/)
- **GitHub** : [github.com/nicolargo/glances](https://github.com/nicolargo/glances)
- **Changelog v4.4** : [github.com/nicolargo/glances/releases/tag/v4.4.0](https://github.com/nicolargo/glances/releases/tag/v4.4.0)