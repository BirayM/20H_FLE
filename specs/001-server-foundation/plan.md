# Implementation Plan: Socle du projet serveur minimal

**Branch**: `001-server-foundation` | **Date**: 2026-11-27 | **Spec**: [specs/001-server-foundation/spec.md](spec.md)

**Input**: Feature specification from `/specs/001-server-foundation/spec.md` et clarifications validées dans `/specs/001-server-foundation/clarifications.md`.

## Summary

Mettre en place le socle serveur minimal exécutable et testable du projet 20H-FLE en TypeScript strict sous Node.js 20 LTS. Le serveur repose sur Express avec rendu côté serveur (SSR-First) pour la page d'accueil (`GET /`, `lang="fr"`, titre « 20H-FLE »), une sonde de supervision (`GET /health` renvoyant `{ "status": "ok" }` sans cache), et un gestionnaire 404 sécurisé sans fuite technique avec négociation de contenu (HTML/JSON). L'architecture sépare l'instance applicative (`app.ts`) du processus d'écoute réseau (`server.ts`) afin de permettre des tests automatisés ultra-rapides en mémoire via Vitest et Supertest sans occuper de port physique.

## Technical Context

**Language/Version**: TypeScript 5.x / 7.x (`strict: true`, `noUncheckedIndexedAccess: true`, `moduleResolution: "NodeNext"`), Node.js >= 20 LTS

**Primary Dependencies**:
- Runtime : `express` (framework web serveur minimal mandaté par la Constitution)
- Dev : `@types/express`, `supertest`, `@types/supertest`, `tsx` (exécution & watch en développement)
- *Note* : Toutes les dépendances sont formellement justifiées dans [research.md](research.md).

**Storage**: N/A (socle initial stateless ; la persistance SQLite et les modèles de données métier sont hors périmètre de cette feature).

**Testing**: Vitest (`npm test`) avec Supertest pour les tests d'intégration HTTP in-memory (sans liaison de socket TCP physique).

**Target Platform**: Linux, macOS, Windows (Node.js 20 LTS).

**Project Type**: Service web avec rendu côté serveur (SSR-First).

**Performance Goals**:
- `GET /` : réponse HTML servie en < 100ms en local (SC-001).
- `GET /health` : réponse JSON servie en < 50ms (SC-002).

**Constraints**:
- Rendu 100% côté serveur sans framework frontend (Principe III).
- 0 JavaScript navigateur requis pour la consultation de la page d'accueil.
- Sécurité défensive : désactivation de `x-powered-by`, 0 trace de pile ou chemin système divulgué lors des erreurs 404 (Principe IV).
- Compilation propre dans `dist/` excluant les tests via `tsconfig.build.json` (FR-006).
- Validation stricte de la variable `PORT` (fallback à 3000 si omis, arrêt avec code 1 si invalide).
- Aucun secret commité ; variables documentées dans `.env.example`.

**Scale/Scope**: Socle serveur initial (3 routes/gestionnaires, ~200 lignes de code applicatif, documentation opérationnelle).

## Constitution Check

*GATE: Evaluated before Phase 0 research, re-checked post-design in Phase 1.*

| Principe Constitutionnel | Exigence | Statut & Justification |
| :--- | :--- | :--- |
| **I. Simplicité et Frugalité (KISS, YAGNI)** | Minimiser les dépendances, refuser la complexité prématurée, justifier chaque ajout. | **CONFORME** : 1 seule dépendance runtime (`express`, imposée par la constitution). Gabarits HTML via fonctions pures TypeScript sans moteur de template lourd. Justification complète dans `research.md`. |
| **II. Tests Automatisés Obligatoires** | 100% de la logique couverte par `npm test` via Vitest et Supertest. | **CONFORME** : Routes `/`, `/health`, 404 (HTML et JSON) et validation du port couvertes par tests in-memory sans écoute socket physique. |
| **III. Rendu Serveur & Amélioration Progressive (SSR-First)** | Rendu SSR via Express, 0 framework frontend, 0 CDN, consultation fonctionnelle sans JS navigateur. | **CONFORME** : Document HTML5 complet généré côté serveur avec `lang="fr"`. 0 JS client dans ce socle. |
| **IV. Sécurité Défensive & Protection des Données** | Headers sécurisés, aucune trace technique divulguée, validation d'entrée, aucun secret commité. | **CONFORME** : Header `x-powered-by` supprimé, 404 sans trace technique, validation stricte de `PORT`, `.env.example` sans données sensibles. |
| **V. Accessibilité Universelle & Standardisation Temporelle** | HTML sémantique, balisage accessible, `lang="fr"`, titres explicites. | **CONFORME** : Balises `<!DOCTYPE html>`, `<html lang="fr">`, `<title>`, `<h1>` systématiques. |
| **VI. Code Lisible & Documentation du « Pourquoi »** | TypeScript strict, nommage clair, documentation des intentions. | **CONFORME** : Architecture découpée (`app.ts`, `server.ts`, `config/`, `views/`), typage strict sans `any`. |

**Résultat de la porte (Gate)** : **SUCCÈS (Toutes les portes passent sans violation)**.

## Project Structure

### Documentation (this feature)

```text
specs/001-server-foundation/
├── spec.md              # Spécification fonctionnelle validée
├── clarifications.md    # Clarifications techniques (404, PORT, titres, cache)
├── checklists/
│   └── requirements.md  # Checklist de conformité de la spécification
├── plan.md              # Plan d'implémentation (ce document)
├── research.md          # Décisions d'architecture et justification des dépendances
├── data-model.md        # Modèles de données (ServerConfig, HealthResponse, ErrorResponse)
├── quickstart.md        # Guide de démarrage et scénarios de validation
└── contracts/
    ├── http-api.yaml    # Contrat d'interface OpenAPI 3.1
    └── http-api.md      # Contrats HTTP et CLI détaillés
```

### Source Code (repository root)

```text
src/
├── app.ts               # Configuration de l'application Express, middlewares, routes, 404
├── server.ts            # Point d'entrée exécutable (app.listen, logging, arrêt propre)
├── config/
│   └── env.ts           # Résolution et validation de la variable PORT et de l'environnement
└── views/
    ├── home.ts          # Fonction de rendu SSR de la page d'accueil (HTML5, lang="fr")
    └── not-found.ts     # Fonction de rendu SSR de la page d'erreur 404

tests/
├── app.test.ts          # Tests d'intégration HTTP (/, /health, 404 HTML/JSON) via Supertest
├── config.test.ts       # Tests unitaires de validation de PORT (défaut 3000, erreurs)
└── smoke.test.ts        # Test de fumée existant

tsconfig.json            # Configuration globale TypeScript (src + tests, typecheck)
tsconfig.build.json      # Configuration de compilation production (exclusion de tests, outDir dist/)
.env.example             # Modèle de configuration documentant PORT
README.md                # Documentation de démarrage, compilation et test du projet
```

**Structure Decision**: Architecture applicative monolithique modulaire (Option 1 adaptée) : séparation claire entre la couche d'assemblage HTTP (`app.ts`) et le cycle de vie du processus serveur (`server.ts`). Ce découpage permet à Supertest d'instancier l'application sans écoute TCP physique, conformément aux exigences du socle.

## Complexity Tracking

> Aucune violation des règles constitutionnelles n'est requise pour cette feature. La solution adopte la forme la plus frugale et directe possible.
