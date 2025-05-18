# Conventions de Codage du Projet [NomProjet]

**Version:** 1.0
**Date de dernière mise à jour:** {{timestamp}}
**Maintenu par:** @architecture-advisor-agent & Tech Lead

## 1. Introduction

Ce document définit les conventions de codage à suivre pour le projet [NomProjet]. L'objectif est d'assurer la lisibilité, la maintenabilité, la cohérence et la qualité du code produit par l'équipe et les agents IA. Toutes les contributions de code doivent adhérer à ces directives.

**Stack Technologique Principale:**
*   **Backend:** .NET Core (C#) - Architecture inspirée de DDD
*   **Frontend:** Angular (TypeScript)
*   **API:** RESTful principles

## 2. Conventions Générales (Applicables à .NET et Angular)

### 2.1. Nommage
*   **Classes, Interfaces, Enums, Types, Décorateurs Angular, Modules Angular:** `PascalCase`
    *   *Exemples .NET:* `OrderService`, `IProductRepository`, `OrderStatus`, `ProductDto`
    *   *Exemples Angular:* `ProductListComponent`, `AuthService`, `OrderStatusPipe`, `AppRoutingModule`
*   **Méthodes, Fonctions, Propriétés, Variables locales:** `camelCase`
    *   *Exemples .NET:* `calculateTotalPrice()`, `public string ProductName { get; set; }`, `int itemCount;`
    *   *Exemples Angular:* `fetchProducts()`, `productName: string;`, `isLoading = false;`
*   **Constantes et champs `readonly static` (.NET):** `PascalCase` ou `ALL_CAPS_SNAKE_CASE` (choisir une convention et s'y tenir, `PascalCase` est souvent préféré en C# moderne).
    *   *Exemple .NET:* `public const int MaxPageSize = 100;`, `public static readonly string DefaultCategory = "General";`
*   **Constantes TypeScript (Angular):** `ALL_CAPS_SNAKE_CASE`
    *   *Exemple Angular:* `export const DEFAULT_PAGE_SIZE = 20;`
*   **Noms de Fichiers:**
    *   **.NET (C#):** `PascalCase.cs` (ex: `OrderService.cs`)
    *   **Angular (TypeScript):** `kebab-case.type.ts` (ex: `product-list.component.ts`, `auth.service.ts`, `order.model.ts`)
    *   **Angular (Templates HTML):** `kebab-case.component.html`
    *   **Angular (Styles SCSS/CSS):** `kebab-case.component.scss`
*   **Préfixes/Suffixes Conventionnels:**
    *   **Interfaces (.NET & TypeScript):** Préfixe `I` (ex: `IOrderService`).
    *   **Classes de Service Angular:** Suffixe `Service` (ex: `ProductService`).
    *   **Composants Angular:** Suffixe `Component` (ex: `ProductListComponent`).
    *   **Pipes Angular:** Suffixe `Pipe` (ex: `OrderStatusPipe`).
    *   **Directives Angular:** Suffixe `Directive` (ex: `HighlightDirective`).
    *   **Modules Angular:** Suffixe `Module` (ex: `OrdersModule`).
    *   **DTOs (.NET):** Suffixe `Dto` (ex: `ProductDto`, `CreateOrderRequestDto`).
    *   **Méthodes Asynchrones (.NET):** Suffixe `Async` (ex: `GetOrderByIdAsync`).

### 2.2. Formatage
*   **Indentation:** 4 espaces (pas de tabulations).
*   **Accolades:** Style Allman (accolades sur leur propre ligne) pour C# ; style K&R (accolade ouvrante sur la même ligne) pour TypeScript/JavaScript.
    *   *Exemple C#:*
        ```csharp
        public class MyClass
        {
            public void MyMethod()
            {
                // code
            }
        }
        ```
    *   *Exemple TypeScript:*
        ```typescript
        export class MyClass {
          myMethod(): void {
            // code
          }
        }
        ```
*   **Longueur de Ligne Maximale:** 120 caractères (indicatif, chercher la lisibilité avant tout).
*   **Utilisation des `using` (.NET) / `import` (TypeScript):**
    *   Placer en haut du fichier.
    *   Trier alphabétiquement (les linters peuvent aider).
    *   Supprimer les `using`/`import` inutilisés.
*   **Linters et Formatteurs:**
    *   **.NET:** Utiliser les analyseurs Roslyn intégrés avec la configuration par défaut ou un `.editorconfig` projet. StyleCop facultatif si décidé par l'équipe.
    *   **Angular:** ESLint et Prettier avec les configurations standard du projet Angular.
    *   Tous les fichiers doivent être exempts d'erreurs de linter avant commit.

### 2.3. Commentaires
*   **Quand commenter :** Commenter le "pourquoi", pas le "comment" (sauf si le "comment" est particulièrement complexe ou non évident).
*   **Style de Commentaire :**
    *   **.NET:** Commentaires XML Docs (`///`) pour les API publiques (classes, méthodes, propriétés). Commentaires `//` pour les explications internes.
    *   **Angular (TypeScript):** TSDoc/JSDoc (`/** ... */`) pour les membres exportés. Commentaires `//` pour les explications internes.
*   **TODOs/FIXMEs:** Utiliser `// TODO: [Description]` ou `// FIXME: [Description]` avec une référence à une tâche Azure DevOps si possible (ex: `// TODO: Azure#45678 Refactor this for performance`).
*   Éviter les commentaires évidents ou les commentaires qui paraphrasent simplement le code.

### 2.4. Gestion des Erreurs et Logging
*   **Exceptions (.NET):**
    *   Utiliser des exceptions spécifiques plutôt que `System.Exception`.
    *   Lever les exceptions de manière appropriée dans la couche Application/Domain. Ne pas catcher les exceptions que l'on ne sait pas gérer.
    *   Les contrôleurs API doivent traduire les exceptions en codes de statut HTTP appropriés.
    *   Utiliser des blocs `try-catch-finally` judicieusement.
*   **Gestion des Erreurs (Angular):**
    *   Utiliser les opérateurs RxJS (`catchError`) pour gérer les erreurs des appels HTTP.
    *   Fournir un feedback utilisateur clair en cas d'erreur.
    *   Éviter les `console.error` en production sans mécanisme de logging centralisé.
*   **Logging:**
    *   Utiliser une librairie de logging standardisée (ex: Serilog pour .NET, `ngx-logger` ou custom service pour Angular).
    *   Logger les informations pertinentes (timestamps, niveau de log, contexte, message d'erreur, stack trace si applicable).
    *   Configurer les niveaux de log par environnement.
    *   Ne pas logger de données sensibles en clair.

## 3. Conventions Spécifiques au Backend .NET (Architecture DDD-like)

L'architecture backend est inspirée de DDD et typiquement structurée en couches :
*   **`ProjectName.Api` (API Layer / Presentation Layer):** Contrôleurs ASP.NET Core, DTOs, configuration API.
*   **`ProjectName.Application` (Application Layer):** Services d'application, commandes, requêtes (CQRS-like), logique d'orchestration, validation des entrées.
*   **`ProjectName.Domain` (Domain Layer):** Entités du domaine, agrégats, value objects, services de domaine, interfaces de repository, logique métier principale. Cœur du système, sans dépendances aux couches externes.
*   **`ProjectName.Infrastructure` (Infrastructure Layer):** Implémentations des repositories (ex: avec EF Core), services externes (email, paiements), accès aux données, configuration de la BDD.

### 3.1. API Layer (`ProjectName.Api`)
*   **Principes RESTful:**
    *   Utiliser les verbes HTTP correctement (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`).
    *   URLs basées sur les ressources, noms au pluriel (ex: `/api/orders`, `/api/products/{id}`).
    *   Utiliser les codes de statut HTTP de manière sémantique (200, 201, 204, 400, 401, 403, 404, 500).
    *   Versionnement de l'API via l'URL (ex: `/api/v1/orders`) ou headers.
    *   DTOs (Data Transfer Objects) spécifiques pour les requêtes et les réponses. Ne pas exposer directement les entités du domaine.
*   **Contrôleurs (`ControllerBase`):**
    *   Mince et focalisés sur la réception des requêtes HTTP, la validation de base des DTOs (Data Annotations ou FluentValidation), et la délégation aux services de la couche Application.
    *   Utiliser `[ApiController]` et `[Route("api/[controller]")]`.
    *   Retourner `ActionResult<T>` ou des types dérivés (`Ok<T>`, `CreatedAtActionResult`, `NotFoundResult`, etc.).
    *   Gérer la traduction des exceptions de la couche Application en réponses HTTP appropriées (via des filtres d'exception globaux ou localement).
*   **DTOs:**
    *   Placer dans un sous-dossier `Dtos` au sein du projet API ou dans un projet de contrats partagés.
    *   Utiliser les Data Annotations (`[Required]`, `[StringLength]`, etc.) pour la validation de base. Pour une validation plus complexe, envisager FluentValidation dans la couche Application.

### 3.2. Application Layer (`ProjectName.Application`)
*   **Services d'Application:**
    *   Orchestrent les opérations, coordonnent les repositories et les services de domaine.
    *   Ne contiennent pas de logique métier complexe (qui devrait être dans le Domaine).
    *   Peuvent utiliser un pattern CQRS-like avec des Commandes (pour les écritures) et des Queries (pour les lectures).
        *   Commandes: ex `CreateOrderCommand`, `UpdateProductCommand`. Handlers de commande.
        *   Queries: ex `GetOrderByIdQuery`, `GetAllProductsQuery`. Handlers de query.
    *   Valider les commandes/requêtes en utilisant FluentValidation.
    *   Gérer les transactions unitaires (Unit of Work) si nécessaire.
*   **Dépendances:** Dépend du Domaine. Ne doit pas dépendre de l'Infrastructure directement (utilise les interfaces de repository définies dans le Domaine).

### 3.3. Domain Layer (`ProjectName.Domain`)
*   **Entités et Agrégats:**
    *   Représentent les concepts clés du métier.
    *   Encapsulent la logique métier et les invariants.
    *   Les agrégats définissent des frontières de consistance.
    *   Utiliser des constructeurs et des méthodes pour modifier l'état, en s'assurant que les invariants sont toujours respectés.
    *   Les entités ne doivent avoir des setters publics que si c'est absolument nécessaire et contrôlé.
*   **Value Objects:** Objets immuables représentant des concepts descriptifs (ex: `Address`, `Money`).
*   **Services de Domaine:**
    *   Contiennent la logique métier qui ne trouve pas naturellement sa place dans une entité ou un agrégat (ex: opérations impliquant plusieurs agrégats).
*   **Interfaces de Repository (`IRepository<T>`):**
    *   Définissent les contrats pour la persistance des agrégats (ex: `IOrderRepository`, `IProductRepository`).
    *   Placées dans le Domaine pour inverser la dépendance vis-à-vis de l'Infrastructure (Dependency Inversion Principle).
*   **Événements de Domaine (Optionnel):** Pour signaler des changements significatifs dans le domaine.
*   **Aucune Dépendance Externe:** Cette couche ne doit dépendre d'aucun framework d'infrastructure (pas d'EF Core, pas d'ASP.NET Core ici).

### 3.4. Infrastructure Layer (`ProjectName.Infrastructure`)
*   **Implémentations des Repositories:**
    *   Implémentent les interfaces de repository définies dans le Domaine, en utilisant EF Core (ou autre ORM/DAL).
    *   Gèrent la traduction entre les entités du domaine et les modèles de persistance (si différents).
    *   Contiennent les requêtes de base de données (LINQ to Entities, Dapper).
*   **EF Core:**
    *   `DbContext` configuré ici.
    *   Configurations d'entités (Fluent API ou Data Annotations).
    *   Migrations.
*   **Services Externes:**
    *   Clients pour des services tiers (email, paiement, services Azure, etc.).
    *   Adaptateurs pour les MCPs.
*   **Dépendances:** Dépend du Domaine (pour implémenter les interfaces). Peut dépendre de librairies spécifiques (EF Core, Azure SDKs).

### 3.5. Tests (.NET)
*   **Projets de Test Séparés:** Un projet de test par projet de code (ex: `ProjectName.Api.Tests`, `ProjectName.Application.Tests`, `ProjectName.Domain.Tests`).
*   **Framework de Test:** [MSTest / NUnit / xUnit - à définir par l'équipe et stocker dans `memoryBank.toolingConfigurations.testFrameworks.dotnet`].
*   **Mocking:** Utiliser Moq ou NSubstitute.
*   **Couverture:** Viser une haute couverture pour les couches Domain et Application. Les contrôleurs API doivent être testés pour le routing et la validation de base.

## 4. Conventions Spécifiques au Frontend Angular

### 4.1. Structure des Modules et Composants
*   **Organisation par Fonctionnalité (Feature Modules):** Préférer une structure où chaque fonctionnalité métier a son propre module Angular.
*   **CoreModule:** Pour les services singletons et les composants utilisés une seule fois dans `AppComponent`.
*   **SharedModule:** Pour les composants, directives, et pipes réutilisables à travers plusieurs modules de fonctionnalité.
*   **Atomic Design (Conceptuel):** Organiser les composants au sein des modules/features en `atoms` (boutons, inputs), `molecules` (champ de recherche avec bouton), `organisms` (formulaire complet, carte produit), `templates`, et `pages`.
*   **Lazy Loading:** Utiliser le chargement paresseux pour les modules de fonctionnalité.

### 4.2. Composants
*   **Single Responsibility Principle:** Les composants doivent être petits et focalisés.
*   **Smart/Dumb Components (Container/Presentational):**
    *   **Smart Components (Containers):** Gèrent l'état, appellent les services, passent les données aux composants de présentation. Souvent les composants de page routés.
    *   **Dumb Components (Presentational):** Reçoivent des données via `@Input()`, émettent des événements via `@Output()`. Pas de dépendances à des services. Faciles à tester et à réutiliser.
*   **`ChangeDetectionStrategy.OnPush`:** Utiliser `OnPush` pour les composants de présentation afin d'améliorer les performances.
*   **Inputs/Outputs:** Utiliser `@Input()` et `@Output()` pour la communication parent-enfant. Éviter `@ViewChild` pour modifier l'état d'un enfant directement.
*   **Cycle de Vie (`OnInit`, `OnDestroy`, etc.):** Implémenter les interfaces de cycle de vie nécessaires. Désinscrire les observables dans `ngOnDestroy` pour éviter les fuites mémoire.

### 4.3. Services
*   Fournis dans `root` (`providedIn: 'root'`) pour les singletons, ou dans un module de fonctionnalité spécifique si leur portée est limitée.
*   Injecter les services via le constructeur.
*   Utiliser des services pour encapsuler la logique métier du frontend, les appels API, et la gestion d'état partagé (si pas de solution de state management dédiée).

### 4.4. Gestion d'État (State Management)
*   **Services RxJS Simples:** Pour un état local ou partagé simple, utiliser des `BehaviorSubject` ou `Subject` dans les services.
*   **Solutions Dédiées (Optionnel, si complexité le justifie):** NgRx, Akita, ou NGXS. La décision d'utiliser une telle librairie doit être un choix d'architecture conscient. Si non spécifié, commencer avec des services RxJS.
*   **Immuabilité:** Pratiquer l'immuabilité pour les données d'état afin de faciliter la détection de changements et le débogage.

### 4.5. Appels HTTP
*   Utiliser `HttpClientModule` et un service wrapper (ex: `ApiService`) pour centraliser les appels API.
*   Utiliser des `HttpInterceptor` pour la gestion globale des erreurs HTTP, l'ajout de headers d'authentification, etc.
*   Typer les requêtes et les réponses avec des interfaces/classes TypeScript.
*   Utiliser RxJS pour gérer les opérations asynchrones.

### 4.6. Routage
*   Utiliser `AppRoutingModule` pour les routes principales, et des modules de routage dédiés pour chaque feature module chargé paresseusement.
*   Utiliser des Route Guards pour protéger les routes (ex: `AuthGuard`).
*   Passer des données aux routes via les paramètres de route ou les résolveurs de route (`Resolve`).

### 4.7. Styles SCSS/CSS
*   Utiliser SCSS (ou CSS si préférence projet).
*   Styles encapsulés par composant (par défaut dans Angular).
*   Utiliser des variables SCSS/CSS pour les thèmes, couleurs, espacements (en lien avec `design_conventions.md` et Tailwind si utilisé).
*   Éviter les sélecteurs CSS trop spécifiques ou globaux (sauf dans `styles.scss` global).
*   Adhérer à une méthodologie comme BEM si pas de Tailwind CSS, ou utiliser les utilitaires Tailwind.

### 4.8. Tests (Angular)
*   **Framework:** [Jasmine/Karma/Jest - à définir par l'équipe et stocker dans `memoryBank.toolingConfigurations.testFrameworks.angular`].
*   **Tests Unitaires (.spec.ts):** Tester la logique des composants, services, pipes. Utiliser `TestBed` pour configurer l'environnement de test et mocker les dépendances.
*   **Tests d'Intégration (Composants):** Tester les interactions entre le template et la classe du composant.
*   **Tests E2E (Optionnel):** Utiliser Protractor (legacy) ou des solutions plus modernes comme Cypress ou Playwright (qui peuvent être pilotées par un agent Pheromind via le Browser Tools MCP).

## 5. Conventions Spécifiques à la Base de Données (MSSQL / Azure SQL)

### 5.1. Nommage
*   **Tables:** `PascalCase`, nom au pluriel (ex: `Orders`, `Products`).
*   **Colonnes:** `PascalCase` (ex: `OrderId`, `ProductName`, `DateCreated`).
*   **Clés Primaires:** `Id` ou `[TableName]Id` (ex: `OrderId`).
*   **Clés Étrangères:** `[ReferencedTableName]Id` (ex: `CustomerId` dans la table `Orders`).
*   **Procédures Stockées:** `usp_VerbNoun` (ex: `usp_GetOrderDetails`, `usp_CreateCustomer`).
*   **Vues:** `vw_Purpose` (ex: `vw_ActiveProducts`).
*   **Index:** `IX_[TableName]_[ColumnNames]` ou `UX_[TableName]_[ColumnNames]` pour unique.

### 5.2. Types de Données
*   Utiliser les types de données les plus appropriés et les plus petits possibles.
*   `NVARCHAR(MAX)` à utiliser avec parcimonie. Préférer des longueurs définies.
*   `DATETIME2` pour les dates/heures au lieu de `DATETIME`.
*   `DECIMAL` pour les montants financiers.

### 5.3. Contraintes
*   Définir des clés primaires sur toutes les tables.
*   Utiliser des clés étrangères pour maintenir l'intégrité référentielle.
*   Utiliser des contraintes `CHECK` et `DEFAULT` lorsque c'est pertinent.
*   Les colonnes `NOT NULL` doivent être la norme, sauf si `NULL` a une signification métier claire.

### 5.4. Procédures Stockées (SPs)
*   Les SPs doivent avoir une logique métier minimale. Préférer la logique dans la couche Application/Domain .NET.
*   Utiliser des SPs principalement pour des opérations complexes de lecture de données (reporting) ou des écritures atomiques si EF Core ne suffit pas.
*   Commenter clairement les SPs (paramètres, objectif, logique complexe).
*   Utiliser `SET NOCOUNT ON`.
*   Gestion des erreurs avec `TRY...CATCH`.

### 5.5. Indexation
*   Créer des index sur les clés étrangères.
*   Créer des index sur les colonnes fréquemment utilisées dans les clauses `WHERE`, `JOIN`, et `ORDER BY`.
*   Analyser les plans d'exécution pour identifier les besoins d'indexation.

## 6. Conventions de Commit Git
*   Adhérer à la spécification "Conventional Commits" (voir [https://www.conventionalcommits.org/](https://www.conventionalcommits.org/)).
*   Format: `type(scope): description`
    *   **Types:** `feat`, `fix`, `build`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf`, `test`.
    *   **Scope (optionnel):** Module ou partie de l'application concernée.
    *   **Description:** Concise, en impératif présent.
*   **Body (optionnel):** Motivation et contexte.
*   **Footer (optionnel):** `BREAKING CHANGE:`, `Resolves: Azure#ID_US`, `Closes: Azure#ID_Tache`.

## 7. Conventions pour Docker et AKS
*   **Dockerfiles:**
    *   Utiliser des builds multi-stages pour des images légères.
    *   Utiliser des images de base officielles et à jour.
    *   Minimiser le nombre de layers.
    *   Ne pas inclure de secrets dans les images.
*   **Manifestes Kubernetes (YAML):**
    *   Utiliser des noms de ressources clairs et cohérents.
    *   Définir les `requests` et `limits` pour CPU/mémoire.
    *   Utiliser des `ConfigMaps` et `Secrets` pour la configuration.
    *   Définir des `Probes` (liveness, readiness).
    *   Suivre les meilleures pratiques de sécurité pour les pods et conteneurs.
*   **Helm Charts (si utilisé):** Suivre les conventions de nommage et de structuration Helm.

## 8. Revue et Mise à Jour des Conventions
Ces conventions seront revues périodiquement par l'équipe (animée par `@architecture-advisor-agent` et le Tech Lead) et mises à jour au besoin. Les propositions de changement doivent être discutées et validées.

---