# Workflow: User Interface (UI) Test Validation (12_UI_Test_Validation.md)

**Objective:** Assist the tester in manual or semi-automated validation of a feature's or User Story's (US) UI. The agent uses the Browser Tools MCP for interactions/captures, compares results against English design specifications and ACs, documents its "chain of thought" in English for significant deviations, and produces a final report in `currentUser.lastInteractionLanguage`. Handles errors or ambiguities in specs.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@tester-ui-validator-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Browser Tools MCP (Puppeteer or Playwright), Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User (Tester/PO) requests UI validation for US/Feature + test env URL (e.g., `"AgilePheromind validate UI US Azure#12323 on https://test.myapp.com"`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Specification & AC Retrieval, English Context Injection.**
        *   UO delegates to `@devops-connector` for US ACs (original language from ADO). Scribe/UO ensures English versions (`acceptanceCriteria_en`) in `memoryBank`.
        *   UO **injects English context** to `@tester-ui-validator-agent`: English ACs, English `design_conventions.md`, English mockups (from `documentationRegistry`/`memoryBank`).
        *   UO evaluates clarity of English UI specs. If ambiguous, UO initiates clarification via `@clarification-agent` (question to PO/Designer in their language).
        *   **onError:** If ADO MCP or specs access fails, notify user (in `userLanguage`), stop.
    *   **Phase 2: Detailed UI Test Scenario Definition (English).**
        *   UO delegates to `@tester-ui-validator-agent` to create an English UI test plan.
    *   **Phase 3: Interactions Execution and Captures via Browser Tools MCP.**
        *   UO delegates to `@tester-ui-validator-agent`.
        *   **onError (Browser Tools MCP):** Agent notes (English), tries to continue, signals in report.
    *   **Phase 4: Results Analysis and Report Generation (English Content, "Chain of Thought" for Deviations, Final Output Localized).**
        *   UO delegates to `@tester-ui-validator-agent`. For major deviations, agent **documents English "chain of thought"**. Agent drafts full English report, then translates to `currentUser.lastInteractionLanguage`.
    *   **Phase 5: Report Recording and Optional Bug Creation.**
        *   Scribe records localized report path. UO may propose ADO bug creation (via `@devops-connector`, with English details for bug, UO can translate summary for ADO if needed).

## Phase Details:

### Phase 1: Specification & AC Retrieval, English Context Injection
*   **Responsible Agent:** UO, `@devops-connector`, Scribe, `@clarification-agent`.
*   **Inputs:** US ID/Feature name, test env URL. `userLanguage`.
*   **Actions (UO & `@devops-connector`):**
    1.  If US ID: `@devops-connector` gets ACs/desc from ADO (**ADO MCP** `get_work_item_details`).
    2.  **onError (ADO MCP):** UO logs (English), notifies user (in `userLanguage`), stops/warns.
    3.  Scribe (with UO help for translation if needed) updates `memoryBank.userStories.{{usId}}` with `acceptanceCriteria_en` and `descriptionFull_en`.
    4.  UO injects to `@tester-ui-validator-agent`: English ACs, path to English `design_conventions.md`, paths to English mockups/prototypes.
*   **Actions (UO - Clarity Evaluation):**
    1.  UO evaluates if English ACs and design specs are coherent/precise.
    2.  **If major ambiguity:** Pause. Delegate to `@clarification-agent` with English context and English question (to be translated to `userLanguage` for PO/Designer). Await response.
*   **Output:** Rich and clarified English context for `@tester-ui-validator-agent`.

### Phase 2: Detailed UI Test Scenario Definition (English)
*   **Responsible Agent:** `@tester-ui-validator-agent`.
*   **Inputs:** Injected English context (Phase 1).
*   **Actions:** Define English validation steps. Document scenario (for English draft of final report).
*   **Output (internal, English):** UI test plan.

### Phase 3: Interactions Execution and Captures via Browser Tools MCP
*   **Responsible Agent:** `@tester-ui-validator-agent`.
*   **Inputs:** English test scenario (Phase 2), test env URL.
*   **Actions (Browser Tools MCP):** For each step: `navigate`, `click`, `fill_form_field`, `take_screenshot`, `execute_script`, `get_console_logs`.
*   **onError (Browser Tools MCP):** Agent logs precise English MCP error. Tries to continue. All MCP failures listed in English in final report.
*   **Output (internal, English context):** Screenshots, browser data, English MCP error logs.

### Phase 4: Results Analysis and Report Generation (English Content, "Chain of Thought" for Deviations, Final Output Localized)
*   **Responsible Agent:** `@tester-ui-validator-agent`.
*   **Inputs:** Data from Phase 3. English specs from Phase 1. `currentUser.lastInteractionLanguage` (as `userLanguageForOutput`).
*   **Actions:**
    1.  Compare captures/data with English mockups/conventions/ACs. Check console logs.
    2.  **Draft English Report Content:** Structure: Scope, Summary, Validation points (English Expected, Observed, Status, English Comment). **English "Chain of Thought"** for major "Failures". List English MCP failures. List English bugs/issues with severity.
    3.  **Translate Report:** Translate entire English report content to `userLanguageForOutput`.
    4.  Save localized report: `ui_validation_report_[US_ID_or_Feature]_[timestamp]_{{userLanguageForOutput}}.md` in dir (e.g., `03_SPECS/UI_Validation_Reports/`).
*   **Output (to Scribe, English NL Summary with path to localized report):** "UI validation '[US_ID/Feature_en]' completed. Global status (English): [Status]. [N_bugs] English bugs identified. Report (in `{{userLanguageForOutput}}`, English reasoning available) at `{{localized_report_path}}`."

### Phase 5: Report Recording and Optional Bug Creation
*   **Responsible Agent:** Scribe, UO, `@devops-connector`.
*   **Inputs:** English NL Summary from `@tester-ui-validator-agent`. `currentUser.lastInteractionLanguage`.
*   **Actions (Scribe):**
    1.  Update `.pheromone` (English data for `memoryBank`):
        *   `documentationRegistry`: Add `{{localized_report_path}}`.
        *   `memoryBank.userStories.{{usId_if_contextual}}.uiValidationHistory_localized[]`: Add `{ reportPath_localized, status_en, timestamp }`.
        *   `memoryBank.userStories.{{usId_if_contextual}}.reasoningChainLinks_en.uiValidation`: Link to English reasoning if kept separate.
        *   (Optional) `memoryBank.identifiedBugs_en`: Add English bug details.
*   **Actions (UO):**
    1.  If major bugs: `ask_followup_question` to user (in `userLanguage`): "UI bugs found for [US_ID/Feature_en]. Create ADO Bug items for: [English list of major bugs, translated for display] ?".
    2.  If yes, UO delegates to `@devops-connector` (**ADO MCP** `create_work_item {type: "Bug", title_en: "...", description_en: "..."}` - UO can translate title/desc for ADO if needed).
*   **Output:** `.pheromone` updated. Bugs potentially created in ADO.

---