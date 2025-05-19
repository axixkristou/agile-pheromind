# Project {{PROJECT_NAME_FROM_PHEROMONE_EN}} Coding Conventions

**Version:** 1.0
**Last Updated:** {{TIMESTAMP_BY_PHEROMIND_AGENT}}
**Maintained by:** @architecture-advisor-agent & Tech Lead

## 1. Introduction

This document defines the coding conventions to be followed for the `{{PROJECT_NAME_FROM_PHEROMONE_EN}}` project. The goal is to ensure readability, maintainability, consistency, and quality of the code produced by the development team and AgilePheromind AI agents. All code contributions must adhere to these guidelines.

This document is a key component of the project's `memoryBank` (referenced via `documentationRegistry`) and will be used by AI agents (notably `@developer-agent` and `@code-reviewer-assistant`) to guide English code generation and analysis.

**Primary Technology Stack (reminder from `memoryBank.projectContext.techStack`):**
*   **Backend:** .NET Core (C#) - DDD-inspired architecture (API -> Application -> Domain <- Infrastructure)
*   **Frontend:** Angular (TypeScript)
*   **API:** RESTful principles
*   **Database:** MSSQL (Azure SQL)

## 2. General Conventions (Applicable to .NET and Angular)

### 2.1. Naming
*   **Clarity and Intent:** Names must clearly reflect the intent and responsibility of the named element. Avoid obscure abbreviations. All identifiers in English.
*   **Language:** English exclusively for all code identifiers.
*   **Casing:**
    *   **Classes, Interfaces, Enums, Types, Angular Decorators, Angular Modules:** `PascalCase`
        *   *.NET Examples:* `OrderProcessingService`, `IProductRepository`, `PaymentStatus`, `ProductViewModel`
        *   *Angular Examples:* `ProductListComponent`, `AuthenticationService`, `OrderTotalPipe`, `SharedComponentsModule`
    *   **Methods, Functions, Public/Protected Properties, Local Variables:** `camelCase`
        *   *.NET Examples:* `calculateFinalPrice()`, `public string CustomerName { get; set; }`, `decimal subTotal;`
        *   *Angular Examples:* `loadAvailableProducts()`, `productDescription: string;`, `isDataLoading = false;`
    *   **Private Fields (.NET & TypeScript):** Prefix `_` followed by `camelCase` (e.g., `_orderItems`, `_httpClient`).
    *   **Constants and `readonly static` fields (.NET):** `PascalCase` (preferred) or `ALL_CAPS_SNAKE_CASE`. Project consistency is key. Default to `PascalCase`.
        *   *.NET Example:* `public const int DefaultRequestTimeoutMs = 5000;`, `public static readonly string ApiVersion = "v1";`
    *   **TypeScript Constants (Angular):** `ALL_CAPS_SNAKE_CASE`
        *   *Angular Example:* `export const MAX_RETRIES = 3;`
*   **File Names (All English):**
    *   **.NET (C#):** `PascalCase.cs` (e.g., `OrderService.cs`, `IProductRepository.cs`).
    *   **Angular (TypeScript):** `kebab-case.type.ts` (e.g., `product-list.component.ts`, `auth.service.ts`, `order.model.ts`, `user-roles.enum.ts`).
    *   **Angular (HTML Templates):** `kebab-case.component.html`.
    *   **Angular (SCSS/CSS Styles):** `kebab-case.component.scss`.
*   **Conventional Prefixes/Suffixes (English):**
    *   **Interfaces (.NET & TypeScript):** `I` prefix (e.g., `IOrderService`, `IUser`).
    *   **Angular Service Classes:** `Service` suffix (e.g., `ProductDataService`).
    *   **Angular Components:** `Component` suffix (e.g., `ProductListComponent`).
    *   **Angular Pipes:** `Pipe` suffix (e.g., `OrderStatusDisplayPipe`).
    *   **Angular Directives:** `Directive` suffix (e.g., `AppHighlightDirective`).
    *   **Angular Modules:** `Module` suffix (e.g., `OrdersFeatureModule`).
    *   **.NET DTOs:** `Dto` suffix (e.g., `ProductDetailDto`, `CreateOrderRequestDto`).
    *   **.NET CQRS-like Requests/Commands:** `Query` / `Command` suffix (e.g., `GetOrderByIdQuery`, `SubmitOrderCommand`).
    *   **.NET Asynchronous Methods:** `Async` suffix (e.g., `GetOrderByIdAsync`, `SaveChangesAsync`).

### 2.2. Formatting
*   **Automated Configuration:** Project MUST use **ESLint+Prettier** for Angular/TypeScript and **Roslyn Analyzers with `.editorconfig`** for .NET.
*   **Indentation:** 4 spaces. No tabs.
*   **Braces:** .NET (C#): Allman style. Angular (TypeScript): K&R / OTBS.
*   **Maximum Line Length:** Target 120 characters. Readability first.
*   **Imports/Usings:** Top of file. Sorted by tools. Unused removed by tools. Avoid wildcards.
*   **Spacing:** Governed by Prettier / Roslyn Analyzers.

### 2.3. Comments (English)
*   **Purpose:** Explain the *why* (intent, complex decisions), not the *how* (code should be self-explanatory).
*   **Style:** .NET: XML Docs (`///`) for public APIs. `//` for internal. Angular (TypeScript): TSDoc/JSDoc (`/** ... */`) for exported members. `//` for internal.
*   **TODOs/FIXMEs:** `// TODO (Azure#Task_ID): [English Description]` or `// FIXME (Azure#Task_ID): [English Bug Description]`. Must link to ADO task.
*   **Quality:** Well-written English, up-to-date. Avoid redundant/obsolete comments.

### 2.4. Error Handling and Logging (English Logs/Messages)
*   **Exceptions (.NET):** Specific types. Throw appropriately. API layer translates to HTTP status codes.
*   **Error Handling (Angular):** RxJS (`catchError`, `retry`). User-friendly (non-technical) error messages displayed to user (can be localized by UI component if needed, but error itself logged in English). Global error handler.
*   **Logging (.NET & Angular):** Structured logging (e.g., Serilog for .NET, `ngx-logger` for Angular). Configurable levels. Correlation IDs. Log relevant context. **NO sensitive data in clear text.** All log messages written by code should be in English.

## 3. .NET Backend Specific Conventions (DDD-inspired Architecture - All English Identifiers)

**Layer Structure Reminder:** `ProjectName.Api`, `ProjectName.Application`, `ProjectName.Domain`, `ProjectName.Infrastructure`.

### 3.1. API Layer (`ProjectName.Api`)
*   **Strict RESTful Principles:** HTTP verbs, resource-based URLs (plural English nouns), semantic HTTP status codes, Content Negotiation (`application/json`), HATEOAS (optional), API Versioning.
*   **Controllers (`ControllerBase`):** Thin. Delegate to Application layer. Use `[ApiController]`, `[Route]`, `[HttpGet]`, etc. DI for App services. Validate input DTOs. Return `ActionResult<T>`.
*   **DTOs (English Naming):** In shared contracts or API project. `Dto`, `RequestDto`, `ResponseDto` suffixes. Data Annotations for basic validation.

### 3.2. Application Layer (`ProjectName.Application`)
*   **Application Services / CQRS Handlers (English Naming):** Orchestrate use cases. MediatR for Command/Query. Commands (writes): return void/ID, validation (FluentValidation), Unit of Work. Queries (reads): return read DTOs. Inject Domain repository interfaces.
*   **Validation (English Messages in Validators):** FluentValidation for complex validation.
*   **Mapping (English Configs):** AutoMapper (or similar) for Entity-DTO, DTO-Command mapping.

### 3.3. Domain Layer (`ProjectName.Domain` - All English Identifiers)
*   **Purity:** No infrastructure framework dependencies.
*   **Entities and Aggregates (English Naming):** Private/protected constructors, creation via static factories/aggregate root methods. State changes via business methods ensuring invariants. Throw specific English domain exceptions.
*   **Value Objects (English Naming):** Immutable, value-based equality.
*   **Domain Services (English Naming):** Business logic spanning aggregates.
*   **Repository Interfaces (`IRepository<T>`, English Naming):** Define persistence contracts.
*   **Domain Events (English Naming, Optional):** Decoupled communication.

### 3.4. Infrastructure Layer (`ProjectName.Infrastructure`)
*   **Repository Implementations (English Naming):** Use EF Core. Inject `DbContext`. EF Core query logic (LINQ). Map domain entities to persistence models.
*   **Entity Framework Core:** `DbContext` here. Fluent API or `IEntityTypeConfiguration<T>`. Migrations.
*   **External Service Implementations (English Naming):** Clients for Azure Services, email, etc. MCP Adapters.

### 3.5. .NET Testing (English Test Names & Comments)
*   **Separate Test Projects.**
*   **Test Types:** Domain (pure unit), Application (unit, mock repos/external), API (lightweight integration, mock App layer).
*   **Test Naming:** `MethodName_Scenario_ExpectedBehavior()`.
*   **Test Framework:** `{{memoryBank.toolingConfigurations.testFrameworks.dotnet}}`.
*   **Mocking Framework:** `{{Project_defined_NET_mocking_framework_en}}`.

## 4. Angular Frontend Specific Conventions (All English Identifiers)

*   **TypeScript Strict Mode:** `strict: true`.
*   **Types and Interfaces (English Naming):** Clear types for models, props, payloads. Shared models in `core/models` or by feature.
*   **RxJS:** Idiomatic use. Manage subscriptions (`takeUntil(this.destroy$)`). Prefer `async` pipe.
*   **Angular Decorators:** According to official docs.
*   **Angular Modules (`NgModule`, English Naming):** `FeatureModules` (lazy-loaded), `SharedModule`, `CoreModule`.
*   **Component Structure:** Follow Atomic Design principles (atoms, molecules, organisms, templates, pages) if adopted. Smart/Dumb component pattern. `ChangeDetectionStrategy.OnPush` for dumb components.
*   **Testing (Angular, English Test Names & Comments):** `.spec.ts` files. `TestBed`. Mock dependencies. Test component logic, template-class interaction, service calls. Test Framework: `{{memoryBank.toolingConfigurations.testFrameworks.angular}}`.

## 5. Database Specific Conventions (MSSQL / Azure SQL - All English Identifiers)

*   **Naming:** Tables (`PascalCase`, plural), Columns (`PascalCase`), PKs (`Id` or `[TableName]Id`), FKs (`[ReferencedTableName]Id`), SPs (`usp_VerbNoun_en`), Views (`vw_Purpose_en`), Indexes (`IX_TableName_ColumnNames`).
*   **Data Types:** Appropriate and smallest possible. `DATETIME2`. `DECIMAL` for currency.
*   **Constraints:** PKs, FKs, `CHECK`, `DEFAULT`. `NOT NULL` as default.
*   **Stored Procedures (English Comments):** Minimal business logic. Primarily for complex reads or atomic writes if EF Core insufficient. `SET NOCOUNT ON`. `TRY...CATCH`.
*   **Indexing:** On FKs, frequently used `WHERE`/`JOIN`/`ORDER BY` columns.

## 6. Git Commit Conventions (English Messages)
*   Adhere to "Conventional Commits". Message MUST include ADO Task ID: `type(scope): english description - Closes Azure#Task_ID`.

## 7. Docker and AKS Conventions (English file names and comments)
*   **Dockerfiles:** Multi-stage builds, official up-to-date base images, minimize layers, no secrets.
*   **Kubernetes Manifests (YAML):** Clear, consistent English resource names. Define requests/limits. Use ConfigMaps/Secrets. Define Probes.
*   **Helm Charts (if used):** Follow Helm conventions.

## 8. Review and Update of Conventions
This English document is a living reference. `@architecture-advisor-agent` proposes updates based on project evolution, team decisions, and new best practices. Major changes validated by Tech Lead. Version and update date maintained.

---