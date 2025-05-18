# Workflow: Pull Request Review Assistance (05_Review_Pull_Request.md)

**Objective:** Help the Tech Lead review an Azure DevOps Pull Request (PR). The system retrieves PR details, analyzes code changes (quality, security, conventions), generates a detailed review report (with "chain of thought" for raised issues), stores it in a branch-specific directory, and handles retrieval or analysis errors.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@code-reviewer-assistant`, `@security-analyst-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Git Tools MCP, Context7 MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The Tech Lead provides the Azure DevOps PR ID (e.g., `"AgilePheromind analyze PR Azure#456"`).
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: PR Information Retrieval and Review Directory Creation/Validation.**
        *   UO delegates to `@devops-connector` for PR details (including `sourceBranchName`).
        *   UO (or `@code-reviewer-assistant`) ensures the existence of the directory `04_PR_REVIEWS/[PR_source_branch_name_sanitized]/`.
        *   **onError:** If ADO MCP fails for `get_pull_request_details`, log, notify, stop.
    *   **Phase 2: Code Changes Retrieval and Preparation.**
        *   UO delegates to `@code-reviewer-assistant` to get the diffs (via ADO or Git MCP).
        *   **onError:** If diffs retrieval fails, log, notify, stop.
    *   **Phase 3: Static, Heuristic, and Security Code Analysis with "Chain of Thought".**
        *   UO **injects targeted context** (coding conventions, relevant technical debt history from `memoryBank`) to `@code-reviewer-assistant`.
        *   `@code-reviewer-assistant` collaborates with `@security-analyst-agent`. Uses **Context7 MCP**. Must **document the "chain of thought"** for major identified issues.
        *   If the analysis is blocked by very ambiguous code, `@code-reviewer-assistant` reports it to the UO, who can initiate clarification via `@clarification-agent` asking the PR developer to explain a code section.
    *   **Phase 4: Detailed Review Report Generation and Storage.**
        *   UO delegates to `@code-reviewer-assistant` to compile and save the report (including the "chain of thought") in the dedicated directory.
    *   **Phase 5: Notification and `.pheromone` Update.**
        *   Scribe records the information and the link to the report.

## Phase Details:

### Phase 1: PR Information Retrieval and Review Directory Creation/Validation
*   **Responsible Agent:** `@devops-connector`, then `🧐 @uber-orchestrator` (or `@code-reviewer-assistant`).
*   **Inputs:** PR ID.
*   **Actions & Tooling (`@devops-connector`):**
    1.  Use **Azure DevOps MCP** `get_pull_request_details {id: PR_ID}` (sourceBranchName, targetBranchName, etc.).
*   **onError Strategy (for UO if `@devops-connector` reports failure):**
    1.  Scribe logs in `activeWorkflow.lastError`.
    2.  UO notifies the user: "Unable to retrieve details for PR Azure#{{prId}}. MCP Error: [Message]. Check ID or ADO connection."
    3.  Stop workflow.
*   **Output (`@devops-connector` to Scribe/UO):** NL Summary: "PR Azure#{{prId}} details retrieved. Source branch: '{{sourceBranchName}}'. Log: `azure_pr_{{prId}}_details_{{timestamp}}.json`."
*   **Actions & Tooling (UO or `@code-reviewer-assistant` after `.pheromone` update by Scribe):**
    1.  Read `activePullRequest.sourceBranchName`. Create `reviewDirPath = "04_PR_REVIEWS/{{activePullRequest.sourceBranchName_sanitized}}/"`. Create directory if absent.
    2.  Store `reviewDirPath` in `.pheromone.activePullRequest.reviewDirectoryPath` (via Scribe).

### Phase 2: Code Changes Retrieval and Preparation
*   **Responsible Agent:** `@code-reviewer-assistant`
*   **Inputs:** `activePullRequest` from `.pheromone`.
*   **Actions & Tooling:**
    1.  Use **Azure DevOps MCP** (`get_pull_request_changed_files`, `get_pull_request_commits`/`get_diff`).
    2.  Or, if needed, **Git Tools MCP** (`git_fetch_origin`, `git_diff`).
*   **onError Strategy (for UO if `@code-reviewer-assistant` reports failure):**
    1.  Scribe logs.
    2.  UO notifies: "Unable to retrieve diffs for PR Azure#{{prId}}. MCP/Git Error: [Message]."
    3.  Stop workflow.
*   **Output (internal to `@code-reviewer-assistant`):** Code diffs ready.

