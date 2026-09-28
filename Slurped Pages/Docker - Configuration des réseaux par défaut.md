---
link: https://slash-root.fr/docker-configuration-des-reseaux-par-defaut/
site: slash-root.fr
date: 2025-06-15T16:17
excerpt: Note sur comment gérer la configuration des réseaux dans docker engine
  et dans un contexte de cluster swarm.
slurped: 2026-09-23T06:53
title: "Docker : Configuration des réseaux par défaut"
---

Salut les admins sys et autres dompteurs de conteneurs ! Aujourd'hui, plongeons dans le fascinant (et parfois frustrant) monde des réseaux Docker. Comme on dit dans notre jargon : "Un conteneur sans réseau, c'est comme un pingouin sans banquise - techniquement possible, mais pas vraiment utile."

## [](#les-r%c3%a9seaux-docker-par-d%c3%a9faut)Les réseaux Docker par défaut

Docker, cet invité un peu sans-gêne, s'installe et crée directement trois réseaux :

- `bridge` : Le réseau par défaut où tous vos conteneurs font la fête (172.17.0.0/16)
- `host` : Pour quand vos conteneurs veulent se sentir comme à la maison
- `none` : Pour les conteneurs asociaux qui préfèrent rester dans leur coin

Le réseau `bridge` s'approprie allègrement la plage 172.17.0.0/16. Mais que se passe-t-il quand cette plage est déjà occupée par votre infrastructure existante ? Eh bien, c'est un peu comme deux familles qui réservent le même bungalow pour les vacances - ça crée des tensions.

### [](#r%c3%a9solution-des-conflits-avec-le-r%c3%a9seau-par-d%c3%a9faut)Résolution des conflits avec le réseau par défaut

Pour modifier la plage d'adresses que Docker utilise par défaut, éditez le fichier `/etc/docker/daemon.json` (si le fichier n'existe pas, créez-le avec la même joie que lorsque vous créez un nouveau répertoire `batman`) :

```
{
  "default-address-pools": [
    {
      "base": "192.168.66.0/24",
      "size": 24
    }
  ]
}
```

Après cette modification, un petit redémarrage s'impose :

```
sudo systemctl restart docker
```

## [](#incr%c3%a9mentation-automatique-des-r%c3%a9seaux-docker)Incrémentation automatique des réseaux Docker

Quand vous créez de nouveaux réseaux Docker avec `docker network create`, Docker fait preuve d'une logique implacable pour l'allocation des sous-réseaux. En fonction de votre configuration `default-address-pools`, Docker va attribuer automatiquement des plages d'adresses IP qui s'incrémentent à chaque nouveau réseau :

```
docker network create mon-super-reseau
```

Pour cette commande, Docker utilisera automatiquement la première plage disponible: 192.168.66.0/24

```
docker network create mon-autre-reseau
```

Pour ce deuxième réseau, Docker passera à la plage suivante: 192.168.67.0/24

C'est comme distribuer des tranches de gâteau - mais en version IP.

Si vous préférez choisir votre part vous-même :

```
docker network create --subnet=192.168.42.0/24 mon-reseau-avec-le-nombre-de-la-vie
```

## [](#les-diff%c3%a9rents-types-de-r%c3%a9seaux-docker)Les différents types de réseaux Docker

### [](#dans-un-docker-engine-standalone)Dans un Docker Engine standalone

Un Docker Engine standard (non-Swarm) propose plusieurs types de réseaux pour répondre à des besoins distincts :

- `bridge` : Le réseau par défaut utilisé par les conteneurs. C'est un réseau virtuel interne qui permet aux conteneurs de communiquer entre eux et avec l'hôte.
    
- `host` : Supprime l'isolation réseau entre le conteneur et l'hôte. Le conteneur utilise directement la pile réseau de l'hôte. Utile pour les performances mais réduit l'isolation.
    
- `none` : Désactive complètement le réseau. Le conteneur n'a aucun accès réseau externe. Utile pour les traitements isolés ne nécessitant pas de communication.
    
- `macvlan` : Attribue une adresse MAC physique aux conteneurs, les faisant apparaître comme des périphériques physiques sur le réseau. Idéal pour migrer des applications legacy qui dépendent d'IPs spécifiques.
    

```
# Créer un réseau bridge personnalisé (recommandé au lieu du bridge par défaut)
docker network create --driver bridge mon-reseau-bridge
```

### [](#dans-un-cluster-docker-swarm)Dans un cluster Docker Swarm

