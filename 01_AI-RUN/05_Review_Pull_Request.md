# Workflow: Pull Request Review Assistance (05_Review_Pull_Request.md)

**Objective:** Assist the Tech Lead in reviewing an Azure DevOps Pull Request (PR). The system retrieves PR details, analyzes code changes (quality, security via `@security-analyst-agent`, conventions), generates a detailed review report (English "chain of thought" for issues, final report translated to user's language), stores it in a PR branch-specific directory, and handles errors. User interaction (notifications) will be in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@code-reviewer-assistant`, `@security-analyst-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Git Tools MCP, Context7 MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Tech Lead provides ADO PR ID (e.g., `"AgilePheromind analyze PR Azure#456"`). `userLanguage` passed by `🎩 @head-orchestrator`.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: PR Info Retrieval & Review Directory Setup.**
        *   UO delegates to `@devops-connector` for PR details (incl. `sourceBranchName`). Scribe stores English `sourceBranchName_en`.
        *   UO/`@code-reviewer-assistant` ensures `04_PR_REVIEWS/[sanitized_PR_source_branch_name_en]/` exists.
        *   **onError:** If ADO MCP fails, log, notify user (in `userLanguage`), stop.
    *   **Phase 2: Code Changes Retrieval.**
        *   UO delegates to `@code-reviewer-assistant` for diffs.
        *   **onError:** If diff retrieval fails, log, notify user (in `userLanguage`), stop.
    *   **Phase 3: Code Analysis (English "Chain of Thought").**
        *   UO **injects English context** to `@code-reviewer-assistant`.
        *   `@code-reviewer-assistant` collaborates with `@security-analyst-agent`. Uses **Context7 MCP**. Documents English "chain of thought" for major issues.
        *   If ambiguous code blocks analysis, `@code-reviewer-assistant` reports (English) to UO for potential clarification via `@clarification-agent`.
    *   **Phase 4: Review Report (English Content, Final Output Localized).**
        *   UO delegates to `@code-reviewer-assistant` to compile English report, translate to `userLanguage`, save localized report.
    *   **Phase 5: `.pheromone` Update (English Data, Localized Report Path).**
        *   Scribe records.

## Phase Details:

### Phase 1: PR Info Retrieval & Review Directory Setup
*   **Responsible Agent:** `@devops-connector`, UO/`@code-reviewer-assistant`, Scribe.
*   **Inputs:** PR ID.
*   **Actions (`@devops-connector`):** **ADO MCP** `get_pull_request_details {id: PR_ID}` (gets `sourceBranchName_origLang`).
*   **onError (ADO MCP):** UO logs (English), notifies user (in `userLanguage`), stops.
*   **Output (`@devops-connector` -> Scribe, English NL Summary with original lang field):** "PR Azure#{{prId}} details retrieved. Source branch (orig lang): '{{sourceBranchName_origLang}}'. Log: `azure_pr_{{prId}}_details_{{timestamp}}.json`."
*   **Actions (Scribe):** Updates `.pheromone.activePullRequest` with details. UO/Scribe conceptually translates `sourceBranchName_origLang` to `sourceBranchName_en` for internal use.
*   **Actions (UO or `@code-reviewer-assistant` after Scribe update):** Read `activePullRequest.sourceBranchName_en`. Sanitize for dir name. Create `reviewDirPath = "04_PR_REVIEWS/{{activePullRequest.sourceBranchName_sanitized_en}}/"`. Store in `activePullRequest.reviewDirectoryPath`.

### Phase 2: Code Changes Retrieval
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** `activePullRequest` from `.pheromone`.
*   **Actions:** Use **ADO MCP** or **Git Tools MCP** for diffs.
*   **onError (MCP):** Scribe logs English error. UO notifies user (in `userLanguage`), stops.
*   **Output (internal, English context):** Code diffs ready.

### Phase 3: Code Analysis (English "Chain of Thought")
*   **Responsible Agent:** `@code-reviewer-assistant` (with `@security-analyst-agent`).
*   **Inputs (Injected by UO, English):** Code changes. `memoryBank` context (English conventions, tech debt, arch decisions).
*   **Actions (`@code-reviewer-assistant`):**
    1.  **Analysis (English logic):** Conventions, quality, smells, bugs, tests. Context7 MCP for library usage.
    2.  **Security (English comms):** Delegate to `@security-analyst-agent`.
    3.  **"Chain of Thought" (English):** For significant issues, document English reasoning.
    4.  **Ambiguity:** If code obscure: Report (English) to UO for `@clarification-agent` (question to PR dev in their language).
*   **Actions (`@security-analyst-agent`):** Security analysis. Concise English report.
*   **Output (compiled by `@code-reviewer-assistant`, English):** Consolidated issues, severity, English "chain of thought".

### Phase 4: Review Report (English Content, Final Output Localized)
*   **Responsible Agent:** `@code-reviewer-assistant`.
*   **Inputs:** Consolidated English issues. `activePullRequest.reviewDirectoryPath`. `currentUser.lastInteractionLanguage` (as `userLanguageForOutput`).
*   **Actions:**
    1.  **Compile English Report Content:** Markdown. Include: PR Info, Summary, Issue Details (severity, location, English reasoning/chain of thought, suggestion).
    2.  **Translate Report:** Translate English report to `userLanguageForOutput`.
    3.  Save localized report: `pr_{{activePullRequest.id}}_review_report_{{timestamp}}_{{userLanguageForOutput}}.md` in `{{activePullRequest.reviewDirectoryPath}}`.
*   **Output (to Scribe, English NL Summary with path to localized report):** "PR Azure#{{activePullRequest.id}} review complete. [Stats]. Report (in `{{userLanguageForOutput}}`, English reasoning available) at `{{localized_fullReportPath}}`. Recommendation (English): [Action]."

### Phase 5: `.pheromone` Update (English Data, Localized Report Path)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** English NL Summary from `@code-reviewer-assistant`.
*   **Actions:** Update `.pheromone` (English data for `memoryBank`):
    *   `documentationRegistry`: Add `{{localized_fullReportPath}}`.
    *   `memoryBank.pullRequests.{{activePullRequest.id}}`: Update `status: "ReviewedByAI"`, `reviewReportPath_localized: "{{localized_fullReportPath}}"` , `lastReviewTimestamp`, `issuesFoundCounts`, `summary_en`, `reasoningChainLinks_en.reviewReport` (link to English reasoning if separate, or note in localized).
    *   (Optional) UO instructs `@devops-connector` to comment on ADO PR (comment can be English or localized by UO).
*   **Output:** `.pheromone` updated. UO informs Tech Lead (in `userLanguage`).

---