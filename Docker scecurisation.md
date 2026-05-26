---
tags:
  - docker
---
Les conteneurs Docker par défaut ne sont pas sécurisés pour la production. Cet article détaille les bonnes pratiques pour durcir vos conteneurs : isolation, limitation des privilèges, scanning des vulnérabilités et gestion des secrets.

#### Plan

- Principes de sécurité des conteneurs
- Utilisateurs non-root et capabilities
- AppArmor et Seccomp profiles
- Scanning d'images avec Trivy
- Gestion des secrets
- Network isolation
- Conclusion

---

#### Principes de sécurité des conteneurs

**Risques majeurs :**

- Conteneurs privilégiés = root sur l'hôte
- Images avec vulnérabilités
- Secrets en clair dans les images
- Réseau non isolé

**Règles d'or :**

1. Ne jamais run en root
2. Scanner les images avant déploiement
3. Limiter les capabilities Linux
4. Utiliser des secrets externes (Vault, etc)
5. Isoler le réseau

---

#### Utilisateurs non-root et capabilities

##### Créer un utilisateur non-root

```dockerfile
# Dockerfile sécurisé
FROM node:20-alpine

# Créer utilisateur dédié
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

# Installer dépendances en root
COPY package*.json ./
RUN npm ci --only=production

# Copier app
COPY --chown=appuser:appgroup . .

# Switcher vers user non-root
USER appuser

EXPOSE 3000
CMD ["node", "server.js"]
```

##### Vérifier l'utilisateur

```bash
# Vérifier qu'on ne run pas en root
docker run --rm myapp id
# uid=1001(appuser) gid=1001(appgroup)

# Interdire les conteneurs root
docker run --user 1001:1001 myapp
```

##### Limiter les capabilities

```bash
# Supprimer toutes les capabilities puis ajouter uniquement celles nécessaires
docker run --rm \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  myapp

# Docker Compose
services:
  app:
    image: myapp
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    security_opt:
      - no-new-privileges:true
```

**Capabilities courantes :**

- `NET_BIND_SERVICE` : bind ports < 1024
- `CHOWN` : changer propriétaires fichiers
- `DAC_OVERRIDE` : bypass permissions fichiers
- `SETUID/SETGID` : changer UID/GID

---

#### AppArmor et Seccomp profiles

##### AppArmor profile personnalisé

```bash
# /etc/apparmor.d/docker-custom
#include <tunables/global>

profile docker-custom flags=(attach_disconnected,mediate_deleted) {
  #include <abstractions/base>

  # Deny toutes les capabilities par défaut
  deny capability,

  # Autoriser uniquement net_bind_service
  capability net_bind_service,

  # Fichiers autorisés
  /app/** r,
  /tmp/** rw,
  /var/log/** w,

  # Deny accès système sensible
  deny /proc/sys/** w,
  deny /sys/** w,
}
```

Charger et utiliser :

```bash
# Charger le profile
sudo apparmor_parser -r /etc/apparmor.d/docker-custom

# Utiliser avec Docker
docker run --security-opt apparmor=docker-custom myapp
```

##### Seccomp profile

```json
{
	"defaultAction": "SCMP_ACT_ERRNO",
	"architectures": ["SCMP_ARCH_X86_64"],
	"syscalls": [
		{
			"names": [
				"accept",
				"accept4",
				"access",
				"bind",
				"brk",
				"chmod",
				"chown",
				"close",
				"connect",
				"dup",
				"epoll_create",
				"epoll_ctl",
				"epoll_wait",
				"exit",
				"exit_group",
				"fcntl",
				"fstat",
				"getdents",
				"getpid",
				"getuid",
				"listen",
				"mmap",
				"open",
				"openat",
				"read",
				"recv",
				"recvfrom",
				"send",
				"sendto",
				"socket",
				"stat",
				"write"
			],
			"action": "SCMP_ACT_ALLOW"
		}
	]
}
```

Utiliser :

```bash
docker run --security-opt seccomp=/path/to/profile.json myapp
```

---

#### Scanning d'images avec Trivy

##### Installation Trivy

```bash
# Debian/Ubuntu
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt update && sudo apt install trivy -y

# RHEL/Rocky
rpm -ivh https://github.com/aquasecurity/trivy/releases/download/v0.48.0/trivy_0.48.0_Linux-64bit.rpm
```

##### Scanner une image

```bash
# Scan complet
trivy image nginx:latest

# Uniquement vulnérabilités HIGH et CRITICAL
trivy image --severity HIGH,CRITICAL nginx:latest

# Format JSON pour automation
trivy image -f json -o results.json nginx:latest

# Fail si vulnérabilités critiques
trivy image --exit-code 1 --severity CRITICAL myapp:latest
```

##### Intégration CI/CD

```yaml
# .gitlab-ci.yml
security_scan:
  stage: test
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 1 --severity CRITICAL,HIGH $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - merge_requests
    - main
```

##### Scanner le filesystem (Dockerfile)

```bash
# Scanner avant le build
trivy fs --severity HIGH,CRITICAL .

# Scanner les dépendances
trivy fs --scanners vuln package-lock.json
```

---

#### Gestion des secrets

##### ❌ Mauvaises pratiques

```dockerfile
# NE JAMAIS FAIRE ÇA
ENV DATABASE_PASSWORD="super_secret_123"
ENV API_KEY="sk_live_abc123xyz"

# NE PAS mettre dans l'image
COPY .env /app/.env
```

##### ✅ Bonnes pratiques

**1. Docker secrets (Swarm/Kubernetes)**

