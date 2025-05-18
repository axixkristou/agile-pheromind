# Workflow: Technical Debt and Code Smells Analysis (11_Analyze_Tech_Debt.md)

**Objective:** Perform an in-depth analysis of the project codebase (or a specific module) to identify technical debt, code smells, and areas requiring refactoring. The system must produce a detailed report with prioritized suggestions and the "chain of thought" justifying the main findings. Error handling for code access or analysis tools is provided.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@code-reviewer-assistant`, `@clarification-agent`.

**MCPs Used:** Git Tools MCP, potentially a dedicated static analysis MCP (e.g., SonarQube), Context7 MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The user (Tech Lead/Architect) requests an analysis (e.g., `"AgilePheromind analyze technical debt project"`). Can be scheduled.
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Scope Definition, Code Collection, and Context Injection.**
        *   UO delegates to `@code-reviewer-assistant` to identify and retrieve target files via **Git Tools MCP**.
        *   UO **injects relevant context** from `memoryBank` (conventions, debt history, architectural decisions) to `@code-reviewer-assistant`.
        *   **onError (Code Access):** If code is inaccessible, log, notify, stop.
    *   **Phase 2: Analysis Planning and Static/Heuristic Analysis (with "Chain of Thought").**
        *   UO delegates to `@code-reviewer-assistant`. The agent uses **Sequential Thinking MCP** to plan its analysis.
        *   Runs linters, analyzers, and heuristics for code smells. Uses **Context7 MCP** for library best practices.
        *   **Must document the "chain of thought"** for identifying the main "smells" or debt areas.
        *   **onError (Analysis Tools):** If an analysis tool (e.g., SonarQube MCP) fails, note and continue with other methods if possible.
    *   **Phase 3: Technical Debt Areas Identification, Prioritization, and Suggestions (with "Chain of Thought").**
        *   UO delegates to `@code-reviewer-assistant`. Evaluates impact, prioritizes.
        *   **Must document the "chain of thought"** for prioritization and refactoring suggestions.
        *   If code areas are too obscure for relevant debt analysis, the agent can report it to the UO for potential clarification via `@clarification-agent` (asking the dev to explain the section).
    *   **Phase 4: Detailed Report Generation.**
        *   UO delegates to `@code-reviewer-assistant`.
    *   **Phase 5: Recording in `.pheromone`.**
        *   Scribe records report and updates `memoryBank.technicalDebtItems`.

## Phase Details:

### Phase 1: Scope Definition, Code Collection, and Context Injection
*   **Responsible Agent:** `@code-reviewer-assistant`, UO.
*   **Inputs:** Analysis scope. `memoryBank.currentProject.repositoryUrl` and `defaultBranch`.
*   **Actions & Tooling (UO and `@code-reviewer-assistant`):**
    1.  Define target files.
    2.  `@code-reviewer-assistant` retrieves code via **Git Tools MCP**.
    3.  **onError (Git Tools MCP):** If failure, UO logs via Scribe, notifies user, stops workflow.
    4.  UO injects context from `memoryBank` to `@code-reviewer-assistant`:
        *   `projectContext.codingConventionsLink`, `designConventionsLink`.
        *   Existing `technicalDebtItems` (to avoid duplicates or see evolution).
        *   Relevant `architecturalDecisions`.
        *   (Optional) Specific configuration for static analysis tools (if stored).
*   **Output:** Source code and injected context ready for `@code-reviewer-assistant`.

### Phase 2: Analysis Planning and Static/Heuristic Analysis (with "Chain of Thought")
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** Source code, injected context.
*   **Actions & Tooling:**
    1.  **Planning (Sequential Thinking MCP):**
        *   `set_goal`: "Analyze the code [scope] for technical debt."
        *   `add_step`: "Run standard linters and analyzers."
        *   `add_step`: "Look for code duplications."
        *   `add_step`: "Analyze cyclomatic complexity (methods/functions)."
        *   `add_step`: "Identify long methods/classes/components."
        *   `add_step`: "Evaluate coupling/cohesion."
        *   `add_step`: "Search for 'dead code'."
        *   `add_step`: "Check TODO/FIXME."
        *   `add_step`: "Consult Context7 for best practices on libraries used in suspicious areas."
        *   `run_sequence`. **Keep this plan as part of the "chain of thought".**
    2.  Execute the steps of the plan.
    3.  **"Chain of Thought":** For each major code smell or debt area identified, document in the report how it was detected and why it is considered debt (e.g., "Method X exceeds 100 lines and has a complexity of Y, indicating a violation of convention Z and a maintainability risk.").
*   **onError (Analysis Tools):**
    *   If an external tool (e.g., SonarQube MCP) fails, `@code-reviewer-assistant` notes it, informs the UO, and continues the analysis with other methods. The final report will mention the failing tool.
*   **Output (internal):** List of code smells, quality issues, with location and beginning of "chain of thought".

### Phase 3: Technical Debt Areas Identification, Prioritization, and Suggestions (with "Chain of Thought")
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** List of code smells (Phase 2).
*   **Actions & Tooling:**
    1.  Group issues, evaluate impact (maintainability, bug risk, performance, extensibility).
    2.  Prioritize debt (Critical, High, Medium, Low).
    3.  For priority items, suggest refactorings or actions.
    4.  **"Chain of Thought":** Document in the report the justification for prioritization of critical/high items and the logic behind refactoring suggestions (e.g., "Refactoring X using pattern Y will improve readability and reduce coupling, as suggested by best practice Z from Context7 for this library.").
    5.  **Ambiguity Management:** If a code area is so obscure that it is impossible to evaluate the debt or suggest relevant refactoring:
        *   Report to the UO: "Unable to analyze debt for [file Z line A]. Obscure code. Suggested question for dev: 'Can you explain the purpose and structure of this section to evaluate its refactoring?'".
        *   The UO can initiate clarification via `@clarification-agent`. Analysis of this specific section is paused.
*   **Output (internal):** Prioritized list of technical debt items with suggestions and justifications.

### Phase 4: Detailed Report Generation
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** Prioritized list of technical debt, justifications ("chains of thought").
*   **Actions & Tooling:**
    1.  Write MD report (`tech_debt_analysis_[scope]_[timestamp].md`) in `03_SPECS/Tech_Debt_Reports/`.
    2.  Structure: Scope, Debt areas summary, Priority items detail (description, location, impact, severity, **reasoning/chain of thought**, refactoring suggestion), Metrics (optional).
    3.  If sections could not be analyzed due to lack of clarity (and clarification did not occur or did not resolve), mention it clearly.
*   **Output (to Scribe):** NL Summary: "Technical debt analysis '[Scope]' completed. [N_total] issues, including [N_crit] critical. Refactoring suggestions included. Report (with chain of thought): `tech_debt_analysis_[scope]_[timestamp].md`."

### Phase 4.5: Prioritization and Remediation Planning
*   **Responsible Agent:** `@architecture-advisor-agent` (in collaboration with `@code-reviewer-assistant`)
*   **Inputs:** Technical debt analysis report (Phase 4).
*   **Actions & Tooling:**
    1.  Use **Sequential Thinking MCP** to methodically evaluate each technical debt item:
        *   Analyze impact on code quality, maintainability, and performance.
        *   Evaluate effort required to fix each item.
        *   Identify dependencies between technical debt items.
    2.  Prioritize items based on impact/effort using an explicit decision matrix:
        *   High priority: High impact / Low effort
        *   Medium priority: High impact / High effort or Low impact / Low effort
        *   Low priority: Low impact / High effort
    3.  Propose a phased remediation plan:
        *   Phase 1: Immediate fixes (high priority)
        *   Phase 2: Medium-term fixes (medium priority)
        *   Phase 3: Long-term fixes (low priority)
    4.  Suggest specific US for the Azure DevOps backlog:
        *   Write titles and descriptions for technical debt remediation US.
        *   Propose acceptance criteria for each US.
    5.  Estimate the impact of each fix on overall code quality.
    6.  **"Chain of Thought":** Explicitly document reasoning for prioritization and remediation plan.
*   **onError Strategy (for UO if `@architecture-advisor-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO notifies user: "Unable to generate a complete remediation plan for technical debt. Error: [Error message]. The analysis report remains available."
    3.  Continue with Phase 5 without the remediation plan.
*   **Output (to Scribe):** NL Summary: "Technical debt remediation plan for '[Scope]' developed. [N] US proposed for backlog, distributed across [M] phases. Plan report: `tech_debt_remediation_plan_[scope]_[timestamp].md`."

### Phase 5: Recording in `.pheromone`
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** NL summaries from `@code-reviewer-assistant` and `@architecture-advisor-agent`.
*   **Actions:**
    1.  Update `.pheromone`:
        *   `documentationRegistry`: Add paths to `tech_debt_analysis...md` and `tech_debt_remediation_plan...md`.
        *   `memoryBank.technicalDebtItems`: Add/Update structured objects for each critical/high debt item identified, including a link to the relevant section of the report (`reasoningChainLink`).
        *   `memoryBank.technicalDebtRemediationPlan`: Add the repayment plan with phases and proposed US.
        *   `memoryBank.projectContext.lastTechDebtAnalysisTimestamp = "{{timestamp}}"`.
*   **Output:** `.pheromone` updated. UO informed.

---