### Phase 3: Static, Heuristic, and Security Code Analysis with "Chain of Thought"
*   **Responsible Agent:** `@code-reviewer-assistant` (collaborates with `@security-analyst-agent`).
*   **Inputs (Injected by UO):**
    *   Code changes (Phase 2).
    *   Context from `memoryBank`: `projectContext.codingConventionsLink`, `projectContext.designConventionsLink`, `technicalDebtItems` (to see if the PR addresses/introduces debt), relevant `architecturalDecisions`.
*   **Actions & Tooling (`@code-reviewer-assistant`):**
    1.  **Conventions & Quality Analysis:** Check vs conventions. Identify code smells, potential bugs. Evaluate readability, maintainability, tests. Use **Context7 MCP** for library usage.
    2.  **Security Coordination:** Delegate security analysis of diffs to `@security-analyst-agent`.
    3.  **"Chain of Thought":** For each significant issue (Major/Critical) identified, briefly document the reasoning: "Issue X detected because [observed condition Y] violates [convention Z] or introduces [risk W]. The best practice (Context7/convention) is [practice]."
    4.  **Ambiguity Management:** If a code section is too complex or its intention is completely obscure, making analysis impossible:
        *   Report to the UO: "Analysis blocked on file [X] line [Y] for PR Azure#{{prId}}. Ambiguous code. Suggested question for dev: 'Can you explain the logic/intention of this section?'".
        *   The UO can then pause the workflow and initiate clarification via `@clarification-agent`.
*   **Actions & Tooling (`@security-analyst-agent`):**
    *   Targeted security analysis (OWASP, etc.). Concise report to `@code-reviewer-assistant`.
*   **Output (compiled by `@code-reviewer-assistant`):** Consolidated list of issues, with severity and "chain of thought" for major points.

### Phase 3.5: Performance and Accessibility Analysis
*   **Responsible Agent:** `@performance-optimization-agent`, `@accessibility-compliance-agent` (if UI modifications)
*   **Inputs:** Code modifications (Phase 2), `memoryBank` context.
*   **Actions & Tooling (`@performance-optimization-agent`):**
    1.  Analyze the impact of changes on performance:
        *   Use **Context7 MCP** to consult performance best practices.
        *   Use **Sequential Thinking MCP** to methodically analyze the impact of changes.
        *   Identify potential performance issues (.NET: excessive allocations, inefficient LINQ, EF Core N+1; Angular: change detection, missing OnPush, inefficient RxJS).
    2.  **"Chain of Thought":** For each identified performance issue, document the reasoning: "Performance issue X detected because [implementation Y] can cause [impact Z] in [context W]. The recommended approach is [alternative]."
    3.  Suggest specific optimizations with code examples.
*   **Actions & Tooling (`@accessibility-compliance-agent`, if UI modifications):**
    1.  Verify WCAG compliance of modified UI components:
        *   Use **Context7 MCP** to consult WCAG guidelines.
        *   Analyze key aspects (contrast, keyboard navigation, alt texts, ARIA).
    2.  Categorize issues by compliance level (A, AA, AAA).
    3.  Propose specific corrections.
*   **Output (to `@code-reviewer-assistant`):** Performance and accessibility analyses to be integrated into the review report.

### Phase 4: Detailed Review Report Generation and Storage
*   **Responsible Agent:** `@code-reviewer-assistant`
*   **Inputs:** Consolidated issues (Phase 3), performance and accessibility analyses (Phase 3.5), `activePullRequest.reviewDirectoryPath`.
*   **Actions & Tooling:**
    1.  Write Markdown report (`pr_{{activePullRequest.id}}_review_report_{{timestamp}}.md`). Include: PR Info, Summary, Issue Details (with severity, location, **reasoning/chain of thought for key points**, suggestion), dedicated sections for performance and accessibility (if applicable).
    2.  Save in `{{activePullRequest.reviewDirectoryPath}}`.
*   **Output (to Scribe):** NL Summary: "PR Azure#{{activePullRequest.id}} review completed. [Issue stats]. [Performance stats]. [Accessibility stats]. Report (with chain of thought): `{{fullReportPath}}`. Recommendation: [Action]."

### Phase 5: Notification and `.pheromone` Update
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** NL summary from `@code-reviewer-assistant`.
*   **Actions & Tooling:**
    1.  Update `.pheromone`:
        *   `documentationRegistry`: Add report path.
        *   `memoryBank.pullRequests.{{activePullRequest.id}}`: Update `status: "ReviewedByAI"`, `reviewReportPath`, `lastReviewTimestamp`, `issuesFoundCounts`, `summary`, and `reasoningChainLinks.reviewReport` (which points to the report containing the chains of thought).
    2.  (Optional) UO instructs `@devops-connector` to comment on the PR in ADO.
*   **Output:** `.pheromone` updated. UO informs Tech Lead.

---