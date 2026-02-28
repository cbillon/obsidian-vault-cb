---
link: https://slash-root.fr/docker-metrics-port-9323/
byline: Julien LOUIS
site: slash-root.fr
date: 2025-06-15T18:55
excerpt: Hmmm des metrics docker miam
slurped: 2026-02-24T09:59
title: "Docker : Metrics port 9323 - slash-root.fr"
tags:
  - docker
  - monitoring
---

L'option `metrics-addr` est une configuration importante du daemon Docker qui permet d'exposer les métriques de performance et d'utilisation des ressources via un endpoint HTTP. Cette fonctionnalité, combinée avec le port par défaut 9323, vous donne accès à une mine d'informations sur votre environnement Docker.

## Comprendre l'option metrics-addr

L'option `metrics-addr` configure l'adresse IP et le port sur lesquels le daemon Docker expose ses métriques. Par défaut, quand cette option est activée, Docker utilise le port 9323.

### Pourquoi le port 9323 ?

- C'est un port non-privilégié (supérieur à 1024)
- Il est officiellement alloué à Docker par l'IANA pour cette fonctionnalité
- Il est peu susceptible d'entrer en conflit avec d'autres services courants
- Il suit la convention de Prometheus pour les ports d'exportation de métriques (9xxx)

## Ce que vous obtenez avec metrics-addr

Activer l'option `metrics-addr` dans le daemon.json vous donne accès à de nombreuses métriques Docker formatées nativement pour Prometheus :

- **Métriques système** : Utilisation CPU, mémoire, disque et réseau par conteneur
- **Métriques du daemon** : État du daemon, nombre d'objets gérés (conteneurs, images, volumes)
- **Métriques d'utilisation** : Temps de démarrage des conteneurs, opérations réseau
- **Métriques de performance** : Latences des opérations de build et de déploiement

Toutes ces métriques sont disponibles au format Prometheus, ce qui les rend facilement intégrables dans un système de monitoring existant.

> ℹ️ **Point technique** : Le format Prometheus utilise des "labels" qui permettent de filtrer et d'agréger les métriques de manière flexible.

## Cas d'utilisation pour metrics-addr

L'activation de `metrics-addr` est particulièrement utile dans les scénarios suivants :

- **Supervision d'infrastructure** : Intégration avec Prometheus pour surveiller l'ensemble de votre environnement Docker
- **Alerting automatisé** : Déclencher des alertes basées sur des seuils de métriques spécifiques
- **Dashboarding** : Création de tableaux de bord Grafana pour visualiser les performances
- **Analyse de tendance** : Collecter des données sur de longues périodes pour identifier les tendances
- **Détection d'anomalies** : Repérer les comportements anormaux dans vos conteneurs

## Configuration de metrics-addr

### Configuration via daemon.json

La méthode recommandée pour activer les métriques Docker est de modifier le fichier de configuration du daemon :

1. Éditez le fichier de configuration Docker :
    
    ```
    sudo vi /etc/docker/daemon.json
    ```
    
2. Ajoutez la configuration pour activer l'API de métriques :
    
    ```
    {
     "metrics-addr" : "127.0.0.1:9323",
     "experimental" : true
    }
    ```
    
    > **Note :** L'adresse `127.0.0.1` limite l'accès à la machine locale. Pour exposer les métriques au réseau, utilisez `0.0.0.0:9323` (avec précaution).
    
3. Redémarrez Docker pour appliquer les changements :
    
    ```
    sudo systemctl restart docker
    ```
    
4. Vérifiez que les métriques sont disponibles :
    
    ```
    curl http://localhost:9323/metrics
    ```
    
    Vous devriez voir une sortie avec de nombreuses métriques au format Prometheus.
    

### Visualisation simple avec Docker Stats

Pour un aperçu rapide des performances de vos conteneurs sans configuration supplémentaire :

```
docker stats
```

Cela affiche en temps réel les statistiques d'utilisation des ressources pour tous les conteneurs :

- CPU% : Pourcentage d'utilisation du CPU
- MEM% : Pourcentage d'utilisation de la mémoire
- NET I/O : Transfert réseau entrant/sortant
- BLOCK I/O : Lectures/écritures sur disque
- PIDS : Nombre de processus

Pour cibler des conteneurs spécifiques :

```
docker stats container1 container2
```

> **Note:** Pour des solutions plus avancées comme Prometheus+Grafana ou cAdvisor, voir la section "Solutions avancées" en fin d'article.

## Les limites à connaître

1. **Limites des métriques natives** :
    
    - Pas de persistance des données historiques
    - Interface de visualisation minimale
    - Absence d'alertes automatiques
2. **Considérations de performance** :
    
    - L'activation des métriques consomme des ressources supplémentaires
    - L'exposition sur le réseau nécessite des précautions de sécurité
    - Le port 9323 doit être protégé par un pare-feu ou limité à localhost
3. **Limites de fonctionnalités** :
    
    - Les métriques Docker ne couvrent pas les métriques applicatives
    - Pas d'information sur la santé des applications à l'intérieur des conteneurs
    - Nécessite des outils complémentaires pour une vision complète

## Bonnes pratiques

1. **Commencez simplement**
    
    - Identifiez les métriques critiques pour votre cas d'usage
    - Augmentez progressivement la complexité
2. **Automatisez les alertes**
    
    - Configurez des alertes sur les seuils critiques
    - Évitez les alertes trop sensibles qui causeraient de la fatigue d'alarme
3. **Corrélation de métriques**
    
    - Combinez les métriques Docker avec des métriques système et applicatives
    - Utilisez des dashboards qui montrent des corrélations entre métriques
4. **Conservation des données**
    
    - Définissez une politique de rétention adaptée à vos besoins
    - Agrégez les données anciennes pour économiser de l'espace

## En résumé

Les métriques Docker sont essentielles pour une gestion efficace des conteneurs en production. La configuration via daemon.json offre un moyen simple d'exposer ces données pour votre monitoring, vous donnant une première couche de visibilité sur vos conteneurs.

> 💡 **Astuce** : Commencez par les métriques natives et docker stats pour comprendre vos besoins, puis évaluez si vous avez besoin de solutions plus avancées.

## Solutions avancées (Annexe)

### 1. Intégration avec Prometheus

Les métriques exposées via daemon.json sont déjà au format Prometheus. Pour les collecter :

```
# prometheus.yml
scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['127.0.0.1:9323']
```

### 2. Utilisation de cAdvisor

Pour une solution prête à l'emploi avec interface web :

```
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  gcr.io/cadvisor/cadvisor:latest
```

Interface disponible sur [http://localhost:8080](http://localhost:8080/)