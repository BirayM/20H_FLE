---
description: "Task list for 001-server-foundation feature implementation"
---

# Tasks: Socle du projet serveur minimal

**Input**: Design documents from `/specs/001-server-foundation/`  
**Prerequisites**: [plan.md](plan.md) (required), [spec.md](spec.md) (required for user stories), [research.md](research.md), [data-model.md](data-model.md), [contracts/http-api.md](contracts/http-api.md), [quickstart.md](quickstart.md), [clarifications.md](clarifications.md)  
**Tests**: Inclus et obligatoires (mandatés par le Principe II de la Constitution et exigés par l'utilisateur avec l'approche TDD : tests rédigés et validés échouants avant les tâches de code correspondantes).  
**Organization**: Tâches découpées par user story afin de garantir une implémentation incrémentale et des tests indépendants pour chaque phase.

## Format: `- [ ] [TaskID] [P?] [Story?] Description with file path`

- **[P]** : Tâche exécutable en parallèle (fichiers distincts, aucune dépendance non résolue)
- **[Story]** : Identifiant de la user story cible ([US1], [US2], [US3], [US4]) — uniquement pour les phases de user stories
- Les chemins de fichiers exacts sont systématiquement inclus dans les descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Initialisation du projet, gestion des dépendances et outillage de compilation TypeScript

- [ ] T001 Installer la dépendance de production express et les dépendances de développement @types/express, supertest, @types/supertest, tsx dans package.json
- [ ] T002 [P] Configurer la compilation de production TypeScript excluant les tests dans tsconfig.build.json
- [ ] T003 Configurer les scripts npm du cycle de vie (dev, build, start) dans package.json

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Infrastructure de base et validation de la configuration d'environnement bloquante pour toutes les user stories

**⚠️ CRITICAL**: Les user stories ne peuvent démarrer tant que cette fondation n'est pas opérationnelle.

### Tests for Foundation (TDD First) ⚠️

> **NOTE: Rédiger ce test EN PREMIER et s'assurer qu'il ÉCHOUE avant d'implémenter le code**

- [ ] T004 [P] Écrire les tests unitaires de validation et résolution de PORT dans tests/config.test.ts (valeur par défaut 3000, chaînes non numériques, valeurs hors plage 1-65535, arrêt via process.exit(1))

### Implementation for Foundation

- [ ] T005 Définir l'interface ServerConfig et implémenter la fonction de résolution de PORT dans src/config/env.ts avec la contrainte : « Doit être un entier strictement numérique conforme à l'expression rationnelle `/^\d+$/`, compris dans l'intervalle `[1, 65535]`. Toute valeur non entière, hors intervalle ou alphanumérique provoque une erreur fatale au démarrage (`process.exit(1)`). `3000` si non spécifié ou chaîne vide dans l'environnement. »
- [ ] T006 [P] Initialiser l'instance applicative Express dans src/app.ts avec désactivation de l'en-tête X-Powered-By (app.disable('x-powered-by')) et export sans app.listen pour permettre les tests Supertest in-memory

**Checkpoint**: Fondation prête — l'implémentation des user stories peut débuter.

---

## Phase 3: User Story 1 - Consultation de la page d'accueil (Priority: P1) 🎯 MVP

**Goal**: Servir la page d'accueil du projet en rendu côté serveur (SSR-First) sur `GET /` avec statut HTTP 200, en-tête `Content-Type: text/html; charset=utf-8`, attribut `lang="fr"` et titre « 20H-FLE ».

**Independent Test**: Requête HTTP `GET /` vérifiant le statut HTTP 200, le type MIME `text/html; charset=utf-8`, la présence de `lang="fr"` sur la balise `<html>`, et le titre « 20H-FLE » dans `<title>` et `<h1>`, avec 0 JavaScript client requis.

### Tests for User Story 1 (TDD First) ⚠️

> **NOTE: Rédiger ces tests EN PREMIER et s'assurer qu'ils ÉCHOUENT avant d'implémenter le code**

