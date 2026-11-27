# Feature Specification: Socle du projet serveur minimal

**Feature Branch**: `001-server-foundation`

**Created**: 2026-11-27

**Status**: Draft

**Input**: User description: "Socle du projet : serveur TypeScript minimal avec page d'accueil et vérification de santé ## 1. Problème / Besoin Le projet n'a pas encore d'application. Il faut un socle exécutable et testable sur lequel les prochaines fonctionnalités (comptes, compétences, séances) viendront s'ajouter. ## 2. Utilisateurs concernés L'équipe de développement, puis tous les futurs utilisateurs du site. ## 3. Critères d'acceptation - **Démarrage** : la commande `npm run dev` démarre le serveur et affiche l'adresse d'écoute dans le terminal. - **Port** : le port vient de la variable d'environnement `PORT`, avec 3000 par défaut. - **Page d'accueil** : `GET /` répond 200 avec une page HTML valide contenant le titre « 20H-FLE » en français (`lang=\"fr\"`). - **Santé** : `GET /health` répond 200 avec du JSON `{ \"status\": \"ok\" }`. - **Erreur** : une adresse inconnue répond 404 avec une page ou un message lisible, sans trace technique. - **Compilation** : `npm run build` produit le code exécutable dans `dist/`, sans les tests, et `npm start` lance cette version compilée. - **Tests** : `npm test` couvre la page d'accueil, la route de santé et le 404 ; l'application doit pouvoir être testée sans démarrer réellement le serveur. - **Types** : `npm run typecheck` passe sans erreur. - **Documentation** : un `README.md` explique en quelques lignes comment installer, lancer, tester et compiler le projet. ## 4. Hors périmètre - Comptes utilisateurs, base de données, compétences, séances, minuteur. - Tout travail de design : la page d'accueil reste volontairement minimale. - Déploiement. ## 5. Contraintes - Respecter la constitution du projet. - Chaque dépendance ajoutée est justifiée dans le plan. - Aucun secret dans le dépôt ; les variables d'environnement sont documentées dans `.env.example`."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Consultation de la page d'accueil (Priority: P1)

En tant qu'utilisateur ou apprenant accédant au service web, je souhaite afficher la page d'accueil du projet afin de vérifier que l'application 20H-FLE est disponible, accessible et présentée en langue française.

**Why this priority**: Il s'agit du point d'entrée principal de l'application et de la première valeur délivrée aux utilisateurs finaux. Sans page d'accueil fonctionnelle, le service n'est pas utilisable.

**Independent Test**: Peut être testé de manière autonome en envoyant une requête sur la racine `/` et en vérifiant la présence du code HTTP 200, d'un document HTML valide contenant le titre « 20H-FLE » et l'attribut de langue française.

**Acceptance Scenarios**:

1. **Given** le serveur est en cours d'exécution, **When** un utilisateur accède à l'adresse racine `/`, **Then** le système répond avec le code HTTP 200 et sert un document HTML valide contenant le titre « 20H-FLE » et défini avec `lang="fr"`.
2. **Given** un navigateur sans exécution JavaScript, **When** la page d'accueil est chargée, **Then** l'ensemble du contenu textuel de la page d'accueil s'affiche lisiblement sans dégradation fonctionnelle.

---

### User Story 2 - Surveillance de la santé du service (Priority: P2)

En tant qu'opérateur ou développeur supervisant la plateforme, je souhaite interroger un indicateur de santé automatique afin de vérifier l'état opérationnel du service sans avoir à analyser une page HTML complète.

**Why this priority**: Essentiel pour la supervision, les sondes automatisées et la validation immédiate de la disponibilité du serveur en environnement d'exécution.

**Independent Test**: Peut être testé indépendamment par une requête HTTP sur `/health` et l'assertion que la réponse est un code 200 avec une charge utile confirmant le statut opérationnel.

**Acceptance Scenarios**:

1. **Given** le service applicatif est actif et prêt, **When** une requête `GET /health` est émise, **Then** le système répond avec le code HTTP 200 et un document JSON contenant `{ "status": "ok" }`.

---

### User Story 3 - Gestion sécurisée des adresses inconnues (Priority: P3)

En tant qu'utilisateur naviguant sur une mauvaise adresse ou tiers envoyant une requête inattendue, je souhaite recevoir un message clair indiquant que la ressource demandée n'existe pas, sans exposition de détails techniques internes du serveur.

**Why this priority**: Assure une expérience utilisateur claire lors des erreurs d'aiguillage tout en respectant les exigences de sécurité défensive (aucun détail d'infrastructure ou pile d'appels divulgué).

**Independent Test**: Peut être testé indépendamment en envoyant une requête vers une ressource non enregistrée (ex: `/adresse-inexistante`) et en confirmant l'obtention d'un code 404 accompagné d'un message compréhensible exempt de traces techniques.

**Acceptance Scenarios**:

1. **Given** le service est en ligne, **When** un client demande une URL inexistante, **Then** le système répond avec le code HTTP 404 et un contenu lisible informant de l'absence de ressource.
2. **Given** une requête vers un chemin non reconnu déclenche une réponse 404, **When** la réponse est analysée, **Then** aucune trace de pile (stack trace), chemin d'accès système ou version logicielle interne n'est exposée.

---

### User Story 4 - Exploitation et vérification du socle par l'équipe de développement (Priority: P4)

En tant que développeur de l'équipe, je souhaite démarrer le serveur en local avec une configuration flexible de port, compiler l'application, et exécuter des tests automatisés en toute autonomie grâce à une documentation claire.

**Why this priority**: Fournit l'environnement d'ingénierie et le cadre de travail indispensables pour implémenter de façon robuste et incrémentale les prochaines fonctionnalités du produit.

