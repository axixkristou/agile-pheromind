# Workflow: AgilePheromind System Optimization Suggestion (15_Optimize_Workflow_Suggestion.md)

**Objective:** Analyze performance metrics, operational history, and `memoryBank` structure of the AgilePheromind system to identify bottlenecks, inefficiencies, or areas for improvement. The system must propose concrete and justified English optimizations (with "chain of thought") for `01_AI-RUN/*.md` scripts, agent definitions in `.roomodes`, or Scribe's interpretation logic in `.swarmConfig`. The final report summary for the user/admin is provided in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@workflow-optimizer-agent`, `@swarm-monitor-agent` (can provide input data).

**MCPs Used:** Sequential Thinking MCP (for structuring optimization analysis).

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Manual:** System Admin/Tech Lead requests optimization analysis (e.g., `"AgilePheromind analyze your workflows for optimization"`). `userLanguage` passed by `🎩 @head-orchestrator`.
    *   **Automatic:** Scheduled trigger or by `@swarm-monitor-agent`. (UO uses default English for report, or configured language for notifications).
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage` if manually triggered.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Swarm Performance Data Collection and Analysis (English Context).**
        *   UO **injects relevant English context** to `@workflow-optimizer-agent`: `.pheromone.activeWorkflow.history`, `memoryBank.workflowPerformanceMetrics`, English reports from `@swarm-monitor-agent`, and (conceptually) current English `.roomodes` and `.swarmConfig` structure.
    *   **Phase 2: Structured Identification of Optimization Axes (English, with "Chain of Thought").**
        *   UO delegates to `@workflow-optimizer-agent`. Agent uses **Sequential Thinking MCP** (English) to identify problems and root causes. **Must document its English "chain of thought."**
        *   **onError:** If performance data insufficient, agent signals (English).
    *   **Phase 3: Generation of Specific Optimization Suggestions (English, with "Chain of Thought").**
        *   UO delegates to `@workflow-optimizer-agent`. **Must document English "chain of thought"** justifying each suggestion.
    *   **Phase 4: Report of Suggestions (Localized Summary) and Recording.**
        *   `@workflow-optimizer-agent` generates English report content. Translates a summary to `userLanguage` for UO.
        *   Scribe records English report and potentially structured English suggestions in `memoryBank`.

## Phase Details:

### Phase 1: Swarm Performance Data Collection and Analysis (English Context)
*   **Responsible Agent:** `@workflow-optimizer-agent`.
*   **Inputs (Injected by UO, English context):** `.pheromone` access, paths to English reports from `@swarm-monitor-agent`, (conceptual) read access to current English `.roomodes` and `.swarmConfig`.
*   **Actions & Tooling:**
    1.  Analyze `activeWorkflow.history`: execution/phase durations, agent errors, script usage frequency.
    2.  Analyze `memoryBank.workflowPerformanceMetrics`.
    3.  Consult `@swarm-monitor-agent` English reports.
    4.  Identify frequently slow/failing MCPs.
*   **Output (internal, English):** Raw/aggregated data on Pheromind system performance.

### Phase 2: Structured Identification of Optimization Axes (English, with "Chain of Thought")
*   **Responsible Agent:** `@workflow-optimizer-agent`.
*   **Inputs:** English performance data (Phase 1).
*   **Actions & Tooling:**
    1.  Use **Sequential Thinking MCP** (English) for methodical analysis: Goal "Identify optimization axes for AgilePheromind." Steps: "Slowest/error-prone scripts/phases? Problematic agents? Scribe interpretation issues? Inefficient `memoryBank` use?" Run.
    2.  **"Chain of Thought" (English):** Detailed output of this Sequential Thinking forms the English "chain of thought" for problem identification.
*   **onError:** If data insufficient, agent reports (English) to UO. UO may indicate limited analysis or postpone.
*   **Output (internal, English):** List of performance issues, inefficiencies, potential root causes, with English "chain of thought".

### Phase 3: Generation of Specific Optimization Suggestions (English, with "Chain of Thought")
*   **Responsible Agent:** `@workflow-optimizer-agent`.
*   **Inputs:** English problems and analysis (Phase 2).
*   **Actions & Tooling:**
    1.  For each identified problem, use **Sequential Thinking MCP** (English) to brainstorm solutions: Goal "Propose optimizations for: [English Problem]." Steps for script, agent, Scribe, or MemoryBank improvements. Run.
    2.  **"Chain of Thought" (English):** MCP output and agent elaboration forms English "chain of thought" for each suggestion.
    3.  Prioritize suggestions (impact vs. feasibility).
*   **Output (internal, English):** List of specific, prioritized English optimization suggestions with justifications.

### Phase 4: Report of Suggestions (Localized Summary) and Recording
*   **Responsible Agent:** `@workflow-optimizer-agent` (report), Scribe (recording).
*   **Inputs:** English optimization suggestions (Phase 3). `currentUser.lastInteractionLanguage` (from UO as `userLanguageForOutput`).
*   **Actions (`@workflow-optimizer-agent`):**
    1.  **Generate English Report Content:** Markdown (`system_optimization_suggestions_[timestamp].md`). Include: Analysis date, performance observations summary, detailed list of English optimization suggestions (problem, proposed solution, **English justification/chain of thought**, expected benefit, priority). Note analysis limitations if data was insufficient.
    2.  **Translate Summary for UO:** Provide UO with a concise English summary of the top 2-3 most impactful suggestions.
    3.  Save the full English report in `02_AI-DOCS/System_Optimization/`.
*   **Output (`@workflow-optimizer-agent` to Scribe, English NL Summary with path to English report):** "AgilePheromind system optimization analysis complete. [N_sugg] English suggestions proposed. Key areas: [list]. Full English Report (with chain of thought): `system_optimization_suggestions_[timestamp].md`."
*   **Actions (Scribe):**
    1.  Record English report in `documentationRegistry`.
    2.  Add entry to `memoryBank.systemImprovementProposals_en`: `{ reportPath_en: "...", timestamp, status_en: "PendingReview", summary_en: "[English Extract]" , reasoningChainLink_en: reportPath_en }`.
*   **Actions (UO):**
    1.  Translate the concise summary (from `@workflow-optimizer-agent`) to `currentUser.lastInteractionLanguage`.
    2.  Notify System Admin/Tech Lead (in `userLanguage`): "[Translated concise summary]. Full English technical report with detailed suggestions and reasoning available at `{{report_path}}`. Review and action are recommended."
*   **Output:** `.pheromone` updated with English data. Admin/Tech Lead informed (in their language).

---