- [ ] T007 [P] [US1] Écrire les tests d'intégration HTTP pour la route GET / dans tests/app.test.ts (vérifiant le statut HTTP 200, l'en-tête Content-Type: text/html; charset=utf-8, la présence de lang="fr" sur la balise <html>, le titre « 20H-FLE » dans <title> et <h1>, et le fonctionnement sans JavaScript navigateur)

### Implementation for User Story 1

- [ ] T008 [P] [US1] Créer le modèle HtmlDocumentModel avec la contrainte « Doit être obligatoirement 'fr' pour respecter la spécification FR-003 » et la fonction de rendu SSR de la page d'accueil dans src/views/home.ts (avec titre et balise h1 fixés à « 20H-FLE »)
- [ ] T009 [US1] Enregistrer la route GET / dans src/app.ts appelant la fonction de rendu de src/views/home.ts pour retourner la page d'accueil en HTTP 200 avec Content-Type text/html; charset=utf-8

**Checkpoint**: User Story 1 complètement fonctionnelle et testable de façon autonome.

---

## Phase 4: User Story 2 - Surveillance de la santé du service (Priority: P2)

**Goal**: Fournir une sonde opérationnelle `GET /health` répondant un statut HTTP 200 avec `{ "status": "ok" }` en JSON et interdiction explicite de mise en cache HTTP.

**Independent Test**: Requête HTTP `GET /health` vérifiant le statut HTTP 200, le type MIME `application/json; charset=utf-8`, le corps `{ "status": "ok" }` et l'en-tête `Cache-Control: no-cache, no-store, must-revalidate`.

### Tests for User Story 2 (TDD First) ⚠️

> **NOTE: Rédiger ces tests EN PREMIER et s'assurer qu'ils ÉCHOUENT avant d'implémenter le code**

- [ ] T010 [US2] Écrire les tests d'intégration HTTP pour la route GET /health dans tests/app.test.ts (vérifiant le statut HTTP 200, Content-Type: application/json; charset=utf-8, la charge utile {"status": "ok"}, et l'en-tête Cache-Control: no-cache, no-store, must-revalidate)

### Implementation for User Story 2

- [ ] T011 [US2] Définir le modèle HealthResponse et implémenter la route GET /health dans src/app.ts avec les contraintes : « Format de sérialisation : JSON (application/json; charset=utf-8). Aucun champ supplémentaire n'est exposé afin de ne divulguer aucune métrique interne non autorisée ou information de plateforme. En-têtes associés : Cache-Control: no-cache, no-store, must-revalidate. » et corps {"status": "ok"}

**Checkpoint**: User Stories 1 et 2 fonctionnelles et testables indépendamment.

---

## Phase 5: User Story 3 - Gestion sécurisée des adresses inconnues (Priority: P3)

**Goal**: Intercepter les requêtes ciblant des adresses non répertoriées ou des méthodes HTTP non supportées avec un code HTTP 404, sans fuite de traces techniques, avec négociation de contenu (HTML ou JSON).

**Independent Test**: Requêtes vers des chemins inexistants (ex: `GET /inconnu`) avec négociation de contenu (`Accept: text/html` et `Accept: application/json`) et requêtes avec méthodes non supportées (ex: `POST /`, `DELETE /health`), vérifiant le statut HTTP 404, l'absence de l'en-tête `X-Powered-By` et 0% de divulgation de traces techniques (stack trace, chemin système, version).

### Tests for User Story 3 (TDD First) ⚠️

> **NOTE: Rédiger ces tests EN PREMIER et s'assurer qu'ils ÉCHOUENT avant d'implémenter le code**

- [ ] T012 [US3] Écrire les tests d'intégration HTTP pour le gestionnaire 404 dans tests/app.test.ts (vérifiant le statut 404, la négociation HTML vs JSON via Accept: application/json, le traitement des méthodes non supportées ex: POST / ou DELETE /health, l'absence de header X-Powered-By et 0% de divulgation de traces techniques)

