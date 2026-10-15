---
link: https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/
byline: Nadia Dubois
site: Tech Insider
date: 2026-03-23T17:21
excerpt: "Tutoriel Docker Compose complet : créez une application
  multi-conteneurs avec FastAPI, PostgreSQL, Redis et Nginx. 12 étapes, code
  fonctionnel et dépannage."
slurped: 2026-10-15T09:59
title: "Tutoriel Docker Compose 2026 : Guide Complet Multi-Conteneurs"
---

23 March 2026

30 min de lecture

Docker Compose est devenu l’outil incontournable pour orchestrer des applications multi-conteneurs. Depuis la sortie de Docker Compose v5.0.0 “Mont Blanc” fin 2025 jusqu’à la v5.5.0, publiée le 17 août 2026 avec la prise en charge native des conteneurs d’initialisation (init containers), l’outil n’a cessé de gagner en maturité : maîtriser sa dernière génération est plus que jamais essentiel pour tout développeur ou DevOps. Ce tutoriel complet vous guide pas à pas, de l’installation à la mise en production, en construisant un projet fonctionnel avec une API Python, une base de données PostgreSQL, un cache Redis et un proxy Nginx.

1. [Prérequis : Versions et Outils Nécessaires](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-0)
2. [Comprendre l’Architecture Docker Compose en 2026](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-1)
3. [Étape 1 : Initialiser la Structure du Projet](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-2)
4. [Étape 2 : Créer l’API FastAPI et son Dockerfile](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-3)
5. [Étape 3 : Configurer PostgreSQL et le Script d’Initialisation](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-4)
6. [Étape 4 : Configurer Nginx comme Reverse Proxy](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-5)
7. [Étape 5 : Écrire le Fichier compose.yaml Complet](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-6)
8. [Étape 6 : Configurer les Variables d’Environnement](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-7)
9. [Étape 7 : Lancer et Vérifier les Services](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-8)
10. [Étape 8 : Utiliser les Commandes Docker Compose Essentielles](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-9)
11. [Étape 9 : Implémenter les Health Checks et la Gestion des Dépendances](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-10)
12. [Étape 10 : Gérer les Volumes et la Persistance des Données](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-11)
13. [Étape 11 : Optimiser pour la Production avec les Profils et Overrides](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-12)
14. [Étape 12 : Mettre en Place la Supervision et les Logs](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-13)
15. [Dépannage : 8 Problèmes Courants et Leurs Solutions](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-14)

1. [Problème 1 : “port is already allocated”](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-15)
2. [Problème 2 : “no matching manifest for linux/arm64”](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-16)
3. [Problème 3 : L’API ne se connecte pas à la base de données](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-17)
4. [Problème 4 : Les modifications du code ne sont pas prises en compte](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-18)
5. [Problème 5 : “insufficient memory” ou conteneur qui crashe](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-19)
6. [Problème 6 : Les données disparaissent après un redémarrage](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-20)
7. [Problème 7 : “network compose-tutorial_backend not found”](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-21)
8. [Problème 8 : Compose ne détecte pas les changements dans le fichier .env](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-22)

17. [Astuces Avancées Docker Compose en 2026](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-23)
18. [Comparaison : Docker Compose vs Alternatives en 2026](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-24)
19. [Bonnes Pratiques de Sécurité Docker Compose](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-25)
20. [FAQ : Questions Fréquentes sur Docker Compose](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-26)

1. [Quelle est la différence entre docker-compose et docker compose ?](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-27)
2. [Docker Compose est-il adapté à la production ?](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-28)
3. [Comment mettre à jour un seul service sans interrompre les autres ?](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-29)
4. [Peut-on utiliser Docker Compose avec Docker Swarm ?](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-30)
5. [Comment accéder à la base de données depuis un outil externe ?](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-31)
6. [Docker Compose est-il gratuit ?](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-32)
7. [Comment déboguer un conteneur qui ne démarre pas ?](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-33)
8. [Quelle est la différence entre volumes et bind mounts ?](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-34)

