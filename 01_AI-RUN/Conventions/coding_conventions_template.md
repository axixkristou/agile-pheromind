# Conventions de Codage du Projet : {{NOM_PROJET_PAR_PHEROMIND}}

**Version:** 1.0
**Date de Dernière Mise à Jour:** {{TIMESTAMP_PAR_PHEROMIND}}
**Maintenu par:** `@architecture-advisor-agent` (avec validation humaine)

## 1. Introduction

Ce document définit les standards et conventions de codage à suivre pour le projet `{{NOM_PROJET_PAR_PHEROMIND}}`. L'objectif est d'assurer la cohérence, la lisibilité, la maintenabilité et la qualité du code produit par tous les membres de l'équipe, y compris les assistants IA.

Le respect de ces conventions est **obligatoire**. Les revues de code (manuelles et assistées par IA) vérifieront leur application.

## 2. Principes Généraux de Codage

*   **Lisibilité avant tout:** Écrire du code que les autres (et votre futur vous) peuvent comprendre facilement.
*   **Simplicité (KISS - Keep It Simple, Stupid):** Préférer les solutions simples et directes aux solutions inutilement complexes.
*   **DRY (Don't Repeat Yourself):** Éviter la duplication de code en utilisant des fonctions, classes, ou composants réutilisables.
*   **SOLID (pour la programmation orientée objet, notamment en .NET):**
    *   **S**ingle Responsibility Principle (SRP)
    *   **O**pen/Closed Principle (OCP)
    *   **L**iskov Substitution Principle (LSP)
    *   **I**nterface Segregation Principle (ISP)
    *   **D**ependency Inversion Principle (DIP)
*   **YAGNI (You Ain't Gonna Need It):** N'implémenter que les fonctionnalités réellement nécessaires pour les exigences actuelles.
*   **Tests:** Tout nouveau code de logique métier ou de fonctionnalité significative doit être accompagné de tests unitaires et/ou d'intégration pertinents.
*   **Sécurité:** La sécurité doit être une préoccupation constante. Suivre les meilleures pratiques de codage sécurisé (voir section dédiée).

## 3. Conventions de Nommage

### 3.1. Général (.NET et Angular/TypeScript)

*   Utiliser des noms descriptifs et non ambigus.
*   Préférer l'anglais pour tous les identifiants (variables, fonctions, classes, etc.).
*   Éviter les abréviations excessives ou cryptiques.

### 3.2. Spécifique à C# / .NET

*   **Classes, Interfaces, Enums, Delegates, Records, Structs:** `PascalCase` (ex: `OrderService`, `ICustomerRepository`).
    *   Interfaces: Préfixer par `I` (ex: `IOrderService`).
*   **Méthodes (Publiques, Protégées, Internes):** `PascalCase` (ex: `CalculateTotalPrice()`, `IsValidOrder()`).
*   **Propriétés (Publiques, Protégées, Internes):** `PascalCase` (ex: `CustomerName`, `IsEnabled`).
*   **Constantes et Champs `readonly` statiques:** `PascalCase` (ex: `DefaultTimeout`, `MaxRetries`).
*   **Variables Locales et Paramètres de Méthodes:** `camelCase` (ex: `orderAmount`, `customerAddress`).
*   **Champs Privés:** `_camelCase` (préfixés par un underscore) (ex: `_connectionString`).
*   **Namespaces:** `PascalCase`, reflétant la structure des dossiers (ex: `SuperApp.Domain.Services`, `SuperApp.Infrastructure.DataAccess`).
*   **Fichiers `.cs`:** Doivent correspondre au nom de la classe publique principale qu'ils contiennent (ex: `OrderService.cs`).

### 3.3. Spécifique à TypeScript / Angular

*   **Classes, Interfaces, Enums, Types Alias:** `PascalCase` (ex: `ProductComponent`, `OrderData`, `UserRole`).
    *   Interfaces: Optionnel de préfixer par `I`, mais la cohérence est clé. Si utilisé, `IProduct`.
*   **Méthodes et Fonctions:** `camelCase` (ex: `calculateTotal()`, `getUserProfile()`).
*   **Propriétés et Variables:** `camelCase` (ex: `productName`, `isLoading`).
    *   Constantes (si exportées ou globales): `UPPER_SNAKE_CASE` (ex: `DEFAULT_TIMEOUT_MS`).
*   **Décorateurs Angular (`@Input`, `@Output`, etc.):** `camelCase`.
*   **Sélecteurs de Composants Angular:** `app-kebab-case` (ex: `selector: 'app-product-list'`).
*   **Fichiers:**
    *   Composants: `component-name.component.ts` (et `.html`, `.scss`, `.spec.ts`).
    *   Services: `service-name.service.ts` (et `.spec.ts`).
    *   Modules: `module-name.module.ts`.
    *   Pipes: `pipe-name.pipe.ts`.
    *   Directives: `directive-name.directive.ts`.
    *   Général: `kebab-case.ts` (ex: `validation-utils.ts`).

## 4. Formatage du Code

*   **Outils:** Utiliser les formateurs automatiques configurés pour le projet.
    *   **.NET:** Intégré à Visual Studio / Rider, potentiellement avec `.editorconfig`.
    *   **Angular/TypeScript:** Prettier (via configuration dans `package.json` et `.prettierrc`).
*   **Indentation:** [Choisir: Espaces (ex: 4) ou Tabulations]. Laisser le formateur gérer.
*   **Longueur de Ligne:** Maximum [ex: 120] caractères.
*   **Accolades:** [Choisir style: même ligne ou nouvelle ligne pour les blocs C# ; même ligne pour TypeScript/JS est courant].
*   **Imports:**
    *   **.NET:** `using` en haut du fichier, triés (d'abord `System`, puis externes, puis internes au projet).
    *   **TypeScript:** `import` en haut du fichier, triés (librairies externes, puis alias de chemin, puis relatifs).

## 5. Commentaires

*   **Quand commenter :**
    *   Pour expliquer la logique complexe ou non évidente ("Pourquoi" et non "Comment").
    *   Pour la documentation d'API publique (XML Docs en C#, TSDoc en TypeScript).
    *   Pour les `TODO:`, `FIXME:`, `NOTE:` avec nom/date et explication.
*   **Éviter les commentaires superflus** qui paraphrasent simplement le code.
*   **Style de Commentaire:**
    *   **.NET XML Documentation Comments:** Pour les types et membres publics/protégés.
        ```csharp
        /// <summary>
        /// Calculates the total price for an order.
        /// </summary>
        /// <param name="orderId">The ID of the order.</param>
        /// <returns>The calculated total price.</returns>
        /// <exception cref="OrderNotFoundException">Thrown if orderId is not found.</exception>
        public decimal CalculateOrderTotal(int orderId) { /* ... */ }
        ```
    *   **TypeScript TSDoc/JSDoc:** Pour les fonctions, classes, méthodes, interfaces exportées.
        ```typescript
        /**
         * Retrieves user details from the backend.
         * @param userId The unique identifier for the user.
         * @returns A Promise resolving to the UserProfile object.
         * @throws Error if the user is not found or network error.
         */
        async function fetchUserProfile(userId: string): Promise<UserProfile> { /* ... */ }
        ```

## 6. Gestion des Erreurs et Logging

### 6.1. Gestion des Erreurs

*   Utiliser les exceptions pour les conditions d'erreur exceptionnelles, pas pour le contrôle de flux normal.
*   Créer des exceptions personnalisées spécifiques au domaine si nécessaire (héritant de `System.Exception` en .NET).
*   Valider les entrées publiques (paramètres d'API, entrées utilisateur).
*   Dans Angular, gérer les erreurs des appels HTTP (ex: `HttpClient`) avec des opérateurs RxJS (`catchError`).

### 6.2. Logging

*   Utiliser une librairie de logging standardisée (ex: Serilog, NLog pour .NET ; une solution basée sur `console` avec des niveaux pour Angular, ou une librairie comme ngx-logger).
*   **Niveaux de Log:**
    *   `Verbose/Trace/Debug`: Pour le débogage détaillé.
    *   `Information`: Pour les événements normaux de l'application.
    *   `Warning`: Pour les situations potentiellement problématiques mais non bloquantes.
    *   `Error`: Pour les erreurs qui ont été gérées mais qui indiquent un problème.
    *   `Fatal/Critical`: Pour les erreurs graves qui empêchent l'application de fonctionner.
*   **Contenu des Logs:** Doivent être informatifs, inclure un timestamp, le contexte (ex: méthode, `userId`), et des données pertinentes (mais **JAMAIS de données sensibles en clair**).

## 7. Tests Unitaires et d'Intégration

*   **Frameworks:**
    *   **.NET:** [MSTest/NUnit/xUnit - à choisir et spécifier, ex: MSTest]. Utiliser Moq ou NSubstitute pour le mocking.
    *   **Angular:** [Jasmine/Karma/Jest - à choisir et spécifier, ex: Jasmine & Karma]. Utiliser `TestBed`, `jasmine.createSpyObj`.
*   **Conventions de Nommage des Tests:**
    *   **.NET:** `ClassNameTests.cs`, Méthodes: `MethodName_Scenario_ExpectedBehavior()`.
    *   **Angular:** `component-name.component.spec.ts`, `describe('ComponentName', ...)` et `it('should expected behavior when scenario', ...)`.
*   **Structure Arrange-Act-Assert (AAA) / Given-When-Then (GWT).**
*   Les tests doivent être indépendants et rapides.
*   Viser une couverture de code élevée pour la logique métier critique.

## 8. Conventions Spécifiques à la Stack

### 8.1. .NET / C#

*   Utilisation de LINQ pour les requêtes sur collections.
*   Utilisation d'async/await pour les opérations I/O-bound. Ne pas bloquer.
*   Utilisation de l'injection de dépendances (DI) fournie par ASP.NET Core.
*   Respecter les conventions d'Entity Framework Core pour les DbContexts, entités, migrations.
*   API Design: Suivre les conventions RESTful pour les APIs Web. Utiliser les attributs de routing et HTTP verb de manière standard.

### 8.2. Angular / TypeScript

*   **Modules:** Utiliser les `NgModule` (ou `standalone components` si Angular 14+) de manière logique pour organiser l'application.
*   **Composants:** Séparer la logique de présentation (template HTML) de la logique métier (classe TypeScript). Préférer les composants "dumb" (de présentation) et "smart" (conteneurs).
*   **Services:** Utiliser les services pour la logique métier partagée, les appels API, la gestion d'état.
*   **RxJS:** Utiliser RxJS et les Observables pour gérer les opérations asynchrones et les flux de données. Utiliser l'opérateur `async` pipe dans les templates. Se désabonner des observables pour éviter les fuites mémoire (`takeUntil`, `OnDestroy`).
*   **Lazy Loading:** Configurer le lazy loading pour les modules afin d'améliorer les performances initiales.
*   **State Management:** [Si une solution de gestion d'état globale est choisie (ex: NgRx, Akita, ou services avec BehaviorSubjects), la spécifier ici et ses conventions d'utilisation].
*   **Typage Fort:** Utiliser TypeScript de manière stricte. Éviter `any` autant que possible. Définir des interfaces et types clairs.

## 9. Sécurité

*   **OWASP Top 10:** Être conscient des vulnérabilités communes et les éviter.
*   **Validation des Entrées:** Valider toutes les données provenant de sources externes (utilisateurs, APIs tierces) côté client ET côté serveur.
*   **Authentification & Autorisation (.NET API):** Utiliser les mécanismes d'ASP.NET Core Identity / JWT. Appliquer les attributs `[Authorize]` de manière appropriée.
*   **Protection XSS (Angular):** Angular fournit une protection par défaut, mais être conscient des cas où la sanitization manuelle est nécessaire (ex: `[innerHTML]`).
*   **Gestion des Secrets:** **NE JAMAIS** stocker de secrets (clés API, connection strings) dans le code source. Utiliser Azure Key Vault, les variables d'environnement, ou les configurations sécurisées d'Azure App Service / AKS.
*   **Dépendances:** Mettre à jour régulièrement les dépendances pour corriger les vulnérabilités connues.

## 10. Conventions Git et Azure DevOps

*   **Branches:**
    *   `main` (ou `master`): Branche de production, stable.
    *   `develop`: Branche d'intégration principale.
    *   `feature/US<ID_US>-<description-courte>`: Pour le développement de nouvelles fonctionnalités.
    *   `fix/TASK<ID_TASK>-<description-courte>`: Pour les corrections de bugs.
    *   `release/vX.Y.Z`: Pour la préparation des releases.
*   **Messages de Commit:** Suivre **Conventional Commits** (voir `06_Finalize_Task_Commit.md`).
    *   Ex: `feat(auth): add Google social login option`
    *   Ex: `fix(order): correct calculation for discounted items`
*   **Pull Requests (PRs) dans Azure Repos:**
    *   Doivent être liées à une User Story ou Tâche dans Azure Boards.
    *   Description claire des changements, du problème résolu, et comment tester.
    *   Doivent passer tous les checks de la build de PR (linters, tests, analyse de code) avant de pouvoir être mergées.
    *   Au moins un autre développeur (ou Tech Lead) doit approuver la PR.
*   **Azure Boards:**
    *   Maintenir les statuts des US et Tâches à jour.
    *   Lier les commits et PRs aux éléments de travail.

## 11. Documentation du Code

*   **Commentaires:** Comme défini en section 5.
*   **README.md:** Chaque projet (.NET API, App Angular) doit avoir un `README.md` à sa racine expliquant son but, comment le configurer, le builder, et le lancer localement.
*   **Diagrammes:** Pour les architectures complexes ou les flux de données importants, utiliser Mermaid dans des fichiers .md pour créer des diagrammes.

## 12. Révision et Mise à Jour de ce Document

Ce document est vivant et sera révisé et mis à jour au besoin par `@architecture-advisor-agent` avec validation de l'équipe (typiquement lors des rétrospectives de sprint ou sur initiative du Tech Lead).

---