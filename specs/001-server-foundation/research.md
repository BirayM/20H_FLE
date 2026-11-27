# Recherche Technique : Socle du projet serveur minimal

Ce document consigne les décisions d'architecture et de conception pour l'implémentation de la feature `001-server-foundation`, conformément à la Constitution du projet et aux spécifications fonctionnelles (`spec.md` et `clarifications.md`).

---

## 1. Séparation Applicative : Instance Express vs Démarrage Serveur

- **Contexte** : Les critères FR-008 et SC-002, ainsi que la section *Edge Cases*, exigent que l'application puisse être testée intégralement sans nécessiter l'ouverture physique d'un port TCP réseau, tout en permettant à `npm run dev` et `npm start` d'écouter sur le port configuré.
- **Décision** : Séparer l'application en deux modules distincts :
  - `src/app.ts` : configure et exporte l'instance Express (`createApp()` ou `app`), avec l'enregistrement des middlewares, des routes (`/`, `/health`) et du gestionnaire 404. Aucun appel à `.listen()` n'y est fait.
  - `src/server.ts` : point d'entrée exécutable qui importe l'application depuis `src/app.ts`, charge et valide la configuration (port d'écoute), lie le socket réseau avec `app.listen()`, et gère la journalisation du démarrage et l'arrêt propre.
- **Raison d'être** :
  - Permet à `supertest` de solliciter directement l'application Express en mémoire via les flux HTTP internes de Node.js, sans occuper de port réseau réel ni provoquer de conflit de port (`EADDRINUSE`) en CI ou en exécution parallèle de tests.
  - Isole la logique de configuration d'environnement et de cycle de vie du processus système dans un fichier dédié.
- **Alternatives considérées** :
  - *Fichier unique `src/server.ts` qui démarre immédiatement l'écoute* : Rejeté car les tests d'intégration devraient soit démarrer un vrai serveur sur un port aléatoire (lourd et sujet à des fuites de sockets), soit nécessiter un serveur externe déjà lancé.
  - *Tests HTTP basés sur `fetch` contre une URL externe* : Rejeté car cela imposerait de maintenir un processus serveur actif en arrière-plan pendant les tests.

---

## 2. Compilation TypeScript et Structure de Build

- **Contexte** : Le critère FR-006 exige que `npm run build` produise les fichiers exécutables dans `dist/` sans inclure les fichiers de tests. Le critère FR-007 exige que `npm start` exécute cette version compilée. Le critère FR-009 impose que `npm run typecheck` vérifie la totalité des fichiers (code source et tests) sans erreur.
- **Décision** :
  - Conserver `tsconfig.json` à la racine comme configuration globale couvrant `src/**/*` et `tests/**/*`, utilisé par l'éditeur et par la commande `npm run typecheck` (`tsc --noEmit`).
  - Définir `tsconfig.build.json` étendant `./tsconfig.json` avec l'exclusion explicite des tests (`"exclude": ["tests/**/*", "**/*.test.ts"]`) et la cible d'émission `"outDir": "dist"`, `"rootDir": "src"`.
  - Configurer les scripts `package.json` :
    - `"build": "tsc -p tsconfig.build.json"`
    - `"start": "node dist/server.js"`
- **Raison d'être** :
  - Garantit l'absence totale de tests compilés dans `dist/` en production.
  - Utilise le compilateur natif `tsc` déjà présent dans `devDependencies`, sans outil tiers (KISS).
  - Préserve un typechecking exhaustif sur les tests lors de `npm run typecheck`.
- **Alternatives considérées** :
  - *Bannir `tsconfig.build.json` et utiliser `tsc` avec un flag d'exclusion en ligne de commande* : `tsc` ne supporte pas d'argument `--exclude` en CLI sans fichier de configuration.
  - *Utiliser un bundler (esbuild, rollup, webpack)* : Rejeté en vertu du principe I (Simplicité et Frugalité) ; `tsc` suffit amplement pour compiler du TypeScript côté serveur Node.js ESM.

---

## 3. Environnement d'Exécution en Développement (`npm run dev`)

- **Contexte** : Le critère FR-001 exige une commande `npm run dev` qui lance le serveur TypeScript et affiche l'adresse d'écoute dans la console avec rechargement automatique en cas de modification.
- **Décision** : Utiliser `tsx` avec la commande `tsx watch src/server.ts`.
- **Raison d'être** :
  - `tsx` est une dépendance de développement légère et spécialisée pour exécuter du TypeScript ESM moderne sous Node.js 20+ sans configuration complexe.
  - Intègre un mode surveillance natif (`watch`) performant et transparent.
  - Respecte la configuration `module: "NodeNext"` et la résolution d'extensions TypeScript.
- **Alternatives considérées** :
  - *`ts-node` + `nodemon`* : Plus lourd, configuration complexe avec ESM et NodeNext, nécessite plusieurs dépendances au lieu d'une seule.
  - *`tsc --watch` couplé à `node --watch`* : Nécessite deux processus synchronisés ou un script shell intermédiaire, plus lent et risque de décalage d'émission de fichiers.
  - *Drapeaux expérimentaux Node.js 20 (`--loader` ou `--experimental-strip-types`)* : Les fonctionnalités de type stripping natif sont stables sous Node.js 22+ mais expérimentales et incomplètes sous Node.js 20 LTS. `tsx` fournit une stabilité garantie pour Node 20.

---

## 4. Rendu Serveur de la Page d'Accueil et Accessibilité (SSR-First)

- **Contexte** : Les critères FR-003, User Story 1 et le Principe III de la Constitution imposent un rendu intégralement côté serveur (SSR) d'une page HTML valide contenant le titre « 20H-FLE » et `lang="fr"`.
- **Décision** : Implémenter le rendu SSR via une fonction pure TypeScript (ex: `renderHomePage()`) générant un document HTML5 sémantique et valide sous forme de template string typée.
  - Métadonnées complètes : `<!DOCTYPE html>`, `<html lang="fr">`, `<meta charset="utf-8">`, `<meta name="viewport" content="width=device-width, initial-scale=1.0">`.
  - Structure sémantique : Balise `<title>20H-FLE</title>` dans `<head>`, balise `<h1>20H-FLE</h1>` dans `<main>`, avec description succincte du projet.
  - En-tête HTTP : `Content-Type: text/html; charset=utf-8`.
- **Raison d'être** :
  - Respecte strictement le principe KISS et de frugalité : aucun moteur de gabarit lourd (EJS, Pug, Handlebars) n'est nécessaire pour des pages serveur simples.
  - Sécurité et maintenabilité maximales : typage strict, aucune injection non contrôlée, zéro dépendance supplémentaire.
  - Conforme aux exigences d'accessibilité (Principe V) et d'amélioration progressive (fonctionnement 100% sans JavaScript navigateur).
- **Alternatives considérées** :
  - *Moteur EJS ou Pug* : Rejeté car ajoute des dépendances externes injustifiées à ce stade du projet.
  - *Fichier HTML statique servi par `express.static`* : Rejeté car la Constitution stipule explicitement un rendu côté serveur (SSR), qui sera étendu lors des prochaines fonctionnalités (affichage de données dynamiques de l'utilisateur).

---

## 5. Gestion et Validation Robuste de la Variable `PORT`

- **Contexte** : FR-002, Edge Cases et Clarification Q3 précisent le comportement d'initialisation du port.
- **Décision** :
  - Extraire la logique de résolution du port dans un utilitaire dédié (ex: `resolvePort(val?: string): number`).
  - Si la variable `PORT` est indéfinie ou vide : retourner la valeur par défaut `3000`.
  - Si la variable `PORT` est renseignée : vérifier qu'il s'agit d'un entier strictement numérique compris entre 1 et 65535 (`/^\d+$/` et `1 <= n <= 65535`).
  - Si la valeur est invalide : écrire un message d'erreur clair sur `stderr` et quitter le processus avec `process.exit(1)`.
- **Raison d'être** :
  - Respecte scrupuleusement la clarification validée (Clarification Q3).
  - Évite tout comportement silencieux erroné ou démarrage sur un port imprévu lors d'une mauvaise configuration d'environnement.
- **Alternatives considérées** :
  - *Repli silencieux sur 3000 si invalide* : Rejeté car cela masque des erreurs de configuration critiques pour les administrateurs et exploitants.

---

## 6. Surveillance de Santé (`/health`) et Gestion Sécurisée des Erreurs 404

- **Contexte** : FR-004, FR-005, SC-002, SC-003, Principe IV (Sécurité Défensive) et Clarifications Q1, Q4, Q5.
- **Décision** :
  - Route `/health` :
    - Répond `200 OK` avec le JSON `{ "status": "ok" }`.
    - En-tête `Cache-Control: no-cache, no-store, must-revalidate` (Clarification Q4).
    - En-tête `Content-Type: application/json; charset=utf-8`.
  - Middleware d'erreur 404 (Route inconnue) :
    - Enregistré en fin de chaîne de middlewares Express.
    - Négociation de contenu (Clarification Q1) :
      - Si `req.accepts(['html', 'json']) === 'json'`, répondre avec du JSON : `{ "error": "Ressource non trouvée" }` (HTTP 404).
      - Sinon (navigateur par défaut), répondre avec un document HTML5 valide avec `lang="fr"`, titre « Page non trouvée - 20H-FLE », `<h1>Page non trouvée</h1>` et message d'accompagnement clair (HTTP 404).
    - Zéro divulgation d'information technique : aucune trace de pile (stack trace), aucun chemin système, aucune version Express/Node.
  - Durcissement sécurité HTTP :
    - Désactiver l'en-tête `x-powered-by` via `app.disable('x-powered-by')`.
  - Méthodes HTTP non gérées (Clarification Q5) :
    - Les requêtes sur des méthodes non prises en charge aboutissent au gestionnaire 404 standard, sans contrôleur spécifique 405 superflu.
- **Raison d'être** :
  - Conformité totale avec le principe Security by Design de la Constitution et les clarifications actées.
  - Garantie que les sondes de monitoring obtiennent un statut temps réel non mis en cache.
- **Alternatives considérées** :
  - *Gestionnaire 405 Method Not Allowed* : Écarté conformément à la Clarification Q5 pour respecter la frugalité (KISS/YAGNI).

---

## 7. Justification Formelle des Dépendances (Principe I de la Constitution)

Conformément au Principe I de la Constitution (*« Toute nouvelle dépendance ajoutée au projet DOIT faire l'objet d'une justification explicite et documentée »*), voici la justification de chaque bibliothèque retenue :

| Dépendance | Type | Rôle | Justification & Bénéfice | Alternative évaluée & Raison du rejet |
| :--- | :--- | :--- | :--- | :--- |
| `express` | Production | Framework HTTP serveur | Mandaté par la Constitution du projet (« Serveur web : Express avec moteur de rendu côté serveur (SSR) »). Robuste, standard, éprouvé et minimal. | `node:http` natif : Rejeté car la Constitution impose Express pour le socle et les futurs middlewares (sessions, CSRF, etc.). |
| `@types/express` | Dév | Typage TypeScript | Indispensable pour la compilation stricte (`strict: true`) du serveur Express en TypeScript. | Typage manuel / `any` : Rejeté par la Constitution (interdiction de contournement de typage). |
| `supertest` | Dév | Client de test d'intégration HTTP | Mandaté par la Constitution (« Supertest pour les tests d'intégration des routes HTTP »). Permet de tester l'instance Express en mémoire sans écoute socket réelle. | `fetch` natif : Rejeté car imposerait d'ouvrir un port réseau réel sur l'hôte pour chaque test. |
| `@types/supertest` | Dév | Typage TypeScript | Typage strict pour Vitest/TypeScript lors des tests d'intégration. | Typage manuel / `any` : Rejeté par la Constitution. |
| `tsx` | Dév | Exécuteur TypeScript et watcher | Permet d'exécuter `npm run dev` sous Node.js 20 ESM avec TypeScript sans étape de compilation préalable et avec rechargement automatique. | `ts-node` + `nodemon` : Plus lourd, configuration complexe en ESM. Drapeau natif Node 20 : instable / incomplet pour Node 20 LTS. |

Aucune autre dépendance n'est ajoutée. Le bundle de production reste limité au strict minimum (`express`).