22. [Couverture Associée](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/#toc-35)

Que vous soyez un développeur qui lance `docker run` manuellement pour chaque conteneur ou un ingénieur DevOps cherchant à standardiser vos environnements, ce guide vous permettra de comprendre Docker Compose en profondeur et de l’utiliser efficacement dans vos projets professionnels. Chaque étape inclut des exemples de code fonctionnels, des sorties attendues et des pièges courants à éviter.


## Prérequis : Versions et Outils Nécessaires

Avant de commencer ce tutoriel Docker Compose, assurez-vous de disposer des outils suivants installés sur votre machine. Chaque version a été testée et validée pour garantir la compatibilité avec les exemples de ce guide. Docker Desktop, disponible sur Windows, macOS et Linux, embarque nativement le plugin Docker Compose – la version 4.79.0, sortie le 22 juin 2026, embarquait déjà Docker Compose v5.4.0 et son nouveau workflow de réconciliation, ce qui simplifie considérablement l’installation et la maintenance des environnements.

|Outil|Version minimale|Version recommandée (mars 2026)|Remarque|
|---|---|---|---|
|Docker Engine|24.0|29.1|Inclut les dernières optimisations de performance|
|Docker Compose|2.25.0|v2.40+ / v5.0.0|Plugin intégré à Docker CLI|
|Docker Desktop|4.30|4.40+|Optionnel mais recommandé sur Windows/macOS|
|Python|3.10|3.12|Pour notre API FastAPI|
|Git|2.30|2.44+|Pour cloner le projet exemple|
|VS Code / Éditeur|–|Dernière version|Extension Docker recommandée|

Pour vérifier que Docker et Docker Compose sont correctement installés, exécutez les commandes suivantes dans votre terminal. Si Docker Compose n’est pas disponible en tant que plugin, vous devrez peut-être le mettre à jour. Depuis la migration vers Compose V2, la commande utilise un espace au lieu d’un tiret : `docker compose` remplace l’ancien `docker-compose`. Cette distinction est fondamentale et source de nombreuses erreurs chez les débutants.

```
# Vérifier les versions installées
docker --version
# Sortie attendue : Docker version 29.1.x, build xxxxxxx

docker compose version
# Sortie attendue : Docker Compose version v2.40.x

# Si vous avez Docker Desktop, tout est déjà inclus
# Sinon, installer le plugin Compose manuellement :
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

Un point critique souvent négligé : assurez-vous que votre utilisateur fait partie du groupe `docker` pour éviter de devoir utiliser `sudo` à chaque commande. Sur Linux, exécutez `sudo usermod -aG docker $USER` puis déconnectez-vous et reconnectez-vous. Sur Docker Desktop (Windows/macOS), cela est géré automatiquement.

## Comprendre l’Architecture Docker Compose en 2026

Docker Compose repose sur un fichier YAML déclaratif – `compose.yaml` (le nom recommandé depuis 2024, remplaçant `docker-compose.yml`) – qui décrit l’ensemble de votre stack applicative. Chaque service correspond à un conteneur, et Compose gère automatiquement le réseau, les volumes et les dépendances entre services. En 2026, avec la sortie de Docker Compose v5.0.0 “Mont Blanc” fin 2025, l’écosystème a connu une refonte majeure incluant un support SDK officiel, la délégation des builds à Docker Bake et le support des ressources OCI et Git distantes. Cette lignée v5 s’est enrichie rapidement : la v5.1.4 du 20 mai 2026 a précédé la v5.2.0 du 23 juin 2026, puis la v5.4.0, publiée le 3 août 2026, a introduit un nouveau workflow de réconciliation pour les volumes et les réseaux, réduisant les dérives de configuration entre l’état déclaré et l’état réel de la stack.

L’architecture de Docker Compose se compose de quatre éléments fondamentaux. Les **services** définissent les conteneurs à exécuter, avec leur image, leurs ports, leurs variables d’environnement et leurs dépendances. Les **réseaux** permettent aux conteneurs de communiquer entre eux de manière isolée, créant des segments réseau virtuels qui protègent vos applications. Les **volumes** assurent la persistance des données en dehors du cycle de vie des conteneurs, ce qui est essentiel pour les bases de données. Enfin, les **configurations** et **secrets** permettent de gérer les paramètres sensibles de manière sécurisée.

La branche v2 a vécu son dernier chapitre avec la v2.40.3, publiée le 30 octobre 2025 – la dernière release de la lignée v2 avant le grand saut vers v5 – et intégrée à Debian experimental (paquet 2.40.3-1) le 11 mars 2026 par la Debian Go Packaging Team, preuve de sa diffusion jusque dans les distributions Linux. La v5.0.0 a ensuite migré vers les modules Moby et mis à jour containerd vers la version 2.2.2, renforçant la stabilité globale de l’outil. Ces évolutions font de Docker Compose un outil de plus en plus mature, utilisé par des millions de développeurs à travers le monde pour standardiser leurs environnements de développement et de production.

## Étape 1 : Initialiser la Structure du Projet

Commençons par créer la structure complète de notre projet. Nous allons construire une application web composée de quatre services : une API REST Python avec FastAPI, une base de données PostgreSQL 16, un cache Redis 7 et un reverse proxy Nginx. Cette architecture reflète une stack de production réaliste que vous retrouverez dans de nombreuses entreprises en 2026.

```
# Créer la structure du projet
mkdir -p compose-tutorial/{api,nginx,db}
cd compose-tutorial

# Créer les fichiers nécessaires
touch compose.yaml
touch api/main.py api/requirements.txt api/Dockerfile
touch nginx/nginx.conf
touch db/init.sql
touch .env

# Structure finale :
# compose-tutorial/
# ├── compose.yaml
# ├── .env
# ├── api/
# │   ├── Dockerfile
# │   ├── main.py
# │   └── requirements.txt
# ├── nginx/
# │   └── nginx.conf
# └── db/
#     └── init.sql
```

Cette organisation respecte les bonnes pratiques Docker en séparant chaque service dans son propre répertoire. Le fichier `compose.yaml` à la racine orchestre l’ensemble, tandis que chaque service dispose de son contexte de build isolé. Le fichier `.env` centralise les variables d’environnement sensibles, évitant ainsi de les coder en dur dans le fichier Compose. Notez que nous utilisons le nom `compose.yaml` – le format officiellement recommandé en 2026 – plutôt que l’ancien `docker-compose.yml`.

**Piège courant n°1 :** Ne placez jamais votre fichier `.env` dans un dépôt Git public. Ajoutez-le immédiatement à votre `.gitignore`. Une erreur fréquente chez les développeurs débutants est de versionner des mots de passe de base de données ou des clés API. En 2026, les outils de scanning de secrets comme GitGuardian détectent automatiquement ces fuites, mais la prévention reste la meilleure approche.

## Étape 2 : Créer l’API FastAPI et son Dockerfile

Notre API Python utilise FastAPI, le framework web le plus populaire en France en 2026, pour créer des endpoints REST performants. FastAPI offre une validation automatique des données, une documentation OpenAPI intégrée et des performances asynchrones natives, ce qui en fait le choix idéal pour notre démonstration Docker Compose.

Commençons par le fichier `api/requirements.txt` qui liste les dépendances Python :

```
# api/requirements.txt
fastapi==0.115.6
uvicorn[standard]==0.34.0
psycopg2-binary==2.9.10
redis==5.2.1
pydantic==2.10.4
```

Ensuite, créez l’API dans `api/main.py`. Cette application se connecte à PostgreSQL et Redis, expose des endpoints CRUD et inclut un health check – un pattern essentiel pour Docker Compose :

```
# api/main.py
import os
import redis
import psycopg2
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from contextlib import asynccontextmanager

# Configuration via variables d'environnement
DB_HOST = os.getenv("DB_HOST", "db")
DB_NAME = os.getenv("POSTGRES_DB", "appdb")
DB_USER = os.getenv("POSTGRES_USER", "appuser")
DB_PASS = os.getenv("POSTGRES_PASSWORD", "changeme")
REDIS_HOST = os.getenv("REDIS_HOST", "redis")

def get_db_connection():
    return psycopg2.connect(
        host=DB_HOST, database=DB_NAME,
        user=DB_USER, password=DB_PASS
    )

redis_client = redis.Redis(host=REDIS_HOST, port=6379, decode_responses=True)

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Initialisation au démarrage
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute("""
        CREATE TABLE IF NOT EXISTS items (
            id SERIAL PRIMARY KEY,
            name VARCHAR(100) NOT NULL,
            description TEXT,
            created_at TIMESTAMP DEFAULT NOW()
        )
    """)
    conn.commit()
    cur.close()
    conn.close()
    yield

app = FastAPI(title="Compose Tutorial API", version="1.0.0", lifespan=lifespan)

class ItemCreate(BaseModel):
    name: str
    description: str = ""

@app.get("/health")
def health_check():
    """Endpoint de santé pour Docker Compose"""
    try:
        conn = get_db_connection()
        conn.close()
        redis_client.ping()
        return {"status": "healthy", "db": "ok", "cache": "ok"}
    except Exception as e:
        raise HTTPException(status_code=503, detail=str(e))

@app.get("/items")
def list_items():
    # Vérifier le cache Redis
    cached = redis_client.get("items_list")
    if cached:
        import json
        return {"source": "cache", "items": json.loads(cached)}
    
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute("SELECT id, name, description FROM items ORDER BY id DESC LIMIT 50")
    rows = cur.fetchall()
    cur.close()
    conn.close()
    
    items = [{"id": r[0], "name": r[1], "description": r[2]} for r in rows]
    import json
    redis_client.setex("items_list", 30, json.dumps(items))
    return {"source": "database", "items": items}

@app.post("/items", status_code=201)
def create_item(item: ItemCreate):
    conn = get_db_connection()
    cur = conn.cursor()
    cur.execute(
        "INSERT INTO items (name, description) VALUES (%s, %s) RETURNING id",
        (item.name, item.description)
    )
    item_id = cur.fetchone()[0]
    conn.commit()
    cur.close()
    conn.close()
    redis_client.delete("items_list")  # Invalider le cache
    return {"id": item_id, "name": item.name}
```

Créez maintenant le Dockerfile multi-étapes pour l’API, une approche qui optimise la taille de l’image finale :

```
# api/Dockerfile
FROM python:3.12-slim AS base

WORKDIR /app

# Installer les dépendances système pour psycopg2
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq-dev gcc \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY main.py .

# Créer un utilisateur non-root pour la sécurité
RUN useradd -m appuser
USER appuser

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Piège courant n°2 :** N’utilisez jamais l’image `python:3.12` complète en production. L’image `slim` pèse environ 150 Mo contre 1 Go pour l’image complète. Pour des images encore plus légères, considérez `python:3.12-alpine`, mais attention : Alpine utilise `musl` au lieu de `glibc`, ce qui peut causer des incompatibilités avec certaines bibliothèques comme `psycopg2`. C’est pourquoi nous utilisons `slim` dans ce tutoriel.

## Étape 3 : Configurer PostgreSQL et le Script d’Initialisation

PostgreSQL est la base de données relationnelle la plus utilisée avec Docker Compose en 2026, grâce à sa robustesse et ses fonctionnalités avancées. Notre configuration utilise PostgreSQL 16, la version LTS la plus récente, avec un script d’initialisation automatique qui crée le schéma de base de données au premier lancement.

Créez le fichier `db/init.sql` qui sera automatiquement exécuté par PostgreSQL lors de la première initialisation du conteneur :

```
-- db/init.sql
-- Script d'initialisation PostgreSQL pour Docker Compose

-- Créer une extension utile
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Table principale
CREATE TABLE IF NOT EXISTS items (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT DEFAULT '',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Index pour les performances
CREATE INDEX IF NOT EXISTS idx_items_name ON items(name);
CREATE INDEX IF NOT EXISTS idx_items_created ON items(created_at DESC);

-- Données de test
INSERT INTO items (name, description) VALUES
    ('Premier élément', 'Créé automatiquement par le script init'),
    ('Deuxième élément', 'Données de démonstration Docker Compose'),
    ('Troisième élément', 'Tutoriel tech-insider.org 2026');

-- Fonction de mise à jour automatique du timestamp
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_items_modtime
    BEFORE UPDATE ON items
    FOR EACH ROW EXECUTE FUNCTION update_modified_column();
```

Le répertoire `/docker-entrypoint-initdb.d/` de l’image officielle PostgreSQL exécute automatiquement tous les fichiers `.sql` et `.sh` qu’il contient lors de la première initialisation. Cela signifie que ce script ne s’exécutera que si le volume de données est vide – un comportement qui surprend souvent les développeurs qui modifient leur script et ne voient pas les changements appliqués.

**Piège courant n°3 :** Si vous modifiez le script `init.sql` après avoir déjà lancé les conteneurs, les changements ne seront pas appliqués car le volume PostgreSQL contient déjà des données. Vous devez supprimer le volume avec `docker compose down -v` puis relancer `docker compose up`. Attention : cette opération détruit toutes les données existantes.

## Étape 4 : Configurer Nginx comme Reverse Proxy

Nginx agit comme point d’entrée unique pour notre application, distribuant les requêtes vers l’API FastAPI. Dans un environnement de production, Nginx gère également la terminaison SSL/TLS, la mise en cache statique et la limitation de débit. Notre configuration Docker Compose simplifie cette mise en place en permettant à Nginx de résoudre automatiquement le nom du service API via le DNS interne de Docker.

```
# nginx/nginx.conf
upstream api_backend {
    server api:8000;
}

server {
    listen 80;
    server_name localhost;

    # Logs
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    # Proxy vers l'API FastAPI
    location /api/ {
        proxy_pass http://api_backend/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 30s;
        proxy_read_timeout 60s;
    }

    # Health check endpoint
    location /health {
        proxy_pass http://api_backend/health;
    }

    # Page d'accueil
    location / {
        return 200 '{"message": "Docker Compose Tutorial API - tech-insider.org"}';
        add_header Content-Type application/json;
    }
}
```

L’élément clé ici est la directive `upstream` qui référence `api:8000`. Docker Compose crée automatiquement un réseau interne où chaque service est accessible par son nom. Ainsi, `api` résout vers l’adresse IP du conteneur FastAPI – aucune configuration réseau manuelle n’est nécessaire. C’est l’un des avantages majeurs de Docker Compose par rapport à l’utilisation manuelle de `docker run` avec des liens réseau explicites.

**Piège courant n°4 :** N’utilisez pas `localhost` ou `127.0.0.1` dans la configuration Nginx pour référencer d’autres conteneurs. Dans le réseau Docker, `localhost` fait référence au conteneur Nginx lui-même, pas aux autres services. Utilisez toujours le nom du service défini dans `compose.yaml`.

## Étape 5 : Écrire le Fichier compose.yaml Complet

Voici le cœur de notre projet : le fichier `compose.yaml` qui orchestre les quatre services. Ce fichier utilise les fonctionnalités modernes de Docker Compose disponibles en 2026, incluant les health checks, les dépendances conditionnelles, les profils et la gestion avancée des volumes. Chaque directive est documentée pour vous permettre de comprendre et d’adapter cette configuration à vos propres projets.

```
# compose.yaml - Docker Compose Tutorial 2026
# Documentation : https://docs.docker.com/compose/

services:
  # --- Base de données PostgreSQL ---
  db:
    image: postgres:16-alpine
    container_name: tutorial-db
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-appdb}
      POSTGRES_USER: ${POSTGRES_USER:-appuser}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Mot de passe requis}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-appuser} -d ${POSTGRES_DB:-appdb}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    networks:
      - backend

  # --- Cache Redis ---
  redis:
    image: redis:7-alpine
    container_name: tutorial-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    command: redis-server --maxmemory 128mb --maxmemory-policy allkeys-lru
    networks:
      - backend

  # --- API FastAPI ---
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    container_name: tutorial-api
    restart: unless-stopped
    environment:
      DB_HOST: db
      REDIS_HOST: redis
      POSTGRES_DB: ${POSTGRES_DB:-appdb}
      POSTGRES_USER: ${POSTGRES_USER:-appuser}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Mot de passe requis}
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    ports:
      - "8000:8000"
    networks:
      - backend
    develop:
      watch:
        - action: sync+restart
          path: ./api/main.py
          target: /app/main.py

  # --- Reverse Proxy Nginx ---
  nginx:
    image: nginx:1.27-alpine
    container_name: tutorial-nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
    depends_on:
      - api
    networks:
      - backend

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

networks:
  backend:
    driver: bridge
```

Plusieurs points importants méritent votre attention dans ce fichier. La syntaxe `${POSTGRES_PASSWORD:?Mot de passe requis}` force Docker Compose à échouer si la variable n’est pas définie – un garde-fou essentiel en production. La directive `depends_on` avec `condition: service_healthy` garantit que l’API ne démarre qu’une fois PostgreSQL et Redis prêts à accepter des connexions. Sans cette condition, l’API pourrait démarrer avant la base de données et crasher immédiatement. Enfin, la section `develop.watch` active la fonctionnalité Compose Watch qui synchronise automatiquement vos fichiers locaux vers le conteneur pendant le développement.

## Étape 6 : Configurer les Variables d’Environnement

La gestion des variables d’environnement est un aspect critique de tout projet Docker Compose. En 2026, les bonnes pratiques imposent de ne jamais coder en dur les mots de passe, clés API ou paramètres sensibles dans le fichier `compose.yaml`. Docker Compose prend en charge plusieurs mécanismes pour injecter ces valeurs de manière sécurisée.

Créez le fichier `.env` à la racine du projet. Ce fichier est automatiquement chargé par Docker Compose :

```
# .env - Variables d'environnement (NE PAS VERSIONNER)
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=MonMotDePasse2026!Securise
REDIS_HOST=redis
API_PORT=8000
```

Docker Compose applique un ordre de priorité précis pour résoudre les variables d’environnement. Comprendre cet ordre est essentiel pour déboguer les problèmes de configuration :

|Priorité|Source|Exemple|Utilisation|
|---|---|---|---|
|1 (la plus haute)|Variables shell de l’hôte|`export POSTGRES_PASSWORD=xxx`|Override temporaire en CI/CD|
|2|Fichier .env spécifié|`docker compose --env-file prod.env up`|Environnements multiples|
|3|Fichier .env par défaut|Fichier `.env` à la racine|Développement local|
|4|Valeurs par défaut dans compose.yaml|`${VAR:-default}`|Fallback sécurisé|
|5 (la plus basse)|Dockerfile ENV|`ENV APP_PORT=8000`|Valeurs de base de l’image|

**Piège courant n°5 :** Les espaces autour du signe `=` dans le fichier `.env` sont inclus dans la valeur. La ligne `POSTGRES_PASSWORD = secret` définira le mot de passe comme `" secret"` (avec un espace devant), ce qui causera des erreurs d’authentification difficiles à diagnostiquer. Utilisez toujours `POSTGRES_PASSWORD=secret` sans espaces.

## Étape 7 : Lancer et Vérifier les Services

Tout est en place. Il est temps de construire les images et de démarrer l’ensemble de la stack. Docker Compose offre plusieurs commandes essentielles que tout développeur doit maîtriser. Voici le processus complet de lancement et de vérification de notre application multi-conteneurs.

```
# Construire et lancer tous les services en arrière-plan
docker compose up -d --build

# Sortie attendue :
# [+] Building 15.2s (12/12) FINISHED
#  => [api] docker image pull python:3.12-slim
#  => [api] COPY requirements.txt .
#  => [api] RUN pip install --no-cache-dir -r requirements.txt
#  => [api] COPY main.py .
# [+] Running 5/5
#  ✔ Network compose-tutorial_backend  Created
#  ✔ Volume "compose-tutorial_postgres_data" Created
#  ✔ Container tutorial-db     Healthy
#  ✔ Container tutorial-redis  Healthy
#  ✔ Container tutorial-api    Started
#  ✔ Container tutorial-nginx  Started

# Vérifier l'état de tous les services
docker compose ps

# Sortie attendue :
# NAME              IMAGE                 STATUS                   PORTS
# tutorial-api      compose-tutorial-api  Up 2 minutes             0.0.0.0:8000->8000/tcp
# tutorial-db       postgres:16-alpine    Up 2 minutes (healthy)   0.0.0.0:5432->5432/tcp
# tutorial-nginx    nginx:1.27-alpine     Up 2 minutes             0.0.0.0:80->80/tcp
# tutorial-redis    redis:7-alpine        Up 2 minutes (healthy)   0.0.0.0:6379->6379/tcp

# Consulter les logs de tous les services
docker compose logs --tail=20

# Tester l'API directement
curl http://localhost/health
# {"status":"healthy","db":"ok","cache":"ok"}

# Tester via Nginx
curl http://localhost/api/items
# {"source":"database","items":[{"id":3,"name":"Troisième élément",...}]}

# Créer un nouvel élément
curl -X POST http://localhost/api/items \
  -H "Content-Type: application/json" \
  -d '{"name": "Test Docker Compose", "description": "Créé via le tutoriel"}'
# {"id":4,"name":"Test Docker Compose"}
```

Si tous les services affichent le statut “Up” et que le health check retourne “healthy”, votre stack Docker Compose fonctionne correctement. Le flag `--build` force la reconstruction des images à chaque lancement, ce qui est utile pendant le développement. En production, vous omettrez ce flag et utiliserez des images pré-construites depuis un registre Docker.

## Étape 8 : Utiliser les Commandes Docker Compose Essentielles

Maîtriser les commandes Docker Compose est indispensable pour gérer efficacement vos applications au quotidien. En 2026, avec la transition complète vers Compose V2 (plugin intégré à Docker CLI), toutes les commandes utilisent la syntaxe `docker compose` avec un espace. Voici les commandes les plus importantes, classées par fréquence d’utilisation.

|Commande|Description|Usage courant|
|---|---|---|
|`docker compose up -d`|Démarrer tous les services en arrière-plan|Lancement quotidien|
|`docker compose down`|Arrêter et supprimer les conteneurs|Fin de journée|
|`docker compose down -v`|Arrêter + supprimer les volumes|Reset complet des données|
|`docker compose logs -f api`|Suivre les logs d’un service en temps réel|Débogage|
|`docker compose exec db psql -U appuser appdb`|Ouvrir un shell PostgreSQL|Requêtes manuelles|
|`docker compose restart api`|Redémarrer un service spécifique|Après modification de config|
|`docker compose build --no-cache`|Reconstruire sans cache|Problèmes de dépendances|
|`docker compose watch`|Activer le mode développement (Watch)|Développement avec hot-reload|
|`docker compose top`|Afficher les processus en cours|Diagnostic de performance|
|`docker compose config`|Valider et afficher la config résolue|Vérifier les variables|

La commande `docker compose config` est particulièrement précieuse car elle affiche le fichier Compose avec toutes les variables d’environnement résolues. Si vous suspectez un problème de configuration, c’est la première commande à exécuter. Elle vous montrera exactement ce que Docker Compose interprète, incluant les valeurs par défaut et les substitutions de variables.

La fonctionnalité **Compose Watch**, introduite dans les versions récentes et améliorée en 2026, révolutionne l’expérience de développement. En exécutant `docker compose watch`, Docker surveille les fichiers locaux et synchronise automatiquement les modifications vers les conteneurs. L’action `sync+restart` que nous avons configurée dans notre `compose.yaml` synchronise le fichier modifié puis redémarre le service – idéal pour les applications Python qui nécessitent un rechargement.

## Étape 9 : Implémenter les Health Checks et la Gestion des Dépendances

Les health checks sont l’une des fonctionnalités les plus importantes de Docker Compose en production. Sans eux, vos conteneurs sont considérés comme “prêts” dès qu’ils démarrent, même si l’application à l’intérieur n’est pas encore opérationnelle. Cette situation provoque des erreurs en cascade : l’API démarre avant que PostgreSQL n’accepte les connexions, les requêtes échouent, et le conteneur redémarre en boucle.

Notre configuration utilise trois types de health checks adaptés à chaque service. Pour PostgreSQL, `pg_isready` vérifie que le serveur accepte les connexions. Pour Redis, `redis-cli ping` confirme que le cache répond. Pour l’API, un endpoint HTTP `/health` teste la connectivité complète vers la base de données et le cache. Les paramètres `interval`, `timeout`, `retries` et `start_period` permettent de contrôler finement le comportement de vérification.

Le paramètre `start_period` est particulièrement important pour PostgreSQL : il accorde un délai de grâce au conteneur pour s’initialiser avant de commencer à compter les échecs. Sans ce paramètre, un conteneur PostgreSQL chargeant un gros script d’initialisation pourrait être considéré comme défaillant et redémarré prématurément. En 2026, la valeur recommandée pour PostgreSQL est de 30 secondes, ce qui laisse le temps à la base de données de s’initialiser même sur des machines modestes.

La directive `depends_on` avec `condition: service_healthy` crée un chaînage intelligent : Docker Compose attend que chaque dépendance soit non seulement démarrée mais aussi en bonne santé avant de lancer le service suivant. C’est une amélioration majeure par rapport à la simple directive `depends_on` sans condition, qui ne garantissait que l’ordre de démarrage, pas la disponibilité effective du service.

## Étape 10 : Gérer les Volumes et la Persistance des Données

La persistance des données est un sujet fondamental dans Docker Compose. Par défaut, les données stockées dans un conteneur sont éphémères : elles disparaissent lorsque le conteneur est supprimé. Les volumes Docker résolvent ce problème en stockant les données sur le système hôte, indépendamment du cycle de vie des conteneurs.

Notre projet utilise deux types de volumes. Les **volumes nommés** (`postgres_data` et `redis_data`) sont gérés par Docker et stockent les données de manière persistante. Les **bind mounts** (`./db/init.sql` et `./nginx/nginx.conf`) lient directement un fichier ou répertoire de l’hôte dans le conteneur, ce qui est idéal pour les fichiers de configuration que vous modifiez fréquemment.

```
# Lister les volumes créés par Docker Compose
docker volume ls --filter name=compose-tutorial
# DRIVER    VOLUME NAME
# local     compose-tutorial_postgres_data
# local     compose-tutorial_redis_data

# Inspecter un volume pour voir où il est stocké
docker volume inspect compose-tutorial_postgres_data
# [{"CreatedAt":"2026-03-23T10:00:00Z",
#   "Driver":"local",
#   "Mountpoint":"/var/lib/docker/volumes/compose-tutorial_postgres_data/_data",
#   "Name":"compose-tutorial_postgres_data"}]

# Sauvegarder les données PostgreSQL
docker compose exec db pg_dump -U appuser appdb > backup_$(date +%Y%m%d).sql

# Restaurer une sauvegarde
cat backup_20260323.sql | docker compose exec -T db psql -U appuser appdb

# Supprimer les volumes orphelins (nettoyage)
docker volume prune --filter "label!=keep"
```

L’option `:ro` (read-only) que nous utilisons pour le script d’initialisation et la configuration Nginx est une bonne pratique de sécurité. Elle empêche le conteneur de modifier ces fichiers, réduisant la surface d’attaque en cas de compromission du conteneur. En production, appliquez `:ro` à tous les bind mounts de configuration.

**Piège courant n°6 :** La commande `docker compose down` ne supprime PAS les volumes par défaut. Vos données PostgreSQL survivent aux arrêts/redémarrages. Si vous voulez un reset complet, utilisez explicitement `docker compose down -v`. C’est un comportement qui surprend beaucoup de développeurs qui pensent repartir de zéro après un `down`.

## Étape 11 : Optimiser pour la Production avec les Profils et Overrides

Docker Compose offre des mécanismes puissants pour gérer différents environnements (développement, staging, production) à partir d’une même base de configuration. Les **profils** et les **fichiers override** sont les deux approches principales en 2026, chacune adaptée à des cas d’usage spécifiques.

Les profils permettent de regrouper des services par contexte. Par exemple, vous pouvez créer un profil “debug” qui inclut des outils de monitoring, ou un profil “test” qui ajoute un conteneur de tests automatisés. Voici comment étendre notre `compose.yaml` avec des profils :

```
# Ajout au compose.yaml - Services avec profils

services:
  # ... services existants ...

  # Outil de monitoring (profil debug uniquement)
  adminer:
    image: adminer:4
    container_name: tutorial-adminer
    ports:
      - "8080:8080"
    networks:
      - backend
    profiles:
      - debug

  # Service de tests (profil test uniquement)
  tests:
    build:
      context: ./api
    container_name: tutorial-tests
    command: python -m pytest tests/ -v
    environment:
      DB_HOST: db
      REDIS_HOST: redis
      POSTGRES_DB: ${POSTGRES_DB:-appdb}
      POSTGRES_USER: ${POSTGRES_USER:-appuser}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Mot de passe requis}
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend
    profiles:
      - test

# Utilisation :
# docker compose up -d                        → Services de base uniquement
# docker compose --profile debug up -d        → + Adminer
# docker compose --profile test run tests     → Exécuter les tests
```

L’approche par fichiers override est complémentaire. Créez un fichier `compose.override.yaml` pour le développement (chargé automatiquement) et un `compose.prod.yaml` pour la production :

```
# compose.override.yaml - Surcharges pour le développement (chargé automatiquement)
services:
  api:
    build:
      target: base
    volumes:
      - ./api:/app  # Hot-reload en développement
    environment:
      - DEBUG=true

# compose.prod.yaml - Configuration de production
services:
  api:
    restart: always
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
    environment:
      - DEBUG=false
  
  db:
    restart: always
    deploy:
      resources:
        limits:
          cpus: "2.0"
          memory: 1G

  nginx:
    restart: always
    ports:
      - "443:443"

# Lancer en production :
# docker compose -f compose.yaml -f compose.prod.yaml up -d
```

En production, les limites de ressources (`deploy.resources.limits`) empêchent un conteneur de consommer toute la mémoire ou le CPU de l’hôte. C’est une configuration essentielle qui prévient les situations où un pic de trafic sur l’API affame la base de données en ressources, provoquant un effondrement en cascade de toute l’application.

## Étape 12 : Mettre en Place la Supervision et les Logs

La supervision est l’ultime étape pour transformer votre projet Docker Compose en une application prête pour la production. En 2026, les bonnes pratiques imposent une centralisation des logs et une surveillance active de la santé des services. Docker Compose facilite grandement cette mise en place grâce à ses pilotes de logging intégrés et sa compatibilité avec les outils de monitoring modernes. D’ailleurs, Docker Desktop 4.70.0, publié le 20 avril 2026 avec Docker Compose v5.1.2, a introduit de nouveaux indices en ligne de commande (CLI hints) pour la commande `compose logs`, facilitant le diagnostic rapide sans quitter le terminal.

Par défaut, Docker utilise le pilote de logging `json-file`, qui stocke les logs localement. Pour éviter que les logs ne remplissent le disque, configurez une rotation automatique dans votre `compose.yaml` :

```
# Ajout à chaque service dans compose.yaml
services:
  api:
    # ... configuration existante ...
    logging:
      driver: json-file
      options:
        max-size: "10m"    # Taille maximale par fichier de log
        max-file: "3"      # Nombre de fichiers de rotation
        tag: "{{.Name}}"   # Tag avec le nom du conteneur

# Commandes de monitoring essentielles :

# Suivre les logs de tous les services en temps réel
docker compose logs -f --tail=50

# Logs d'un seul service avec timestamps
docker compose logs -f --timestamps api

# Statistiques de ressources en temps réel
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"
# NAME              CPU %     MEM USAGE / LIMIT     NET I/O
# tutorial-api      0.15%     85.2MiB / 512MiB      1.2kB / 850B
# tutorial-db       0.08%     45.7MiB / 1GiB        2.1kB / 1.5kB
# tutorial-nginx    0.02%     12.3MiB / 256MiB      980B / 720B
# tutorial-redis    0.05%     8.1MiB / 128MiB       450B / 320B

# Vérifier les événements Docker
docker compose events --json
```

Pour une supervision plus avancée, vous pouvez intégrer Prometheus et Grafana à votre stack Docker Compose. Ces outils permettent de collecter des métriques, de créer des tableaux de bord et de configurer des alertes. Cependant, pour la plupart des projets de taille moyenne, les commandes `docker stats` et `docker compose logs` combinées avec une rotation des logs constituent une solution suffisante et bien plus simple à maintenir.

## Dépannage : 8 Problèmes Courants et Leurs Solutions

Même avec une configuration soignée, vous rencontrerez inévitablement des problèmes lors de l’utilisation de Docker Compose. Voici les huit situations les plus fréquentes, avec leurs causes et solutions détaillées, basées sur les retours de milliers de développeurs en 2026.

### Problème 1 : “port is already allocated”

**Cause :** Un autre processus (ou un ancien conteneur) utilise déjà le port demandé sur l’hôte. C’est le problème le plus fréquent lors du premier lancement.

**Solution :** Identifiez le processus avec `sudo lsof -i :5432` (Linux/macOS) ou `netstat -ano | findstr :5432` (Windows). Arrêtez le processus conflictuel ou modifiez le port dans `compose.yaml` : `"5433:5432"` pour mapper sur un port différent de l’hôte.

### Problème 2 : “no matching manifest for linux/arm64”

**Cause :** Vous utilisez un Mac Apple Silicon (M1/M2/M3/M4) et l’image Docker n’est pas disponible pour l’architecture ARM64. Fréquent avec les images communautaires non officielles.

**Solution :** Ajoutez `platform: linux/amd64` au service concerné dans `compose.yaml`. Docker Desktop émuler l’architecture x86 via Rosetta 2, avec une légère perte de performance. Privilégiez toujours les images officielles qui supportent le multi-architecture.

### Problème 3 : L’API ne se connecte pas à la base de données

**Cause :** L’API démarre avant que PostgreSQL ne soit prêt, ou les variables d’environnement sont incorrectes. Sans health check, `depends_on` ne garantit que l’ordre de démarrage, pas la disponibilité.

**Solution :** Vérifiez que vos health checks sont en place avec `condition: service_healthy`. Exécutez `docker compose config` pour vérifier que les variables d’environnement sont correctement résolues. Testez la connectivité manuellement : `docker compose exec api python -c "import psycopg2; psycopg2.connect(host='db', user='appuser', password='...', dbname='appdb')"`.

### Problème 4 : Les modifications du code ne sont pas prises en compte

**Cause :** Le cache de build Docker réutilise une couche précédente. C’est le cas lorsque vous modifiez le code source mais que le Dockerfile n’a pas changé au niveau de l’instruction `COPY`.

**Solution :** Utilisez `docker compose up --build` pour forcer la reconstruction, ou `docker compose build --no-cache api` pour un rebuild complet sans cache. En développement, activez Compose Watch avec `docker compose watch` pour la synchronisation automatique des fichiers.

### Problème 5 : “insufficient memory” ou conteneur qui crashe

**Cause :** Docker Desktop est configuré avec des ressources insuffisantes. Par défaut, Docker Desktop limite la mémoire à 2 Go, ce qui peut être insuffisant pour une stack avec PostgreSQL, Redis et une API.

**Solution :** Dans Docker Desktop, allez dans Settings → Resources et augmentez la mémoire à 4 Go minimum. Sur Linux, vérifiez avec `docker info | grep -i memory`. Ajoutez des limites de ressources dans `compose.yaml` pour éviter qu’un service ne monopolise toute la mémoire disponible.

### Problème 6 : Les données disparaissent après un redémarrage

**Cause :** Vous utilisez `docker compose down -v` qui supprime les volumes, ou vos données sont stockées dans le conteneur au lieu d’un volume nommé.

**Solution :** Vérifiez que vos volumes sont correctement définis dans la section `volumes:` du `compose.yaml`. Utilisez `docker compose down` (sans `-v`) pour conserver les données. Vérifiez avec `docker volume ls` que les volumes existent.

### Problème 7 : “network compose-tutorial_backend not found”

**Cause :** Un réseau orphelin d’une exécution précédente interfère avec le nouveau déploiement. Cela se produit souvent après un crash ou un arrêt forcé.

**Solution :** Nettoyez les réseaux orphelins avec `docker network prune`. Si le problème persiste, supprimez manuellement le réseau avec `docker network rm compose-tutorial_backend` puis relancez `docker compose up`.

### Problème 8 : Compose ne détecte pas les changements dans le fichier .env

**Cause :** Docker Compose charge le fichier `.env` au moment de l’analyse du fichier Compose, pas au démarrage des conteneurs. Si vous modifiez `.env` pendant que les services tournent, les changements ne sont pas appliqués.

**Solution :** Après toute modification du fichier `.env`, vous devez recréer les conteneurs avec `docker compose up -d`. Pour vérifier que les variables sont correctement chargées, utilisez `docker compose config` qui affiche la configuration résolue avec toutes les substitutions appliquées.

## Astuces Avancées Docker Compose en 2026

Au-delà des bases, Docker Compose offre des fonctionnalités avancées qui transforment votre workflow de développement et de déploiement. Voici les techniques que les ingénieurs DevOps les plus expérimentés utilisent en 2026.

**Compose Bridge vers Kubernetes :** L’outil Compose Bridge, mis en avant dans la documentation Docker 2026, transforme vos fichiers `compose.yaml` en manifestes Kubernetes. C’est un excellent moyen de migrer progressivement vers Kubernetes sans réécrire toute votre configuration. Exécutez `docker compose bridge` pour générer les fichiers YAML Kubernetes correspondant à votre stack.

**Support SDK officiel :** Docker Compose v5.0.0 introduit un SDK officiel qui permet d’intégrer Compose dans des outils tiers via des paramètres fonctionnels et des API documentées. Cela ouvre la porte à l’automatisation programmatique de vos déploiements, par exemple dans des scripts Python ou des workflows CI/CD personnalisés.

**Ressources OCI et Git distantes :** La v5.0.0 apporte le support officiel des ressources OCI et Git distantes dans les fichiers Compose. Vous pouvez désormais référencer des configurations stockées dans un registre OCI ou un dépôt Git distant, facilitant le partage de configurations standardisées à travers plusieurs équipes et projets.

**Builds avec Docker Bake :** La délégation des builds à Docker Bake dans la v5.0.0 aligne le comportement de Compose avec celui de `docker build`. Docker Bake offre des fonctionnalités avancées comme les builds parallèles, les matrices de build et les cibles multiples, ce qui accélère considérablement les temps de construction pour les projets multi-services.

**Inclure des fichiers Compose externes :** La directive `include` permet de décomposer votre configuration en fichiers modulaires. Par exemple, vous pouvez avoir un fichier `compose.monitoring.yaml` pour Prometheus/Grafana et l’inclure uniquement dans certains environnements. C’est une approche modulaire qui améliore la maintenabilité des configurations complexes.

## Comparaison : Docker Compose vs Alternatives en 2026

Pour situer Docker Compose dans l’écosystème des outils d’orchestration en 2026, voici un comparatif avec les principales alternatives. Chaque outil a ses forces et ses cas d’usage privilégiés. Docker Compose excelle dans le développement local et les déploiements simples, tandis que Kubernetes domine pour l’orchestration à grande échelle en production.

|Critère|Docker Compose|Kubernetes|Docker Swarm|Podman Compose|
|---|---|---|---|---|
|Courbe d’apprentissage|Faible|Élevée|Moyenne|Faible|
|Cas d’usage principal|Développement local, CI/CD, petite production|Production à grande échelle|Production moyenne|Développement local sans daemon|
|Scaling automatique|Manuel (`--scale`)|Automatique (HPA)|Basique|Manuel|
|Haute disponibilité|Non native|Oui (multi-nœuds)|Oui (multi-nœuds)|Non native|
|Configuration|1 fichier YAML|Multiples manifestes|1 fichier YAML + init|1 fichier YAML|
|Communauté (2026)|Très large|Massive|En déclin|Croissante|
|Support GPU|Oui (runtime nvidia)|Oui (device plugins)|Limité|Oui|

En 2026, la trajectoire est claire : Docker Compose pour le développement et les environnements simples, Kubernetes pour la production à grande échelle. Docker Swarm, bien que toujours fonctionnel, voit sa communauté décliner au profit de Kubernetes. Podman Compose gagne en popularité dans les environnements où un daemon root n’est pas souhaitable, notamment dans les entreprises soumises à des contraintes de sécurité strictes comme celles régies par la directive NIS2 en Europe.

## Bonnes Pratiques de Sécurité Docker Compose

La sécurité des conteneurs est devenue une priorité absolue en 2026, notamment en Europe avec l’entrée en vigueur de la directive NIS2 et le renforcement du Cyber Resilience Act. Voici les bonnes pratiques de sécurité à appliquer systématiquement dans vos projets Docker Compose.

Premièrement, ne jamais exécuter de processus en tant que root dans vos conteneurs. Notre Dockerfile inclut déjà la création d’un utilisateur dédié avec `USER appuser`. Cette pratique limite l’impact d’une éventuelle compromission : même si un attaquant exploite une vulnérabilité dans votre application, il n’aura pas les privilèges root dans le conteneur.

Deuxièmement, utilisez toujours des images avec des tags spécifiques plutôt que `:latest`. La notation `postgres:16-alpine` est préférable à `postgres:latest` car elle garantit la reproductibilité de vos déploiements. Un changement majeur de version dans une image `:latest` peut casser votre application sans avertissement. Pour une sécurité maximale, utilisez le digest SHA256 de l’image : `postgres@sha256:abc123...`.

Troisièmement, scannez régulièrement vos images avec Docker Scout ou Trivy. Docker Scout, intégré à Docker Desktop depuis 2025, analyse automatiquement vos images et signale les vulnérabilités CVE connues. Exécutez `docker scout quickview` pour obtenir un résumé rapide des vulnérabilités dans vos images.

Quatrièmement, limitez les capacités système de vos conteneurs. Par défaut, Docker accorde un ensemble de capacités Linux qui ne sont pas nécessaires pour la plupart des applications. Ajoutez `cap_drop: [ALL]` puis `cap_add` uniquement les capacités requises. Pour une API web, vous n’avez généralement besoin d’aucune capacité supplémentaire au-delà de `NET_BIND_SERVICE`.

Cinquièmement, activez le mode `read_only: true` pour le système de fichiers du conteneur lorsque c’est possible, et définissez des volumes `tmpfs` pour les répertoires qui nécessitent une écriture temporaire. Cette approche renforce considérablement la posture de sécurité de vos conteneurs en empêchant toute modification non autorisée du système de fichiers.

## FAQ : Questions Fréquentes sur Docker Compose

### Quelle est la différence entre docker-compose et docker compose ?

`docker-compose` (avec tiret) est l’ancienne version standalone écrite en Python, officiellement dépréciée depuis 2023. `docker compose` (avec espace) est la version V2 intégrée comme plugin à Docker CLI, écrite en Go. En 2026, vous devez utiliser exclusivement `docker compose`. Si vous avez des scripts utilisant l’ancienne syntaxe, mettez-les à jour immédiatement.

### Docker Compose est-il adapté à la production ?

Oui, pour des déploiements de petite à moyenne envergure sur un serveur unique. Docker Compose gère très bien les applications avec 5 à 15 services sur une même machine. Pour des architectures nécessitant du scaling horizontal, de la haute disponibilité multi-nœuds ou des déploiements blue-green, migrez vers Kubernetes. L’outil Compose Bridge facilite cette transition en convertissant vos fichiers Compose en manifestes Kubernetes.

### Comment mettre à jour un seul service sans interrompre les autres ?

Utilisez `docker compose up -d --no-deps --build api`. Le flag `--no-deps` empêche Docker Compose de recréer les services dont dépend l’API. Le flag `--build` force la reconstruction de l’image. Les autres services continuent de fonctionner normalement pendant la mise à jour.

### Peut-on utiliser Docker Compose avec Docker Swarm ?

Oui, mais avec des limitations. `docker stack deploy -c compose.yaml mystack` permet de déployer un fichier Compose sur un cluster Swarm. Cependant, certaines directives comme `build` ou `depends_on` ne sont pas supportées en mode Swarm. En 2026, la plupart des équipes choisissent Kubernetes plutôt que Swarm pour les déploiements multi-nœuds.

### Comment accéder à la base de données depuis un outil externe ?

Si le port est mappé dans `compose.yaml` (par exemple `"5432:5432"`), vous pouvez accéder à PostgreSQL depuis l’hôte via `localhost:5432` avec n’importe quel client SQL (DBeaver, pgAdmin, DataGrip). En production, retirez ce mapping de port pour des raisons de sécurité et accédez uniquement via `docker compose exec`.

### Docker Compose est-il gratuit ?

Docker Compose en tant que plugin CLI est entièrement gratuit et open source. Docker Desktop, qui inclut Compose, est gratuit pour les particuliers, l’éducation et les petites entreprises (moins de 250 employés et moins de 10 millions de dollars de revenus). Les grandes entreprises doivent souscrire à un abonnement Docker Business (à partir de 24 $/mois par utilisateur en 2026). Sur Linux, vous pouvez utiliser Docker Engine + Compose plugin sans Docker Desktop, entièrement gratuitement.

### Comment déboguer un conteneur qui ne démarre pas ?

Suivez cette séquence de diagnostic : 1) `docker compose logs service_name` pour voir les messages d’erreur. 2) `docker compose config` pour vérifier la configuration résolue. 3) `docker compose run --rm service_name sh` pour ouvrir un shell dans un nouveau conteneur avec la même configuration. 4) `docker inspect container_name` pour voir l’état détaillé du conteneur, incluant le code de sortie et la raison de l’arrêt.

