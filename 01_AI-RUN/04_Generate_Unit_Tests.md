# Workflow: Unit Test Skeleton Generation (04_Generate_Unit_Tests.md)

**Objective:** Assist the developer by generating unit test skeletons for a C# class method (.NET) or a TypeScript service/component method/function (Angular). The agent relies on source code, functional specifications (US/task ACs), project test conventions, and documents its "chain of thought" for test case selection.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@test-generator-agent`, `@clarification-agent`.

**MCPs Used:** Git Tools MCP (to read target source code), Context7 MCP (for documentation of .NET/Angular test frameworks and mocking libraries), Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The developer (Dev) specifies the target for test generation (e.g., `"AgilePheromind generate unit tests for the calculatePrice method in OrderService"`).
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Context Injection and Preliminary Analysis.**
        *   UO retrieves the target source code (via **Git Tools MCP**) and relevant context from `.pheromone` (`activeTask`, `activeUserStory` ACs, `memoryBank.toolingConfigurations.testFrameworks`, `coding_conventions.md`).
        *   UO evaluates if the source code or specs are clear enough for `@test-generator-agent`. If ambiguous, UO can initiate clarification via `@clarification-agent`.
        *   UO delegates to `@test-generator-agent` with the code and injected context.
        *   **onError:** If Git Tools MCP fails or if the code is not found, notify the user and stop.
    *   **Phase 2: Target Code, Specifications, and Test Framework Analysis by the Agent.**
        *   `@test-generator-agent` analyzes the target, its dependencies, the ACs. Uses **Context7 MCP** for test/mocking frameworks docs.
    *   **Phase 3: Methodical Identification of Test Cases with "Chain of Thought".**
        *   `@test-generator-agent` uses **Sequential Thinking MCP** and must **document its "chain of thought"** for case selection.
        *   **onError:** If the agent cannot deduce test cases due to persistent ambiguities, it reports to the UO. The UO can restart clarification or notify the developer.
    *   **Phase 4: Generation of Test Files and Method Skeletons.**
        *   `@test-generator-agent` generates the files.
    *   **Phase 5: Recording and Reporting.**
        *   Scribe records the information (including the link to the scenarios/chain of thought report) in `.pheromone`.

## Phase Details:

### Phase 1: Context Injection and Preliminary Analysis
*   **Responsible Agent:** `🧐 @uber-orchestrator` (for collection and evaluation), `@clarification-agent` (if needed).
*   **Inputs:** Name of the method/class/component/service and name of the specific method/function to test (from the user).
*   **Actions & Tooling (UO):**
    1.  **Locate and Read Target Code:**
        *   Identify the source file path.
        *   Use **Git Tools MCP** (`get_file_contents {filePath}`) to read the code.
        *   **onError (Git Tools MCP):** If failure, log via Scribe, notify the user "Unable to read source file [filePath]. Check the path and permissions.", stop workflow.
    2.  **Collect Context from `.pheromone`:**
        *   `activeTask.description` and `activeUserStory.acceptanceCriteria`.
        *   `memoryBank.toolingConfigurations.testFrameworks` (.NET and Angular).
        *   Link to `coding_conventions.md` from `memoryBank.projectContext`.
        *   (Optional) Existing tests for similar methods/components in `memoryBank.tasks.{{any_task}}.testCasesGenerated`.
    3.  **Clarity Evaluation:**
        *   The UO performs an initial evaluation: is the source code present and readable? Are the ACs precise enough to guide test generation?
        *   **If ambiguity detected** (e.g., heavily commented source code, ACs too vague to deduce testable behaviors):
            *   Pause workflow (`activeWorkflow.status: 'PendingClarification_TestGen'`).
            *   Delegate to `@clarification-agent` with the context and a question for the developer (e.g., "The code for the `calculatePrice` method seems incomplete. Can you provide a more finalized version or clarify its expected behavior for [case X]?").
            *   Wait for response via `01_AI-RUN/XX_Handle_Clarification_Response.md`.
    4.  If clear, delegate to `@test-generator-agent` by injecting the source code and all collected context.
*   **Output:** If clear, task delegated to `@test-generator-agent` with rich context. If ambiguous, workflow paused.

### Phase 2: Target Code, Specifications, and Test Framework Analysis by the Agent
*   **Responsible Agent:** `@test-generator-agent`
*   **Inputs (Injected by UO):** Target source code, US/task ACs, test framework to use, coding conventions.
*   **Actions & Tooling:**
    1.  Analyze the method/function signature, injected dependencies, internal logic.
    2.  Use **Context7 MCP** (`get_library_docs`) for documentation of the specific test framework (e.g., MSTest, Jasmine) and mocking libraries (e.g., Moq, `jasmine.createSpyObj`).
