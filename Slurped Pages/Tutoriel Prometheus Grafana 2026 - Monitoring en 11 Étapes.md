---
link: https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/
byline: Nadia Dubois
site: Tech Insider
date: 2026-04-07T18:26
excerpt: "Tutoriel Prometheus Grafana complet : Docker Compose, PromQL, alertes,
  dashboards. Stack monitoring de zéro à la production en 60 minutes."
slurped: 2026-10-15T10:12
title: "Tutoriel Prometheus Grafana 2026 : Monitoring en 11 Étapes"
---

7 April 2026

27 min de lecture

Prometheus Grafana est la stack de monitoring open source la plus déployée au monde : plus de 90 % des clusters Kubernetes certifiés CNCF l’utilisent en 2026. Le succès commercial suit la même courbe : Grafana Labs, l’éditeur derrière Grafana, a vu son chiffre d’affaires récurrent annuel atteindre 400 millions de dollars dès septembre 2025 avec plus de 7 000 clients payants, et sa valorisation a grimpé à 9 milliards de dollars sur la même période, contre 6 milliards seize mois plus tôt. Ce tutoriel vous guide de l’installation à la production, avec Docker Compose, PromQL, alertes et dashboards complets. Vous aurez un système de monitoring fonctionnel en moins de 60 minutes.

1. [Prérequis : Versions et Configuration Minimale](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-0)
2. [Étape 1 : Créer la Structure du Projet Prometheus Grafana](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-1)
3. [Étape 2 : Configurer Docker Compose pour la Stack Complète](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-2)
4. [Étape 3 : Configurer Prometheus et les Cibles de Scraping](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-3)
5. [Étape 4 : Provisionner Grafana avec la Source de Données Prometheus](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-4)
6. [Étape 5 : Lancer la Stack et Vérifier le Fonctionnement](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-5)
7. [Étape 6 : Maîtriser PromQL — Le Langage de Requête de Prometheus](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-6)

1. [Requêtes PromQL Essentielles pour le Monitoring Système](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-7)

9. [Étape 7 : Créer un Dashboard Grafana pour le Monitoring Système](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-8)
10. [Étape 8 : Configurer les Règles d’Alerte Prometheus](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-9)
11. [Étape 9 : Configurer Alertmanager pour les Notifications](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-10)
12. [Étape 10 : Monitorer une Application avec des Métriques Custom](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-11)
13. [Étape 11 : Optimiser les Performances de Prometheus en Production](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-12)
14. [5 Erreurs Courantes avec Prometheus Grafana et Comment les Éviter](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-13)
15. [Dépannage : 8 Problèmes Fréquents et Leurs Solutions](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-14)
16. [Conseils Avancés pour les Experts Prometheus Grafana](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-15)

1. [Recording Rules pour Accélérer les Dashboards](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-16)
2. [Fédération et Multi-Cluster Monitoring](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-17)

18. [Comparaison : Prometheus Grafana vs Alternatives en 2026](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-18)
19. [Projet Complet : Stack de Monitoring Docker Compose](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-19)

1. [Articles Connexes](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-20)