### Quelle est la différence entre volumes et bind mounts ?

Les **volumes nommés** (`postgres_data:/var/lib/postgresql/data`) sont gérés par Docker, stockés dans `/var/lib/docker/volumes/`, et offrent les meilleures performances. Les **bind mounts** (`./config:/etc/app/config`) lient un chemin de l’hôte au conteneur, idéal pour les fichiers de configuration et le développement. Utilisez les volumes nommés pour les données persistantes (bases de données) et les bind mounts pour les fichiers que vous modifiez fréquemment.

## Couverture Associée

Pour approfondir vos connaissances sur Docker, le DevOps et l’écosystème cloud en 2026, consultez nos articles connexes :

- [Docker vs Kubernetes 2026 : Le Comparatif Définitif des Conteneurs](https://tech-insider.org/fr/docker-vs-kubernetes-2026/) – Comprendre quand passer de Compose à Kubernetes
- [Tutoriel Ansible 2026 : Automatiser Votre Infrastructure de Zéro à la Production](https://tech-insider.org/fr/tutoriel-ansible-automatiser-infrastructure-2026/) – Compléter Docker Compose avec l’automatisation Ansible
- [GitLab vs GitHub 2026 : Le Comparatif Définitif des Plateformes DevOps](https://tech-insider.org/fr/gitlab-vs-github-2026/) – Choisir la meilleure plateforme CI/CD pour vos workflows Docker
- [Tutoriel FastAPI Python 2026 : Créer une API REST Complète](https://tech-insider.org/fr/tutoriel-fastapi-python-api-rest-2026/) – Approfondir FastAPI, le framework utilisé dans notre tutoriel
- [Cloud Souverain vs Cloud Public 2026 : Le Comparatif pour la France et l’Europe](https://tech-insider.org/fr/cloud-souverain-vs-cloud-public-2026/) – Comprendre les enjeux du cloud pour les déploiements Docker en entreprise
- [AWS vs Azure vs Google Cloud 2026 : Le Comparatif Définitif](https://tech-insider.org/fr/aws-vs-azure-vs-google-cloud-2026/) – Choisir le bon cloud provider pour héberger vos conteneurs

Pour aller plus loin avec Docker Compose, consultez la [documentation officielle Docker Compose](https://docs.docker.com/compose/), le [dépôt GitHub de Docker Compose](https://github.com/docker/compose) et le [Docker Hub](https://hub.docker.com/) pour trouver des images officielles. La communauté [Docker Forums](https://forums.docker.com/) est également une excellente ressource pour obtenir de l’aide sur des problèmes spécifiques.

_Dernière mise à jour : 19 août 2026, suite à la sortie de Docker Compose v5.5.0 le 17 août 2026. Entre-temps, l’outil a connu une cadence de publication soutenue : Docker Desktop 4.66.1 (26 mars 2026) embarquait la v5.1.1, suivi de Docker Desktop 4.71.0 (20 avril 2026) avec la v5.1.3. Ce tutoriel est maintenu et mis à jour régulièrement pour refléter les dernières versions de Docker Compose et les bonnes pratiques actuelles._


[Voir tous les articles](https://tech-insider.org/fr/author/nadia-dubois/)