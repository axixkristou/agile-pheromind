# Workflow: Legacy Code Analysis for Migration Project (09_Legacy_Migration_Analysis.md)

**Objective:** Provide a detailed and traceable English analysis of a legacy codebase (VB6, old .NET, COM+, MSSQL SPs) for migration to a modern .NET Core/Angular stack. The analysis must identify key components, dependencies, encapsulated business logic, database interactions, and propose complexity, risks, and potential migration strategies, with clear English "chain of thought" documentation. Manages errors in accessing sources or MCPs. The final report provided to the user will be in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@migration-analyst-agent`, `@clarification-agent`.

**MCPs Used:** Git Tools MCP, Context7 MCP, Fetch MCP, MSSQL MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User (Tech Lead/Architect) specifies path to legacy source code and target stack (e.g., `"AgilePheromind analyze VB6 at [path/repo] for migration to .NET Core/Angular."`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Analysis Planning, Code Ingestion, and Context Injection.**
        *   UO **injects English context** to `@migration-analyst-agent`. Agent uses **Sequential Thinking MCP** (English) to structure analysis. Attempts code ingestion.
        *   **onError (Code Ingestion):** If fails, agent reports (English) to UO. UO may ask user (in `userLanguage`) to verify path/access via `@clarification-agent`. Workflow pauses.
    *   **Phase 2: Detailed Legacy Component Analysis (with English "Chain of Thought").**
        *   UO delegates to `@migration-analyst-agent`. English analysis of modules, UI, logic, DB (MSSQL MCP), dependencies (Fetch MCP). **Must document English "chain of thought"**.
        *   **onError (MCP):** Agent notes missing info (English), continues if possible, signals in report.
    *   **Phase 3: Mapping to Modern Stack and Strategies (with English "Chain of Thought").**
        *   UO delegates to `@migration-analyst-agent`. English proposals, challenges, strategies. Uses **Context7 MCP**. **Must document English "chain of thought"**.
    *   **Phase 4: Complexity Estimation and Risks (with English "Chain of Thought").**
        *   UO delegates to `@migration-analyst-agent`. **Must document English "chain of thought"**.
    *   **Phase 5: Generation of Migration Analysis Report (English Content, Final Output Localized).**
        *   UO delegates to `@migration-analyst-agent`. Agent drafts full English report, then translates it to `currentUser.lastInteractionLanguage`.
    *   **Phase 6: Report Recording (English Data for Scribe, Localized Report Path).**
        *   Scribe records.

## Phase Details:

### Phase 1: Analysis Planning, Code Ingestion, and Context Injection
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs (Injected by UO):** Legacy code path/repo, target stack info. English context from `memoryBank` (past migration analyses, detailed target tech stack).
*   **Actions & Tooling:**
    1.  Use **Sequential Thinking MCP** (English) for analysis plan. Output forms first part of English "chain of thought".
    2.  **Ingest Code:** If Git repo, **Git Tools MCP** (`clone_repository`). Else, file access.
*   **onError (Code Ingestion):** If failure, `@migration-analyst-agent` reports (English) to UO: "Unable to access legacy code at [path/repo]: [Error]." UO pauses, delegates to `@clarification-agent` to ask user (in `userLanguage`) to verify/provide valid access.
*   **Output (internal if success, English):** Analysis plan, accessible legacy code.

### Phase 2: Detailed Legacy Component Analysis (with English "Chain of Thought")
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs:** Legacy code, English analysis plan.
*   **Actions & Tooling:**
    1.  Static code analysis (English observations).
    2.  SPs analysis (MSSQL MCP). **onError (MSSQL MCP):** Note inability to analyze SPs (English), continue if possible. Signal in report.
    3.  Dependency Analysis (Fetch/Context7 MCPs). **onError (Fetch/Context7):** Note dependency as "unknown/at risk" (English).
    4.  **"Chain of Thought" (English):** Document in report how main components were identified, their English functional interpretation, and dependency tracing.
*   **Output (internal, English):** Detailed inventory, logic, dependencies, with English analysis justification.

### Phase 3: Mapping to Modern Stack and Strategies (with English "Chain of Thought")
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs:** English component analysis (Phase 2). Target stack info.
*   **Actions & Tooling:**
    1.  **Functional Mapping (English):** Propose rewrites/replacements.
    2.  **Modern Documentation (Context7 MCP):** For .NET Core/Angular to guide English proposals.
    3.  **Migration Strategies (English):** Propose (Big Bang, Strangler Fig, Phased). Suggest order, PoCs.
    4.  **"Chain of Thought" (English):** Document in report why certain English mapping approaches preferred, alternatives, strategy logic.
*   **Output (internal, English):** Technical mapping proposals, migration strategies, with English justifications.

### Phase 4: Complexity Estimation and Risks (with English "Chain of Thought")
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs:** English component analysis, English mapping proposals.
*   **Actions & Tooling:**
    1.  **Complexity Estimation (English):** By component/feature (Low, Medium, High, Very High). Justify.
    2.  **Risk Identification (English):** Technical, data, coexistence risks.
    3.  **"Chain of Thought" (English):** Document in report how English complexity was evaluated and English risks identified/assessed.
*   **Output (internal, English):** Complexity evaluations, risk list, with English justifications.

### Phase 5: Generation of Migration Analysis Report (English Content, Final Output Localized)
*   **Responsible Agent:** `@migration-analyst-agent`.
*   **Inputs:** All English information from previous phases. `currentUser.lastInteractionLanguage` (from UO as `userLanguageForOutput`).
*   **Actions & Tooling:**
    1.  **Compile English Report Content:** Full structured English Markdown report. Include: Intro, Legacy Overview, Component Inventory, Detailed Analysis, Mapping Proposals, Strategy(ies), Complexity Estimates, Risks, Recommendations. **Must include all English "Chain of Thought" sections.**
    2.  **Translate Report:** Translate the entire English report content to `userLanguageForOutput`, preserving Markdown and technical accuracy.
    3.  Save localized report as `legacy_analysis_[legacy_project_name_en]_[timestamp]_{{userLanguageForOutput}}.md` in `02_AI-DOCS/Migration_Analyses/`.
*   **Output (to Scribe, English NL Summary with path to localized report):** "Legacy migration analysis '[LegacyProjectName_en]' to .NET Core/Angular complete. Complexity (English): [Overall_en]. [N_risks] major risks. Report (in `{{userLanguageForOutput}}`, with English reasoning chain available or summarized within) at `{{localized_report_path}}`. Key English recommendation: [1-2 key recommendations]."

### Phase 6: Report Recording (English Data for Scribe, Localized Report Path)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** English NL Summary from `@migration-analyst-agent`.
*   **Actions:**
    1.  Update `.pheromone` (English data for `memoryBank`):
        *   `documentationRegistry`: Add `{{localized_report_path}}`.
        *   `memoryBank.legacyCodeAnalyses.{{legacy_project_name_en}}`: Add `{ summary_en: "[English summary extract]", linkToReport_localized: "{{localized_report_path}}", timestamp, overallComplexity_en, keyRisksCount, reasoningChainLink_en: "{{english_version_of_report_or_core_reasoning_doc_path}}" }`. (The agent should ideally make its core English reasoning available, even if the main report is translated).
        *   If specific risks identified, create preliminary English entries in `memoryBank.riskRegister_en`.
*   **Output:** `.pheromone` updated. UO informed (can notify user in `userLanguage`).

---