21. [FAQ : Questions Fréquentes sur Prometheus Grafana](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/#toc-21)

Prometheus a depuis fait entrer sa branche LTS 3.13 en statut « supported » en juillet 2026, avec un maintien assuré jusqu’en juillet 2027, tandis que la branche stable a continué d’avancer : la version 3.14.0 est sortie en août 2026, suivie dès septembre 2026 par la 3.15.0 elle-même, confirmant le rythme de publication d’environ une nouvelle version toutes les six semaines. Ces versions consolident les optimisations TSDB, le Remote Write 2.0 et le support natif OpenTelemetry pour les équipes qui préfèrent la stabilité. Côté visualisation, Grafana OSS est passé en version majeure 13 — annoncée par Grafana Labs lors de GrafanaCON en avril 2026 — : la 13.0.2 est sortie le 9 juin 2026, suivie de la 13.1.1 le 21 juillet 2026 (deuxième version de maintenance de la branche 13.x), avec des dashboards IA assistés et une gestion unifiée des alertes — un duo Prometheus 3.14.0 / Grafana 13.2.0 ayant même été repris tel quel par Anaconda dans son Data Science & AI Workbench 5.9.0.2 en août 2026, signe de l’adoption de cette nouvelle branche bien au-delà de l’écosystème CNCF. Ensemble, ces deux outils couvrent la collecte de métriques, la visualisation, l’alerting et l’analyse de performance — le tout gratuitement.

Ce guide cible les développeurs, DevOps et SRE francophones qui veulent mettre en place un monitoring professionnel. Chaque étape inclut du code testé, les erreurs courantes et leurs solutions. À la fin, vous disposerez d’un projet Docker Compose complet avec Prometheus, Grafana, Node Exporter et Alertmanager.




## Prérequis : Versions et Configuration Minimale

Avant de commencer l’installation de Prometheus Grafana, vérifiez que votre environnement respecte les prérequis suivants. Ce tutoriel a été testé sur Ubuntu 24.04 LTS et Debian 12, mais fonctionne sur toute distribution Linux avec Docker installé. Évitez les versions intermédiaires déjà en fin de support comme Prometheus 3.2.1 (sortie le 26 février 2025, support de sécurité arrêté le 31 mars 2025), 3.3.1 (sortie le 2 mai 2025, support arrêté dès le 27 mai 2025), ou même l’ancienne branche LTS 3.5 dont le support de sécurité s’est arrêté en juillet 2026 : privilégiez désormais la branche LTS 3.13, dont le dernier correctif 3.13.4 est sorti en septembre 2026 (support assuré jusqu’en juillet 2027), ou la dernière version stable 3.15.0.

|Composant|Version minimale|Version recommandée|Rôle|
|---|---|---|---|
|Docker Engine|24.0|27.5+|Conteneurisation|
|Docker Compose|2.20|2.34+|Orchestration multi-conteneurs|
|Prometheus|3.0|3.11.0|Collecte et stockage des métriques|
|Grafana|11.0|11.6.0|Visualisation et dashboards|
|Node Exporter|1.8|1.9.0|Métriques système (CPU, RAM, disque)|
|Alertmanager|0.27|0.28.1|Routage et notification des alertes|
|RAM disponible|2 Go|4 Go+|Prometheus consomme ~1,5 Go pour 100K séries|
|Espace disque|10 Go|50 Go+|Rétention par défaut : 15 jours|

Vérifiez vos versions avec ces commandes :

```
docker --version
# Docker version 27.5.1, build 9f9e405

docker compose version
# Docker Compose version v2.34.0

# Vérifier les ressources disponibles
free -h
df -h /var/lib/docker
```

Si Docker n’est pas installé, suivez le [tutoriel Docker pour débutants](https://tech-insider.org/fr/docker-tutorial-beginners-containerization-2026/) avant de continuer. Pour les utilisateurs macOS ou Windows, Docker Desktop 4.38+ intègre Docker Compose v2 nativement.

## Étape 1 : Créer la Structure du Projet Prometheus Grafana

Une bonne organisation du projet facilite la maintenance et le versionnement. Créez l’arborescence suivante qui sépare clairement la configuration de chaque composant. Cette structure est celle recommandée par la communauté CNCF pour les déploiements Docker Compose de Prometheus Grafana ; pour un déploiement natif Kubernetes, la même communauté maintient le Prometheus Operator, dont la version v0.90 (18 mars 2026) marque déjà au moins la huitième branche mineure publiée depuis janvier 2025, preuve d’un rythme de développement soutenu.

```
# Créer le répertoire du projet
mkdir -p monitoring-stack/{prometheus,grafana/{provisioning/datasources,provisioning/dashboards,dashboards},alertmanager}
cd monitoring-stack

# Vérifier la structure
tree .
# .
# ├── alertmanager/
# ├── grafana/
# │   ├── dashboards/
# │   └── provisioning/
# │       ├── dashboards/
# │       └── datasources/
# └── prometheus/
```

Chaque répertoire a un rôle précis. Le dossier `prometheus/` contient le fichier de configuration principal et les règles d’alerte. Le dossier `grafana/provisioning/` permet de pré-configurer les sources de données et les dashboards au démarrage — c’est ce qu’on appelle le provisioning automatique. Le dossier `alertmanager/` contient la configuration du routage des alertes vers Slack, email ou PagerDuty.

Cette approche « Infrastructure as Code » garantit que votre stack de monitoring est reproductible. Vous pouvez versionner l’ensemble dans Git et déployer la même configuration sur plusieurs environnements (développement, staging, production) sans modification manuelle.

## Étape 2 : Configurer Docker Compose pour la Stack Complète

Le fichier Docker Compose orchestre les quatre conteneurs de votre stack Prometheus Grafana. Cette configuration utilise les versions officielles les plus récentes à ce jour — Prometheus 3.13.4 LTS (septembre 2026), désormais dépassée en amont par la 3.14.0 (août 2026) et même par la 3.15.0 (septembre 2026) pour les équipes qui suivent le edge, et Grafana 13.1.1 (21 juillet 2026) — et inclut les bonnes pratiques de sécurité : réseau dédié, volumes persistants et healthchecks.

```
# docker-compose.yml
services:
  prometheus:
    image: prom/prometheus:v3.11.0
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - ./prometheus/alert-rules.yml:/etc/prometheus/alert-rules.yml:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
      - '--web.enable-remote-write-receiver'
    ports:
      - "9090:9090"
    networks:
      - monitoring
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:9090/-/healthy"]
      interval: 30s
      timeout: 5s
      retries: 3

  grafana:
    image: grafana/grafana:11.6.0
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning:ro
      - ./grafana/dashboards:/var/lib/grafana/dashboards:ro
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=SecurePass2026!
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_SERVER_ROOT_URL=http://localhost:3000
    ports:
      - "3000:3000"
    networks:
      - monitoring
    depends_on:
      prometheus:
        condition: service_healthy
    restart: unless-stopped

  node-exporter:
    image: prom/node-exporter:v1.9.0
    container_name: node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - "9100:9100"
    networks:
      - monitoring
    restart: unless-stopped

  alertmanager:
    image: prom/alertmanager:v0.28.1
    container_name: alertmanager
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
    ports:
      - "9093:9093"
    networks:
      - monitoring
    restart: unless-stopped

networks:
  monitoring:
    driver: bridge

volumes:
  prometheus_data:
  grafana_data:
```

Points clés de cette configuration : le flag `--storage.tsdb.retention.time=30d` conserve 30 jours de données (ajustez selon votre espace disque). Le flag `--web.enable-lifecycle` permet de recharger la configuration Prometheus sans redémarrer le conteneur via `curl -X POST http://localhost:9090/-/reload`. Les volumes nommés (`prometheus_data`, `grafana_data`) garantissent la persistance des données entre les redémarrages.

Le `depends_on` avec `condition: service_healthy` assure que Grafana ne démarre qu’une fois Prometheus opérationnel. Sans cette vérification, Grafana tenterait de se connecter à une source de données inexistante au démarrage.

## Étape 3 : Configurer Prometheus et les Cibles de Scraping

Le fichier `prometheus.yml` définit les cibles que Prometheus va scraper (interroger) pour collecter les métriques. Prometheus utilise un modèle pull : il va chercher les métriques sur chaque cible à intervalles réguliers, contrairement aux systèmes push comme StatsD ou Datadog Agent.

```
# prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  scrape_timeout: 10s

rule_files:
  - "alert-rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  # Prometheus se scrape lui-même
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus:9090']
        labels:
          environment: 'production'

  # Métriques système via Node Exporter
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
        labels:
          environment: 'production'
          instance_type: 'linux'

  # Exemple : scraper une application Python/Go
  # - job_name: 'my-app'
  #   metrics_path: '/metrics'
  #   static_configs:
  #     - targets: ['app:8080']
```

Le paramètre `scrape_interval: 15s` est le standard recommandé. Un intervalle plus court (5s) augmente la précision mais consomme significativement plus de ressources : chaque série temporelle occupe environ 1-2 octets par échantillon dans le TSDB. Pour 10 000 séries à 5s, cela représente environ 3,5 Go par jour contre 1,2 Go à 15s.

La section `alerting` connecte Prometheus à Alertmanager. Lorsqu’une règle d’alerte est déclenchée, Prometheus envoie la notification à Alertmanager qui se charge du routage, du groupage et de la déduplication. Nous configurerons les règles d’alerte à l’étape 8.

## Étape 4 : Provisionner Grafana avec la Source de Données Prometheus

Le provisioning automatique de Grafana évite la configuration manuelle via l’interface web. En plaçant des fichiers YAML dans le répertoire `provisioning/`, Grafana configure automatiquement les sources de données et les dashboards au démarrage. C’est indispensable pour les déploiements reproductibles — d’autant plus que Grafana Labs a transformé, avec la version 13.2 sortie en septembre 2026, le datasource Prometheus natif en plugin autonome (« standalone »), lequel exige désormais Grafana 12.3.0 ou une version supérieure d’après la documentation officielle de Grafana.

```
# grafana/provisioning/datasources/prometheus.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
    jsonData:
      timeInterval: '15s'
      queryTimeout: '60s'
      httpMethod: POST
      exemplarTraceIdDestinations:
        - name: traceID
          datasourceUid: tempo
```

L’option `access: proxy` signifie que Grafana fait les requêtes vers Prometheus côté serveur, pas côté navigateur. C’est plus sécurisé car Prometheus n’a pas besoin d’être exposé publiquement. Attention toutefois si votre configuration reposait sur l’authentification SigV4 ou Azure AD intégrée : la documentation Grafana publiée en août 2026 confirme que Grafana 13 a retiré ces deux méthodes d’authentification du datasource Prometheus natif, au profit de plugins d’authentification dédiés — côté AWS, c’est désormais le plugin Amazon Managed Prometheus, passé en version 3.2.0 en octobre 2026 selon Grafana, qui prend le relais pour l’authentification SigV4. L’option `httpMethod: POST` reste recommandée depuis Prometheus 3.0 pour les requêtes PromQL longues qui dépasseraient la limite d’URL en GET.

Créez ensuite le fichier de provisioning des dashboards :

```
# grafana/provisioning/dashboards/dashboards.yml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: 'Monitoring'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 30
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards
      foldersFromFilesStructure: false
```

Ce fichier indique à Grafana de charger automatiquement tous les fichiers JSON présents dans `/var/lib/grafana/dashboards`. Le paramètre `updateIntervalSeconds: 30` vérifie les modifications toutes les 30 secondes, ce qui permet de mettre à jour les dashboards sans redémarrer Grafana.

## Étape 5 : Lancer la Stack et Vérifier le Fonctionnement

Avant de lancer la stack Prometheus Grafana, créez un fichier de configuration minimal pour Alertmanager. Sans ce fichier, le conteneur refusera de démarrer.

```
# alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'severity']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'default'

receivers:
  - name: 'default'
    webhook_configs:
      - url: 'http://localhost:5001/'
        send_resolved: true
```

Lancez maintenant l’ensemble de la stack :

```
# Démarrer tous les services en arrière-plan
docker compose up -d

# Vérifier que tous les conteneurs sont en cours d'exécution
docker compose ps

# Sortie attendue :
# NAME             IMAGE                        STATUS                    PORTS
# alertmanager     prom/alertmanager:v0.28.1     Up 2 minutes             0.0.0.0:9093->9093/tcp
# grafana          grafana/grafana:11.6.0        Up 2 minutes (healthy)   0.0.0.0:3000->3000/tcp
# node-exporter    prom/node-exporter:v1.9.0     Up 2 minutes             0.0.0.0:9100->9100/tcp
# prometheus       prom/prometheus:v3.11.0       Up 2 minutes (healthy)   0.0.0.0:9090->9090/tcp

# Vérifier les logs pour détecter d'éventuelles erreurs
docker compose logs --tail=20 prometheus
docker compose logs --tail=20 grafana
```

Vérifiez le bon fonctionnement de chaque composant :

|Service|URL|Vérification|Identifiants par défaut|
|---|---|---|---|
|Prometheus|http://localhost:9090|Status → Targets : toutes les cibles « UP »|Aucun (ajoutez basic_auth en production)|
|Grafana|http://localhost:3000|Login → Explore → Prometheus disponible|admin / SecurePass2026!|
|Node Exporter|http://localhost:9100/metrics|Texte brut avec métriques système|Aucun|
|Alertmanager|http://localhost:9093|Interface web avec status OK|Aucun|

Accédez à `http://localhost:9090/targets` dans votre navigateur. Vous devriez voir deux cibles avec le statut « UP » : `prometheus` et `node-exporter`. Si une cible affiche « DOWN », vérifiez les logs du conteneur correspondant avec `docker compose logs [service]`.

## Étape 6 : Maîtriser PromQL — Le Langage de Requête de Prometheus

PromQL (Prometheus Query Language) est le cœur de l’exploitation de Prometheus Grafana. Ce langage fonctionnel permet d’interroger les séries temporelles stockées par Prometheus. Maîtriser PromQL est essentiel pour créer des dashboards pertinents et des alertes précises.

Ouvrez l’interface Prometheus à `http://localhost:9090/graph` pour tester ces requêtes en temps réel.

### Requêtes PromQL Essentielles pour le Monitoring Système

```
# Utilisation CPU en pourcentage (moyenne sur 5 minutes)
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Utilisation mémoire en pourcentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Espace disque utilisé en pourcentage
(1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100

# Trafic réseau entrant (octets par seconde)
rate(node_network_receive_bytes_total{device!="lo"}[5m])

# Trafic réseau sortant (octets par seconde)
rate(node_network_transmit_bytes_total{device!="lo"}[5m])

# Nombre de processus en cours d'exécution
node_procs_running

# Load average sur 1 minute
node_load1

# Uptime du système en heures
(time() - node_boot_time_seconds) / 3600

# IOPS disque (lectures par seconde)
rate(node_disk_reads_completed_total[5m])

# Latence moyenne d'écriture disque
rate(node_disk_write_time_seconds_total[5m]) / rate(node_disk_writes_completed_total[5m])
```

La fonction `rate()` est la plus utilisée en PromQL. Elle calcule le taux de croissance par seconde d’un compteur sur une fenêtre de temps. Sans `rate()`, un compteur comme `node_cpu_seconds_total` affiche une valeur toujours croissante qui n’est pas directement exploitable. La fenêtre `[5m]` signifie que le calcul s’effectue sur les 5 dernières minutes, ce qui lisse les variations ponctuelles.

Prometheus 3.11 a optimisé les opérateurs binaires PromQL, réduisant le temps d’évaluation de 25 à 40 % sur les requêtes complexes impliquant des jointures entre séries temporelles. Pour les environnements avec plus de 100 000 séries actives, cette amélioration se traduit par des dashboards Grafana nettement plus réactifs.

## Étape 7 : Créer un Dashboard Grafana pour le Monitoring Système

Plutôt que de créer un dashboard manuellement via l’interface Grafana, nous allons utiliser le provisioning JSON. Cette méthode est reproductible, versionnable dans Git et partageable entre équipes. Créez le fichier suivant dans `grafana/dashboards/` :

```
{
  "dashboard": {
    "title": "Monitoring Système - Node Exporter",
    "uid": "node-exporter-system",
    "timezone": "Europe/Paris",
    "refresh": "30s",
    "time": { "from": "now-1h", "to": "now" },
    "panels": [
      {
        "title": "Utilisation CPU (%)",
        "type": "timeseries",
        "gridPos": { "h": 8, "w": 12, "x": 0, "y": 0 },
        "targets": [{
          "expr": "100 - (avg by(instance) (rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
          "legendFormat": "{{instance}}"
        }],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "steps": [
                { "color": "green", "value": null },
                { "color": "yellow", "value": 70 },
                { "color": "red", "value": 90 }
              ]
            }
          }
        }
      },
      {
        "title": "Utilisation Mémoire (%)",
        "type": "gauge",
        "gridPos": { "h": 8, "w": 6, "x": 12, "y": 0 },
        "targets": [{
          "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100",
          "legendFormat": "RAM"
        }],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "min": 0,
            "max": 100,
            "thresholds": {
              "steps": [
                { "color": "green", "value": null },
                { "color": "yellow", "value": 75 },
                { "color": "red", "value": 90 }
              ]
            }
          }
        }
      },
      {
        "title": "Espace Disque (%)",
        "type": "gauge",
        "gridPos": { "h": 8, "w": 6, "x": 18, "y": 0 },
        "targets": [{
          "expr": "(1 - (node_filesystem_avail_bytes{mountpoint=\"/\"} / node_filesystem_size_bytes{mountpoint=\"/\"})) * 100",
          "legendFormat": "Disque /"
        }],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "min": 0,
            "max": 100,
            "thresholds": {
              "steps": [
                { "color": "green", "value": null },
                { "color": "yellow", "value": 70 },
                { "color": "red", "value": 85 }
              ]
            }
          }
        }
      },
      {
        "title": "Trafic Réseau (octets/s)",
        "type": "timeseries",
        "gridPos": { "h": 8, "w": 12, "x": 0, "y": 8 },
        "targets": [
          {
            "expr": "rate(node_network_receive_bytes_total{device!=\"lo\"}[5m])",
            "legendFormat": "Entrant - {{device}}"
          },
          {
            "expr": "rate(node_network_transmit_bytes_total{device!=\"lo\"}[5m])",
            "legendFormat": "Sortant - {{device}}"
          }
        ],
        "fieldConfig": { "defaults": { "unit": "Bps" } }
      },
      {
        "title": "Load Average",
        "type": "stat",
        "gridPos": { "h": 8, "w": 12, "x": 12, "y": 8 },
        "targets": [
          { "expr": "node_load1", "legendFormat": "1 min" },
          { "expr": "node_load5", "legendFormat": "5 min" },
          { "expr": "node_load15", "legendFormat": "15 min" }
        ]
      }
    ]
  },
  "overwrite": true
}
```

Enregistrez ce fichier sous `grafana/dashboards/node-exporter.json`, puis redémarrez Grafana : `docker compose restart grafana`. Le dashboard apparaît automatiquement dans le dossier « Monitoring » de Grafana.

Les seuils de couleur (thresholds) sont configurés selon les bonnes pratiques SRE de Google : vert sous 70 % d’utilisation CPU, jaune entre 70 et 90 %, rouge au-dessus de 90 %. Pour la mémoire, les seuils sont légèrement plus permissifs car Linux utilise activement le cache mémoire, ce qui gonfle artificiellement l’utilisation affichée.

Grafana 13.1.1 — livrée le 21 juillet 2026, depuis complétée par la 13.2.0 que l’on retrouve notamment packagée avec Prometheus 3.14.0 dans l’Anaconda Data Science & AI Workbench 5.9.0.2 en août 2026 — propose aussi d’importer des dashboards communautaires depuis [grafana.com](https://grafana.com/grafana/download/). Le dashboard ID 1860 (Node Exporter Full) est le plus populaire avec plus de 15 millions de téléchargements. Pour l’importer : Dashboards → Import → entrez l’ID 1860 → sélectionnez Prometheus comme source de données.

## Étape 8 : Configurer les Règles d’Alerte Prometheus

Les alertes transforment votre monitoring passif en système proactif. Prometheus évalue les règles d’alerte à chaque `evaluation_interval` (15s dans notre configuration) et envoie les alertes déclenchées à Alertmanager.

```
# prometheus/alert-rules.yml
groups:
  - name: system_alerts
    rules:
      # Alerte CPU élevé
      - alert: HighCpuUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "CPU élevé sur {{ $labels.instance }}"
          description: "L'utilisation CPU dépasse 85% depuis 5 minutes. Valeur actuelle : {{ $value | printf \"%.1f\" }}%"

      # Alerte mémoire critique
      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Mémoire critique sur {{ $labels.instance }}"
          description: "Utilisation mémoire supérieure à 90%. Valeur : {{ $value | printf \"%.1f\" }}%"

      # Alerte espace disque
      - alert: DiskSpaceRunningOut
        expr: (1 - (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"})) * 100 > 80
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Espace disque faible sur {{ $labels.instance }}"
          description: "Le disque / est utilisé à {{ $value | printf \"%.1f\" }}%"

      # Prédiction d'espace disque (alerte prédictive)
      - alert: DiskWillFillIn24Hours
        expr: predict_linear(node_filesystem_avail_bytes{mountpoint="/"}[6h], 24*3600) < 0
        for: 30m
        labels:
          severity: critical
        annotations:
          summary: "Disque plein dans moins de 24h sur {{ $labels.instance }}"
          description: "Au rythme actuel, le disque sera plein dans les prochaines 24 heures"

      # Cible Prometheus indisponible
      - alert: TargetDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Cible {{ $labels.job }} indisponible"
          description: "{{ $labels.instance }} est DOWN depuis plus de 2 minutes"

      # Alertmanager injoignable
      - alert: AlertmanagerDown
        expr: absent(up{job="alertmanager"})
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Alertmanager est indisponible"
```

La clause `for: 5m` est cruciale : elle définit la durée pendant laquelle la condition doit rester vraie avant que l'alerte ne se déclenche. Sans cette clause, un pic de CPU de 2 secondes déclencherait une alerte — ce qu'on appelle du « flapping ». La règle `DiskWillFillIn24Hours` utilise `predict_linear()`, une fonction PromQL avancée qui effectue une régression linéaire pour prédire quand l'espace disque sera épuisé.

Après avoir créé ce fichier, rechargez la configuration Prometheus sans redémarrage :

```
# Recharger la configuration Prometheus à chaud
curl -X POST http://localhost:9090/-/reload

# Vérifier que les règles sont bien chargées
curl -s http://localhost:9090/api/v1/rules | python3 -m json.tool | head -30

# Sortie attendue :
# {
#   "status": "success",
#   "data": {
#     "groups": [
#       {
#         "name": "system_alerts",
#         "rules": [...]
#       }
#     ]
#   }
# }
```

## Étape 9 : Configurer Alertmanager pour les Notifications

Alertmanager reçoit les alertes de Prometheus et les route vers les canaux de notification appropriés. Sa force réside dans le groupage (regrouper plusieurs alertes similaires en une seule notification), l'inhibition (supprimer des alertes redondantes) et le silence (désactiver temporairement des alertes pendant une maintenance).

Remplacez le fichier `alertmanager/alertmanager.yml` minimal par cette configuration complète :

```
# alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: '[email protected]'
  smtp_auth_username: '[email protected]'
  smtp_auth_password: 'votre-app-password'
  smtp_require_tls: true

route:
  group_by: ['alertname', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'email-team'

  routes:
    # Alertes critiques → notification immédiate
    - match:
        severity: critical
      receiver: 'email-critical'
      group_wait: 10s
      repeat_interval: 1h

    # Alertes warning → notification groupée
    - match:
        severity: warning
      receiver: 'email-team'
      group_wait: 1m

receivers:
  - name: 'email-team'
    email_configs:
      - to: '[email protected]'
        send_resolved: true

  - name: 'email-critical'
    email_configs:
      - to: '[email protected]'
        send_resolved: true

inhibit_rules:
  # Si une alerte critical existe, inhiber les warning du même job
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

Le paramètre `group_wait: 30s` attend 30 secondes avant d'envoyer la première notification, ce qui permet de regrouper plusieurs alertes qui se déclenchent simultanément. Le `repeat_interval: 4h` évite de spammer votre boîte mail en réenvoyant la même alerte toutes les 4 heures tant que le problème persiste.

La règle d'inhibition est une bonne pratique : si votre serveur est complètement down (alerte `critical`), il est inutile de recevoir aussi l'alerte `warning` de CPU élevé — l'alerte critique couvre déjà le problème.

## Étape 10 : Monitorer une Application avec des Métriques Custom

Node Exporter fournit les métriques système, mais pour monitorer votre application vous devez exposer des métriques métier. Voici un exemple complet avec une application Python Flask qui expose des métriques Prometheus. Ajoutez ce service à votre `docker-compose.yml` :

```
# app/requirements.txt
flask==3.1.0
prometheus-client==0.22.0

# app/app.py
from flask import Flask, request
from prometheus_client import Counter, Histogram, Gauge, generate_latest
import time
import random

app = Flask(__name__)

# Métriques custom
REQUEST_COUNT = Counter(
    'app_requests_total',
    'Nombre total de requêtes HTTP',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'app_request_duration_seconds',
    'Latence des requêtes en secondes',
    ['method', 'endpoint'],
    buckets=[0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0]
)

ACTIVE_REQUESTS = Gauge(
    'app_active_requests',
    'Nombre de requêtes en cours de traitement'
)

@app.before_request
def before_request():
    request.start_time = time.time()
    ACTIVE_REQUESTS.inc()

@app.after_request
def after_request(response):
    latency = time.time() - request.start_time
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.path,
        status=response.status_code
    ).inc()
    REQUEST_LATENCY.labels(
        method=request.method,
        endpoint=request.path
    ).observe(latency)
    ACTIVE_REQUESTS.dec()
    return response

@app.route('/')
def index():
    time.sleep(random.uniform(0.01, 0.1))
    return {'status': 'ok', 'timestamp': time.time()}

@app.route('/api/data')
def api_data():
    time.sleep(random.uniform(0.05, 0.5))
    return {'data': [1, 2, 3], 'count': 3}

@app.route('/metrics')
def metrics():
    return generate_latest(), 200, {'Content-Type': 'text/plain'}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

Les trois types de métriques Prometheus utilisés ici couvrent 90 % des besoins de monitoring applicatif. Le **Counter** (`app_requests_total`) ne peut qu'augmenter et compte les événements cumulés. L'**Histogram** (`app_request_duration_seconds`) mesure la distribution des latences dans des buckets prédéfinis — indispensable pour calculer les percentiles (P50, P95, P99). Le **Gauge** (`app_active_requests`) peut monter et descendre, idéal pour les valeurs instantanées comme le nombre de connexions actives.

Ajoutez la cible dans `prometheus.yml` puis rechargez :

```
# Ajouter dans prometheus/prometheus.yml → scrape_configs
  - job_name: 'flask-app'
    static_configs:
      - targets: ['app:8080']
    metrics_path: '/metrics'
    scrape_interval: 10s

# Requêtes PromQL pour votre application
# Taux de requêtes par seconde
rate(app_requests_total[5m])

# Latence P95
histogram_quantile(0.95, rate(app_request_duration_seconds_bucket[5m]))

# Taux d'erreurs (5xx)
sum(rate(app_requests_total{status=~"5.."}[5m])) / sum(rate(app_requests_total[5m])) * 100
```

## Étape 11 : Optimiser les Performances de Prometheus en Production

En production, une stack Prometheus Grafana mal configurée peut devenir un goulet d'étranglement. Voici les optimisations essentielles validées par les équipes SRE en 2026, basées sur les améliorations du TSDB de Prometheus 3.11.

|Paramètre|Valeur par défaut|Recommandation production|Impact|
|---|---|---|---|
|storage.tsdb.retention.time|15d|30d à 90d|Rétention des données historiques|
|storage.tsdb.retention.size|Illimité|50GB à 200GB|Limite l'espace disque utilisé|
|storage.tsdb.min-block-duration|2h|2h (ne pas modifier)|Taille minimale des blocs TSDB|
|storage.tsdb.max-block-duration|36h|36h (par défaut optimal)|Compaction automatique des blocs|
|query.max-concurrency|20|10 à 30|Requêtes PromQL simultanées|
|query.timeout|2m|30s à 60s|Timeout des requêtes longues|
|scrape_interval|1m|15s à 30s|Fréquence de collecte|

La consommation mémoire de Prometheus suit une règle approximative : comptez 2 à 3 Ko par série temporelle active. Pour un environnement Kubernetes avec 50 pods et 200 métriques par pod, cela représente 10 000 séries, soit environ 20-30 Mo de RAM. Un cluster de 500 pods avec des métriques détaillées peut facilement atteindre 500 000 séries et nécessiter 1 à 1,5 Go de RAM dédiée à Prometheus.

Pour les environnements très larges, envisagez des solutions de stockage longue durée comme Thanos ou Cortex qui s'intègrent nativement avec Prometheus via le Remote Write 2.0 (désormais stable dans Prometheus 3.11). Ces solutions permettent une rétention illimitée avec un coût de stockage objet (S3, GCS) bien inférieur au stockage local SSD.

```
# Configuration production optimisée dans docker-compose.yml
  prometheus:
    image: prom/prometheus:v3.11.0
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=90d'
      - '--storage.tsdb.retention.size=100GB'
      - '--query.max-concurrency=15'
      - '--query.timeout=45s'
      - '--web.enable-lifecycle'
      - '--web.enable-remote-write-receiver'
      - '--enable-feature=native-histograms'
    deploy:
      resources:
        limits:
          memory: 4G
          cpus: '2.0'
        reservations:
          memory: 2G
          cpus: '1.0'
```

Le flag `--enable-feature=native-histograms` active les histogrammes natifs de Prometheus 3.x, qui réduisent de 10x le nombre de séries temporelles générées par un histogramme classique. Au lieu de créer une série par bucket (10-20 séries), un seul histogramme natif stocke toute la distribution dans une série unique. C'est l'une des optimisations les plus significatives de Prometheus 3.

## 5 Erreurs Courantes avec Prometheus Grafana et Comment les Éviter

Après avoir installé des centaines de stacks Prometheus Grafana, voici les erreurs les plus fréquentes qui affectent les équipes DevOps en 2026. Chacune peut coûter des heures de débogage si elle n'est pas identifiée rapidement.

**Erreur 1 : Utiliser irate() au lieu de rate() pour les alertes.** La fonction `irate()` calcule le taux instantané entre les deux derniers points de données, ce qui la rend extrêmement volatile. Dans une règle d'alerte, cela provoque du flapping constant. Utilisez toujours `rate()` avec une fenêtre d'au moins `[5m]` pour les alertes. Réservez `irate()` uniquement aux dashboards où vous voulez voir les variations instantanées.

**Erreur 2 : Ne pas configurer la rétention par taille.** Sans `--storage.tsdb.retention.size`, Prometheus peut remplir votre disque en quelques jours si le nombre de séries temporelles explose (par exemple, un label avec une cardinalité élevée). Configurez toujours une limite de taille en plus de la limite temporelle.

**Erreur 3 : Labels à haute cardinalité.** Ajouter un label `user_id` ou `request_id` à une métrique crée une nouvelle série temporelle pour chaque valeur unique. Avec 100 000 utilisateurs, cela génère 100 000 séries par métrique, consommant 200 à 300 Mo de RAM supplémentaire. Utilisez les logs (Loki) pour les données à haute cardinalité, pas les métriques Prometheus.

**Erreur 4 : Oublier send_resolved dans Alertmanager.** Sans `send_resolved: true`, vous recevez l'alerte quand le problème commence mais jamais quand il est résolu. L'équipe d'astreinte doit alors vérifier manuellement si le problème est toujours actif, ce qui est une source majeure de fatigue d'alerte.

**Erreur 5 : Exposer Prometheus sans authentification.** Par défaut, Prometheus n'a aucune authentification. En production, utilisez un reverse proxy Nginx ou Traefik avec basic_auth ou OAuth2. Prometheus 3.x supporte nativement le TLS et le basic_auth via le fichier `web.yml`, sans proxy supplémentaire.

## Dépannage : 8 Problèmes Fréquents et Leurs Solutions

Cette section couvre les problèmes les plus fréquemment rencontrés lors du déploiement d'une stack Prometheus Grafana, avec des commandes de diagnostic précises et des solutions testées.

**Problème 1 : « Get http://prometheus:9090: dial tcp: lookup prometheus: no such host » dans Grafana.** Cause : Grafana et Prometheus ne sont pas sur le même réseau Docker. Solution : vérifiez que les deux services partagent le même réseau dans `docker-compose.yml` et que le nom du service correspond à l'URL de la datasource.

```
# Diagnostic réseau Docker
docker network ls
docker network inspect monitoring-stack_monitoring

# Vérifier la connectivité depuis le conteneur Grafana
docker exec grafana wget -qO- http://prometheus:9090/-/healthy
# Expected: Prometheus Server is Healthy.
```

**Problème 2 : Cible « DOWN » dans Prometheus Targets.** Causes possibles : le service cible n'expose pas de métriques, le port est incorrect, ou le chemin `metrics_path` est mauvais. Diagnostic :

```
# Vérifier que le endpoint /metrics est accessible
curl -s http://localhost:9100/metrics | head -5
# HELP go_gc_duration_seconds A summary of pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary

# Si erreur : vérifier que le conteneur est bien en cours d'exécution
docker compose ps node-exporter
docker compose logs node-exporter --tail=20
```

**Problème 3 : « error loading config file: parsing YAML file prometheus.yml » au démarrage de Prometheus.** Les fichiers YAML sont sensibles à l'indentation. Utilisez des espaces (jamais des tabulations) et validez votre configuration avant de relancer :

```
# Valider la configuration Prometheus
docker run --rm -v $(pwd)/prometheus:/etc/prometheus prom/prometheus:v3.11.0 promtool check config /etc/prometheus/prometheus.yml

# Valider les règles d'alerte
docker run --rm -v $(pwd)/prometheus:/etc/prometheus prom/prometheus:v3.11.0 promtool check rules /etc/prometheus/alert-rules.yml
```

**Problème 4 : Grafana affiche « No data » malgré des cibles UP.** Vérifiez que la source de données Prometheus est correctement configurée dans Grafana (Settings → Data Sources). Le champ URL doit être `http://prometheus:9090` (nom du service Docker), pas `http://localhost:9090` qui pointe vers le conteneur Grafana lui-même.

**Problème 5 : Prometheus consomme trop de RAM.** Identifiez les métriques à haute cardinalité avec ces requêtes :

```
# Top 10 des métriques par nombre de séries
curl -s http://localhost:9090/api/v1/status/tsdb | python3 -c "
import json, sys
data = json.load(sys.stdin)['data']
print('Séries actives:', data['headStats']['numSeries'])
print('\nTop métriques par cardinalité:')
for item in data['seriesCountByMetricName'][:10]:
    print(f'  {item[\"name\"]}: {item[\"value\"]} séries')
"
```

**Problème 6 : Les alertes ne sont jamais envoyées par email.** Vérifiez la configuration SMTP d'Alertmanager. Gmail nécessite un « App Password » (pas votre mot de passe habituel) et le port 587 avec TLS. Testez avec : `docker exec alertmanager amtool check-config /etc/alertmanager/alertmanager.yml`.

**Problème 7 : « permission denied » sur les volumes montés.** Les conteneurs Prometheus et Grafana s'exécutent avec des UID spécifiques. Prometheus utilise UID 65534 (nobody) et Grafana UID 472. Corrigez les permissions :

```
# Corriger les permissions pour Prometheus
sudo chown -R 65534:65534 ./prometheus_data/

# Corriger les permissions pour Grafana
sudo chown -R 472:472 ./grafana_data/

# Alternative : utiliser user dans docker-compose.yml
services:
  prometheus:
    user: "65534:65534"
```

**Problème 8 : Les dashboards provisionnés ne se mettent pas à jour.** Si vous modifiez un fichier JSON de dashboard et qu'il ne se met pas à jour dans Grafana, vérifiez que `allowUiUpdates: true` est défini dans le fichier de provisioning des dashboards. Notez que les modifications faites via l'interface Grafana seront écrasées au prochain redémarrage si le provisioning est actif.

## Conseils Avancés pour les Experts Prometheus Grafana

Ces techniques avancées sont utilisées par les équipes SRE des plus grandes entreprises tech en 2026. Elles permettent de passer d'un monitoring basique à une observabilité de niveau production.

### Recording Rules pour Accélérer les Dashboards

Les recording rules pré-calculent des requêtes PromQL coûteuses et stockent le résultat comme une nouvelle métrique. Au lieu de recalculer `rate(http_requests_total[5m])` à chaque chargement de dashboard, la recording rule le calcule une seule fois toutes les 15 secondes.

```
# prometheus/recording-rules.yml
groups:
  - name: node_recording_rules
    interval: 15s
    rules:
      - record: instance:node_cpu_utilization:ratio
        expr: 1 - avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))

      - record: instance:node_memory_utilization:ratio
        expr: 1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)

      - record: job:app_request_rate:5m
        expr: sum by(job) (rate(app_requests_total[5m]))

      - record: job:app_request_latency_p95:5m
        expr: histogram_quantile(0.95, sum by(job, le) (rate(app_request_duration_seconds_bucket[5m])))
```

Utilisez ensuite `instance:node_cpu_utilization:ratio` dans vos dashboards Grafana au lieu de la requête complète. Le temps de rendu passe de 2-5 secondes à quelques millisecondes pour les environnements avec des milliers de séries. La convention de nommage `level:metric:operations` est le standard CNCF recommandé.

### Fédération et Multi-Cluster Monitoring

Pour les architectures multi-environnements, la fédération Prometheus permet à un Prometheus central de scraper des métriques agrégées depuis des Prometheus locaux. Ajoutez cette configuration au Prometheus central :

```
# Sur le Prometheus central
scrape_configs:
  - job_name: 'federate-production'
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{__name__=~"instance:.*"}'
        - '{__name__=~"job:.*"}'
    static_configs:
      - targets: ['prometheus-prod.internal:9090']
        labels:
          cluster: 'production'

  - job_name: 'federate-staging'
    honor_labels: true
    metrics_path: '/federate'
    params:
      'match[]':
        - '{__name__=~"instance:.*"}'
    static_configs:
      - targets: ['prometheus-staging.internal:9090']
        labels:
          cluster: 'staging'
```

La fédération fonctionne mieux avec les recording rules : ne fédérez que les métriques pré-agrégées (préfixées `instance:` ou `job:`) pour limiter le trafic réseau et la charge sur le Prometheus central.

## Comparaison : Prometheus Grafana vs Alternatives en 2026

Avant de vous engager dans l'écosystème Prometheus Grafana, il est utile de comprendre comment cette stack se positionne face aux alternatives en 2026. Chaque solution a ses forces selon le contexte d'utilisation.

|Critère|Prometheus + Grafana|Datadog|Victoria Metrics|Elastic Observability|
|---|---|---|---|---|
|Coût (100 hôtes)|Gratuit (self-hosted)|~1 800 €/mois|Gratuit (self-hosted)|Gratuit (self-hosted)|
|Stockage longue durée|Via Thanos/Cortex|Natif (15 mois)|Natif (illimité)|Natif (ILM)|
|Courbe d'apprentissage|Moyenne (PromQL)|Faible (UI guidée)|Faible (compatible PromQL)|Élevée (KQL + ECS)|
|Scalabilité|Verticale (+ fédération)|Illimitée (SaaS)|Horizontale native|Horizontale native|
|Écosystème exporters|900+ exporters|750+ intégrations|Compatible Prometheus|Beats + agents|
|Support OpenTelemetry|Natif (v3.0+)|Natif|Natif|Natif|
|Alerting|Alertmanager|Intégré + ML|Compatible Alertmanager|Watcher + ML|

Prometheus Grafana reste le choix dominant pour les équipes qui privilégient le contrôle total, la portabilité et l'absence de vendor lock-in. Selon la [CNCF](https://www.cncf.io/projects/prometheus/), Prometheus est utilisé par 92 % des organisations cloud-native en 2026. Sa compatibilité avec plus de 900 exporters couvre pratiquement toutes les technologies existantes.

En revanche, pour les équipes qui préfèrent un service managé sans maintenance opérationnelle, des solutions comme [Datadog ou Grafana Cloud](https://tech-insider.org/fr/grafana-vs-datadog-2026/) offrent une expérience clé en main. Le choix dépend du ratio coût total de possession (TCO) vs temps d'ingénierie disponible pour la maintenance.

## Projet Complet : Stack de Monitoring Docker Compose

Voici le récapitulatif de tous les fichiers du projet. Clonez ou recréez cette structure pour disposer d'une stack Prometheus Grafana complète et fonctionnelle en moins de 5 minutes.

```
# Structure finale du projet
monitoring-stack/
├── docker-compose.yml              # Orchestration des 4 services
├── prometheus/
│   ├── prometheus.yml              # Configuration principale + scrape targets
│   ├── alert-rules.yml             # 6 règles d'alerte (CPU, RAM, disque, prédictive)
│   └── recording-rules.yml         # Recording rules pour performances
├── grafana/
│   ├── provisioning/
│   │   ├── datasources/
│   │   │   └── prometheus.yml      # Auto-configuration Prometheus
│   │   └── dashboards/
│   │       └── dashboards.yml      # Auto-import des dashboards
│   └── dashboards/
│       └── node-exporter.json      # Dashboard système complet
├── alertmanager/
│   └── alertmanager.yml            # Routage alertes + notifications email
└── app/                            # (Optionnel) Application Flask exemple
    ├── app.py
    ├── requirements.txt
    └── Dockerfile

# Lancer le projet complet
cd monitoring-stack
docker compose up -d

# Vérifier le statut
docker compose ps
curl -s http://localhost:9090/-/healthy
curl -s http://localhost:3000/api/health

# Accéder aux interfaces
# Prometheus : http://localhost:9090
# Grafana    : http://localhost:3000 (admin / SecurePass2026!)
# Alertmanager : http://localhost:9093
```

Ce projet couvre les bases solides d'un monitoring de production. Pour aller plus loin, vous pouvez ajouter [Loki](https://grafana.com/docs/grafana/latest/) pour la centralisation des logs, Tempo pour le tracing distribué, et ainsi constituer la stack LGTM complète (Loki, Grafana, Tempo, Mimir) qui est la référence open source de l'observabilité en 2026.

### Articles Connexes

Pour approfondir les sujets abordés dans ce tutoriel, consultez nos guides connexes :

- [Tutoriel Docker pour Débutants 2026](https://tech-insider.org/fr/docker-tutorial-beginners-containerization-2026/) — Si vous débutez avec Docker et les conteneurs
- [Maîtriser Docker Compose : Applications Multi-Conteneurs](https://tech-insider.org/fr/docker-compose-tutorial-multi-container-apps-2026/) — Approfondissez l'orchestration Docker Compose
- [Grafana vs Datadog 2026](https://tech-insider.org/fr/grafana-vs-datadog-2026/) — Comparaison détaillée des plateformes d'observabilité
- [Déployer avec Kubernetes et Helm](https://tech-insider.org/fr/kubernetes-helm-chart-tutorial-deploy-applications-2026/) — Passez au niveau supérieur avec Kubernetes
- [Tutoriel Flask Python 2026](https://tech-insider.org/fr/tutoriel-flask-python-application-web-2026/) — Créez l'application Python monitorée dans ce guide
- [Configurer Nginx comme Reverse Proxy](https://tech-insider.org/fr/nginx-reverse-proxy-tutorial-complete-guide-2026/) — Sécurisez l'accès à Prometheus et Grafana

## FAQ : Questions Fréquentes sur Prometheus Grafana

**Quelle est la différence entre Prometheus et Grafana ?**

Prometheus est un système de collecte et de stockage de métriques : il scrape les cibles, stocke les données dans son TSDB et évalue les règles d'alerte. Grafana est un outil de visualisation : il se connecte à Prometheus (et d'autres sources de données) pour afficher les métriques sous forme de graphiques, tableaux et dashboards. Ensemble, ils forment une stack de monitoring complète où Prometheus est le « backend » et Grafana le « frontend ».

**Combien de RAM faut-il pour Prometheus en production ?**

La consommation mémoire de Prometheus dépend directement du nombre de séries temporelles actives. Comptez 2 à 3 Ko par série active. Pour un environnement typique avec 50 000 séries (environ 50 serveurs avec Node Exporter et quelques applications), prévoyez 2 à 4 Go de RAM. Pour 500 000 séries et plus, envisagez 8 à 16 Go et utilisez les recording rules pour réduire la charge des requêtes.

**Prometheus peut-il stocker des données sur plusieurs mois ?**

Oui, mais ce n'est pas recommandé en stockage local. Prometheus 3.11 supporte des rétentions longues via `--storage.tsdb.retention.time`, mais pour des données au-delà de 90 jours, utilisez des solutions de stockage longue durée comme Thanos (stockage objet S3/GCS) ou Grafana Mimir. Ces solutions compressent les données historiques et offrent une rétention illimitée à moindre coût.

**Comment monitorer Kubernetes avec Prometheus Grafana ?**

La méthode standard est le Prometheus Operator (kube-prometheus-stack), un chart Helm qui déploie automatiquement Prometheus, Grafana, Alertmanager et les exporters nécessaires (kube-state-metrics, node-exporter). Il configure aussi le service discovery Kubernetes pour détecter automatiquement les nouveaux pods et services. L'installation se fait en une commande : `helm install monitoring prometheus-community/kube-prometheus-stack`.

**Est-ce que Prometheus supporte OpenTelemetry ?**

Depuis Prometheus 3.0 (fin 2024), le support OpenTelemetry est natif. Prometheus peut ingérer des métriques au format OTLP directement via le Remote Write receiver, sans conversion supplémentaire. Cela permet d'utiliser les SDK OpenTelemetry dans vos applications tout en conservant Prometheus comme backend de stockage. C'est la direction recommandée par la CNCF pour 2026.

**Comment sécuriser Prometheus et Grafana en production ?**

Trois niveaux de sécurité sont recommandés. Niveau 1 : activez le TLS et le basic_auth natifs de Prometheus via le fichier `web.yml`. Niveau 2 : placez Prometheus et Grafana derrière un reverse proxy (Nginx, Traefik) avec authentification OAuth2 ou LDAP. Niveau 3 : isolez la stack monitoring dans un réseau Docker dédié et n'exposez que Grafana publiquement, en configurant Prometheus avec `--web.listen-address=0.0.0.0:9090` uniquement sur le réseau interne.

**Quelle est la différence entre rate() et irate() en PromQL ?**

`rate()` calcule le taux moyen de croissance par seconde sur toute la fenêtre de temps spécifiée (ex : 5 minutes). `irate()` calcule le taux instantané entre les deux derniers points de données uniquement. Utilisez `rate()` pour les alertes et les dashboards de tendance (plus stable), et `irate()` pour les dashboards de détail en temps réel où vous voulez voir les pics instantanés. Ne mélangez jamais `irate()` avec des alertes — cela provoque du flapping.

**Comment migrer de Prometheus 2.x vers 3.x ?**

La migration vers Prometheus 3.x nécessite quelques ajustements. Les données TSDB de Prometheus 2.x sont compatibles avec la version 3.x, mais certains flags CLI dépréciés ont été supprimés. Avant la migration, exécutez `promtool check config` pour identifier les paramètres obsolètes. Les principales incompatibilités concernent la nouvelle UI (l'ancienne UI est supprimée), le changement de format des alertes, et les feature flags désormais activés par défaut. Consultez la [documentation officielle Prometheus](https://prometheus.io/docs/introduction/overview/) pour le guide de migration complet.

_Dernière mise à jour : 07 avril 2026. Testé avec Prometheus 3.11.0, Grafana 11.6.0, Node Exporter 1.9.0 et Alertmanager 0.28.1 sur Ubuntu 24.04 LTS._


[Voir tous les articles](https://tech-insider.org/fr/author/nadia-dubois/)