```bash
# Créer un secret
echo "my_db_password" | docker secret create db_password -

# Docker Compose
services:
  app:
    image: myapp
    secrets:
      - db_password

secrets:
  db_password:
    external: true
```

Lire dans l'app :

```javascript
const fs = require('fs');
const dbPassword = fs.readFileSync('/run/secrets/db_password', 'utf8').trim();
```

**2. HashiCorp Vault**

```bash
# Lancer Vault en dev
docker run --cap-add=IPC_LOCK -e 'VAULT_DEV_ROOT_TOKEN_ID=myroot' -p 8200:8200 vault

# Stocker un secret
export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN='myroot'
vault kv put secret/myapp db_password="super_secret"

# Lire depuis l'app
curl -H "X-Vault-Token: $VAULT_TOKEN" \
  $VAULT_ADDR/v1/secret/data/myapp
```

**3. Variables d'environnement runtime**

```bash
# Passer au runtime (jamais dans l'image)
docker run -e DATABASE_PASSWORD="$DB_PASS" myapp

# Docker Compose avec .env (ne pas commit .env)
services:
  app:
    environment:
      - DATABASE_PASSWORD=${DATABASE_PASSWORD}
```

**4. Secrets pour builds multi-stage**

```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:20 AS builder

# Monter secret au build (ne reste pas dans l'image)
RUN --mount=type=secret,id=npmtoken \
    echo "//registry.npmjs.org/:_authToken=$(cat /run/secrets/npmtoken)" > .npmrc && \
    npm ci && \
    rm .npmrc

FROM node:20-alpine
COPY --from=builder /app /app
```

Build :

```bash
docker build --secret id=npmtoken,src=.npmtoken -t myapp .
```

---

#### Network isolation

##### Créer des réseaux isolés

```bash
# Réseau frontend (DMZ)
docker network create --driver bridge frontend

# Réseau backend (isolé)
docker network create --driver bridge --internal backend

# App web : accès public
docker run -d --name web --network frontend nginx

# API : accès uniquement depuis web
docker run -d --name api --network backend myapi

# Connecter web à backend pour accès API
docker network connect backend web
```

##### Docker Compose avec isolation

```yaml
services:
  web:
    image: nginx
    networks:
      - frontend
      - backend
    ports:
      - '80:80'

  api:
    image: myapi
    networks:
      - backend
      - database

  db:
    image: postgres
    networks:
      - database
    # Pas de ports exposés = isolé

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true # Pas d'accès internet
  database:
    driver: bridge
    internal: true
```

##### Firewall rules

```bash
# Bloquer tout trafic sortant sauf DNS
docker run --cap-add=NET_ADMIN --rm alpine sh -c '
  apk add iptables
  iptables -P OUTPUT DROP
  iptables -A OUTPUT -p udp --dport 53 -j ACCEPT
  iptables -A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
'
```

---

#### Bonnes pratiques additionnelles

##### Images minimales

```dockerfile
# Utiliser distroless ou alpine
FROM gcr.io/distroless/nodejs:18

# Ou alpine
FROM node:20-alpine

# Installer uniquement le nécessaire
RUN apk add --no-cache \
    ca-certificates \
    && rm -rf /var/cache/apk/*
```

##### Read-only filesystem

```bash
# Forcer read-only sauf volumes
docker run --read-only \
  --tmpfs /tmp \
  --tmpfs /var/run \
  myapp
```

Docker Compose :

```yaml
services:
  app:
    image: myapp
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
```

##### Limiter les ressources

```yaml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    pids_limit: 100 # Limiter nombre de process
```

##### Health checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

---

#### Checklist de sécurité Docker

✅ **Images :**

- [ ]  Utiliser images officielles ou vérifiées
- [ ]  Scanner avec Trivy avant déploiement
- [ ]  Images minimales (alpine/distroless)
- [ ]  Multi-stage builds

✅ **Runtime :**

- [ ]  Utilisateur non-root
- [ ]  Capabilities limitées (--cap-drop=ALL)
- [ ]  AppArmor/Seccomp profiles
- [ ]  Read-only filesystem
- [ ]  no-new-privileges

✅ **Secrets :**

- [ ]  Jamais dans l'image
- [ ]  Variables runtime ou Vault
- [ ]  Secrets Docker/K8s

✅ **Réseau :**

- [ ]  Réseaux isolés
- [ ]  Pas de --net=host
- [ ]  Firewall rules

✅ **Ressources :**

- [ ]  CPU/Memory limits
- [ ]  PID limits
- [ ]  Health checks

---

#### Compléments et ressources

Pour approfondir chaque aspect :

- [Profils AppArmor et seccomp détaillés](https://www.shpv.fr/blog/docker-security-profile/)
- [Scanner vos images avec Trivy](https://www.shpv.fr/blog/trivy-container-scanning/)
- [Registry sécurisée avec Harbor](https://www.shpv.fr/blog/harbor-registry/)
- [Alternative rootless avec Podman](https://www.shpv.fr/blog/podman-2026/)
- [Gestion des secrets Kubernetes avec External Secrets](https://www.shpv.fr/blog/k8s-external-secrets/)

#### Conclusion

Sécuriser Docker nécessite une approche en profondeur : images scannées, utilisateurs non-root, capabilities limitées, isolation réseau et gestion externe des secrets. Appliquez ces pratiques dès le développement et automatisez les scans en CI/CD.

**Actions prioritaires :**

1. Scanner toutes les images avec Trivy
2. Passer tous les conteneurs en non-root
3. Implémenter Vault pour les secrets
4. Créer AppArmor/Seccomp profiles customs
5. Isoler les réseaux par fonction