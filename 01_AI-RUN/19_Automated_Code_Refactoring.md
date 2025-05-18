# Workflow: Automated Code Refactoring (19_Automated_Code_Refactoring.md)

**Objective:** Automate the refactoring of specific code portions to improve quality, reduce technical debt, or modernize obsolete approaches. The system analyzes the target code, plans modifications, generates preliminary tests to ensure non-regression, implements changes, and validates the result.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@developer-agent`, `@code-reviewer-assistant`, `@test-generator-agent`, `@performance-optimization-agent`.

**MCPs Used:** Git Tools MCP, Context7 MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User specifies the refactoring scope (e.g., `"AgilePheromind refactor AuthService module to use Dependency Injection"`).
2.  **`🧐 @uber-orchestrator`** (UO) takes control.
    *   **Phase 1: Target Code Analysis and Context Injection.**
        *   UO delegates to `@code-reviewer-assistant` to analyze the target code.
        *   UO **injects relevant context** from `memoryBank` (conventions, patterns, architectural decisions).
        *   **onError:** If code inaccessible, log, notify, stop.
    *   **Phase 2: Refactoring Planning (with "Chain of Thought").**
        *   UO delegates to `@developer-agent` to elaborate a detailed plan.
        *   **onError:** If plan impossible to elaborate, log, notify, stop.
    *   **Phase 3: Preliminary Test Generation.**
        *   UO delegates to `@test-generator-agent` to create/complete tests.
        *   **onError:** If test generation fails, evaluate if critical, log, continue or stop.
    *   **Phase 4: Refactoring Execution.**
        *   UO delegates to `@developer-agent` to implement the modifications.
        *   **onError:** If implementation fails, log, notify, stop.
    *   **Phase 5: Validation and Testing.**
        *   UO delegates to `@test-generator-agent` to run tests.
        *   UO delegates to `@code-reviewer-assistant` to verify quality.
        *   UO delegates to `@performance-optimization-agent` to evaluate performance impact.
        *   **onError:** If tests fail, log, notify, propose corrections or rollback.
    *   **Phase 6: Finalization.**
        *   UO delegates to `@developer-agent` to create a branch and PR.
        *   **onError:** If branch/PR creation fails, log, notify.

## Phase Details:

### Phase 1: Target Code Analysis and Context Injection
*   **Responsible Agent:** `@code-reviewer-assistant`
*   **Inputs:** Module/component specification to refactor. `memoryBank.projectContext.repositoryUrl` and `defaultBranch`.
*   **Actions & Tooling:**
    1.  Use **Git Tools MCP** to access source code.
    2.  Analyze code structure, dependencies, and patterns used.
    3.  Use **Context7 MCP** to consult refactoring best practices and patterns.
    4.  Identify problematic areas and improvement opportunities.
*   **onError Strategy (for UO if `@code-reviewer-assistant` reports failure):**
    1.  Scribe logs error in `activeWorkflow.lastError` and `memoryBank.agentActivityLog`.
    2.  UO notifies user: "Unable to analyze code for module [Module]. Error: [Error message]. Please check path or permissions."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Code analysis for '[Module]' completed. Structure: [structure summary]. Dependencies: [list]. Refactoring opportunities identified: [list]. Analysis report: `code_analysis_[Module]_[timestamp].md`."

### Phase 2: Refactoring Planning (with "Chain of Thought")
*   **Responsible Agent:** `@developer-agent`
*   **Inputs:** Analysis report from `@code-reviewer-assistant`. Context injected by UO from `memoryBank`.
*   **Actions & Tooling:**
    1.  Use **Sequential Thinking MCP** to methodically plan the refactoring.
    2.  Elaborate a detailed plan of modifications to make, with justification for each change.
    3.  **Detail the "Chain of Thought":** Explicitly document reasoning behind each refactoring decision.
    4.  Identify potential risks and mitigation strategies.
*   **onError Strategy (for UO if `@developer-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO notifies user: "Unable to elaborate refactoring plan for [Module]. Reason: [Reason]. Please review refactoring scope."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Refactoring plan for '[Module]' elaborated. [N] planned modifications. Identified risks: [list]. Detailed report (with chain of thought): `refactoring_plan_[Module]_[timestamp].md`."

### Phase 3: Preliminary Test Generation
*   **Responsible Agent:** `@test-generator-agent`
*   **Inputs:** Source code, refactoring plan.
*   **Actions & Tooling:**
    1.  Analyze existing test coverage.
    2.  Identify test scenarios needed to ensure non-regression.
    3.  Use **Context7 MCP** to consult testing best practices.
    4.  Generate or complete unit/integration tests.
*   **onError Strategy (for UO if `@test-generator-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO evaluates if test generation is critical for this refactoring.
    3.  If critical, notify user and stop. Otherwise, log a warning and continue.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Preliminary tests for '[Module]' refactoring generated. [N] unit tests, [M] integration tests. Estimated coverage: [percentage]. Test files: [list]."

### Phase 4: Refactoring Execution
*   **Responsible Agent:** `@developer-agent`
*   **Inputs:** Source code, refactoring plan, preliminary tests.
*   **Actions & Tooling:**
    1.  Implement modifications according to the plan.
    2.  Use **Context7 MCP** to consult best practices and patterns.
    3.  Apply modifications incrementally, checking compilation at each step.
*   **onError Strategy (for UO if `@developer-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO notifies user: "Failed to implement refactoring for [Module]. Error: [Message]. Please check the report for more details."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Refactoring of '[Module]' implemented. [N] files modified. Main modifications: [summary]. Implementation report: `refactoring_implementation_[Module]_[timestamp].md`."

### Phase 5: Validation and Testing
*   **Responsible Agent:** `@test-generator-agent`, `@code-reviewer-assistant`, `@performance-optimization-agent`
*   **Inputs:** Refactored code, preliminary tests.
*   **Actions & Tooling (`@test-generator-agent`):**
    1.  Run tests to verify non-regression.
    2.  Document test results.
*   **Actions & Tooling (`@code-reviewer-assistant`):**
    1.  Verify refactored code quality (conventions, readability, maintainability).
    2.  Use **Context7 MCP** to validate adherence to best practices.
*   **Actions & Tooling (`@performance-optimization-agent`):**
    1.  Evaluate refactoring impact on performance.
    2.  Identify potential additional optimizations.
*   **onError Strategy (for UO if tests fail):**
    1.  Scribe logs error.
    2.  UO notifies user: "Non-regression tests failed after refactoring [Module]. [N] failing tests. Please consult the report for details."
    3.  Propose corrections or rollback.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Validation of '[Module]' refactoring completed. Tests: [result]. Code quality: [evaluation]. Performance impact: [summary]. Complete report: `refactoring_validation_[Module]_[timestamp].md`."

### Phase 6: Finalization
*   **Responsible Agent:** `@developer-agent`
*   **Inputs:** Validated refactored code.
*   **Actions & Tooling:**
    1.  Use **Git Tools MCP** to create a branch (e.g., `refactor/[Module]-[Description]`).
    2.  Commit changes with descriptive message.
    3.  Create a Pull Request with detailed refactoring description.
*   **onError Strategy (for UO if `@developer-agent` reports Git failure):**
    1.  Scribe logs error.
    2.  UO notifies user: "Error creating branch/PR for [Module] refactoring. Git error: [Message]. Changes are available locally."
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Refactoring of '[Module]' finalized. Branch: '[branchName]'. PR: '[prLink]'. Description: '[description]'. Refactoring is ready for review."

---
