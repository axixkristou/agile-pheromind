# Conventions de Codage du Projet {{NOM_PROJET_PAR_PHEROMIND}}

**Version:** 1.0
**Date de dernière mise à jour:** {{TIMESTAMP_PAR_PHEROMIND}}
**Maintenu par:** @architecture-advisor-agent & Tech Lead

## 1. Introduction

Ce document définit les conventions de codage à suivre pour le projet `{{NOM_PROJET_PAR_PHEROMIND}}`. L'objectif est d'assurer la lisibilité, la maintenabilité, la cohérence et la qualité du code produit par l'équipe et les agents IA d'AgilePheromind. Toutes les contributions de code doivent adhérer à ces directives.

Ce document est une composante clé de la `memoryBank` du projet et sera utilisé par les agents IA (notamment `@developer-agent` et `@code-reviewer-assistant`) pour guider la génération et l'analyse du code.

**Stack Technologique Principale (rappel de `memoryBank.projectContext.techStack`):**
*   **Backend:** .NET Core (C#) - Architecture inspirée de DDD (API -> Application -> Domain <- Infrastructure)
*   **Frontend:** Angular (TypeScript)
*   **API:** Principes RESTful
*   **Base de Données:** MSSQL (Azure SQL)

## 2. Conventions Générales (Applicables à .NET et Angular)

### 2.1. Nommage
*   **Clarté et Intention:** Les noms doivent clairement refléter l'intention et la responsabilité de l'élément nommé. Éviter les abréviations obscures.
*   **Langue:** Anglais exclusivement pour tous les identifiants de code (variables, méthodes, classes, etc.).
*   **Casse (Casing):**
    *   **Classes, Interfaces, Enums, Types, Décorateurs Angular, Modules Angular:** `PascalCase`
        *   *Exemples .NET:* `OrderProcessingService`, `IProductRepository`, `PaymentStatus`, `ProductViewModel`
        *   *Exemples Angular:* `ProductListComponent`, `AuthenticationService`, `OrderTotalPipe`, `SharedComponentsModule`
    *   **Méthodes, Fonctions, Propriétés publiques/protégées, Variables locales:** `camelCase`
        *   *Exemples .NET:* `calculateFinalPrice()`, `public string CustomerName { get; set; }`, `decimal subTotal;`
        *   *Exemples Angular:* `loadAvailableProducts()`, `productDescription: string;`, `isDataLoading = false;`
    *   **Champs privés (.NET & TypeScript):** Préfixe `_` suivi de `camelCase` (ex: `_orderItems`, `_httpClient`).
    *   **Constantes et champs `readonly static` (.NET):** `PascalCase` (préféré) ou `ALL_CAPS_SNAKE_CASE`. Cohérence au sein du projet.
        *   *Exemple .NET:* `public const int DefaultRequestTimeoutMs = 5000;`, `public static readonly string ApiVersion = "v1";`
    *   **Constantes TypeScript (Angular):** `ALL_CAPS_SNAKE_CASE`
        *   *Exemple Angular:* `export const MAX_RETRIES = 3;`
*   **Noms de Fichiers:**
    *   **.NET (C#):** `PascalCase.cs` (ex: `OrderService.cs`, `IProductRepository.cs`).
    *   **Angular (TypeScript):** `kebab-case.type.ts` (ex: `product-list.component.ts`, `auth.service.ts`, `order.model.ts`, `user-roles.enum.ts`).
    *   **Angular (Templates HTML):** `kebab-case.component.html`.
    *   **Angular (Styles SCSS/CSS):** `kebab-case.component.scss`.
*   **Préfixes/Suffixes Conventionnels:**
    *   **Interfaces (.NET & TypeScript):** Préfixe `I` (ex: `IOrderService`, `IUser`).
    *   **Classes de Service Angular:** Suffixe `Service` (ex: `ProductDataService`).
    *   **Composants Angular:** Suffixe `Component` (ex: `ProductListComponent`).
    *   **Pipes Angular:** Suffixe `Pipe` (ex: `OrderStatusDisplayPipe`).
    *   **Directives Angular:** Suffixe `Directive` (ex: `AppHighlightDirective`).
    *   **Modules Angular:** Suffixe `Module` (ex: `OrdersFeatureModule`).
    *   **DTOs (.NET):** Suffixe `Dto` (ex: `ProductDetailDto`, `CreateOrderRequestDto`).
    *   **Requêtes/Commandes (CQRS-like .NET):** Suffixe `Query` / `Command` (ex: `GetOrderByIdQuery`, `SubmitOrderCommand`).
    *   **Méthodes Asynchrones (.NET):** Suffixe `Async` (ex: `GetOrderByIdAsync`, `SaveChangesAsync`).

### 2.2. Formatage
*   **Configuration Automatique:** Le projet doit être configuré pour utiliser **ESLint+Prettier** pour Angular/TypeScript et les **analyseurs Roslyn avec `.editorconfig`** pour .NET afin d'automatiser le formatage autant que possible.
*   **Indentation:** 4 espaces. Pas de tabulations.
*   **Accolades:**
    *   **.NET (C#):** Style Allman (accolades sur leur propre ligne).
    *   **Angular (TypeScript):** Style K&R / One True Brace Style (OTBS) (accolade ouvrante sur la même ligne que l'instruction).
*   **Longueur de Ligne Maximale:** Objectif de 120 caractères. La lisibilité prime.
*   **Imports/Usings:**
    *   Placer en haut du fichier.
    *   Trier (par les outils : alphabétiquement, puis `System` d'abord pour .NET).
    *   Supprimer les imports/usings inutilisés (les outils doivent le faire).
    *   Éviter les imports wildcard (`*`) sauf si absolument nécessaire et justifié.
*   **Espacement:** Utiliser les règles de Prettier / Roslyn Analyzers. En général, un espacement cohérent autour des opérateurs, après les virgules, avant les accolades ouvrantes.

### 2.3. Commentaires
*   **Objectif:** Expliquer le *pourquoi* du code (intention, décisions de design complexes), pas le *comment* (le code doit être auto-explicatif pour le comment).
*   **Style:**
    *   **.NET:** Commentaires XML Docs (`///`) pour toutes les API publiques (types, méthodes, propriétés, événements). Commentaires `//` pour les explications internes.
    *   **Angular (TypeScript):** TSDoc/JSDoc (`/** ... */`) pour tous les membres exportés (classes, interfaces, fonctions, propriétés, méthodes). Commentaires `//` pour les explications internes.
*   **TODOs/FIXMEs:** `// TODO (Azure#ID_Tache): [Description]` ou `// FIXME (Azure#ID_Tache): [Description du bug et pourquoi c'est un fix temporaire]`. Doivent être liés à une tâche dans Azure DevOps.
*   **Qualité:** Les commentaires doivent être bien écrits, en anglais, et maintenus à jour avec le code. Éviter les commentaires redondants ou obsolètes.

### 2.4. Gestion des Erreurs et Logging
*   **Exceptions (.NET):**
    *   Utiliser des types d'exception spécifiques et sémantiques (ex: `ArgumentNullException`, `InvalidOperationException`, exceptions personnalisées du domaine si pertinent). Éviter de catcher `System.Exception` sauf au plus haut niveau pour logging.
    *   Lever les exceptions avec des messages clairs et contextuels.
    *   La couche API (`ProjectName.Api`) est responsable de traduire les exceptions en codes de statut HTTP appropriés (via des filtres d'exception globaux).
*   **Gestion des Erreurs (Angular):**
    *   Utiliser les opérateurs RxJS (`catchError`, `retry`) pour gérer les erreurs des `HttpClient`.
    *   Afficher des messages d'erreur conviviaux à l'utilisateur. Ne pas exposer de détails techniques bruts.
    *   Utiliser un service de gestion d'erreurs global pour intercepter les erreurs non gérées et les logger.
*   **Logging (.NET & Angular):**
    *   Utiliser une librairie de logging structuré (ex: Serilog pour .NET, `ngx-logger` ou un service custom basé sur `console` pour Angular).
    *   Configurer les niveaux de log (DEBUG, INFO, WARN, ERROR, FATAL) par environnement.
    *   Inclure un ID de corrélation dans les logs pour tracer une requête à travers les couches/services.
    *   Logger les informations contextuelles utiles au débogage (mais PAS de données sensibles en clair comme mots de passe, tokens, PII).

## 3. Conventions Spécifiques au Backend .NET (Architecture DDD-like)

**Rappel de la Structure des Couches:**
*   `ProjectName.Api` (Présentation/API)
*   `ProjectName.Application` (Services d'Application, CQRS, Validation)
*   `ProjectName.Domain` (Cœur Métier, Entités, Agrégats, Interfaces de Repository)
*   `ProjectName.Infrastructure` (Persistance via EF Core, Services Externes)

### 3.1. API Layer (`ProjectName.Api`)
*   **Principes RESTful Stricts:**
    *   Utilisation correcte des verbes HTTP (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`).
    *   URLs basées sur les ressources, noms au pluriel (ex: `/api/v1/orders`, `/api/v1/products/{productId}`).
    *   Utilisation sémantique des codes de statut HTTP.
    *   Content Negotiation (supporter `application/json` par défaut).
    *   HATEOAS (liens hypermédia) pour les ressources liées, si pertinent.
    *   Versionnement de l'API (ex: `/api/v1/...`).
*   **Contrôleurs (`ControllerBase`):**
    *   Doivent être "minces". Déléguer la logique métier à la couche Application.
    *   Utiliser `[ApiController]`, `[Route]`, `[HttpGet]`, `[HttpPost]`, etc.
    *   Utiliser l'injection de dépendances pour les services Application.
    *   Valider les DTOs d'entrée (Data Annotations ou `ModelState.IsValid`).
    *   Retourner `ActionResult<T>` ou des types spécifiques (`OkObjectResult`, `CreatedAtRouteResult`, `NotFoundResult`, `BadRequestObjectResult`).
*   **DTOs (Data Transfer Objects):**
    *   Définis dans un projet partagé (ex: `ProjectName.Contracts`) ou dans le projet API.
    *   Suffixés par `Dto` (ex: `OrderSummaryDto`), `RequestDto` (ex: `CreateOrderRequestDto`), `ResponseDto` (ex: `OrderCreatedResponseDto`).
    *   Utiliser les Data Annotations pour la validation de base.

### 3.2. Application Layer (`ProjectName.Application`)
*   **Services d'Application / Handlers CQRS:**
    *   Orchestrent les cas d'usage. Peuvent utiliser MediatR pour implémenter les patterns Commande et Requête.
    *   Commandes (pour les écritures) : ne retournent généralement rien ou un ID. Gèrent la validation métier (ex: FluentValidation) et la coordination des unités de travail/transactions.
    *   Requêtes (pour les lectures) : retournent des DTOs de lecture (projection des données).
    *   Injectent les interfaces de repository (du Domaine) et autres services d'application/domaine.
*   **Validation:** Utiliser FluentValidation pour la validation complexe des commandes/requêtes.
*   **Mapping:** Utiliser AutoMapper (ou similaire) pour mapper entre Entités du Domaine et DTOs, ou entre DTOs de requête et Commandes.

### 3.3. Domain Layer (`ProjectName.Domain`)
*   **Pureté:** Ne doit avoir aucune dépendance vers des frameworks d'infrastructure (pas d'EF Core, pas d'ASP.NET Core). Uniquement du C# pur et des librairies "métier" si besoin.
*   **Entités et Agrégats:**
    *   Constructeurs privés/protégés, création via des factory methods statiques ou des méthodes de l'agrégat racine.
    *   Les setters de propriétés doivent être privés ou protégés, modifications d'état via des méthodes métier qui garantissent les invariants.
    *   Lever des exceptions de domaine spécifiques si un invariant est violé.
*   **Value Objects:** Immuables, égalité basée sur les valeurs.
*   **Services de Domaine:** Logique métier impliquant plusieurs agrégats ou qui n'appartient pas à une entité unique.
*   **Interfaces de Repository (`IRepository<T>`):** Définissent les contrats de persistance. Ex: `Task<Order?> GetByIdAsync(OrderId id);`, `Task AddAsync(Order order);`.
*   **Événements de Domaine (Optionnel):** Pour la communication découplée entre agrégats ou pour déclencher des side-effects.

### 3.4. Infrastructure Layer (`ProjectName.Infrastructure`)
*   **Implémentations des Repositories:**
    *   Utilisent EF Core. Injectent le `DbContext`.
    *   Contiennent la logique de requête EF Core (LINQ).
    *   Mappent les entités du domaine vers les entités EF Core (si différentes, bien que souvent identiques).
*   **Entity Framework Core:**
    *   `DbContext` dans ce projet. Configuration des entités via Fluent API dans `OnModelCreating` ou des classes `IEntityTypeConfiguration<T>`.
    *   Migrations gérées via `dotnet ef migrations add` et `dotnet ef database update`.
*   **Implémentations de Services Externes:** Clients pour Azure Services (Blobs, Queues, etc.), services d'email, etc.

### 3.5. Tests (.NET)
*   **Projets de Test:** `ProjectName.Domain.Tests`, `ProjectName.Application.Tests`, `ProjectName.Api.Tests`.
*   **Types de Tests:**
    *   **Domain:** Tests unitaires purs sur les entités, agrégats, value objects, services de domaine. Pas de mocks (ou très peu pour des dépendances pures).
    *   **Application:** Tests unitaires sur les services d'application/handlers CQRS. Mocker les repositories et services externes. Vérifier la logique d'orchestration, la validation, les événements de domaine levés.
    *   **API:** Tests d'intégration légers (utilisant `WebApplicationFactory`) pour tester les contrôleurs, le routing, la validation des DTOs, la sérialisation, et la réponse HTTP. Mocker la couche Application.
*   **Conventions de Nommage des Tests:** `MethodName_Scenario_ExpectedBehavior()`.

## 4. Conventions Spécifiques au Frontend Angular
*   **(Similaire à la section Angular du fichier `design_conventions_template.md` pour la structure des modules, composants, services, gestion d'état, HTTP, routage, mais avec un focus sur le code TypeScript et les patterns.)**
*   **TypeScript Strict:** Activer toutes les options strictes du compilateur TypeScript (`strict: true` dans `tsconfig.json`).
*   **Types et Interfaces:** Définir des types clairs pour les modèles de données, les props des composants, les payloads des services. Placer les modèles partagés dans un répertoire `core/models` ou par feature.
*   **RxJS:** Utiliser RxJS de manière idiomatique. Gérer les souscriptions (ex: `takeUntil(this.destroy$)` et `this.destroy$.next(); this.destroy$.complete();` dans `ngOnDestroy`). Préférer les pipes async dans les templates.
*   **Décorateurs:** Utiliser les décorateurs Angular (`@Component`, `@Injectable`, `@Input`, `@Output`, etc.) conformément à la documentation officielle.
*   **Modules Angular (`NgModule`):**
    *   Organiser en `FeatureModules` (lazy-loaded), `SharedModule` (pour composants/pipes/directives communs), `CoreModule` (pour services singletons, importé une seule fois dans `AppModule`).
    *   Déclarer les composants, directives, pipes dans le module approprié. Exporter ce qui doit être utilisé par d'autres modules.
*   **Tests (Angular):**
    *   Fichiers `.spec.ts` à côté des fichiers qu'ils testent.
    *   Utiliser `TestBed` pour configurer les modules de test.
    *   Mocker les dépendances (services, `HttpClient`).
    *   Tester la logique du composant, les interactions template-classe, les appels de service.
    *   Vérifier le rendu conditionnel et les bindings.

## 5. Conventions Spécifiques à la Base de Données (MSSQL / Azure SQL)
*   **(Similaire à la section DB du fichier `design_conventions_template.md` pour le nommage, types de données, contraintes, mais peut être étendu ici avec des règles sur l'écriture de T-SQL, l'optimisation des requêtes, etc.)**
*   **Transactions:** Utiliser des transactions explicites (`BEGIN TRANSACTION`, `COMMIT`, `ROLLBACK`) dans les SPs pour les opérations atomiques.
*   **Performance:** Éviter les `SELECT *`. Utiliser des `JOIN` explicites. S'assurer que les clauses `WHERE` sont SARGable (peuvent utiliser des index).
*   **Sécurité:** Ne pas utiliser de SQL dynamique construit à partir d'entrées utilisateur sans paramétrage strict pour éviter les injections SQL (généralement géré par EF Core, mais important pour les SPs ou requêtes brutes). Accorder les permissions minimales nécessaires aux utilisateurs de la base de données.

## 6. Conventions de Commit Git
*   Adhérer à "Conventional Commits" (voir `design_conventions_template.md` Section 6). Message doit inclure l'ID de la tâche Azure DevOps (ex: `feat(orders): add endpoint for order creation - Closes Azure#45678`).

## 7. Conventions pour Docker et AKS
*   **(Similaire à `design_conventions_template.md` Section 7).**
*   Les variables d'environnement pour les conteneurs doivent être gérées via les ConfigMaps/Secrets Kubernetes et injectées, pas hardcodées.

## 8. Revue et Mise à Jour des Conventions
Ce document est une référence vivante. L'agent `@architecture-advisor-agent` est chargé de proposer des mises à jour basées sur l'évolution du projet, les décisions de l'équipe, et les nouvelles bonnes pratiques. Toute modification majeure doit être validée par le Tech Lead. La version et la date de mise à jour doivent être actualisées.

---