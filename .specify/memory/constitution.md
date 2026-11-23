<!--
SYNC IMPACT REPORT
==================
- Version change: Uninitialized (template) -> 1.0.0
- List of modified principles:
  * [PRINCIPLE_1_NAME] -> I. Simplicité et Frugalité (KISS, YAGNI, Dépendances Justifiées)
  * [PRINCIPLE_2_NAME] -> II. Tests Automatisés Obligatoires (NON-NÉGOCIABLE)
  * [PRINCIPLE_3_NAME] -> III. Rendu Serveur & Amélioration Progressive (SSR-First)
  * [PRINCIPLE_4_NAME] -> IV. Sécurité Défensive & Protection des Données (Security by Design)
  * [PRINCIPLE_5_NAME] -> V. Accessibilité Universelle & Standardisation Temporelle
  * [Added Principle] -> VI. Code Lisible & Documentation du « Pourquoi »
- Added sections:
  * Stack Technique & Contraintes d'Architecture (remplace [SECTION_2_NAME])
  * Workflow de Développement & Portes de Qualité (remplace [SECTION_3_NAME])
- Removed sections:
  * Aucun (tous les blocs du template ont été concrétisés)
- Follow-up TODOs:
  * Aucun (toutes les valeurs requises ont été définies sans placeholder différé)
-->

# 20H-FLE Constitution

## Core Principles

### I. Simplicité et Frugalité (KISS, YAGNI, Dépendances Justifiées)
- L'architecture et le code DOIVENT appliquer strictement les principes KISS (*Keep It Simple, Stupid*) et YAGNI (*You Aren't Gonna Need It*). Aucune fonctionnalité spéculative, couche d'abstraction prématurée ou complexité non directement requise ne sera acceptée.
- Les dépendances externes DOIVENT être réduites au strict minimum. Toute nouvelle dépendance ajoutée au projet DOIT faire l'objet d'une justification explicite et documentée (besoin avéré, coût/bénéfice, impact sur la sécurité et le bundle).
- *Raison d'être* : En tant que projet pédagogique, la clarté et la maîtrise complète du code source prévalent sur l'accumulation de bibliothèques tierces opaques.

### II. Tests Automatisés Obligatoires (NON-NÉGOCIABLE)
- Toute logique métier, modèle de données, route d'application et calcul DOIT être couverte par des tests automatisés.
- L'ensemble des tests DOIT s'exécuter via la commande unique `npm test` (utilisant Vitest pour les tests unitaires/intégration et Supertest pour les tests HTTP).
- Tout bogue corrigé ou nouvelle fonctionnalité DOIT être accompagné de tests automatisés démontrant le comportement attendu et empêchant toute régression.
- *Raison d'être* : L'intégrité fonctionnelle et la confiance dans le code reposent sur une suite de tests rapide, reproductible et exécutable en environnement local comme en CI.

### III. Rendu Serveur & Amélioration Progressive (SSR-First)
- Le rendu des pages DOIT être effectué intégralement côté serveur (SSR via Express).
- Le JavaScript/TypeScript côté navigateur est STRICTEMENT RESTREINT : il n'est autorisé que pour le minuteur interactif et les comportements dynamiques strictement impossibles à exécuter côté serveur.
- Tout script client DOIT être écrit en TypeScript sans framework frontend, compilé localement en un fichier statique unique servi en local, sans aucun recours à des CDN externes.
- Amélioration progressive obligatoire : la consultation du site et la saisie manuelle du temps de pratique DOIVENT rester pleinement fonctionnelles et utilisables même si le JavaScript navigateur est désactivé ou indisponible.
- *Raison d'être* : Garantir la résilience, la légèreté, l'indépendance vis-à-vis d'infrastructures tierces et l'accès universel à l'application.

### IV. Sécurité Défensive & Protection des Données (Security by Design)
- Les mots de passe DOIVENT TOUJOURS être hachés avec un algorithme robuste et éprouvé (ex. bcrypt/argon2), JAMAIS stockés ou journalisés en clair.
- L'authentification et les sessions DOIVENT reposer sur des cookies sécurisés avec l'attribut `HttpOnly` (et les drapeaux `SameSite` et `Secure` appropriés).
- Une protection anti-CSRF DOIT être mise en place sur toutes les routes modifiant un état (POST, PUT, DELETE).
- Toutes les entrées utilisateurs DOIVENT être validées et assainies rigoureusement côté serveur avant tout traitement ou stockage.
- Zéro secret dans le dépôt Git : aucune clé, mot de passe ou jeton de configuration ne doit être commité. La configuration DOIT passer par des variables d'environnement, avec un modèle exhaustif maintenu dans `.env.example`.
- La base de données SQLite locale (`*.db`, `*.sqlite`) ainsi que les fichiers générés localement DOIVENT être ignorés par Git (`.gitignore`).
- *Raison d'être* : Protéger les données utilisateurs et prémunir le projet contre les vulnérabilités web courantes dès sa conception.

