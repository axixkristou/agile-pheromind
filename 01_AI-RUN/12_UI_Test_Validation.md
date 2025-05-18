# Workflow: User Interface Validation (UI Test Validation) (12_UI_Test_Validation.md)

**Objective:** Assist the tester in manual or semi-automated validation of the user interface (UI) of a feature or User Story (US). The agent uses the Browser Tools MCP for interactions/captures, compares against design specifications (with a "chain of thought" for significant discrepancies), and handles errors or ambiguities in the specs.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@tester-ui-validator-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Browser Tools MCP (Puppeteer or Playwright), Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Tester/PO requests UI validation for US/Feature + test environment URL (e.g., `"AgilePheromind validate UI US Azure#12323 on https://test.myapp.com"`).
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Specifications Retrieval, ACs and Context Injection.**
        *   UO delegates to `@devops-connector` for ACs (via **ADO MCP**).
        *   UO **injects context** to `@tester-ui-validator-agent`: ACs, `design_conventions.md`, mockups (from `documentationRegistry`/`memoryBank`).
        *   UO evaluates clarity of UI specs. If ambiguous (e.g., design contradicts ACs), UO initiates clarification via `@clarification-agent` with PO/Designer.
        *   **onError:** If ADO MCP or specs access fails, notify, stop.
    *   **Phase 2: Detailed UI Test Scenario Definition.**
        *   UO delegates to `@tester-ui-validator-agent`.
    *   **Phase 3: Interactions Execution and Captures via Browser Tools MCP.**
        *   UO delegates to `@tester-ui-validator-agent`.
        *   **onError (Browser Tools MCP):** If MCP fails (e.g., element not found, navigation impossible), the agent notes it, tries to continue if possible, and reports it in the report.
    *   **Phase 4: Results Analysis and Report Generation (with "Chain of Thought" for Discrepancies).**
        *   UO delegates to `@tester-ui-validator-agent`. For major discrepancies, the agent **documents its "chain of thought"** (why it considers it a deviation from specs).
    *   **Phase 5: Report Recording and Bug Creation (Optional).**
        *   Scribe records. UO can propose bug creation in ADO via `@devops-connector`.

## Phase Details:

### Phase 1: Specifications Retrieval, ACs and Context Injection
*   **Responsible Agent:** UO, `@devops-connector`, `@clarification-agent`.
*   **Inputs:** US/Feature ID, test env URL.
*   **Actions (UO & `@devops-connector`):**
    1.  `@devops-connector` retrieves ACs of the US (if US ID) via **ADO MCP** `get_work_item_details`.
    2.  **onError (ADO MCP):** UO logs via Scribe, notifies, stops/continues with warning.
    3.  Scribe updates `memoryBank.userStories.{{usId}}.acceptanceCriteria`.
    4.  UO injects to `@tester-ui-validator-agent`:
        *   Retrieved ACs.
        *   Path to `design_conventions.md` (from `documentationRegistry`).
        *   Paths to mockups/prototypes for the US/feature (from `memoryBank.userStories.{{usId}}.designArtifactLinks` or `documentationRegistry`).
        *   (Optional) Previous UI validation reports for similar features.
*   **Actions (UO - Clarity Evaluation):**
    1.  The UO evaluates if the ACs and injected design specs are consistent and sufficiently precise.
    2.  **If major ambiguity** (e.g., AC describes behavior X, mockup shows Y):
        *   Pause workflow (`activeWorkflow.status: 'PendingClarification_UITestSpec'`).
        *   Delegate to `@clarification-agent` with context and a question for the PO/Designer (e.g., "For US Azure#{{usId}}, AC X says [behavior], but mockup Y shows [something else]. What is the correct expectation for UI validation?").
        *   Wait for response via `01_AI-RUN/XX_Handle_Clarification_Response.md`.
*   **Output:** Rich and clarified (if needed) context for `@tester-ui-validator-agent`.

### Phase 2: Detailed UI Test Scenario Definition
*   **Responsible Agent:** `@tester-ui-validator-agent`.
*   **Inputs:** Injected context (Phase 1).
*   **Actions:**
    1.  Define validation steps (actions, visual/functional expectations) based on ACs and design specs.
    2.  Document scenario (for final report).
*   **Output (internal):** UI test plan.

### Phase 3: Interactions Execution and Captures via Browser Tools MCP
*   **Responsible Agent:** `@tester-ui-validator-agent`.
*   **Inputs:** Test scenario (Phase 2), test env URL.
*   **Actions (Browser Tools MCP - Puppeteer/Playwright):**
    1.  For each step: `navigate`, `click`, `fill_form_field`, `take_screenshot` (different viewports, in `04_PR_REVIEWS/[branch]/screenshots/` or `03_SPECS/UI_Validation_Screenshots/[feature]/`), `execute_script` (CSS, content), `get_console_logs`.
*   **onError (Browser Tools MCP):**
    *   If an MCP action fails (e.g., `click {selector}` because element not found):
        *   The agent logs the precise MCP error.
        *   Tries to continue the scenario if the failure is not blocking for subsequent steps.
        *   All MCP failures will be explicitly listed in the validation report.
*   **Output (internal):** Screenshots, browser data, MCP error logs.

### Phase 4: Results Analysis and Report Generation (with "Chain of Thought" for Discrepancies)
*   **Responsible Agent:** `@tester-ui-validator-agent`.
*   **Inputs:** Data from Phase 3, specs from Phase 1.
*   **Actions:**
    1.  Compare captures/data with mockups/conventions/ACs. Check console logs.
    2.  Write MD report (`ui_validation_report_[US_ID_or_Feature]_[timestamp].md`) in `03_SPECS/UI_Validation_Reports/` or PR dir.
        *   Structure: Scope, Summary, Validation points (Expected, Observed (with screenshot/data), Status, Comment).
        *   **"Chain of Thought" for Significant Discrepancies:** For each major "Failure", explain *why* it's a deviation from specs (e.g., "Button X uses color `colors.red['300']` while `design_conventions.md` Section 4.1 specifies `colors.red['500']` for destructive actions. This lacks contrast and doesn't respect the defined semantics.").
        *   List MCP failures from Phase 3.
        *   List bugs/issues with severity.
*   **Output (to Scribe):** NL Summary: "UI validation '[US_ID/Feature]' completed. Status: [Global]. [N_bugs] bugs. Report (with chain of thought for discrepancies): `ui_validation_report...md`."

### Phase 5: Report Recording and Bug Creation (Optional)
*   **Responsible Agent:** Scribe, UO, `@devops-connector`.
*   **Inputs:** NL summary from `@tester-ui-validator-agent`.
*   **Actions (Scribe):**
    1.  Update `.pheromone`:
        *   `documentationRegistry`: Add report path.
        *   `memoryBank.userStories.{{usId}}.uiValidationHistory[]`: Add `{reportPath, status, timestamp}`.
        *   `memoryBank.userStories.{{usId}}.reasoningChainLinks.uiValidation`: Link to report (containing chain of thought for discrepancies).
        *   (Optional) `memoryBank.identifiedBugs`: Add listed bugs.
*   **Actions (UO):**
    1.  If major bugs: `ask_followup_question` to user "UI bugs found. Do you want to create Bug items in ADO for: [list of major bugs]?".
    2.  If yes, UO delegates to `@devops-connector` (**ADO MCP** `create_work_item {type: "Bug", ...}`).
*   **Output:** `.pheromone` updated. Bugs potentially created in ADO.

---