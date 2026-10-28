# bsf-portail

Portail B-School Factory : pilotage du cycle de vie des projets pédagogiques, de leur dépôt à leur capitalisation.

> **État : amorçage.** Le dépôt est créé (ticket BSF-43). Il ne contient pas encore de code.

## Parcours métier couvert

Déposer → Préqualifier → Orienter → Cadrer → Réaliser → Démontrer → Évaluer → Capitaliser.

Un *sujet* est une demande pas encore acceptée ; il devient un *projet* lorsqu'il est accepté et cadré.

## Contenu du dépôt

| Dossier | Contenu |
|---|---|
| `frontend/` | Application Angular |
| `backend/` | API Spring Boot |
| `docs/` | Documentation technique, décisions d'architecture (ADR) |
| `.github/` | Modèle de pull request |

## Stack

Angular (front), Java avec Spring Boot (back), PostgreSQL (base).
Les versions, l'outil de build et les bibliothèques de test seront fixés par l'ADR-002 (ticket BSF-44).

## Lancer le projet

Rien à lancer à ce jour. Cette section sera complétée avec les premières commandes de build et de démarrage local (Docker Compose prévu).

## Dépendances avec les autres dépôts

- `bsf-auth` : identité, comptes et rôles, partagés avec les autres produits du programme.
- `bsf-contracts` : contrats d'API (OpenAPI) échangés avec les autres groupes. Le Portail **consomme** ces services, il ne les réimplémente pas.

## Données

Toutes les données de développement, de test et de démonstration sont **fictives**.

## Contribuer

Voir [CONTRIBUTING.md](CONTRIBUTING.md) : branches courtes, commits référençant le ticket Jira `BSF-xx`, pull requests vers `main`.

## Maintenance

Squad Portail. Le suivi du travail se fait dans le projet Jira `BSF`.
