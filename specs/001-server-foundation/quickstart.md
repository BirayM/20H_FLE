# Guide de Démarrage et de Validation Rapide : Socle Serveur

**Feature**: `001-server-foundation`  
**Date**: 2026-11-27  

Ce guide présente les scénarios de vérification bout-en-bout permettant de valider le fonctionnement opérationnel du socle minimal du projet. Pour les détails des schémas et en-têtes, se référer aux [Contrats d'interface](contracts/http-api.md) et au [Modèle de données](data-model.md).

---

## 1. Prérequis Système

- **Node.js** : version 20 LTS ou supérieure (`node -v` >= 20.0.0).
- **npm** : version 10 ou supérieure (`npm -v`).
- **Fichier d'environnement** : Copier `.env.example` vers `.env` si une personnalisation est souhaitée (facultatif car les valeurs par défaut sont prévues).

---

## 2. Procédure d'Installation et Initialisation

Depuis la racine du dépôt :

```bash
# Installation des dépendances du projet
npm install
```

---

## 3. Scénarios de Validation Opérationnelle

### Scénario 1 : Contrôle de conformité des types TypeScript

- **Commande** :
  ```bash
  npm run typecheck
  ```
- **Résultat attendu** :
  - La commande se termine avec un code de sortie `0`.
  - Aucune erreur de typage ou avertissement n'est affiché.

---

### Scénario 2 : Exécution de la suite de tests automatisés

- **Commande** :
  ```bash
  npm test
  ```
- **Résultat attendu** :
  - 100% des tests réussissent.
  - Couverture des cas suivants validée en mémoire via Supertest (sans ouverture de port TCP physique) :
    1. `GET /` renvoie le statut `200`, le type `text/html; charset=utf-8`, la balise `<html lang="fr">` et le titre `20H-FLE`.
    2. `GET /health` renvoie le statut `200`, le type `application/json; charset=utf-8`, le corps `{ "status": "ok" }` et l'en-tête `Cache-Control: no-cache, no-store, must-revalidate`.
    3. `GET /inconnu` renvoie le statut `404` avec du HTML lisible sans trace technique.
    4. `GET /inconnu` avec `Accept: application/json` renvoie le statut `404` avec `{ "error": "Ressource non trouvée" }`.
    5. La résolution de `PORT` utilise 3000 par défaut et rejette les valeurs non valides.

---

### Scénario 3 : Compilation et exécution de production

1. **Compilation des artefacts** :
   ```bash
   npm run build
   ```
   - **Vérification** : Le dossier `dist/` est créé et contient les fichiers JavaScript compilés (`server.js`, `app.js`, etc.). Aucun fichier de test (`*.test.js` ou dossier `tests/`) n'y est présent.

2. **Démarrage en mode production** :
   ```bash
   PORT=4000 npm start
   ```
   - **Vérification console** : Le serveur affiche un message confirmant l'écoute sur le port `4000` (ex: `[server] Écoute sur http://localhost:4000`).

3. **Vérification HTTP externe** :
   ```bash
   curl -i http://localhost:4000/
   curl -i http://localhost:4000/health
   curl -i http://localhost:4000/route-invalide
   ```
   - `/` : Code 200, HTML français contenant « 20H-FLE ».
   - `/health` : Code 200, JSON `{ "status": "ok" }`.
   - `/route-invalide` : Code 404, message clair sans trace de pile.

---

### Scénario 4 : Lancement en mode développement avec rechargement à chaud

- **Commande** :
  ```bash
  npm run dev
  ```
- **Résultat attendu** :
  - Le serveur démarre via `tsx watch` sur le port par défaut (3000 si `PORT` non renseigné).
  - La console affiche l'adresse d'écoute locale.
  - La modification d'un fichier source redémarre automatiquement le processus.
