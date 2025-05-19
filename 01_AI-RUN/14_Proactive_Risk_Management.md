# Workflow: Proactive Project Risk Management (14_Proactive_Risk_Management.md)

**Objective:** Proactively identify, assess, and track potential project risks in English. This workflow utilizes analysis of `.pheromone` (English `memoryBank`) and Azure DevOps data to maintain an up-to-date English risk register. It documents the "chain of thought" in English for risk assessment and proposes English mitigations. The final summary/report for the user is provided in `currentUser.lastInteractionLanguage`. Error handling and clarifications are integrated.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@risk-manager-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Manual (Scrum Master/Tech Lead: `"AgilePheromind run project risk analysis"`) or automatic. `userLanguage` passed by `🎩 @head-orchestrator`.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Data Collection and Risk Indicator Identification (English Focus).**
        *   UO **injects English context** to `@risk-manager-agent`. Agent scans `.pheromone` (English `memoryBank`).
        *   `@risk-manager-agent` requests `@devops-connector` for "Risk"/"Impediment" items from ADO.
        *   **onError (ADO MCP):** Notify user (in `userLanguage`), analysis proceeds on `.pheromone` data only, with English warning.
    *   **Phase 2: Risk Assessment and Prioritization (English, with "Chain of Thought").**
        *   UO delegates to `@risk-manager-agent`. Uses **Sequential Thinking MCP** (English). **Must document English "chain of thought"**.
        *   If indicator ambiguous, agent reports (English) to UO for clarification via `@clarification-agent` (question to user in `userLanguage`).
    *   **Phase 3: Risk Register Update (`memoryBank.riskRegister_en` - English Data).**
        *   UO delegates to `@risk-manager-agent` for updates. Scribe updates `.pheromone`.
    *   **Phase 4: Mitigation Plan Proposal (English, with "Chain of Thought", Optional).**
        *   If critical risks, UO requests `@risk-manager-agent` for English mitigations (via **Sequential Thinking MCP**), documenting English "chain of thought".
    *   **Phase 5: Report (Localized Summary) and Notification.**
        *   `@risk-manager-agent` generates English report content. Translates summary/key findings to `userLanguage` for UO.
        *   Scribe records English report. UO notifies stakeholders (in `userLanguage`).

## Phase Details:

### Phase 1: Data Collection and Risk Indicator Identification (English Focus)
*   **Responsible Agent:** `@risk-manager-agent` (coordinates with `@devops-connector`).
*   **Inputs (Injected by UO):** `currentSprint.id`, `currentProject.id`. English context from `memoryBank` (risk matrix, past risks).
*   **Actions (`@risk-manager-agent`):**
    1.  **Scan `.pheromone.memoryBank` (English data):** `tasks` (Blocked, Delayed), `technicalDebtItems_en`, `sprintRetrospectivesSummaries_en`, `architecturalDecisions_en`.
    2.  **Scan `.pheromone.activeWorkflow.history`:** Frequent agent/script failures.
    3.  **Request ADO Data via `@devops-connector` (English instruction):** "Fetch 'Risk' or 'Impediment' work items from ADO project `{{currentProject.name}}`."
*   **onError (ADO MCP):** `@devops-connector` signals failure (English). `@risk-manager-agent` includes English warning in summary: "ADO sync for risks/impediments failed. Analysis on Pheromind data only."
*   **Output (internal, English):** List of observations and risk indicators.

### Phase 2: Risk Assessment and Prioritization (English, with "Chain of Thought")
*   **Responsible Agent:** `@risk-manager-agent`, UO, `@clarification-agent`.
*   **Inputs:** English risk indicators (Phase 1). `memoryBank.projectContext.riskMatrix_en` (injected).
*   **Actions (`@risk-manager-agent`):**
    1.  Use **Sequential Thinking MCP** (English) for each major indicator to formalize risk: Goal "Assess risk indicator: [English Indicator]." Steps: Describe English risk, category, causes, impact (justify), probability (justify), score. **Retain MCP output as English "chain of thought".**
    2.  Prioritize English risks.
    3.  **Ambiguity Management:** If indicator evaluation blocked: Report (English) to UO: "Cannot fully assess indicator '[Indicator_en]' due to [missing_info_en]. Suggest question for [Role_en]: '[Precise English question]'". UO may use `@clarification-agent`.
*   **Output (internal, English):** List of formalized, assessed, prioritized English risks (each with English "chain of thought").

### Phase 3: Risk Register Update (`memoryBank.riskRegister_en` - English Data)
*   **Responsible Agent:** `@risk-manager-agent` (preparation), Scribe (writing).
*   **Inputs:** Assessed English risks (Phase 2). Existing English `memoryBank.riskRegister_en` (injected).
*   **Actions (`@risk-manager-agent`):** Prepare additions/updates for English `memoryBank.riskRegister_en`. Each English entry: `id`, `description_en`, `category_en`, `impactLevel_en`, `probabilityLevel_en`, `riskScore_en`, `status_en`, dates, `owner_en`, `mitigationPlanLink_en`, `reasoningChainSummary_en`.
*   **Output (`@risk-manager-agent` -> Scribe, English):** Structured data for `memoryBank.riskRegister_en`. English NL summary of changes.
*   **Actions (Scribe):** Update `memoryBank.riskRegister_en` in `.pheromone`.

### Phase 4: Mitigation Plan Proposal (English, with "Chain of Thought", Optional)
*   **Responsible Agent:** `@risk-manager-agent`.
*   **Inputs:** Critical/high English risks from `memoryBank.riskRegister_en` (injected).
*   **Actions:** For targeted risks, use **Sequential Thinking MCP** (English) to brainstorm English mitigations. **"Chain of Thought" (English):** Document English reasoning.
*   **Output (internal, for Phase 5 report, English):** Suggested English mitigation plans with justifications.

### Phase 5: Report (Localized Summary) and Notification
*   **Responsible Agent:** `@risk-manager-agent` (report), Scribe (recording), UO (notification).
*   **Inputs:** Updated English risk register, English mitigation proposals. `currentUser.lastInteractionLanguage`.
*   **Actions (`@risk-manager-agent`):**
    1.  **Generate English Report Content:** Markdown (`risk_assessment_report_[timestamp].md`). Include: Summary of critical English risks, full English risk register, **detailed English "chain of thought" for major risk assessments**, proposed English mitigation plans (with English "chain of thought"). Note data collection failures.
    2.  **Translate Key Findings/Summary for UO:** Provide UO with concise English summary of critical findings and overall risk posture.
    3.  Save full English report in `03_SPECS/Risk_Management/`.
*   **Output (`@risk-manager-agent` -> Scribe, English NL Summary with path to English report):** "Proactive risk analysis complete. [N_total_open] open English risks, [N_high_critical] critical/high. [N_new] identified. [N_mitigations] English mitigations proposed. English Report (with chains of thought): `risk_assessment_report_[timestamp].md`."
*   **Actions (Scribe):** Record English report in `documentationRegistry`. Ensure `memoryBank.riskRegister_en` is up-to-date. Link English `reasoningChainLink_en` for risks to report sections.
*   **Actions (UO):** Translate concise summary (from `@risk-manager-agent`) to `currentUser.lastInteractionLanguage`. Notify stakeholders (in `userLanguage`): "[Translated concise summary]. Full English technical report available at `{{report_path}}`. Review of critical risks requested." Use `ask_followup_question`.
*   **Output:** `.pheromone` updated with English risk data. Stakeholders informed in their language.

---