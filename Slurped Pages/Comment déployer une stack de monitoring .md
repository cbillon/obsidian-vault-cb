---
link: https://www.shpv.fr/blog/deployer-stack-monitoring/
byline: SHPV
site: SHPV
date: 2025-05-15T12:00
excerpt: Guide complet pour déployer une stack de monitoring avec Prometheus, Grafana et Alertmanager sur un serveur Debian avec Docker
twitter: https://twitter.com/@SHPVFR
tags:
  - linux
  - monitoring
slurped: 2025-10-12T19:17
title: Comment déployer une stack de monitoring ?
---

Mettre en place une stack de monitoring permet de superviser l'état de vos serveurs, applications et services critiques. Ce guide vous présente comment déployer une solution de monitoring moderne et efficace avec Prometheus, Grafana et Alertmanager.

## Prérequis

Avant de commencer, assurez-vous d'avoir :

- Un serveur Debian 12 (ou équivalent)
- Docker et Docker Compose installés
- Un domaine (optionnel pour Grafana via HTTPS)

## Installation des dépendances

```
# Mise à jour du système
sudo apt update && sudo apt upgrade -y

# Installation des dépendances Docker
sudo apt install -y ca-certificates curl gnupg lsb-release

# Ajout de la clé GPG Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Ajout du dépôt Docker
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian   $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Installation de Docker et Docker Compose
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Vérification
docker --version
docker compose version
```

## Création de la stack de monitoring

```
mkdir ~/monitoring && cd ~/monitoring
```

Créez un fichier `docker-compose.yml` :

```
version: '3.8'
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus:/etc/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - "9090:9090"

  alertmanager:
    image: prom/alertmanager
    volumes:
      - ./alertmanager:/etc/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
    ports:
      - "9093:9093"

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - ./grafana:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

## Configuration de Prometheus

Créez un dossier `prometheus/` avec un fichier `prometheus.yml` :

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

## Configuration d'Alertmanager

Créez un dossier `alertmanager/` avec un fichier `alertmanager.yml` :

```
global:
  resolve_timeout: 5m

route:
  receiver: 'default'

receivers:
  - name: 'default'
```

## Lancement de la stack

```
docker compose up -d
```

## Accès aux interfaces

- **Prometheus** : http://<ip_serveur>:9090
- **Grafana** : http://<ip_serveur>:3000 (admin / admin)
- **Alertmanager** : http://<ip_serveur>:9093

## Exemple de dashboard Grafana

Une fois connecté à Grafana, ajoutez Prometheus comme source de données et importez un dashboard officiel depuis [grafana.com/dashboards](https://grafana.com/grafana/dashboards).

## Sauvegarde et restauration

```
# Exemple de sauvegarde
tar -czf monitoring_backup_$(date +%Y%m%d).tar.gz prometheus/ alertmanager/ grafana/
```

## Mise à jour des images

```
docker compose pull
docker compose up -d
```

## Comparaison des outils de monitoring

|Fonctionnalité|Prometheus|Zabbix|Nagios|Datadog|
|---|---|---|---|---|
|**Type**|Open Source|Open Source|Open Source|Propriétaire|
|**Interface web**|✅ Oui|✅ Oui|✅ Oui|✅ Oui|
|**Alerting intégré**|✅ Oui|✅ Oui|✅ Basique|✅ Oui|
|**Scalabilité**|⭐⭐⭐⭐|⭐⭐⭐|⭐⭐|⭐⭐⭐⭐⭐|
|**Complexité initiale**|⭐⭐|⭐⭐⭐⭐|⭐⭐⭐⭐|⭐⭐|
|**Prix**|Gratuit|Gratuit|Gratuit|Payant|

## Avantages de cette stack

1. **Open-source** : Aucun coût de licence
2. **Extensible** : Intégration facile avec d'autres outils (Node Exporter, Blackbox, etc.)
3. **Visualisation avancée** : Grâce à Grafana
4. **Alerting puissant** : Personnalisable avec Alertmanager
5. **Support communautaire large**

## Alternatives à considérer

- **Zabbix** : Pour un environnement plus orienté entreprise
- **Nagios** : Si vous préférez une solution plus ancienne et éprouvée
- **Datadog** : Pour une solution clé en main avec support professionnel

## Conclusion

Déployer une stack de monitoring avec Prometheus, Grafana et Alertmanager est simple, puissant et adapté à la majorité des cas d’usage. Vous avez maintenant une base solide pour surveiller vos services, détecter les anomalies et réagir rapidement.