Docker Swarm, c'est comme une colocation de conteneurs répartis sur plusieurs serveurs. La communication entre nœuds nécessite des réseaux spéciaux :

- `overlay` : Permet aux conteneurs de différents nœuds de communiquer comme s'ils étaient sur le même hôte. Crée un réseau virtual "au-dessus" de l'infrastructure physique.
    
- `ingress` : Un réseau overlay spécial qui facilite le routage du trafic entrant vers les services appropriés, indépendamment du nœud qui héberge le conteneur. Gère également l'équilibrage de charge interne.
    

```
# Créer un réseau overlay pour les services Swarm
docker network create --driver overlay --attachable mon-reseau-multi-hote
```

Tous ces réseaux peuvent coexister sur le même hote ou cluster, permettant ainsi une architecture réseau flexible et adaptée à chaque cas d'usage.

### [](#le-r%c3%a9seau-ingress-et-la-plage-100008)Le réseau Ingress et la plage 10.0.0.0/8

Par défaut, ce cher réseau `ingress` s'installe confortablement dans la plage 10.0.0.0/8. Si vous utilisez déjà cette plage, les symptômes sont clairs :

- Docker vous lance des erreurs cryptiques avec "network overlap"
- Vos conteneurs deviennent subitement muets
- Vous commencez à douter de vos choix de carrière

### [](#reconfiguration-du-r%c3%a9seau-ingress)Reconfiguration du réseau Ingress

Contrairement à ce qu'on pourrait croire, **pas besoin de quitter le swarm** pour résoudre ce problème ! Une approche beaucoup plus civilisée consiste à :

1. Supprimer poliment le réseau ingress existant :

```
docker network rm ingress
```

2. Créer un nouveau réseau ingress avec votre propre plage :

```
docker network create \
  --driver overlay \
  --ingress \
  --subnet=172.30.0.0/16 \
  --gateway=172.30.0.1 \
  ingress
```

Voilà qui est beaucoup moins traumatisant pour votre swarm que de tout démolir et reconstruire !

## [](#bonnes-pratiques-pour-la-gestion-des-r%c3%a9seaux-docker)Bonnes pratiques pour la gestion des réseaux Docker

1. **Planifiez votre allocation d'adresses IP** : Un schéma bien pensé vaut mieux qu'une nuit blanche de débogage
    
2. **Choisissez le bon type de réseau** : Ne vous limitez pas au `bridge` par défaut.
    
    - **Bridge personnalisé** (`docker network create my_bridge`) : Résolution DNS entre conteneurs, isolation, idéal pour les applications sur un seul hôte
    - **Overlay** (`docker network create -d overlay my_overlay`) : Communication entre hôtes dans Swarm
    - **Macvlan** (`docker network create -d macvlan --subnet=192.168.40.0/24 --gateway=192.168.40.1 -o parent=eth0 my_macvlan`) : Intégration au réseau physique
3. **Sécurisez votre infrastructure** : **Évitez d'exposer les ports de base de données** (comme MySQL, PostgreSQL, MongoDB) sur votre réseau hôte. C'est une faille de sécurité courante et totalement évitable. Dans les réseaux Docker Compose ou Swarm, la communication inter-services se fait nativement via le nom du service. En pratique, votre service d'application peut simplement utiliser `db:3306` comme adresse de connexion, sans jamais avoir à publier le port 3306 sur l'interface de l'hôte. Dans Swarm, cette communication fonctionne même entre des conteneurs déployés sur différents nœuds physiques via les réseaux overlay
    
4. **Adoptez une nomenclature claire** : Préférez `project_frontend_net` ou `prod_db_network` à des noms génériques comme `net1`
    
5. **Documentez votre architecture réseau** : Votre futur vous (ou votre remplaçant après votre burnout) vous remerciera
    

## [](#conclusion)Conclusion

La gestion des réseaux Docker ressemble parfois à un jeu de Tetris avec des blocs CIDR - challenging mais satisfaisant une fois maîtrisé. En comprenant comment Docker assigne et gère les adresses IP, vous pourrez éviter ces moments où, tard dans la nuit, vous fixez votre écran en murmurant "mais pourquoi ça ne communique pas ?".

Une bonne configuration réseau vous permettra de dormir tranquille, sachant que vos conteneurs peuvent socialiser entre eux sans créer d'incident diplomatique sur votre infrastructure.

Et souvenez-vous : dans un monde de conteneurs éphémères, seule une bonne configuration réseau est éternelle (ou du moins, jusqu'au prochain `docker-compose down`).