# Workflow: Accessibility Compliance Check (21_Accessibility_Compliance_Check.md)

**Objective:** Analyze application UI components to verify their compliance with WCAG accessibility standards and propose improvements. This process involves scope identification, UI component analysis, accessibility issue detection, recommendation generation, and production of a detailed compliance report.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@accessibility-compliance-agent`, `@developer-agent`.

**MCPs Used:** Context7 MCP, Sequential Thinking MCP, Git Tools MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User requests an accessibility check (e.g., `"AgilePheromind check accessibility of UserProfile module"`).
2.  **`🧐 @uber-orchestrator`** (UO) takes control.
    *   **Phase 1: Scope Identification and Code Retrieval.**
        *   UO identifies relevant UI components.
        *   UO delegates to `@developer-agent` to retrieve source code.
        *   **onError:** If code inaccessible, log, notify, stop.
    *   **Phase 2: Component Analysis and Issue Detection.**
        *   UO delegates to `@accessibility-compliance-agent` to analyze each component.
        *   **onError:** If analysis fails, log, notify, stop.
    *   **Phase 3: Recommendation Generation.**
        *   UO delegates to `@accessibility-compliance-agent` to propose corrections.
        *   **onError:** If generation fails, log, continue with partial recommendations.
    *   **Phase 4: Compliance Report Generation.**
        *   UO delegates to `@accessibility-compliance-agent` to produce a detailed report.
        *   **onError:** If generation fails, log, notify.
    *   **Phase 5: `memoryBank` Update.**
        *   UO delegates to Scribe to update `memoryBank.accessibilityCompliance`.
        *   **onError:** If update fails, log, notify.

## Phase Details:

### Phase 1: Scope Identification and Code Retrieval
*   **Responsible Agent:** `🧐 @uber-orchestrator`, `@developer-agent`
*   **Inputs:** User directive specifying module/component to check.
*   **Actions & Tooling (UO):**
    1.  Identify relevant UI components:
        *   If module specified, identify all UI components in the module.
        *   If component specified, focus on that component.
    2.  Consult `memoryBank.projectContext` for project context.
*   **Actions & Tooling (`@developer-agent`):**
    1.  Use **Git Tools MCP** to retrieve source code of identified components.
    2.  Organize files by component to facilitate analysis.
*   **onError Strategy (for UO if `@developer-agent` reports failure):**
    1.  Scribe logs error in `activeWorkflow.lastError` and `memoryBank.agentActivityLog`.
    2.  UO notifies user: "Unable to retrieve source code for [Module/Component]. Error: [Error message]. Please check path or permissions."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Source code retrieved for [N] UI components of '[Module/Component]'. Identified components: [list]. Ready for accessibility analysis."

### Phase 2: Component Analysis and Issue Detection
*   **Responsible Agent:** `@accessibility-compliance-agent`
*   **Inputs:** UI component source code, project context.
*   **Actions & Tooling:**
    1.  Use **Context7 MCP** to consult WCAG guidelines and accessibility best practices.
    2.  Use **Sequential Thinking MCP** for methodical analysis of each component.
    3.  For each component, check key accessibility aspects:
        *   **Perception:** Contrast, alt texts for images, captions for media.
        *   **Operability:** Keyboard navigation, sufficient time, seizure prevention.
        *   **Understandability:** Readability, predictability, input assistance.
        *   **Robustness:** Compatibility with assistive technologies.
    4.  Categorize each issue by:
        *   WCAG compliance level (A, AA, AAA).
        *   Issue type (perception, operability, understandability, robustness).
        *   Severity (critical, major, minor).
    5.  **Detail the "Chain of Thought":** Explicitly document reasoning for each identified issue.
*   **onError Strategy (for UO if `@accessibility-compliance-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO notifies user: "Error during accessibility analysis of [Module/Component]. Error: [Error message]."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Accessibility analysis of '[Module/Component]' completed. [N] issues identified: [X] critical, [Y] major, [Z] minor. Analysis report: `accessibility_analysis_[Module]_[timestamp].md`."

### Phase 3: Recommendation Generation
*   **Responsible Agent:** `@accessibility-compliance-agent`
*   **Inputs:** Accessibility issues identified in Phase 2.
*   **Actions & Tooling:**
    1.  For each identified issue:
        *   Propose a specific correction with code example.
        *   Explain the impact of the correction on accessibility.
        *   Provide references to relevant WCAG guidelines.
    2.  Prioritize recommendations based on:
        *   WCAG compliance level (A, AA, AAA).
        *   Issue severity.
        *   Implementation ease.
    3.  Suggest general improvements for the entire module/component.
*   **onError Strategy (for UO if `@accessibility-compliance-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO evaluates if partial recommendations have been generated.
    3.  If yes, continue with available recommendations. Otherwise, notify user and stop.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Accessibility recommendations for '[Module/Component]' generated. [N] corrections proposed, prioritized by impact. Recommendations report: `accessibility_recommendations_[Module]_[timestamp].md`."

### Phase 4: Compliance Report Generation
*   **Responsible Agent:** `@accessibility-compliance-agent`
*   **Inputs:** Analysis and recommendations from previous phases.
*   **Actions & Tooling:**
    1.  Compile a comprehensive compliance report including:
        *   Executive summary with overall compliance score.
        *   Table of issues by component, WCAG level, and severity.
        *   Detailed recommendations with before/after code examples.
        *   Prioritized action plan.
    2.  Generate visualizations if possible (e.g., issue distribution charts).
    3.  Include references to relevant accessibility resources.
*   **onError Strategy (for UO if `@accessibility-compliance-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO notifies user: "Unable to generate complete compliance report. Error: [Error message]. Individual analyses and recommendations are available."
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Accessibility compliance report for '[Module/Component]' generated. Overall score: [score]/100. WCAG compliance: [level]. Complete report: `accessibility_audit_[Module]_[timestamp].md`."

### Phase 5: `memoryBank` Update
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Compliance report from Phase 4.
*   **Actions & Tooling:**
    1.  Update `memoryBank.accessibilityCompliance.wcagCheckHistory` with a new record.
    2.  Update `memoryBank.accessibilityCompliance.componentAccessibilityStatus` for each analyzed component.
    3.  Add the report to `documentationRegistry`.
*   **onError Strategy (for UO if Scribe reports failure):**
    1.  UO logs error.
    2.  UO notifies user: "Unable to update memoryBank with accessibility results. Error: [Error message]. The report is available but not indexed."
*   **Output (internal to Scribe):** `memoryBank` updated with accessibility results.

---
