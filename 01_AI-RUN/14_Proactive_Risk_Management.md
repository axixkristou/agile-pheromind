# Workflow: Proactive Project Risk Management (14_Proactive_Risk_Management.md)

**Objective:** Proactively identify, assess, and track potential project risks. This workflow utilizes analysis of `.pheromone` (English `memoryBank`, task/US states, workflow history) and Azure DevOps data to maintain an up-to-date risk register (in English). It documents the "chain of thought" for risk assessment and proposes mitigations. Error handling and clarifications (user interaction in `currentUser.lastInteractionLanguage`) are integrated.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@risk-manager-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Manual:** User (Scrum Master/Tech Lead) requests risk management session (e.g., `"AgilePheromind run project risk analysis"`). `userLanguage` passed by `🎩 @head-orchestrator`.
    *   **Automatic:** Scheduled trigger. (UO uses default English for report summaries, or configured language).
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage` if manually triggered.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Data Collection and Risk Indicator Identification.**
        *   UO **injects English context** to `@risk-manager-agent`: `currentSprint.id`, `currentProject.id`, `memoryBank.projectContext.riskMatrix_en` (if defined), past risk history.
        *   `@risk-manager-agent` scans `.pheromone` (English `memoryBank`).
        *   `@risk-manager-agent` requests `@devops-connector` to retrieve "Risk"/"Impediment" items from Azure DevOps (via **ADO MCP**).
        *   **onError (ADO MCP):** If fails, notify user (in `userLanguage`), analysis proceeds based on `.pheromone` only, with a warning.
    *   **Phase 2: Risk Assessment and Prioritization (with English "Chain of Thought").**
        *   UO delegates to `@risk-manager-agent`. Attributes impact, probability. Uses **Sequential Thinking MCP** (English) for structured assessment. **Must document English "chain of thought"** for each major risk.
        *   If an indicator is ambiguous, `@risk-manager-agent` may report (English) to UO for clarification via `@clarification-agent` (question to user in `userLanguage`).
    *   **Phase 3: Risk Register Update (English Data).**
        *   UO delegates to `@risk-manager-agent` to prepare updates.
        *   Scribe updates `memoryBank.riskRegister` in `.pheromone` (all English entries).
    *   **Phase 4: Mitigation Plan Proposal (English, with "Chain of Thought", Optional).**
        *   If critical risks, UO requests `@risk-manager-agent` to suggest English mitigations (via **Sequential Thinking MCP**), documenting English "chain of thought".
    *   **Phase 5: Report (Localized Summary) and Notification.**
        *   `@risk-manager-agent` generates English report content (including English "chains of thought"). Translates a summary/key findings to `userLanguage` for UO.
        *   Scribe records English report. UO notifies stakeholders (in `userLanguage`).

## Phase Details:

### Phase 1: Data Collection and Risk Indicator Identification
*   **Responsible Agent:** `@risk-manager-agent` (coordinates with `@devops-connector`).
*   **Inputs (Injected by UO):** `currentSprint.id`, `currentProject.id`. English context from `memoryBank` (risk matrix, past risks).
*   **Actions (`@risk-manager-agent`):**
    1.  **Scan `.pheromone.memoryBank` (English data):** `tasks` (Blocked, Delayed), `technicalDebtItems_en` (critical), `sprintRetrospectivesSummaries_en` (recurring impediments), `architecturalDecisions_en` (risky ones).
    2.  **Scan `.pheromone.activeWorkflow.history`:** Frequently failing agents/scripts.
    3.  **Request ADO Data via `@devops-connector`:** "Fetch 'Risk' or 'Impediment' work items from ADO project `{{currentProject.name}}`."
*   **onError (ADO MCP via `@devops-connector`):** `@devops-connector` signals failure (English) to `@risk-manager-agent`. Agent includes English warning in its final summary: "Azure DevOps sync for risks/impediments failed. Analysis based on Pheromind data only."
*   **Output (internal to `@risk-manager-agent`, English):** List of observations and risk indicators.

### Phase 2: Risk Assessment and Prioritization (with English "Chain of Thought")
*   **Responsible Agent:** `@risk-manager-agent`, UO, `@clarification-agent`.
*   **Inputs:** English risk indicators (Phase 1). `memoryBank.projectContext.riskMatrix_en` (injected by UO).
*   **Actions (`@risk-manager-agent`):**
    1.  Use **Sequential Thinking MCP** (English) for each major indicator to formalize risk:
        *   `set_goal`: "Assess risk indicator: [English Indicator]."
        *   Steps: Describe English risk, category, causes, impact (Low, Medium, High, Critical - justify), probability (Low, Medium, High, Very High - justify), calculate score (if matrix).
        *   **Retain MCP output as English "chain of thought".**
    2.  Prioritize English risks.
    3.  **Ambiguity Management:** If indicator evaluation blocked by missing info: Report (English) to UO: "Cannot fully assess indicator '[Indicator_en]' due to [missing info_en]. Suggest question for [PO/TechLead/Dev_en]: '[Precise English question]'". UO may use `@clarification-agent` (translating question for user).
*   **Output (internal, English):** List of formalized, assessed, prioritized English risks (each with "chain of thought").

### Phase 3: Risk Register Update (English Data)
*   **Responsible Agent:** `@risk-manager-agent` (preparation), Scribe (writing).
*   **Inputs:** Assessed English risks (Phase 2). Existing English `memoryBank.riskRegister` (injected by UO).
*   **Actions (`@risk-manager-agent`):**
    1.  Compare, prepare additions/updates for English `memoryBank.riskRegister`.
    2.  Each English entry: `id`, `description_en`, `category_en`, `impactLevel_en`, `probabilityLevel_en`, `riskScore_en`, `status_en`, `dateIdentified`, `lastAssessed`, `owner_en` (optional), `mitigationPlanLink_en` (optional), `reasoningChainSummary_en` (or link to report section).
*   **Output (`@risk-manager-agent` -> Scribe, English):** Structured data for `memoryBank.riskRegister`. English NL summary of changes.
*   **Actions (Scribe):** Update `memoryBank.riskRegister` in `.pheromone` (all English).

### Phase 4: Mitigation Plan Proposal (English, with "Chain of Thought", Optional)
*   **Responsible Agent:** `@risk-manager-agent`.
*   **Inputs:** Critical/high English risks from `memoryBank.riskRegister` (injected by UO).
*   **Actions:**
    1.  For targeted risks, use **Sequential Thinking MCP** (English) to brainstorm English mitigations.
    2.  **"Chain of Thought" (English):** Document English reasoning for each proposed mitigation plan.
*   **Output (internal, for Phase 5 report, English):** Suggested English mitigation plans with justifications.

### Phase 5: Report (Localized Summary) and Notification
*   **Responsible Agent:** `@risk-manager-agent` (report), Scribe (recording), UO (notification).
*   **Inputs:** Updated English risk register, English mitigation proposals. `currentUser.lastInteractionLanguage`.
*   **Actions (`@risk-manager-agent`):**
    1.  **Generate English Report Content:** Markdown (`risk_assessment_report_[timestamp].md`). Include: Summary of critical English risks, full English risk register (or significant changes), **detailed English "chain of thought" for major risk assessments**, proposed English mitigation plans (with their "chain of thought"). Note data collection failures (e.g., ADO).
    2.  **Translate Key Findings/Summary for UO:** Provide UO with a concise English summary of the most critical findings and the overall risk posture.
    3.  The full detailed report remains in English for the `memoryBank` and technical team.
*   **Output (`@risk-manager-agent` -> Scribe, English NL Summary with path to English report):** "Proactive risk analysis complete. [N_total_open] open English risks, [N_high_critical] critical/high. [N_new] new risks. [N_mitigations] English mitigations proposed. English Report (with chains of thought): `risk_assessment_report_[timestamp].md`." (Path in `03_SPECS/Risk_Management/`).
*   **Actions (Scribe):**
    1.  Record English report in `documentationRegistry`.
    2.  Ensure `memoryBank.riskRegister` (English) is up-to-date. Link English `reasoningChainLink_en` for risks to report sections.
*   **Actions (UO):**
    1.  Translate the concise summary (from `@risk-manager-agent`) to `currentUser.lastInteractionLanguage`.
    2.  Notify stakeholders (Scrum Master, PO, Tech Lead) in `userLanguage`: "[Translated concise summary]. Full English technical report available at `{{report_path}}`. Review of critical risks requested." Use `ask_followup_question` for actions on critical risks.
*   **Output:** `.pheromone` updated with English risk data. Stakeholders informed in their language.

---