# Workflow: Continuous Learning Integration (20_Continuous_Learning_Integration.md)

**Objective:** Proactively enrich the `memoryBank` by analyzing recurring patterns, effective decisions, and frequent problems to improve future system recommendations. This process involves analyzing data accumulated in the `memoryBank`, identifying patterns and insights, abstracting them into general principles, and integrating this knowledge into the system's knowledge base.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@learning-integration-agent`, `@architecture-advisor-agent`.

**MCPs Used:** Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Periodic trigger (weekly) or manual (`"AgilePheromind analyze and integrate recent learnings"`).
2.  **`🧐 @uber-orchestrator`** (UO) takes control.
    *   **Phase 1: Analysis Scope Definition and Data Preparation.**
        *   UO defines analysis scope (period, focus).
        *   UO prepares relevant data from `memoryBank`.
    *   **Phase 2: Recurring Pattern Analysis and Insight Identification.**
        *   UO delegates to `@learning-integration-agent` to analyze data and identify patterns.
        *   **onError:** If analysis fails, log, notify, stop.
    *   **Phase 3: Knowledge Abstraction and Generalization.**
        *   UO delegates to `@learning-integration-agent` to transform specific insights into general principles.
        *   **onError:** If abstraction fails, log, notify, stop.
    *   **Phase 4: Insight Validation and Enrichment.**
        *   UO delegates to `@architecture-advisor-agent` to validate and enrich insights.
        *   **onError:** If validation fails, log, continue with validated insights.
    *   **Phase 5: Knowledge Base Integration.**
        *   UO delegates to Scribe to integrate insights into `memoryBank`.
        *   **onError:** If integration fails, log, notify, propose manual integration.
    *   **Phase 6: Learning Report Generation.**
        *   UO delegates to `@learning-integration-agent` to generate a report summarizing new learnings.
        *   **onError:** If generation fails, log, notify.

## Phase Details:

### Phase 1: Analysis Scope Definition and Data Preparation
*   **Responsible Agent:** `🧐 @uber-orchestrator`
*   **Inputs:** User directive (if manual) or periodic trigger.
*   **Actions & Tooling:**
    1.  Define analysis scope:
        *   Period: By default, the last 2-4 weeks of activity.
        *   Focus: General or specific (e.g., "development", "code review", "architecture").
    2.  Prepare relevant data from `memoryBank`:
        *   Recently completed `userStories`.
        *   Recently completed `tasks`.
        *   Recently reviewed `pullRequests`.
        *   Recently identified `technicalDebtItems`.
        *   Recent `architecturalDecisions`.
        *   Recent `clarificationHistory`.
*   **Output (internal to UO):** Analysis scope defined and data prepared for `@learning-integration-agent`.

### Phase 2: Recurring Pattern Analysis and Insight Identification
*   **Responsible Agent:** `@learning-integration-agent`
*   **Inputs:** Analysis scope and data prepared by UO.
*   **Actions & Tooling:**
    1.  Use **Sequential Thinking MCP** to methodically analyze data.
    2.  Identify recurring patterns:
        *   Frequently encountered technical problems.
        *   Effective solutions applied to similar problems.
        *   Frequently asked clarification questions.
        *   Architectural decisions that worked well.
    3.  Document each identified pattern with concrete examples.
*   **onError Strategy (for UO if `@learning-integration-agent` reports failure):**
    1.  Scribe logs error in `activeWorkflow.lastError` and `memoryBank.agentActivityLog`.
    2.  UO notifies user: "Unable to analyze patterns in recent data. Error: [Error message]. Please try again with a narrower scope."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Pattern analysis for period [start] to [end] completed. [N] patterns identified: [list]. Analysis report: `learning_patterns_analysis_[timestamp].md`."

### Phase 3: Knowledge Abstraction and Generalization
*   **Responsible Agent:** `@learning-integration-agent`
*   **Inputs:** Patterns identified in Phase 2.
*   **Actions & Tooling:**
    1.  Use **Sequential Thinking MCP** to abstract specific patterns into general principles.
    2.  For each pattern:
        *   Identify underlying principles.
        *   Formulate general rules or best practices.
        *   Evaluate applicability in different contexts.
    3.  Categorize insights by domain (development, architecture, process, etc.).
    4.  **Detail the "Chain of Thought":** Explicitly document reasoning for each abstraction.
*   **onError Strategy (for UO if `@learning-integration-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO notifies user: "Unable to abstract identified patterns into general principles. Error: [Error message]."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Abstraction of patterns into general principles completed. [N] principles formulated: [list]. Abstraction report (with chain of thought): `learning_abstractions_[timestamp].md`."

### Phase 4: Insight Validation and Enrichment
*   **Responsible Agent:** `@architecture-advisor-agent`
*   **Inputs:** General principles formulated in Phase 3.
*   **Actions & Tooling:**
    1.  Evaluate validity and relevance of each principle.
    2.  Enrich principles with architectural considerations.
    3.  Identify links with existing architectural decisions.
    4.  Suggest improvements or nuances to formulated principles.
*   **onError Strategy (for UO if `@architecture-advisor-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO evaluates if some principles were validated before failure.
    3.  If yes, continue with validated principles. Otherwise, notify user and stop.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Validation and enrichment of principles completed. [N] principles validated, [M] enriched, [P] rejected. Validation report: `learning_validation_[timestamp].md`."

### Phase 5: Knowledge Base Integration
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Validated and enriched principles from Phase 4.
*   **Actions & Tooling:**
    1.  Update relevant sections of `memoryBank`:
        *   `memoryBank.learningInsights.recurringPatterns`
        *   `memoryBank.learningInsights.effectiveSolutions`
        *   `memoryBank.learningInsights.commonPitfalls`
    2.  Update best practice documents if necessary.
*   **onError Strategy (for UO if Scribe reports failure):**
    1.  UO logs error.
    2.  UO notifies user: "Unable to integrate insights into memoryBank. Error: [Error message]. Please consult the report for manual integration."
*   **Output (internal to Scribe):** `memoryBank` updated with new insights.

### Phase 6: Learning Report Generation
*   **Responsible Agent:** `@learning-integration-agent`
*   **Inputs:** Principles integrated into `memoryBank`.
*   **Actions & Tooling:**
    1.  Generate a report summarizing new learnings.
    2.  Include concrete examples to illustrate each principle.
    3.  Suggest practical applications for future tasks.
*   **onError Strategy (for UO if `@learning-integration-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO notifies user: "Unable to generate learning report. Error: [Error message]. Insights have been integrated into memoryBank."
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Learning analysis for [focus] completed. [N] patterns identified, [M] best practices formulated. Report (with reasoning): `learning_insights_[focus]_[timestamp].md`. MemoryBank has been enriched with these insights."

---
