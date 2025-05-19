# Workflow: Unit Test Skeleton Generation (04_Generate_Unit_Tests.md)

**Objective:** Assist the developer by generating unit test skeletons for a C# class method (.NET) or a TypeScript service/component method/function (Angular). The agent relies on source code, functional specifications (US/task ACs in English from `memoryBank`), project test conventions (English), and documents its "chain of thought" (English) for test case selection. Handles cases where source information is incomplete or ambiguous. User interaction is in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@test-generator-agent`, `@clarification-agent`.

**MCPs Used:** Git Tools MCP, Context7 MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Developer (Dev) specifies the target for test generation (e.g., `"AgilePheromind generate unit tests for method calculatePrice in OrderService"`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Context Injection and Preliminary Analysis.**
        *   UO retrieves target source code (via **Git Tools MCP**) and relevant English context from `.pheromone` (`activeTask`, `activeUserStory` English ACs, `memoryBank.toolingConfigurations.testFrameworks`, English `coding_conventions.md` link).
        *   UO evaluates if source code or English specs are clear enough for `@test-generator-agent`. If ambiguous, UO may initiate clarification via `@clarification-agent` (question translated to `userLanguage`).
        *   UO delegates to `@test-generator-agent` with code and injected English context.
        *   **onError:** If Git Tools MCP fails or code is not found, notify user (in `userLanguage`), stop.
    *   **Phase 2: Target Code, Specifications, and Test Framework Analysis by Agent (English).**
        *   `@test-generator-agent` analyzes target, dependencies, English ACs. Uses **Context7 MCP** for test/mocking framework docs.
    *   **Phase 3: Methodical Test Case Identification with "Chain of Thought" (English).**
        *   `@test-generator-agent` uses **Sequential Thinking MCP**. Must **document its English "chain of thought"** for case selection.
        *   **onError:** If agent cannot deduce test cases due to persistent ambiguities, it reports to UO (English). UO may restart clarification or notify developer (in `userLanguage`).
    *   **Phase 4: Generation of Test File Skeletons and Methods (English Comments).**
        *   `@test-generator-agent` generates files with English comments. The scenarios report is generated in English and then translated to `userLanguage` for the final output file.
    *   **Phase 5: Recording and Reporting (English Data for Scribe, Localized Report Path).**
        *   Scribe records info (including link to localized scenarios/chain of thought report) in `.pheromone`.

## Phase Details:

### Phase 1: Context Injection and Preliminary Analysis
*   **Responsible Agent:** `🧐 @uber-orchestrator` (for collection & evaluation), `@clarification-agent` (if needed).
*   **Inputs:** Target for test generation. `userLanguage`.
*   **Actions & Tooling (UO):**
    1.  **Locate & Read Target Code:** Identify source file path. Use **Git Tools MCP** (`get_file_contents {filePath}`).
    2.  **onError (Git Tools MCP):** If failure, Scribe logs English error. UO notifies user (in `userLanguage`): "Unable to read source file [filePath]. Check path/permissions.", stop workflow.
    3.  **Collect English Context from `.pheromone`:** `activeTask.description_en`, `activeUserStory.acceptanceCriteria_en`. `memoryBank.toolingConfigurations.testFrameworks`. Link to English `coding_conventions.md`. Optionally, existing tests for similar methods.
    4.  **Clarity Evaluation:** UO assesses if English code/specs are clear.
        *   **If ambiguity detected:** Pause workflow. Delegate to `@clarification-agent` with English context and English question (to be translated for user). Await response.
    5.  If clear, delegate to `@test-generator-agent` with source code and injected English context, and `currentUser.lastInteractionLanguage` as `userLanguageForOutput`.
*   **Output:** Task delegated to `@test-generator-agent` with rich English context and output language, or workflow paused.

### Phase 2: Target Code, Specifications, and Test Framework Analysis by Agent (English)
*   **Responsible Agent:** `@test-generator-agent`.
*   **Inputs (Injected by UO):** Target source code, English US/task ACs, test framework, English coding conventions.
*   **Actions & Tooling:**
    1.  Analyze method/function signature, dependencies, internal logic (all English context).
    2.  Use **Context7 MCP** (`get_library_docs`) for specific test framework and mocking library documentation.
*   **Output (internal, English):** In-depth understanding of target and test tools.

### Phase 3: Methodical Test Case Identification with "Chain of Thought" (English)
*   **Responsible Agent:** `@test-generator-agent`.
*   **Inputs:** English analysis from Phase 2.
*   **Actions & Tooling:**
    1.  Use **Sequential Thinking MCP** to identify English test cases.
    2.  **Document "Chain of Thought" (English):** For each major test case, explain *why* it was selected. This forms part of the English scenarios report content.
*   **onError Strategy (Agent to UO, English):** If unable to deduce test cases: Agent formulates English blocking point. UO may restart clarification or notify developer (in `userLanguage`).
*   **Output (internal, English):** Structured list of test cases and their English justification.

### Phase 4: Generation of Test File Skeletons and Methods (English Comments)
*   **Responsible Agent:** `@test-generator-agent`.
*   **Inputs:** English test cases and justification (Phase 3). Test framework knowledge. `userLanguageForOutput`.
*   **Actions & Tooling:**
    1.  Determine test file name/location. Create/open.
    2.  For each test case, generate test method (descriptive English name, English `// Arrange`, `// Act`, `// Assert` comments), suggest initializations, mocks, relevant assertions.
    3.  Save test file (code comments remain English).
    4.  Compile English scenarios/chain of thought report content.
    5.  **Translate Scenarios Report:** Translate the English report content to `userLanguageForOutput`, preserving Markdown. Save as `unit_tests_scenarios_{{targetName_en}}_{{timestamp}}_{{userLanguageForOutput}}.md` in `03_SPECS/Test_Scenarios/`.
*   **Output (to Scribe, English NL Summary with path to localized report):** "Skeletons for [Number] unit tests generated for `{{targetName_en}}` in `{{TestFilePath}}`. Scenarios report (in `{{userLanguageForOutput}}`) at `{{localized_scenario_report_path}}`. Developer to complete assertions."

### Phase 5: Recording and Reporting (English Data for Scribe, Localized Report Path)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** English NL Summary from `@test-generator-agent`, including path to the localized report.
*   **Actions & Tooling:**
    1.  Interpret summary via `.swarmConfig`.
    2.  Update `.pheromone` (English data):
        *   `documentationRegistry`: Add `{{TestFilePath}}` (English comments) and `{{localized_scenario_report_path}}`.
        *   `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated_en`: Add `{ path: "{{TestFilePath}}", scenarioReportPath_localized: "{{localized_scenario_report_path}}", description_en: "Unit test skeletons generated. Scenarios and reasoning in linked report.", timestamp: "{{timestamp}}" }`.
        *   `memoryBank.tasks.{{activeTask.id}}.reasoningChainLinks_en.unitTestGeneration`: Link to localized scenarios report (Scribe notes language).
*   **Output:** `.pheromone` updated. UO informed.

---