### V. Accessibilité Universelle & Standardisation Temporelle
- L'interface utilisateur DOIT respecter les bonnes pratiques d'accessibilité (a11y) : formulaires avec libellés explicites (`<label for="...">`), contrastes visuels conformes et structure sémantique HTML irréprochable.
- La navigation intégrale DOIT être possible et fluide au clavier (ordre de focus logique, indicateurs de focus visibles).
- Les données et états essentiels (statut du minuteur, cumuls d'heures) DOIVENT être restitués clairement aux technologies d'assistance (lecteurs d'écran) sans déclencher d'annonces répétitives ou de pollutions sonores.
- Toutes les dates et estampilles temporelles DOIVENT être enregistrées et traitées en UTC en base de données, puis formatées selon le fuseau horaire et la locale de l'utilisateur à l'affichage.
- *Raison d'être* : Garantir une expérience inclusive pour tous les apprenants et une cohérence temporelle sans ambiguïté.

### VI. Code Lisible & Documentation du « Pourquoi »
- Tout le code applicatif DOIT être écrit en TypeScript avec le mode strict activé (`strict: true`).
- Le code source DOIT privilégier la lisibilité, l'expressivité et des noms explicites.
- Les commentaires de code DOIVENT expliquer le « pourquoi » (raisons métier, contraintes techniques, choix d'architecture sous-jacents) plutôt que de paraphraser le « comment » déjà évident dans la syntaxe.
- *Raison d'être* : Favoriser la maintenabilité, l'apprentissage continu et la collaboration sans ambiguïté sur les intentions du code.

## Stack Technique & Contraintes d'Architecture

- **Environnement d'exécution** : Node.js version 20 LTS (garantie via `.nvmrc` et `engines` dans `package.json`).
- **Langage** : TypeScript en mode strict (`strict: true`), sans `any` implicite ni contournement de typage non justifié.
- **Serveur web** : Express avec moteur de rendu côté serveur (SSR).
- **Persistance** : SQLite local avec accès typé, fichiers de base exclus de Git.
- **Tests** : Vitest pour les tests unitaires et d'intégration métier ; Supertest pour les tests d'intégration des routes HTTP.
- **Client Web** : Script TypeScript unique compilé statiquement en local, aucun framework (Vanilla DOM pur), aucun CDN externe.
- **Gestion des configurations** : `.env` local pour l'exécution, `.env.example` versionné servant de contrat pour les variables d'environnement requises.

## Workflow de Développement & Portes de Qualité

- **Vérification systématique avant livraison** :
  - `npm test` DOIT s'exécuter avec succès (100% des tests passants).
  - `npm run typecheck` DOIT s'exécuter sans aucune erreur de compilation TypeScript.
- **Revue des dépendances** : Tout ajout dans `dependencies` ou `devDependencies` DOIT comporter une justification écrite dans la PR / documentation associée.
- **Revue d'accessibilité et de dégradation gracieuse** :
  - Validation du fonctionnement complet en navigation clavier.
  - Validation du fonctionnement du formulaire de saisie et de consultation avec JavaScript désactivé.
- **Revue de sécurité** : Vérification de la présence des protections CSRF, de la validation d'entrée et de l'absence totale de secrets ou de données personnelles en clair.

## Governance

- **Primauté de la Constitution** : Cette constitution prévaut sur toute autre documentation, convention informelle ou décision ponctuelle de développement.
- **Processus d'amendement** :
  - Toute modification des principes, contraintes ou règles de gouvernance DOIT faire l'objet d'une proposition documentée et validée.
  - La version de la constitution est régie par le versionnement sémantique (SemVer) :
    - **MAJOR** : Suppression, affaiblissement ou modification incompatible d'un principe fondamental ou d'une contrainte architecturale.
    - **MINOR** : Ajout d'un nouveau principe, d'une nouvelle contrainte d'architecture ou expansion significative des directives.
    - **PATCH** : Clarifications rédactionnelles, corrections typographiques ou reformulations non sémantiques.
- **Contrôle de conformité** : Chaque contribution ou revue de code DOIT explicitement vérifier la conformité avec les principes établis dans ce document. Toute dérogation technique DOIT être formellement documentée avec justification et plan de remédiation.

**Version**: 1.0.0 | **Ratified**: 2026-11-23 | **Last Amended**: 2026-11-23
