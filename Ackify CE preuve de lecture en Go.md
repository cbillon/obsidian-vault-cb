---
tags:
  - golang
---
-  [Ackify.eu](https://linuxfr.org/redirect/116600 "https://ackify.eu/fr") (273 clics)
-  [Github](https://linuxfr.org/redirect/116601 "https://github.com/btouchard/ackify-ce") (59 clics)
-  [Documentation](https://linuxfr.org/redirect/116602 "https://github.com/btouchard/ackify-ce/tree/main/docs/fr") (51 clics)

Ackify CE : preuve de lecture cryptographique en Go + Vue3

Ackify CE est une plateforme open-source (AGPL v3) permettant de générer des preuves de lecture cryptographiquement vérifiables pour des documents internes.

Le problème
Les organisations doivent souvent prouver qu'un collaborateur a lu un document (politique RGPD, charte de sécurité, formation obligatoire). Les solutions existantes sont soit trop lourdes (signature électronique qualifiée comme DocuSign à 10-30€/utilisateur/mois), soit non sécurisées (simple email).

La solution
Ackify génère des preuves de lecture cryptographiques avec :

Signatures Ed25519 (même algo que SSH)
Horodatage immutable (PostgreSQL triggers)
Hash chain blockchain-like
Vérification offline possible
Cas d'usage
Validation de politiques internes (sécurité, RGPD)
Attestations de formation obligatoire
Prise de connaissance de procédures
Accusés de réception contractuels
Différence avec DocuSign
Ackify n'est pas une alternative à DocuSign pour des contrats juridiques. C'est une solution simple pour des besoins internes où la signature qualifiée est overkill.
## Le problème

Les organisations doivent souvent prouver qu'un collaborateur a lu un document (politique RGPD, charte de sécurité, formation obligatoire). Les solutions existantes sont soit trop lourdes (signature électronique qualifiée comme DocuSign à 10-30€/utilisateur/mois), soit non sécurisées (simple email).

## La solution

Ackify génère des **preuves de lecture cryptographiques** avec :

- Signatures Ed25519 (même algo que SSH)
- Horodatage immutable (PostgreSQL triggers)
- Hash chain blockchain-like
- Vérification offline possible

## Cas d'usage

- Validation de politiques internes (sécurité, RGPD)
- Attestations de formation obligatoire
- Prise de connaissance de procédures
- Accusés de réception contractuels

## Différence avec DocuSign

Ackify n'est pas une alternative à DocuSign pour des contrats juridiques. C'est une solution simple pour des besoins internes où la signature qualifiée est overkill.

N'hésitez pas si vous avez des questions techniques !

## Installation

```
curl -fsSL https://raw.githubusercontent.com/btouchard/ackify-ce/main/install/install.sh | bash
cd ackify-ce
nano .env  # Configurer OAuth2
docker compose up -d
```

Installation complète en ~5 minutes.

## Stack technique

**Backend**

- Go 1.24 (Clean Architecture / DDD)
- PostgreSQL 16
- Chi Router
- OAuth2 (Google, GitHub, GitLab, custom) ou Magic Link (passwordless)

**Frontend**

- Vue 3 + TypeScript
- Tailwind CSS
- i18n (FR, EN, ES, DE, IT)

**DevOps**

- Docker distroless < 30 MB
- CI/CD GitHub Actions
- Tests : 72,6% couverture (180 tests unitaires + 33 intégration)