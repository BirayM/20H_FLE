# Spécification des Contrats d'Interface : Socle Serveur

**Feature**: `001-server-foundation`  
**Date**: 2026-11-27  

Ce document décrit formellement l'ensemble des contrats d'interface exposés par le socle minimal : endpoints HTTP, en-têtes réseau, négociation de contenu, et contrat d'interface système/CLI.

---

## 1. Contrats d'Interface HTTP

### 1.1. Route d'accueil `GET /`

- **Objectif** : Point d'entrée utilisateur en rendu côté serveur (SSR-First).
- **Méthode** : `GET`
- **Chemin** : `/`
- **Paramètres** : Aucun.
- **Réponse HTTP 200 OK** :
  - **En-tête `Content-Type`** : `text/html; charset=utf-8`
  - **Corps de la réponse** : Document HTML5 valide respectant la structure :
    ```html
    <!DOCTYPE html>
    <html lang="fr">
      <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>20H-FLE</title>
      </head>
      <body>
        <main>
          <h1>20H-FLE</h1>
          <p>Socle du projet 20H-FLE.</p>
        </main>
      </body>
    </html>
    ```
- **Contraintes** :
  - `lang="fr"` obligatoire sur la balise `<html>`.
  - « 20H-FLE » présent dans `<title>` et `<h1>`.
  - Aucun JavaScript requis pour afficher ou lire le contenu.

---

### 1.2. Sonde de santé opérationnelle `GET /health`

- **Objectif** : Sonde automatisée pour la supervision et l'orchestration.
- **Méthode** : `GET`
- **Chemin** : `/health`
- **Paramètres** : Aucun.
- **Réponse HTTP 200 OK** :
  - **En-têtes obligatoires** :
    - `Content-Type: application/json; charset=utf-8`
    - `Cache-Control: no-cache, no-store, must-revalidate`
  - **Corps de la réponse** :
    ```json
    {
      "status": "ok"
    }
    ```
- **Contraintes** :
  - Interdiction de toute mise en cache (Clarification Q4).
  - Temps de réponse < 50ms (Critère SC-002).

---

### 1.3. Gestion des routes et méthodes non enregistrées (404 Not Found)

- **Objectif** : Gestion sécurisée des requêtes ciblant une ressource inconnue ou avec méthode non supportée (FR-005, Clarifications Q1 et Q5).
- **Méthodes capturées** : Toutes méthodes sur chemin non enregistré ou méthodes non gérées sur routes existantes (ex: `POST /`, `DELETE /health`).
- **Négociation de contenu** :
  - **Cas A : Requête préférant le JSON** (`Accept: application/json` prioritaire) :
    - **Statut** : `404 Not Found`
    - **En-tête** : `Content-Type: application/json; charset=utf-8`
    - **Corps** :
      ```json
      {
        "error": "Ressource non trouvée"
      }
      ```
  - **Cas B : Requête standard / navigateur** (`Accept: text/html` ou en-tête omis/générique) :
    - **Statut** : `404 Not Found`
    - **En-tête** : `Content-Type: text/html; charset=utf-8`
    - **Corps** : Document HTML5 valide avec `lang="fr"`, titre « Page non trouvée - 20H-FLE », `<h1>Page non trouvée</h1>`, et message explicite.
- **Règles de sécurité défensive** :
  - Absence totale d'en-tête `X-Powered-By`.
  - Zéro trace technique : aucune pile d'exécution (*stack trace*), chemin de fichier système ou version de bibliothèque.

---

## 2. Contrat d'Interface Système et Ligne de Commande (CLI)

### 2.1. Variables d'Environnement

Le serveur lit sa configuration via les variables d'environnement suivantes :

| Variable | Type | Requis | Valeur par défaut | Description & Règle de validation |
| :--- | :--- | :--- | :--- | :--- |
| `PORT` | Entier | Non | `3000` | Port d'écoute TCP. Si renseigné, doit être une chaîne strictement numérique comprise entre `1` et `65535`. |
| `NODE_ENV` | Chaîne | Non | `development` | Environnement d'exécution (`development`, `production`, `test`). |

En cas de variable `PORT` invalide (non numérique ou hors intervalle) :
- Écriture d'un message d'erreur clair sur `process.stderr` : `[config] Erreur: Le port spécifié ("...") est invalide. Doit être un entier compris entre 1 et 65535.`
- Sortie immédiate du processus avec le code de sortie `process.exit(1)`.

### 2.2. Commandes du cycle de vie du projet (`package.json`)

| Commande | Rôle | Comportement attendu |
| :--- | :--- | :--- |
| `npm run dev` | Lancement en développement | Lance `tsx watch src/server.ts`, écoute sur le port configuré et affiche l'URL locale sur `stdout`. |
| `npm run build` | Compilation de production | Lance `tsc -p tsconfig.build.json` et émet les fichiers exécutables dans `dist/` sans aucun fichier de test. |
| `npm start` | Lancement en production | Lance `node dist/server.js` à partir des artefacts compilés. |
| `npm test` | Exécution des tests | Lance Vitest sur l'ensemble de la suite de tests avec 100% de succès sans nécessiter l'ouverture d'un port TCP. |
| `npm run typecheck` | Contrôle strict des types | Lance `tsc --noEmit` sur l'ensemble du projet sans aucune erreur de compilation. |
