# Workflow: Pull Request Review Assistance (05_Review_Pull_Request.md)

**Objective:** Assist the Tech Lead in reviewing an Azure DevOps Pull Request (PR). The system retrieves PR details, analyzes code changes (quality, security via `@security-analyst-agent`, conventions), generates a detailed review report (with English "chain of thought" for raised issues, then translated to user's language), stores it in a PR branch-specific directory, and handles retrieval or analysis errors. User interaction (notifications) will be in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@code-reviewer-assistant`, `@security-analyst-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Git Tools MCP, Context7 MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Tech Lead provides Azure DevOps PR ID (e.g., `"AgilePheromind analyze PR Azure#456"`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: PR Information Retrieval and Review Directory Creation/Validation.**
        *   UO delegates to `@devops-connector` for PR details (including `sourceBranchName`).
        *   UO (or `@code-reviewer-assistant`) ensures existence of `04_PR_REVIEWS/[sanitized_PR_source_branch_name]/` directory.
        *   **onError:** If ADO MCP fails, log, notify user (in `userLanguage`), stop.
    *   **Phase 2: Code Changes Retrieval and Preparation.**
        *   UO delegates to `@code-reviewer-assistant` for diffs (via ADO or Git MCP).
        *   **onError:** If diff retrieval fails, log, notify user (in `userLanguage`), stop.
    *   **Phase 3: Static, Heuristic, and Security Code Analysis with "Chain of Thought" (English).**
        *   UO **injects targeted English context** to `@code-reviewer-assistant`.
        *   `@code-reviewer-assistant` collaborates with `@security-analyst-agent`. Uses **Context7 MCP**. Must **document English "chain of thought"** for major issues.
        *   If analysis blocked by ambiguous code, `@code-reviewer-assistant` reports (English) to UO, who may initiate clarification via `@clarification-agent` (question to PR dev in their language).
    *   **Phase 4: Detailed Review Report Generation (English Content, Final Output Localized) and Storage.**
        *   UO delegates to `@code-reviewer-assistant` to compile English report (including "chain of thought"), translate it to `currentUser.lastInteractionLanguage`, and save localized report in the dedicated directory.
    *   **Phase 5: Notification and `.pheromone` Update (English Data for Scribe, Localized Report Path).**
        *   Scribe records information and link to localized report.

## Phase Details:

### Phase 1: PR Information Retrieval and Review Directory Creation/Validation
*   **Responsible Agent:** `@devops-connector`, then UO (or `@code-reviewer-assistant`).
*   **Inputs:** PR ID from UO.
*   **Actions & Tooling (`@devops-connector`):**
    1.  Use **Azure DevOps MCP** `get_pull_request_details {id: PR_ID}` (sourceBranchName, targetBranchName, etc.).
*   **onError Strategy (for UO):** If ADO MCP fails, Scribe logs English error. UO notifies user (in `currentUser.lastInteractionLanguage`): "Unable to retrieve PR Azure#{{prId}} details. MCP Error: [Message].", stop workflow.
*   **Output (`@devops-connector` to Scribe/UO, English NL Summary):** "PR Azure#{{prId}} details retrieved. Source branch: '{{sourceBranchName}}'. Log: `azure_pr_{{prId}}_details_{{timestamp}}.json`."
*   **Actions & Tooling (UO or `@code-reviewer-assistant` after Scribe updates `.pheromone`):**
    1.  Read `activePullRequest.sourceBranchName_en`. Sanitize it for directory name.
    2.  Create `reviewDirPath = "04_PR_REVIEWS/{{activePullRequest.sourceBranchName_sanitized}}/"`. Ensure directory exists.
    3.  Store `reviewDirPath` in `.pheromone.activePullRequest.reviewDirectoryPath` (via Scribe).

### Phase 2: Code Changes Retrieval and Preparation
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** `activePullRequest` from `.pheromone`.
*   **Actions & Tooling:**
    1.  Use **Azure DevOps MCP** (`get_pull_request_changed_files`, `get_pull_request_commits`/`get_diff`).
    2.  Or, **Git Tools MCP** (`git_fetch_origin`, `git_diff`).
*   **onError Strategy (for UO):** If diff retrieval fails, Scribe logs English error. UO notifies user (in `currentUser.lastInteractionLanguage`): "Unable to retrieve diffs for PR Azure#{{prId}}. MCP/Git Error: [Message].", stop workflow.
*   **Output (internal to `@code-reviewer-assistant`, English context):** Code diffs ready.

### Phase 3: Static, Heuristic, and Security Code Analysis with "Chain of Thought" (English)
*   **Responsible Agent:** `@code-reviewer-assistant` (collaborates with `@security-analyst-agent`).
*   **Inputs (Injected by UO, English context):** Code changes. `memoryBank` context: English `coding_conventions.md`, `technicalDebtItems_en`, relevant English `architecturalDecisions_en`.
*   **Actions & Tooling (`@code-reviewer-assistant`):**
    1.  **Analysis (English logic):** Conventions, quality, smells, bugs, tests. Use **Context7 MCP**.
    2.  **Security Coordination (English comms):** Delegate to `@security-analyst-agent`.
    3.  **"Chain of Thought" (English):** For significant issues, document English reasoning.
    4.  **Ambiguity Management:** If code obscure: Report (English) to UO for potential clarification via `@clarification-agent`.
*   **Actions (`@security-analyst-agent`):** Security analysis. Concise English report to `@code-reviewer-assistant`.
*   **Output (compiled by `@code-reviewer-assistant`, English):** Consolidated list of issues, severity, English "chain of thought".

### Phase 4: Detailed Review Report Generation (English Content, Final Output Localized) and Storage
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** Consolidated English issues (Phase 3). `activePullRequest.reviewDirectoryPath`. `currentUser.lastInteractionLanguage` (passed from UO as `userLanguageForOutput`).
*   **Actions & Tooling:**
    1.  **Compile English Report Content:** Draft full English Markdown report. Include: PR Info, Summary, Issue Details (severity, location, English reasoning/chain of thought, suggestion).
    2.  **Translate Report:** Translate the entire English report content to `userLanguageForOutput`, preserving Markdown and ensuring technical accuracy.
    3.  Save localized report as `pr_{{activePullRequest.id}}_review_report_{{timestamp}}_{{userLanguageForOutput}}.md` in `{{activePullRequest.reviewDirectoryPath}}`.
*   **Output (to Scribe, English NL Summary with path to localized report):** "PR Azure#{{activePullRequest.id}} review complete. [Stats]. Report (in `{{userLanguageForOutput}}`, English reasoning available or summarized) at `{{localized_fullReportPath}}`. Recommendation (English): [Action]."

### Phase 5: Notification and `.pheromone` Update (English Data for Scribe, Localized Report Path)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** English NL Summary from `@code-reviewer-assistant`, including path to localized report.
*   **Actions & Tooling:**
    1.  Update `.pheromone` (Scribe uses English for internal `memoryBank` fields where specified):
        *   `documentationRegistry`: Add `{{localized_fullReportPath}}`.
        *   `memoryBank.pullRequests.{{activePullRequest.id}}`: Update `status: "ReviewedByAI"`, `reviewReportPath_localized: "{{localized_fullReportPath}}"` (new field), `lastReviewTimestamp`, `issuesFoundCounts` (from English stats), `summary_en` (English summary of review), `reasoningChainLinks_en.reviewReport` (can point to an English version of the reasoning if agent produced one, or the localized report if reasoning is embedded and translated).
    2.  (Optional) UO instructs `@devops-connector` to comment on PR in ADO (comment can be English or UO can request translation for `userLanguage`).
*   **Output:** `.pheromone` updated. UO informs Tech Lead (in `userLanguage`).

---