### Implementation for User Story 3

- [ ] T013 [P] [US3] Définir le modèle ErrorResponse et créer la fonction de rendu SSR 404 dans src/views/not-found.ts avec HtmlDocumentModel (lang="fr", titre « Page non trouvée - 20H-FLE », <h1>Page non trouvée</h1>)
- [ ] T014 [US3] Implémenter le middleware de capture 404 dans src/app.ts avec négociation de contenu et les contraintes : « Format de sérialisation : JSON (application/json; charset=utf-8). Contrainte de sécurité : L'objet ne DOIT JAMAIS comporter de pile d'appels (stack), chemin système, version logicielle ou identifiant interne. Valeur nominale : { "error": "Ressource non trouvée" }. » ou rendu HTML via src/views/not-found.ts

**Checkpoint**: User Stories 1, 2 et 3 fonctionnelles et validées en isolation.

---

## Phase 6: User Story 4 - Exploitation et vérification du socle par l'équipe de développement (Priority: P4)

**Goal**: Permettre le démarrage du serveur en local (`npm run dev` et `npm start`), la prise en compte flexible du port, la compilation propre (`npm run build`) et la documentation d'exploitation complète pour l'équipe de développement.

**Independent Test**: Démarrage du serveur avec `npm run dev` sur port spécifié, compilation avec `npm run build` produisant `dist/` sans tests, lancement de `npm start`, exécution de `npm run typecheck`, et conformité de la documentation `README.md`.

### Tests for User Story 4 (TDD First) ⚠️

> **NOTE: Rédiger ces tests EN PREMIER et s'assurer qu'ils ÉCHOUENT avant d'implémenter le code**

- [ ] T015 [P] [US4] Écrire les tests d'intégration et de cycle de vie du serveur dans tests/server.test.ts (démarrage du serveur HTTP sur le port résolu, vérification de l'écoute réseau, et arrêt propre du processus)

### Implementation for User Story 4

- [ ] T016 [US4] Implémenter le point d'entrée exécutable du serveur dans src/server.ts (importation de l'application depuis src/app.ts, résolution du port via src/config/env.ts, app.listen, affichage de l'URL d'écoute locale sur stdout, et gestion propre des signaux SIGINT/SIGTERM)
- [ ] T017 [P] [US4] Renseigner le fichier modèle d'environnement .env.example avec les variables supportées (PORT=3000, NODE_ENV=development) sans secret ni donnée sensible
- [ ] T018 [P] [US4] Rédiger la documentation d'exploitation (installation, npm run dev, npm run build, npm start, npm test, npm run typecheck, architecture) dans README.md

**Checkpoint**: L'ensemble des 4 user stories est opérationnel, testé et documenté.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Validation finale transversale, conformité stricte du typage et exécution des scénarios bout-en-bout

