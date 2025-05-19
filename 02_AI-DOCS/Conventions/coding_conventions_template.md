# Project {{PROJECT_NAME_FROM_PHEROMONE}} Coding Conventions

**Version:** 1.0
**Last Updated:** {{TIMESTAMP_BY_PHEROMIND_AGENT}}
**Maintained by:** @architecture-advisor-agent & Tech Lead

## 1. Introduction

This document defines the coding conventions to be followed for the `{{PROJECT_NAME_FROM_PHEROMONE}}` project. The goal is to ensure readability, maintainability, consistency, and quality of the code produced by the development team and AgilePheromind AI agents. All code contributions must adhere to these guidelines.

This document is a key component of the project's `memoryBank` and will be used by AI agents (notably `@developer-agent` and `@code-reviewer-assistant`) to guide code generation and analysis.

**Primary Technology Stack (reminder from `memoryBank.projectContext.techStack`):**
*   **Backend:** .NET Core (C#) - DDD-inspired architecture (API -> Application -> Domain <- Infrastructure)
*   **Frontend:** Angular (TypeScript)
*   **API:** RESTful principles
*   **Database:** MSSQL (Azure SQL)

## 2. General Conventions (Applicable to .NET and Angular)

### 2.1. Naming
*   **Clarity and Intent:** Names must clearly reflect the intent and responsibility of the named element. Avoid obscure abbreviations.
*   **Language:** English exclusively for all code identifiers (variables, methods, classes, etc.).
*   **Casing:**
    *   **Classes, Interfaces, Enums, Types, Angular Decorators, Angular Modules:** `PascalCase`
        *   *.NET Examples:* `OrderProcessingService`, `IProductRepository`, `PaymentStatus`, `ProductViewModel`
        *   *Angular Examples:* `ProductListComponent`, `AuthenticationService`, `OrderTotalPipe`, `SharedComponentsModule`
    *   **Methods, Functions, Public/Protected Properties, Local Variables:** `camelCase`
        *   *.NET Examples:* `calculateFinalPrice()`, `public string CustomerName { get; set; }`, `decimal subTotal;`
        *   *Angular Examples:* `loadAvailableProducts()`, `productDescription: string;`, `isDataLoading = false;`
    *   **Private Fields (.NET & TypeScript):** Prefix `_` followed by `camelCase` (e.g., `_orderItems`, `_httpClient`).
    *   **Constants and `readonly static` fields (.NET):** `PascalCase` (preferred) or `ALL_CAPS_SNAKE_CASE`. Maintain consistency within the project.
        *   *.NET Example:* `public const int DefaultRequestTimeoutMs = 5000;`, `public static readonly string ApiVersion = "v1";`
    *   **TypeScript Constants (Angular):** `ALL_CAPS_SNAKE_CASE`
        *   *Angular Example:* `export const MAX_RETRIES = 3;`
*   **File Names:**
    *   **.NET (C#):** `PascalCase.cs` (e.g., `OrderService.cs`, `IProductRepository.cs`).
    *   **Angular (TypeScript):** `kebab-case.type.ts` (e.g., `product-list.component.ts`, `auth.service.ts`, `order.model.ts`, `user-roles.enum.ts`).
    *   **Angular (HTML Templates):** `kebab-case.component.html`.
    *   **Angular (SCSS/CSS Styles):** `kebab-case.component.scss`.
*   **Conventional Prefixes/Suffixes:**
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
*   **Automated Configuration:** The project MUST be configured to use **ESLint+Prettier** for Angular/TypeScript and **Roslyn Analyzers with `.editorconfig`** for .NET to automate formatting as much as possible.
*   **Indentation:** 4 spaces. No tabs.
*   **Braces:**
    *   **.NET (C#):** Allman style (braces on their own line).
    *   **Angular (TypeScript):** K&R / One True Brace Style (OTBS) (opening brace on the same line as the statement).
*   **Maximum Line Length:** Target 120 characters. Readability is paramount.
*   **Imports/Usings:**
    *   Place at the top of the file.
    *   Sort (by tools: alphabetically, then `System` first for .NET).
    *   Remove unused imports/usings (tools should do this).
    *   Avoid wildcard imports (`*`) unless absolutely necessary and justified.
*   **Spacing:** Use Prettier / Roslyn Analyzer rules. Generally, consistent spacing around operators, after commas, before opening braces.

### 2.3. Comments
*   **Purpose:** Explain the *why* of the code (intent, complex design decisions), not the *how* (code should be self-explanatory for the how).
*   **Style:**
    *   **.NET:** XML Docs (`///`) for all public APIs (types, methods, properties, events). `//` comments for internal explanations.
    *   **Angular (TypeScript):** TSDoc/JSDoc (`/** ... */`) for all exported members. `//` comments for internal explanations.
*   **TODOs/FIXMEs:** `// TODO (Azure#Task_ID): [Description]` or `// FIXME (Azure#Task_ID): [Description of bug and why it's a temporary fix]`. Must be linked to a task in Azure DevOps.
*   **Quality:** Comments must be well-written, in English, and kept up-to-date with the code. Avoid redundant or obsolete comments.

### 2.4. Error Handling and Logging
*   **Exceptions (.NET):**
    *   Use specific, semantic exception types (e.g., `ArgumentNullException`, `InvalidOperationException`, custom domain exceptions if relevant). Avoid catching `System.Exception` except at the highest level for logging.
    *   Throw exceptions appropriately in the Application/Domain layer. Do not catch exceptions you cannot handle.
    *   API controllers must translate exceptions into appropriate HTTP status codes (via global exception filters).
*   **Error Handling (Angular):**
    *   Use RxJS operators (`catchError`, `retry`) to handle `HttpClient` errors.
    *   Display user-friendly error messages. Do not expose raw technical details.
    *   Use a global error handling service to catch unhandled errors and log them.
*   **Logging (.NET & Angular):**
    *   Use a standardized structured logging library (e.g., Serilog for .NET, `ngx-logger` or a custom console-based service for Angular).
    *   Configure log levels (DEBUG, INFO, WARN, ERROR, FATAL) per environment.
    *   Include a correlation ID in logs to trace requests across layers/services.
    *   Log relevant contextual information useful for debugging (but NO sensitive data like passwords, tokens, PII in clear text).

## 3. .NET Backend Specific Conventions (DDD-inspired Architecture)

**Layer Structure Reminder:**
*   `ProjectName.Api` (Presentation/API)
*   `ProjectName.Application` (Application Services, CQRS, Validation)
*   `ProjectName.Domain` (Business Core, Entities, Aggregates, Repository Interfaces)
*   `ProjectName.Infrastructure` (Persistence via EF Core, External Services)

### 3.1. API Layer (`ProjectName.Api`)
*   **Strict RESTful Principles:**
    *   Correct use of HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`).
    *   Resource-based URLs, plural nouns (e.g., `/api/v1/orders`, `/api/v1/products/{productId}`).
    *   Semantic use of HTTP status codes.
    *   Content Negotiation (support `application/json` by default).
    *   HATEOAS for related resources, if relevant.
    *   API Versioning (e.g., `/api/v1/...`).
*   **Controllers (`ControllerBase`):**
    *   Keep them "thin". Delegate business logic to Application layer.
    *   Use `[ApiController]`, `[Route]`, `[HttpGet]`, `[HttpPost]`, etc.
    *   Use Dependency Injection for Application services.
    *   Validate input DTOs (Data Annotations or `ModelState.IsValid`).
    *   Return `ActionResult<T>` or specific types (`OkObjectResult`, `CreatedAtRouteResult`, etc.).
*   **DTOs (Data Transfer Objects):**
    *   Define in a shared contracts project (e.g., `ProjectName.Contracts`) or API project.
    *   Suffix with `Dto`, `RequestDto`, `ResponseDto`.
    *   Use Data Annotations for basic validation.

### 3.2. Application Layer (`ProjectName.Application`)
*   **Application Services / CQRS Handlers:**
    *   Orchestrate use cases. May use MediatR for Command/Query patterns.
    *   Commands (for writes): typically return void or an ID. Handle business validation (e.g., FluentValidation) and coordinate Unit of Work/transactions.
    *   Queries (for reads): return read DTOs (data projections).
    *   Inject repository interfaces (from Domain) and other app/domain services.
*   **Validation:** Use FluentValidation for complex validation of commands/queries.
*   **Mapping:** Use AutoMapper (or similar) for mapping between Domain Entities and DTOs, or Request DTOs and Commands.

### 3.3. Domain Layer (`ProjectName.Domain`)
*   **Purity:** No dependencies on infrastructure frameworks (no EF Core, no ASP.NET Core). Pure C# and business-related libraries only.
*   **Entities and Aggregates:**
    *   Private/protected constructors, creation via static factory methods or aggregate root methods.
    *   Property setters should be private or protected; state changes via business methods ensuring invariants.
    *   Throw specific domain exceptions if invariants are violated.
*   **Value Objects:** Immutable, value-based equality.
*   **Domain Services:** Business logic involving multiple aggregates or not fitting a single entity.
*   **Repository Interfaces (`IRepository<T>`):** Define persistence contracts. E.g., `Task<Order?> GetByIdAsync(OrderId id);`.
*   **Domain Events (Optional):** For decoupled communication.

### 3.4. Infrastructure Layer (`ProjectName.Infrastructure`)
*   **Repository Implementations:**
    *   Use EF Core. Inject `DbContext`.
    *   Contain EF Core query logic (LINQ).
    *   Map domain entities to persistence models (if different).
*   **Entity Framework Core:**
    *   `DbContext` configured here. Entity configurations (Fluent API or `IEntityTypeConfiguration<T>`). Migrations.
*   **External Service Implementations:** Clients for Azure Services, email, etc. Adapters for MCPs.

### 3.5. .NET Testing
*   **Separate Test Projects:** `ProjectName.Domain.Tests`, `ProjectName.Application.Tests`, `ProjectName.Api.Tests`.
*   **Test Types:**
    *   **Domain:** Pure unit tests. Few/no mocks.
    *   **Application:** Unit tests for app services/CQRS handlers. Mock repositories/external services.
    *   **API:** Lightweight integration tests (`WebApplicationFactory`) for controllers, routing, DTO validation, serialization. Mock Application layer.
*   **Test Naming:** `MethodName_Scenario_ExpectedBehavior()`.
*   **Test Framework:** `{{memoryBank.toolingConfigurations.testFrameworks.dotnet}}` (e.g., MSTest).
*   **Mocking Framework:** `{{Project_defined_mocking_framework_for_.NET, e.g., Moq}}`.

## 4. Angular Frontend Specific Conventions

*   **(Refer to the Angular section of `design_conventions_template.md` for module/component structure, services, state management, HTTP, routing, but with a focus here on TypeScript code and patterns.)**
*   **TypeScript Strict Mode:** Enforce `strict: true` in `tsconfig.json`.
*   **Types and Interfaces:** Define clear types for data models, component props, service payloads. Shared models in `core/models` or by feature.
*   **RxJS:** Idiomatic use. Manage subscriptions (e.g., `takeUntil(this.destroy$)`). Prefer `async` pipe in templates.
*   **Angular Decorators:** Use according to official documentation.
*   **Angular Modules (`NgModule`):** Organize into `FeatureModules` (lazy-loaded), `SharedModule`, `CoreModule`.
*   **Testing (Angular):**
    *   `.spec.ts` files alongside tested files.
    *   Use `TestBed`. Mock dependencies. Test component logic, template-class interaction, service calls.
    *   Test Framework: `{{memoryBank.toolingConfigurations.testFrameworks.angular}}` (e.g., Jasmine/Karma).

## 5. Database Specific Conventions (MSSQL / Azure SQL)

*   **(Refer to DB section of `design_conventions_template.md` for naming, data types, constraints. Extend here with T-SQL rules, query optimization, etc.)**
*   **Transactions:** Explicit transactions in SPs for atomic operations.
*   **Performance:** Avoid `SELECT *`. Use explicit `JOINs`. Ensure `WHERE` clauses are SARGable.
*   **Security:** No dynamic SQL from user input without strict parameterization (usually handled by EF Core, but critical for SPs/raw queries). Grant minimal necessary DB user permissions.

## 6. Git Commit Conventions
*   Adhere to "Conventional Commits". Message MUST include Azure DevOps Task ID that it closes (e.g., `feat(orders): add endpoint for order creation - Closes Azure#45678`).

## 7. Docker and AKS Conventions
*   **(Refer to Docker/AKS section of `design_conventions_template.md`).**
*   Environment variables for containers managed via Kubernetes ConfigMaps/Secrets.

## 8. Review and Update of Conventions
This is a living document. `@architecture-advisor-agent` is responsible for proposing updates based on project evolution, team decisions, and new best practices. Major changes must be validated by the Tech Lead. Version and update date must be maintained.

---