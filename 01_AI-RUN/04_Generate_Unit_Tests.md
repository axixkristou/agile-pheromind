# Workflow: Unit Test Skeleton Generation (04_Generate_Unit_Tests.md)

**Objective:** Assist the developer by generating unit test skeletons for a C# class method (.NET) or a TypeScript service/component method/function (Angular). The agent relies on source code, functional specifications (US/task ACs in English from `memoryBank`), project test conventions (English), and documents its "chain of thought" (English) for test case selection. Handles cases where source information is incomplete or ambiguous. The final scenarios report is provided to the user in `currentUser.lastInteractionLanguage`. Test code comments remain in English.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@test-generator-agent`, `@clarification-agent`.

**MCPs Used:** Git Tools MCP, Context7 MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Developer (Dev) specifies the target for test generation (e.g., `"AgilePheromind generate unit tests for method calculatePrice in OrderService"`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Context Injection and Preliminary Analysis.**
        *   UO retrieves target source code (via **Git Tools MCP**) and relevant English context from `.pheromone`.
        *   UO evaluates if source code or English specs are clear enough for `@test-generator-agent`. If ambiguous, UO may initiate clarification via `@clarification-agent` (question translated to `userLanguage`).
        *   UO delegates to `@test-generator-agent` with code, injected English context, and `currentUser.lastInteractionLanguage` as `userLanguageForOutputReport`.
        *   **onError:** If Git Tools MCP fails or code not found, notify user (in `userLanguage`), stop.
    *   **Phase 2: Target Code, Specifications, and Test Framework Analysis by Agent (English).**
        *   `@test-generator-agent` analyzes target, dependencies, English ACs. Uses **Context7 MCP**.
    *   **Phase 3: Methodical Test Case Identification with "Chain of Thought" (English).**
        *   `@test-generator-agent` uses **Sequential Thinking MCP**. Must **document English "chain of thought"**.
        *   **onError:** If agent cannot deduce test cases, it reports to UO (English). UO may restart clarification or notify developer (in `userLanguage`).
    *   **Phase 4: Generation of Test File Skeletons (English Comments) and Scenarios Report (English content, then localized).**
        *   `@test-generator-agent` generates test files with English comments. It drafts the scenarios/reasoning report in English, then translates it to `userLanguageForOutputReport`.
    *   **Phase 5: Recording and Reporting (English Data for Scribe, Localized Report Path).**
        *   Scribe records info in `.pheromone`.

## Phase Details:

### Phase 1: Context Injection and Preliminary Analysis
*   **Responsible Agent:** `🧐 @uber-orchestrator`, `@clarification-agent`.
*   **Inputs:** Target for test generation. `userLanguage`.
*   **Actions & Tooling (UO):**
    1.  **Locate & Read Target Code (Git Tools MCP `get_file_contents`).**
    2.  **onError (Git Tools MCP):** Scribe logs English error. UO notifies user (in `userLanguage`), stop.
    3.  **Collect English Context from `.pheromone`:** `activeTask.description_en`, `activeUserStory.acceptanceCriteria_en`, `memoryBank.toolingConfigurations.testFrameworks`, English `coding_conventions.md` link.
    4.  **Clarity Evaluation:** UO assesses if English code/specs are clear.
        *   **If ambiguity:** Pause workflow. Delegate to `@clarification-agent` with English context and English question (to be translated for user). Await response.
    5.  If clear, delegate to `@test-generator-agent` with source code, injected English context, and `currentUser.lastInteractionLanguage` as `userLanguageForOutputReport`.
*   **Output:** Task delegated or workflow paused.

### Phase 2: Target Code, Specifications, and Test Framework Analysis by Agent (English)
*   **Responsible Agent:** `@test-generator-agent`.
*   **Inputs (Injected by UO):** Target source code, English US/task ACs, test framework, English coding conventions.
*   **Actions & Tooling:** Analyze method/function signature, dependencies, logic (English context). Use **Context7 MCP** for test/mocking framework docs.
*   **Output (internal, English):** In-depth understanding of target and test tools.

### Phase 3: Methodical Test Case Identification with "Chain of Thought" (English)
*   **Responsible Agent:** `@test-generator-agent`.
*   **Inputs:** English analysis from Phase 2.
*   **Actions & Tooling:** Use **Sequential Thinking MCP** for English test cases. **Document English "Chain of Thought"** for case selection in scenarios report draft.
*   **onError Strategy (Agent to UO, English):** If unable to deduce test cases: Agent formulates English blocking point. UO may restart clarification or notify developer (in `userLanguage`).
*   **Output (internal, English):** Structured list of test cases and English justification.

### Phase 4: Generation of Test File Skeletons (English Comments) and Scenarios Report (English content, then localized)
*   **Responsible Agent:** `@test-generator-agent`.
*   **Inputs:** English test cases and justification (Phase 3). Test framework knowledge. `userLanguageForOutputReport`.
*   **Actions & Tooling:**
    1.  Determine test file name/location. Create/open. Generate test methods (descriptive English name, English `// Arrange/Act/Assert` comments). Save test file.
    2.  Compile English scenarios/chain of thought report content.
    3.  **Translate Scenarios Report:** Translate English report content to `userLanguageForOutputReport`. Save as `unit_tests_scenarios_{{targetName_en}}_{{timestamp}}_{{userLanguageForOutputReport}}.md` in `03_SPECS/Test_Scenarios/`.
*   **Output (to Scribe, English NL Summary with path to localized report):** "Skeletons for [Number] unit tests (English comments) generated for `{{targetName_en}}` in `{{TestFilePath}}`. Scenarios report (in `{{userLanguageForOutputReport}}`) at `{{localized_scenario_report_path}}`. Developer to complete."

### Phase 5: Recording and Reporting (English Data for Scribe, Localized Report Path)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** English NL Summary from `@test-generator-agent`.
*   **Actions & Tooling:** Update `.pheromone` (English data):
    *   `documentationRegistry`: Add `{{TestFilePath}}` and `{{localized_scenario_report_path}}`.
    *   `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated_en`: Add `{ path: "{{TestFilePath}}", scenarioReportPath_localized: "{{localized_scenario_report_path}}", description_en: "Unit test skeletons generated. Scenarios/reasoning in report." ... }`.
    *   `memoryBank.tasks.{{activeTask.id}}.reasoningChainLinks_en.unitTestGeneration`: Link to localized report (Scribe notes language).
*   **Output:** `.pheromone` updated. UO informed.

---