# Modèle de Données : Socle du projet serveur minimal

**Feature**: `001-server-foundation`  
**Date**: 2026-11-27  
**Périmètre de persistance** : Aucune persistance en base de données pour ce socle initial (les comptes, séances et compétences sont expressément hors périmètre). Ce document modélise les structures de données en mémoire, les contrats de configuration système et les modèles de réponse HTTP.

---

## 1. Modèle de Configuration d'Exécution (`ServerConfig`)

Modélise les paramètres d'exécution injectés via l'environnement au démarrage du processus serveur.

### Définition TypeScript

```typescript
export interface ServerConfig {
  /**
   * Port TCP d'écoute du serveur.
   * Résolu depuis process.env.PORT ou 3000 par défaut.
   */
  readonly port: number;

  /**
   * Environnement d'exécution Node.js (ex: 'development', 'production', 'test').
   */
  readonly nodeEnv: string;
}
```

### Règles de validation

| Champ | Type | Présence | Contraintes & Règles de validation | Valeur de repli (défaut) |
| :--- | :--- | :--- | :--- | :--- |
| `port` | `number` | Obligatoire (résolu) | Doit être un entier strictement numérique conforme à l'expression rationnelle `/^\d+$/`, compris dans l'intervalle `[1, 65535]`. Toute valeur non entière, hors intervalle ou alphanumérique provoque une erreur fatale au démarrage (`process.exit(1)`). | `3000` si non spécifié ou chaîne vide dans l'environnement. |
| `nodeEnv` | `string` | Optionnel | Chaîne indicative (`process.env.NODE_ENV`). | `'development'` si indéfini. |

---

## 2. Modèle de Réponse de Santé (`HealthResponse`)

Structure de données renvoyée par la sonde de disponibilité opérationnelle `GET /health`.

### Définition TypeScript

```typescript
export interface HealthResponse {
  /**
   * Statut d'état du service.
   * Valeur fixe 'ok' lorsque le service Express est disponible.
   */
  readonly status: "ok";
}
```

### Règles de validation et de sérialisation

- **Format de sérialisation** : JSON (`application/json; charset=utf-8`).
- **Contrainte** : Aucun champ supplémentaire n'est exposé afin de ne divulguer aucune métrique interne non autorisée ou information de plateforme.
- **En-têtes associés** : `Cache-Control: no-cache, no-store, must-revalidate`.

---

## 3. Modèle d'Erreur HTTP (`ErrorResponse`)

Structure de données renvoyée lors d'une route non trouvée (HTTP 404) pour les clients demandant explicitement une réponse JSON.

### Définition TypeScript

```typescript
export interface ErrorResponse {
  /**
   * Message fonctionnel lisible décrivant l'absence de ressource.
   */
  readonly error: string;
}
```

### Règles de validation et de sécurité

- **Format de sérialisation** : JSON (`application/json; charset=utf-8`).
- **Contrainte de sécurité** : L'objet ne DOIT JAMAIS comporter de pile d'appels (`stack`), chemin système, version logicielle ou identifiant interne.
- **Valeur nominale** : `{ "error": "Ressource non trouvée" }`.

---

## 4. Modèle de Page HTML Rendu Serveur (`HtmlDocumentModel`)

Structure de données interne utilisée par les fonctions de gabarit SSR pour générer les pages HTML5 valides.

### Définition TypeScript

```typescript
export interface HtmlDocumentModel {
  /**
   * Attribut de langue du document (conforme BCP 47).
   * Doit être obligatoirement 'fr' pour respecter la spécification FR-003.
   */
  readonly lang: "fr";

  /**
   * Titre affiché dans l'onglet du navigateur (<title>).
   */
  readonly title: string;

  /**
   * Titre de niveau principal dans le corps de la page (<h1>).
   */
  readonly heading: string;

  /**
   * Description textuelle sémantique (<p> ou contenu principal).
   */
  readonly description?: string;
}
```

### Instances types

1. **Page d'accueil (`GET /`)** :
   ```typescript
   const homePageModel: HtmlDocumentModel = {
     lang: "fr",
     title: "20H-FLE",
     heading: "20H-FLE",
     description: "Application d'apprentissage et de suivi de pratique du Français Langue Étrangère.",
   };
   ```

2. **Page d'erreur 404 (`GET /inconnu` avec acceptation HTML)** :
   ```typescript
   const notFoundPageModel: HtmlDocumentModel = {
     lang: "fr",
     title: "Page non trouvée - 20H-FLE",
     heading: "Page non trouvée",
     description: "La ressource demandée n'existe pas ou a été déplacée.",
   };
   ```

---

## 5. Cycle de Vie et Transitions d'État

En l'absence de base de données dans ce socle initial, les modèles sont **entièrement immutables et sans état persistant** (*stateless*). Chaque requête HTTP instancie ou évalue les structures de données à la volée avant de répondre au client.
