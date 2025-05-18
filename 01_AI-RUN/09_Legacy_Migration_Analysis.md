# Workflow: Legacy Code Analysis for Migration Project (09_Legacy_Migration_Analysis.md)

**Objective:** Provide a detailed and traceable analysis of a legacy codebase (VB6, older .NET, COM+, MSSQL SPs) for migration to .NET Core/Angular. The analysis must identify key components, dependencies, business logic, DB interactions, and propose complexity, risks, and migration strategies, with clear documentation of the agent's "chain of thought". Handle errors in accessing sources or MCPs.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@migration-analyst-agent`, `@clarification-agent`.

**MCPs Used:** Git Tools MCP, Context7 MCP, Fetch MCP, MSSQL MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The user (Tech Lead/Architect) specifies the path/repo of the legacy code and the target stack (e.g., `"AgilePheromind analyze VB6 at [path] for migration to .NET Core/Angular."`).
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Analysis Planning, Code Ingestion, and Context Injection.**
        *   UO **injects context** (target stack from `memoryBank.projectContext`, past similar migration analyses from `memoryBank.legacyCodeAnalyses`) to `@migration-analyst-agent`.
        *   `@migration-analyst-agent` uses **Sequential Thinking MCP** to structure its approach. Attempts to ingest the code.
        *   **onError (Code Ingestion):** If access/reading of legacy code fails, the agent reports it to the UO. The UO may ask the user to verify the path/access or upload the code, potentially via `@clarification-agent`. Workflow paused.
    *   **Phase 2: Detailed Analysis of Legacy Components (with "Chain of Thought").**
        *   UO delegates to `@migration-analyst-agent`. Syntactic analysis, modules, UI, business logic, DB access (with **MSSQL MCP**), dependencies (with **Fetch MCP**). **Must document the "chain of thought"** for the identification and interpretation of components.
        *   **onError (MCP):** If an MCP (MSSQL, Fetch) fails, the agent notes the missing information, continues if possible, and reports it in its report.
    *   **Phase 3: Mapping to Modern Stack and Strategies (with "Chain of Thought").**
        *   UO delegates to `@migration-analyst-agent`. Proposal of equivalents, challenges, strategies. Use of **Context7 MCP**. **Must document the "chain of thought"** for mapping proposals.
    *   **Phase 4: Complexity Estimation and Risks (with "Chain of Thought").**
        *   UO delegates to `@migration-analyst-agent`. **Must document the "chain of thought"** for evaluations.
    *   **Phase 5: Generation of Analysis Report (including "Chain of Thought").**
        *   UO delegates to `@migration-analyst-agent`.
    *   **Phase 6: Report Recording.**
        *   Scribe records.

## Phase Details:

### Phase 1: Analysis Planning, Code Ingestion, and Context Injection
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs (Injected by UO):** Legacy code path/repo, target stack. `memoryBank` context (past migration analyses, detailed target tech stack).
*   **Actions & Tooling:**
    1.  **Sequential Thinking MCP** to plan the analysis (steps: ingestion, module identification, dependencies, DB, mapping, strategy, complexity/risks). **The output of this MCP will constitute the first part of the "chain of thought".**
    2.  **Code Ingestion:** If Git repo, **Git Tools MCP** (`clone_repository`). Otherwise, file access.
*   **onError (Code Ingestion):**
    *   If failure, `@migration-analyst-agent` reports to the UO: "Unable to access legacy code at [path/repo]: [Error].".
    *   UO pauses workflow, delegates to `@clarification-agent` to ask the user to verify/provide valid access.
*   **Output (internal if successful):** Analysis plan, accessible legacy code.

### Phase 2: Detailed Analysis of Legacy Components (with "Chain of Thought")
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs:** Legacy code, analysis plan.
*   **Actions & Tooling:**
    1.  Static analysis (VB6, .NET Framework etc.): modules, UI, logic, data access.
    2.  SPs analysis (if applicable): **MSSQL MCP** (`get_database_schema`, `list_stored_procedures`, `get_stored_procedure_definition`). **onError (MSSQL MCP):** If failure, note the inability to analyze SPs and continue if possible. Report it in the report.
    3.  Dependencies Analysis: DLLs, COM+, third-party libs. For obscure ones, **Fetch MCP** (web scraping) or **Context7 MCP**. **onError (Fetch/Context7):** If doc not found, note the dependency as "unknown/at risk".
    4.  **"Chain of Thought":** Document in the report (dedicated section or appendix) how the main components were identified, what was their functional interpretation, and how dependencies were traced.
*   **Output (internal):** Detailed inventory of components, logic, dependencies, with analysis justification.

### Phase 3: Mapping to Modern Stack and Strategies (with "Chain of Thought")
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs:** Component analysis (Phase 2). Target stack.
*   **Actions & Tooling:**
    1.  **Functional Mapping:** Propose rewrite/replacement (VB6/.NET Logic -> .NET Core Services; VB6/WinForms UI -> Angular Components; SPs -> .NET Core API/EF Core).
    2.  **Modern Documentation (Context7 MCP):** `get_library_docs` for .NET Core/Angular to guide proposals.
    3.  **Migration Strategies:** Propose (Big Bang, Strangler Fig, Phased). Suggest order, PoCs.
    4.  **"Chain of Thought":** Document in the report why certain mapping approaches are preferred, alternatives considered and discarded, and the logic behind the proposed migration strategy.
*   **Output (internal):** Technical mapping proposals, migration strategies, with justifications.

### Phase 4: Complexity Estimation and Risks (with "Chain of Thought")
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs:** Component analysis, mapping proposals.
*   **Actions & Tooling:**
    1.  **Complexity Estimation:** By component/feature group (Low, Medium, High, Very High). Justify.
    2.  **Risk Identification:** Technical (undocumented logic, irreplaceable dependencies), data, coexistence.
    3.  **"Chain of Thought":** Document in the report how complexity was evaluated for each component (which factors weighed most) and how risks were identified and assessed.
*   **Output (internal):** Complexity evaluations, risk list, with justifications.

### Phase 5: Generation of Migration Analysis Report (including "Chain of Thought")
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs:** All information from previous phases.
*   **Actions & Tooling:**
    1.  Write MD report (`legacy_analysis_[legacy_project_name]_[timestamp].md`) in `02_AI-DOCS/Migration_Analyses/`.
    2.  Structure: Intro, Legacy Overview, Component Inventory, Detailed Analysis, Mapping Proposals, Migration Strategy(ies), Complexity Estimates, Risks, Recommendations.
    3.  **Explicitly integrate "Chain of Thought" sections** or references to appendices documenting the reasoning for each major analysis and proposal step.
*   **Output (to Scribe):** NL Summary: "Migration analysis '[LegacyProjectName]' to .NET Core/Angular completed. Complexity: [Overall]. [N_risks] major risks. Report (with detailed chain of thought): `legacy_analysis_[legacy_project_name]_[timestamp].md`. Recommendation: [Key]."

### Phase 6: Report Recording
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** NL summary from `@migration-analyst-agent`.
*   **Actions:**
    1.  Update `.pheromone`:
        *   `documentationRegistry`: Add path to `legacy_analysis...md`.
        *   `memoryBank.legacyCodeAnalyses.{{legacy_project_name}}`: Add `{ summary, linkToReport, timestamp, overallComplexity, keyRisksCount, reasoningChainLink: reportPath }`.
        *   If specific risks are identified and can be added to the `riskRegister`, create preliminary entries.
*   **Output:** `.pheromone` updated. UO informed.

---