- [ ] T019 Exécuter la vérification stricte du typage sur l'ensemble des fichiers TypeScript avec npm run typecheck dans tsconfig.json
- [ ] T020 Exécuter l'intégralité de la suite de tests automatisés avec npm test et vérifier 100% de réussite dans tests/
- [ ] T021 Valider la compilation de production avec npm run build et vérifier la conformité du dossier dist/ sans aucun fichier de test via tsconfig.build.json
- [ ] T022 Exécuter et valider l'ensemble des scénarios de validation opérationnelle décrits dans specs/001-server-foundation/quickstart.md

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)** : Aucune dépendance — peut démarrer immédiatement.
- **Foundational (Phase 2)** : Dépend de la Phase 1 (Setup). **BLOQUE** toutes les user stories.
- **User Stories (Phases 3 à 6)** : Dépendent toutes de l'achèvement complet de la Phase 2 (Foundational).
  - US1 (P1) peut démarrer dès la fin de Phase 2 (priorité MVP).
  - US2 (P2) peut démarrer dès la fin de Phase 2 (indépendante de US1, routes distinctes).
  - US3 (P3) peut démarrer dès la fin de Phase 2 (middleware terminal s'enregistrant après les routes).
  - US4 (P4) dépend de l'application Express (`src/app.ts`) et de la configuration (`src/config/env.ts`) pour lier le serveur.
- **Polish (Phase 7)** : Dépend de l'achèvement de toutes les user stories.

### User Story Dependencies

- **User Story 1 (P1)** : Démarre après Phase 2. Aucune dépendance sur les autres stories.
- **User Story 2 (P2)** : Démarre après Phase 2. Indépendante de US1 au niveau modèle et route.
- **User Story 3 (P3)** : Démarre après Phase 2. Se positionne en fin de chaîne de middlewares d'Express.
- **User Story 4 (P4)** : Démarre après Phase 2 et s'appuie sur `src/app.ts` pour assembler l'exécutable `src/server.ts`.

### Within Each User Story (Strict TDD Order)

1. **Tests rédigés en premier** : Chaque suite de tests est rédigée et exécutée pour vérifier son échec avant toute implémentation.
2. **Modèles de données & vues SSR** avant les contrôleurs/routes.
3. **Contrôleurs et middlewares** avant l'enregistrement dans l'application.
4. **Validation autonome** de la user story avant passage à la suivante.

### Parallel Opportunities

- **Phase 1** : T002 (`tsconfig.build.json`) peut s'exécuter en parallèle de T001 (`package.json`).
- **Phase 2** : T004 (`tests/config.test.ts`) et T006 (`src/app.ts`) peuvent être travaillés en parallèle.
- **Phase 3 (US1)** : T007 (`tests/app.test.ts`) et T008 (`src/views/home.ts`) peuvent être créés en parallèle.
- **Phase 5 (US3)** : T013 (`src/views/not-found.ts`) peut être développé en parallèle de T012 (`tests/app.test.ts`).
- **Phase 6 (US4)** : T015 (`tests/server.test.ts`), T017 (`.env.example`) et T018 (`README.md`) peuvent s'exécuter en parallèle.

---

## Parallel Example: User Story 1

```bash
# Rédiger le test HTTP d'intégration pour US1 :
Task: "T007 [P] [US1] Écrire les tests d'intégration HTTP pour la route GET / dans tests/app.test.ts"

# Créer simultanément le gabarit SSR pour US1 :
Task: "T008 [P] [US1] Créer le modèle HtmlDocumentModel et la fonction de rendu SSR dans src/views/home.ts"

# Puis brancher la route dans l'application :
Task: "T009 [US1] Enregistrer la route GET / dans src/app.ts"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Exécuter Phase 1 : Setup (dépendances et compilation).
2. Exécuter Phase 2 : Foundational (validation de configuration PORT et squelette `src/app.ts`).
3. Exécuter Phase 3 : User Story 1 (tests T007, vue T008, route T009).
4. **VALIDER LE MVP** : `npm test` doit valider la route `GET /` avec code 200 et HTML français en mémoire.

### Incremental Delivery

1. Setup + Foundational terminés → Socle prêt.
2. Ajout US1 (Page d'accueil) → Test autonome → MVP délivré.
3. Ajout US2 (Sonde `/health`) → Test autonome → Supervision activée.
4. Ajout US3 (Gestion 404 sécurisée) → Test autonome → Robustesse et sécurité défensive en place.
5. Ajout US4 (Exécution serveur & documentation) → Test cycle de vie → Prêt pour les prochaines features.
6. Phase 7 (Polish) → Exécution de `typecheck`, `test`, `build` et vérification de `quickstart.md`.

---

## Notes

- Tous les tests HTTP s'exécutent en mémoire via Supertest sans écoute socket TCP physique.
- Le mode TDD est rigoureusement appliqué : chaque test doit échouer avant l'implémentation correspondante.
- Aucune donnée technique ni secret ne doit être divulgué ou commité.
