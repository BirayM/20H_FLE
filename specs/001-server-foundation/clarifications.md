# Clarifications de spécification : Socle du projet serveur minimal

Ce document recense les ambiguïtés et points de décision identifiés dans la spécification fonctionnelle (`spec.md`) de la feature `001-server-foundation`. Chaque question est accompagnée d'une réponse par défaut proposée et de son impact technique et fonctionnel pour guider l'implémentation et les tests automatisés sans modifier le fichier de spécification d'origine.

---

### Question 1 : Format et négociation de contenu pour les réponses 404 (FR-005, User Story 3)

- **Question** : Quel format de contenu (`Content-Type`) et quelle présentation doivent être retournés lors d'une réponse 404 pour une ressource non trouvée ?
- **Pourquoi c'est important** : Le critère `FR-005` exige un message clair exempt de traces techniques, mais ne précise pas si la réponse doit être servie en HTML (pour un navigateur), en JSON (pour une sonde ou client programmatique), ou dépendre de l'en-tête `Accept`.
- **Réponse par défaut proposée** : Implémenter une négociation de contenu simple : si l'en-tête `Accept` demande expressément du JSON (`application/json`) sans privilégier le HTML, répondre avec `{ "error": "Ressource non trouvée" }` en `application/json; charset=utf-8`. Dans tous les autres cas (notamment navigation web standard), répondre avec une page HTML valide minimale avec `lang="fr"` et un message clair (« Page non trouvée »).
- **Impact** : Structure le middleware de gestion des erreurs 404 dans Express, assure une expérience utilisateur propre en accord avec le principe SSR-First de la Constitution, et fixe les assertions pour les tests d'intégration HTTP.

---

### Question 2 : Sémantique et présence du titre sur la page d'accueil (FR-003, User Story 1)

- **Question** : Où et sous quelles balises HTML le titre « 20H-FLE » doit-il être présent sur la page racine `GET /` ?
- **Pourquoi c'est important** : L'exigence `FR-003` impose une page HTML valide contenant le titre « 20H-FLE ». Les suites de tests automatisés peuvent cibler l'élément `<title>` dans le `<head>`, la balise de niveau principal `<h1>` dans le `<body>`, ou les deux.
- **Réponse par défaut proposée** : Le document HTML5 valide doit comporter « 20H-FLE » à la fois dans la balise `<title>` de l'en-tête et dans un titre de premier niveau `<h1>` dans le corps de la page, avec les balises métadonnées indispensables (`<!DOCTYPE html>`, `<html lang="fr">`, `<meta charset="utf-8">`, `<meta name="viewport" content="width=device-width, initial-scale=1.0">`).
- **Impact** : Clarifie les sélecteurs de tests automatisés, garantit la validité syntaxique HTML5 et assure la conformité avec les règles d'accessibilité universelle (Principe V de la Constitution).

---

### Question 3 : Comportement face à une variable d'environnement `PORT` invalide (FR-002, Edge Cases)

- **Question** : Quel comportement exact le serveur doit-il adopter si la variable d'environnement `PORT` est définie mais contient une valeur invalide (chaîne alphabétique, nombre hors de la plage valide 1-65535, etc.) ?
- **Pourquoi c'est important** : La section *Edge Cases* indique que le serveur "s'arrête proprement avec un message d'erreur explicite dans le journal d'exécution", tandis que `FR-002` indique un port 3000 par défaut. Il convient de clarifier si une valeur invalide fournie doit basculer sur 3000 ou arrêter le processus.
- **Réponse par défaut proposée** : Si la variable `PORT` est présente mais non convertible en entier valide compris entre 1 et 65535, le serveur doit journaliser une erreur explicite sur `stderr` et interrompre le processus avec un code de sortie d'échec (`process.exit(1)`). Le port par défaut 3000 ne s'applique que lorsque la variable `PORT` est omise ou vide.
- **Impact** : Empêche un démarrage silencieux sur un port inattendu en cas de mauvaise configuration de déploiement, renforce la sécurité opérationnelle et permet de valider le comportement d'arrêt via des tests unitaires/d'intégration.

---

### Question 4 : Gestion du cache HTTP et en-têtes de la route de santé (FR-004, User Story 2)

- **Question** : La route `GET /health` doit-elle interdire explicitement toute mise en cache HTTP de sa réponse ?
- **Pourquoi c'est important** : La route `/health` est destinée aux sondes de supervision automatisées. Si un proxy inverse, un cache navigateur ou une passerelle intermédiaire met en cache la réponse `200 OK`, une défaillance ultérieure du serveur pourrait être masquée.
- **Réponse par défaut proposée** : La réponse JSON de `GET /health` doit inclure systématiquement l'en-tête HTTP `Cache-Control: no-cache, no-store, must-revalidate` en plus de `Content-Type: application/json; charset=utf-8`.
- **Impact** : Garantit la fiabilité temps réel de la sonde de santé sans risque d'état obsolète mis en cache (critère SC-002) et définit les assertions attendues dans les tests HTTP Supertest.

---

### Question 5 : Traitement des méthodes HTTP non autorisées sur les routes exposées (FR-003, FR-004, Edge Cases)

- **Question** : Quel code de statut HTTP doit être renvoyé lorsqu'un client soumet une méthode HTTP non supportée sur les routes déclarées (par exemple `POST /` ou `DELETE /health`) ?
- **Pourquoi c'est important** : La spécification précise que les méthodes non gérées doivent recevoir une réponse adaptée sans provoquer d'interruption du serveur. La norme HTTP suggère un code `405 Method Not Allowed`, alors que le comportement standard et minimaliste d'Express renvoie un code `404 Not Found` en l'absence de routeur dédié.
- **Réponse par défaut proposée** : Par application stricte du principe de simplicité et de frugalité (KISS/YAGNI) défini dans la Constitution du projet, les requêtes utilisant une méthode non gérée sont traitées par le middleware standard de route non trouvée et répondent avec un code HTTP 404 propre sans fuite d'information technique, sans complexifier le socle avec un gestionnaire spécifique de code 405.
- **Impact** : Évite l'ajout d'abstractions prématurées tout en garantissant la résilience du serveur face aux requêtes inattendues et en clarifiant les scénarios de tests négatifs.
