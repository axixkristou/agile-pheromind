# Workflow: Technical Debt and Code Smells Analysis (11_Analyze_Tech_Debt.md)

**Objective:** Perform an in-depth English analysis of the project codebase (or a specific module) to identify technical debt, code smells, and areas requiring refactoring. The system must produce a detailed report (final output in `currentUser.lastInteractionLanguage`) with prioritized suggestions and the English "chain of thought" justifying key findings. Manages errors in code access or analysis tools.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@code-reviewer-assistant`, `@clarification-agent`.

**MCPs Used:** Git Tools MCP, potentially a dedicated static analysis MCP (e.g., SonarQube), Context7 MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User (Tech Lead/Architect) requests analysis (e.g., `"AgilePheromind analyze project tech debt"`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Scope Definition, Code Collection, and English Context Injection.**
        *   UO delegates to `@code-reviewer-assistant` to identify and retrieve target files via **Git Tools MCP**.
        *   UO **injects relevant English context** from `memoryBank` to `@code-reviewer-assistant`.
        *   **onError (Code Access):** If code inaccessible, log, notify user (in `userLanguage`), stop.
    *   **Phase 2: Analysis Planning and Static/Heuristic Analysis (English, with "Chain of Thought").**
        *   UO delegates to `@code-reviewer-assistant`. Agent uses **Sequential Thinking MCP** (English) for planning.
        *   Runs linters, analyzers, heuristics for code smells. Uses **Context7 MCP**. **Must document English "chain of thought"**.
        *   **onError (Analysis Tools):** If tool fails, note, continue with other methods if possible.
    *   **Phase 3: Tech Debt Identification, Prioritization & Suggestions (English, with "Chain of Thought").**
        *   UO delegates to `@code-reviewer-assistant`. Evaluates impact, prioritizes. **Must document English "chain of thought"**.
        *   If code areas too obscure for analysis, agent may report (English) to UO for potential clarification via `@clarification-agent` (question to dev in their language).
    *   **Phase 4: Detailed Report Generation (English Content, Final Output Localized).**
        *   UO delegates to `@code-reviewer-assistant`. Agent drafts full English report, then translates to `currentUser.lastInteractionLanguage`.
    *   **Phase 5: Recording in `.pheromone` (English Data for Scribe, Localized Report Path).**
        *   Scribe records report path and updates `memoryBank.technicalDebtItems_en` (English).

## Phase Details:

### Phase 1: Scope Definition, Code Collection, and English Context Injection
*   **Responsible Agent:** `@code-reviewer-assistant`, UO.
*   **Inputs:** Analysis scope. `memoryBank.currentProject.repositoryUrl` and `defaultBranch`.
*   **Actions & Tooling (UO and `@code-reviewer-assistant`):**
    1.  Define target files. `@code-reviewer-assistant` retrieves code via **Git Tools MCP**.
    2.  **onError (Git Tools MCP):** UO logs (English) via Scribe, notifies user (in `userLanguage`), stops.
    3.  UO injects English context to `@code-reviewer-assistant`: English `coding_conventions.md`, existing English `technicalDebtItems_en`, relevant English `architecturalDecisions_en`.
*   **Output:** Source code and injected English context ready for `@code-reviewer-assistant`.

### Phase 2: Analysis Planning and Static/Heuristic Analysis (English, with "Chain of Thought")
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** Source code, injected English context.
*   **Actions & Tooling:**
    1.  **Planning (Sequential Thinking MCP, English):** Set goal "Analyze code [scope] for tech debt." Add steps for linters, duplication, complexity, long methods, coupling, dead code, TODOs, Context7 checks. Run. **Retain plan as part of English "chain of thought".**
    2.  Execute plan steps.
    3.  **"Chain of Thought" (English):** For each major code smell/debt area, document in report how it was detected and why it's debt.
*   **onError (Analysis Tools):** If external tool fails, `@code-reviewer-assistant` notes (English), informs UO, continues with other methods. Report will mention failing tool.
*   **Output (internal, English):** List of code smells, quality issues, with location and English "chain of thought".

### Phase 3: Tech Debt Identification, Prioritization & Suggestions (English, with "Chain of Thought")
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** List of English code smells (Phase 2).
*   **Actions & Tooling:**
    1.  Group issues, evaluate impact. Prioritize debt (Critical, High, Medium, Low - English terms).
    2.  For priority items, suggest English refactorings/actions.
    3.  **"Chain of Thought" (English):** Document in report justification for prioritization and refactoring suggestions.
    4.  **Ambiguity Management:** If code area too obscure: Report (English) to UO for `@clarification-agent` (question to dev in their language).
*   **Output (internal, English):** Prioritized list of English tech debt items with suggestions and justifications.

### Phase 4: Detailed Report Generation (English Content, Final Output Localized)
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** Prioritized English tech debt list, justifications. `currentUser.lastInteractionLanguage` (from UO as `userLanguageForOutput`).
*   **Actions & Tooling:**
    1.  **Compile English Report Content:** Full structured English Markdown report. Structure: Scope, Debt Summary, Priority Item Details (desc, location, impact, severity, English reasoning/chain of thought, refactoring suggestion), Metrics (optional). Note unanalyzed sections.
    2.  **Translate Report:** Translate entire English report content to `userLanguageForOutput`.
    3.  Save localized report as `tech_debt_analysis_[scope]_[timestamp]_{{userLanguageForOutput}}.md` in `03_SPECS/Tech_Debt_Reports/`.
*   **Output (to Scribe, English NL Summary with path to localized report):** "Tech debt analysis '[Scope]' complete. [N_total] English issues, [N_crit] critical. Refactoring suggestions included. Report (in `{{userLanguageForOutput}}`, English reasoning available) at `{{localized_report_path}}`."

### Phase 5: Recording in `.pheromone` (English Data for Scribe, Localized Report Path)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** English NL Summary from `@code-reviewer-assistant`.
*   **Actions:**
    1.  Update `.pheromone`:
        *   `documentationRegistry`: Add `{{localized_report_path}}`.
        *   `memoryBank.technicalDebtItems_en`: Add/Update structured English objects for each critical/high debt item (e.g., `{ id: "TD_UUID", description_en: "[English Problem]", ..., reasoningChainLink_en: "{{english_reasoning_doc_or_report_section_link}}"}`).
        *   `memoryBank.projectContext.lastTechDebtAnalysisTimestamp_en = "{{timestamp}}"`.
*   **Output:** `.pheromone` updated. UO informed (can notify user in `userLanguage`).

---