**Independent Test**: Peut être testé de manière autonome en exécutant les commandes documentées pour le démarrage, la configuration du port d'écoute, l'exécution des tests automatisés et le contrôle strict des types.

**Acceptance Scenarios**:

1. **Given** une absence de variable `PORT` dans l'environnement, **When** la commande de démarrage en développement `npm run dev` est exécutée, **Then** le serveur démarre sur le port 3000 par défaut et affiche l'adresse d'écoute dans la console.
2. **Given** une variable d'environnement `PORT` définie à une valeur spécifique (ex: 8080), **When** le serveur démarre, **Then** il écoute sur le port configuré.
3. **Given** les sources du projet, **When** la commande de compilation `npm run build` est lancée, **Then** les artefacts exécutables sont produits dans le dossier `dist/` sans les fichiers de tests, et `npm start` exécute ces fichiers compilés.
4. **Given** l'application, **When** la commande de test `npm test` est exécutée, **Then** les routes d'accueil, de santé et d'erreur 404 sont testées avec succès sans nécessiter l'ouverture obligatoire d'un port réseau physique.
5. **Given** l'ensemble des fichiers source, **When** la vérification de typage `npm run typecheck` est lancée, **Then** aucune erreur de typage n'est signalée.
6. **Given** le dépôt du projet, **When** un développeur consulte le fichier `README.md`, **Then** les instructions d'installation, d'exécution, de compilation et de test sont présentées de façon claire et concise.

---

### Edge Cases

- **Port réseau occupé ou invalide** : Si la variable d'environnement `PORT` contient une valeur invalide (ex: non numérique) ou si le port est déjà réservé par un autre processus système, le serveur s'arrête proprement avec un message d'erreur explicite dans le journal d'exécution.
- **Accès sans ouverture de socket réseau réel** : L'architecture de test doit permettre d'instancier et de solliciter la couche de traitement des requêtes HTTP en mémoire sans lier de port TCP physique, évitant ainsi les conflits de ports en intégration continue ou en exécution concurrente.
- **Requêtes avec méthodes non autorisées ou chemins malformés** : Les requêtes avec des caractères inattendus ou des méthodes non gérées reçoivent une réponse d'erreur adaptée sans bloquer ni interrompre le serveur.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Le système DOIT fournir une commande d'exécution pour le développement (`npm run dev`) démarrant le serveur et affichant son adresse d'écoute dans le terminal.
- **FR-002**: Le système DOIT déterminer son port d'écoute via la variable d'environnement `PORT`, avec une valeur de repli fixée à 3000 si la variable n'est pas définie.
- **FR-003**: Le système DOIT répondre avec un code HTTP 200 à la requête `GET /` et retourner une page HTML valide contenant le titre « 20H-FLE » et l'attribut `lang="fr"`.
- **FR-004**: Le système DOIT répondre avec un code HTTP 200 à la requête `GET /health` et retourner un document JSON avec le contenu `{ "status": "ok" }`.
- **FR-005**: Le système DOIT répondre avec un code HTTP 404 à toute requête ciblant une adresse inexistante, avec un message clair et compréhensible exempt de toute information technique ou trace de débogage interne.
- **FR-006**: Le système DOIT fournir une commande de compilation (`npm run build`) générant les fichiers exécutables prêts pour la production dans le dossier `dist/`, en excluant les fichiers de test.
- **FR-007**: Le système DOIT fournir une commande de démarrage (`npm start`) qui exécute l'application à partir des fichiers compilés dans `dist/`.
- **FR-008**: Le système DOIT fournir une commande de test (`npm test`) validant les fonctionnalités de la page d'accueil, du contrôle de santé et de la gestion du code 404, avec la capacité de tester l'application sans écoute réseau physique réelle.
- **FR-009**: Le système DOIT fournir une commande de vérification des types (`npm run typecheck`) garantissant une conformité sans avertissement ni erreur de compilation.
- **FR-010**: Le projet DOIT inclure un modèle de configuration `.env.example` documentant les variables d'environnement supportées, sans aucun secret ni donnée sensible.
- **FR-011**: Le projet DOIT comporter une documentation `README.md` résumant les instructions d'installation, de démarrage (développement et production), de test et de compilation.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% des requêtes adressées à la page d'accueil (`/`) retournent une page HTML valide et lisible avec un code HTTP 200 en moins de 100 millisecondes en condition nominale locale.
- **SC-002**: 100% des requêtes de vérification de santé (`/health`) reçoivent une confirmation JSON d'état opérationnel en moins de 50 millisecondes.
- **SC-003**: 100% des requêtes sollicitant une ressource inexistante aboutissent à une réponse HTTP 404 conviviale, avec 0% de divulgation de trace technique, chemin système ou pile d'appels.
- **SC-004**: Un nouveau contributeur peut configurer, démarrer le serveur et exécuter avec succès l'ensemble de la suite de vérification en moins de 5 minutes à partir du document `README.md`.
- **SC-005**: 100% des tests automatisés du socle réussissent lors de l'exécution de la suite de vérification.
- **SC-006**: 0 variable secrète ou sensible n'est enregistrée dans le référentiel de code versionné.

## Assumptions

- Le service s'exécute dans un environnement Node.js 20 LTS conforme aux prérequis du projet.
- Le port réseau configuré (par défaut 3000) est libre lors du démarrage du serveur en environnement local.
- Les utilisateurs accèdent au service via un navigateur ou un client HTTP standard compatible avec les spécifications HTTP/1.1+.
- La page d'accueil initiale sert de socle de validation technique et fonctionnelle et ne comporte pas à ce stade de travail de stylisation élaboré ni de composants graphiques complexes.
- Les fonctionnalités avancées (gestion des comptes, compétences, séances, minuteur dynamique côté client et base de données) sont expressément hors périmètre de ce socle initial.