*   **Output (internal to `@test-generator-agent`):** In-depth understanding of the target and test tools.

### Phase 3: Methodical Identification of Test Cases with "Chain of Thought"
*   **Responsible Agent:** `@test-generator-agent`
*   **Inputs:** Analysis from Phase 2.
*   **Actions & Tooling:**
    1.  Use **Sequential Thinking MCP** to identify cases (nominal, boundary, error, dependencies, UI states for Angular).
    2.  **Document the "Chain of Thought":** For each major identified test case, briefly explain *why* it was selected (e.g., "This case tests the boundary condition X identified in AC Y", "This case verifies the behavior if dependency Z returns an error, which is an expected failure scenario"). This "chain of thought" will be included in the scenarios report.
*   **onError Strategy (for the agent, to report to UO):**
    1.  If, despite the provided context, it is impossible to deduce relevant test cases (e.g., too complex and uncommented logic, still vague ACs):
        *   The agent formulates the precise blocking point.
        *   The UO can then restart clarification via `@clarification-agent` or notify the developer that manual intervention is necessary to define the test cases.
*   **Output (internal to `@test-generator-agent`):** Structured list of test cases and their justification ("chain of thought").

### Phase 4: Generation of Test Files and Method Skeletons
*   **Responsible Agent:** `@test-generator-agent`
*   **Inputs:** List of test cases and their justification (Phase 3). Knowledge of the test framework.
*   **Actions & Tooling:**
    1.  Determine name and location of the test file. Create or open it.
    2.  For each test case, generate the test method (with descriptive name, Arrange/Act/Assert comments), suggest initializations, mocks, and relevant assertions.
    3.  Save the test file.
    4.  Compile the scenarios/chain of thought report (`unit_tests_scenarios_{{targetName}}_{{timestamp}}.md`) in `03_SPECS/Test_Scenarios/`.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "[Number] unit test skeletons generated for `{{targetName}}` in `{{TestFilePath}}`. Scenarios report (with chain of thought) available at `unit_tests_scenarios_{{targetName}}_{{timestamp}}.md`. The developer must complete the sections and finalize the assertions."

### Phase 4.5: Test Coverage and Quality Analysis
*   **Responsible Agent:** `@test-generator-agent`
*   **Inputs:** Generated test skeletons (Phase 4), target source code.
*   **Actions & Tooling:**
    1.  Analyze coverage of generated tests:
        *   Evaluate code line coverage (which parts of the code are tested).
        *   Evaluate branch coverage (which conditions/execution paths are tested).
        *   Identify missing or insufficiently covered test scenarios.
    2.  Evaluate test quality according to criteria:
        *   **Isolation:** Are tests properly isolated (appropriate mocks, no unmocked external dependencies)?
        *   **Determinism:** Will tests always produce the same result under the same conditions?
        *   **Readability:** Are test names and structure clear and descriptive?
        *   **Maintainability:** Are tests easy to maintain when code changes?
    3.  Identify potential improvements:
        *   Propose additional tests for uncovered scenarios.
        *   Suggest improvements for existing tests (better mocks, more precise assertions, etc.).
    4.  Compile a coverage and quality analysis report (`test_coverage_analysis_{{targetName}}_{{timestamp}}.md`) in `03_SPECS/Test_Coverage/`.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Test coverage and quality analysis for `{{targetName}}` completed. Estimated coverage: [percentage]%. [N] well-covered scenarios, [M] partially covered scenarios, [P] missing scenarios. Analysis report available at `test_coverage_analysis_{{targetName}}_{{timestamp}}.md`."

### Phase 5: Recording and Reporting
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** NL summaries from `@test-generator-agent` (generation and analysis).
*   **Actions & Tooling:**
    1.  Interpret summaries via `.swarmConfig`.
    2.  Update `.pheromone`:
        *   `documentationRegistry`: Add `{{TestFilePath}}`, `unit_tests_scenarios_{{targetName}}_{{timestamp}}.md` and `test_coverage_analysis_{{targetName}}_{{timestamp}}.md`.
        *   `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated`: Add `{ path: "{{TestFilePath}}", scenarioReportPath: "unit_tests_scenarios_{{targetName}}_{{timestamp}}.md", coverageReportPath: "test_coverage_analysis_{{targetName}}_{{timestamp}}.md", description: "Unit test skeletons with chain of thought for scenarios and coverage analysis.", timestamp: "{{timestamp}}", estimatedCoverage: "[percentage]%" }`.
        *   `memoryBank.tasks.{{activeTask.id}}.reasoningChainLinks.unitTestGeneration`: Link to scenario and coverage reports.
*   **Output:** `.pheromone` updated. UO informed.

---