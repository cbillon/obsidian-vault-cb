---
link: https://slash-root.fr/docker-configuration-et-gestion-des-logs/
site: slash-root.fr
date: 2025-06-15T17:33
excerpt: Alalala, les logs ! Voici comment les configurer finement avec docker
  et tous les conteneurs.
slurped: 2026-09-23T06:54
title: "Docker : Configuration et gestion des logs"
---

[![Featured image of post Docker : Configuration et gestion des logs](app://obsidian.md/docker-configuration-et-gestion-des-logs/cover.png)](app://obsidian.md/docker-configuration-et-gestion-des-logs/)

[Docker](app://obsidian.md/categories/docker/)

### Alalala, les logs ! Voici comment les configurer finement avec docker et tous les conteneurs.

Salut les admins sys et autres dompteurs de conteneurs ! Aujourd'hui, plongeons dans l'univers des logs Docker. Comme on dit dans notre jargon : "Sans logs, c'est comme chercher une erreur dans un datacenter éteint - théoriquement possible, mais pas vraiment efficace."

## [](#comprendre-les-logs-dans-docker)Comprendre les logs dans Docker

Docker propose plusieurs pilotes de journalisation pour gérer les logs de vos conteneurs. Par défaut, tous vos messages sont acheminés vers le pilote `json-file`, qui stocke les logs sous forme de fichiers JSON dans `/var/lib/docker/containers/[container-id]/`.

Le problème ? Ces fichiers peuvent grossir... grossir... et encore grossir, jusqu'à dévorer tout l'espace disque de votre serveur. Une véritable histoire d'horreur pour tout admin système qui se respecte.

## [](#configuration-du-pilote-de-logs-par-d%c3%a9faut)Configuration du pilote de logs par défaut

La configuration des logs se fait dans le fichier `/etc/docker/daemon.json`. Voici comment éviter le scénario catastrophe du disque plein :

```
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

> **Note :** Cette configuration limite la taille de chaque fichier de log à 10 Mo et permet une rotation sur 3 fichiers maximum.

Après cette modification, un petit redémarrage s'impose :

```
sudo systemctl restart docker
```

## [](#les-diff%c3%a9rents-pilotes-de-logs)Les différents pilotes de logs

Docker ne se limite pas au format JSON. Voici les principaux pilotes disponibles, chacun avec ses forces et ses faiblesses :

1. **json-file** : Le pilote par défaut
    
    - Stocke les logs dans des fichiers JSON
    - Accessible via `docker logs`
    - Exemple de configuration personnalisée :
        
        ```
        docker run --log-driver=json-file \
                --log-opt max-size=10m \
                --log-opt max-file=3 \
                --log-opt labels=app_name \
                nginx
        ```
        
    - Parfait pour le développement et les petits déploiements
2. **local** : Le petit frère optimisé
    
    - Format binaire plus compact que JSON
    - Toujours accessible via `docker logs`
    - Exemple d'utilisation :
        
        ```
        docker run --log-driver=local \
                --log-opt max-size=10m \
                --log-opt max-file=5 \
                --log-opt compress=true \
                nginx
        ```
        
    - Idéal pour les environnements de production avec espace disque limité
3. **syslog** : L'option du barbu expérimenté
    
    - Envoie les logs au service syslog de l'hôte
    - Configuration type avec rotation via logrotate :
        
        ```
        # Dans /etc/docker/daemon.json
        {
        "log-driver": "syslog",
        "log-opts": {
         "syslog-address": "udp://192.168.1.100:514",
         "tag": "{{.ImageName}}/{{.Name}}/{{.ID}}",
         "syslog-facility": "local7"
        }
        }
        ```
        
    - Configuration équivalente dans docker-compose :
        
        ```
        services:
        app:
         image: nginx
         logging:
           driver: syslog
           options:
             syslog-address: "udp://192.168.1.100:514"
             tag: "{{.ImageName}}/{{.Name}}/{{.ID}}"
        ```
        
    - Petit prix à payer : `docker logs` ne fonctionne plus directement
4. **journald** : L'ami des systèmes modernes
    
    - Parfait pour les systèmes basés sur systemd
    - Exemple de configuration :
        
        ```
        # Activer journald dans /etc/docker/daemon.json
        {
        "log-driver": "journald",
        "log-opts": {
         "tag": "{{.ImageName}}/{{.Name}}/{{.ID}}",
         "labels": "app_name,environment"
        }
        }
        ```
        
    - Pour consulter les logs :
        
        ```
        # Voir les logs d'un conteneur
        journalctl CONTAINER_NAME=mon_conteneur
        
        # Suivre les logs en temps réel
        journalctl -f CONTAINER_NAME=mon_conteneur
        
        # Filtrer par priorité
        journalctl -p err CONTAINER_NAME=mon_conteneur
        ```
        
    - Le choix évident si vous utilisez déjà systemd pour tout le reste
5. **Les poids lourds** : pour les environnements critiques
    
    **Splunk** :
    
    ```
    docker run --log-driver=splunk \
              --log-opt splunk-token=YOUR_TOKEN \
              --log-opt splunk-url=https://your-splunk:8088 \
              --log-opt tag="{{.ImageName}}/{{.Name}}/{{.ID}}" \
              nginx
    ```
    
    **AWS CloudWatch Logs** :
    
    ```
    docker run --log-driver=awslogs \
              --log-opt awslogs-region=eu-west-1 \
              --log-opt awslogs-group=my-log-group \
              --log-opt awslogs-stream=my-stream \
              nginx
    ```
    
    **Fluentd** :
    
    ```
    # docker-compose.yml
    version: '3'
    services:
     app:
       image: nginx
       logging:
         driver: "fluentd"
         options:
           fluentd-address: localhost:24224
           tag: docker.{{.Name}}
    
     fluentd:
       image: fluent/fluentd
       volumes:
         - ./fluentd/conf:/fluentd/etc
       ports:
         - "24224:24224"
    ```
    
    **GELF** (Graylog Extended Log Format) :
    
    ```
    docker run --log-driver=gelf \
              --log-opt gelf-address=udp://graylog:12201 \
              --log-opt tag="{{.ImageName}}/{{.Name}}" \
              nginx
    ```
    
    Ces solutions sont idéales pour les environnements de production à grande échelle nécessitant une analyse avancée des logs.
    

## [](#configuration-par-conteneur)Configuration par conteneur

Parfois, un conteneur mérite un traitement spécial. Vous pouvez configurer le pilote de logs individuellement, comme un couturier qui ajuste un costume sur mesure :

```
docker run --log-driver=journald --log-opt labels=app_id --name=mon-app nginx
```

Ou dans un fichier docker-compose.yml :

```
version: '3'
services:
  mon_service:
    image: mon_image
    logging:
      driver: json-file
      options:
        max-size: "10m"  # Taille maximale d'un fichier de log
        max-file: "3"     # Nombre maximum de fichiers de rotation
```

## [](#bonnes-pratiques-pour-la-gestion-des-logs)Bonnes pratiques pour la gestion des logs

1. **La rotation, c'est la vie**
    
    - Configurez toujours `max-size` et `max-file`
    - Évitez les fichiers de logs de 42 Go qui font pleurer les disques durs
2. **Étiquetez, c'est gagné**
    
    - Utilisez des tags personnalisés pour chaque conteneur
    - Ajoutez des labels pour plus de contexte
3. **Les logs, c'est comme le bon vin : ça se conserve**
    
    - Centralisez avec ELK, Graylog ou Loki
    - Pensez à la rétention et à l'archivage
4. **La surveillance, c'est comme le café du matin : indispensable**
    
    - Surveillez l'espace disque utilisé par les logs
    - Configurez des alertes pour les cas critiques, surtout en production
5. **Structurez vos logs** : encouragez vos développeurs à générer des logs au format JSON pour faciliter leur analyse
    

## [](#conclusion)Conclusion

Les logs, c'est un peu comme le journal intime de vos conteneurs. Une bonne configuration peut transformer une enquête policière en simple coup d'œil. Ou l'inverse.

Comme dirait un vieux barbu de l'IT : "Les logs, c'est comme le papier toilette - on ne réalise leur importance que quand on en a plus." Alors, à vos fichiers de configuration, et que la force des logs soit avec vous !

Et souvenez-vous : dans un monde de conteneurs éphémères, seuls les logs racontent l'histoire complète (du moins jusqu'à ce qu'ils soient eux aussi supprimés par rotation).