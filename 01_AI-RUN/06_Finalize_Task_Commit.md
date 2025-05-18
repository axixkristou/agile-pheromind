# Workflow: Finalize Task and Prepare Commit (06_Finalize_Task_Commit.md)

**Objective:** Guide the developer through the steps of finalizing a technical task. This includes rigorous verification that all tests (unit, integration) pass and that linters are satisfied. Then, prepare a "Conventional Commits" compliant commit message, execute the commit, and update statuses in `.pheromone` and Azure DevOps, with error handling for each step.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@developer-agent`, `@commit-pr-formatter`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Git Tools MCP, Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The developer (Dev) signals the completion of a task and the associated US (e.g., `"AgilePheromind: Task Azure#23223 is completed. Prepare commit for US Azure#12323."`).
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Thorough Pre-Commit Verification (Tests and Linters).**
        *   UO delegates to `@developer-agent`.
        *   **onError:** If failures, notify the dev, store failure details in `memoryBank.tasks.{{activeTask.id}}.lastVerificationFailure`, stop the workflow for this action.
    *   **Phase 2: Commit Message Generation.**
        *   UO **injects context** (US/task titles from `.pheromone`) to `@commit-pr-formatter`.
    *   **Phase 3: User Validation and Commit Execution.**
        *   UO presents the message to the dev via `ask_followup_question`.
        *   If validated, UO instructs `@developer-agent` (or `@commit-pr-formatter`) to use **Git Tools MCP**.
        *   **onError (Git Commit):** If `commit_files` fails (e.g., pre-commit hook fails, unmanaged merge conflict), log the error, notify the dev, suggest actions (e.g., `git status`, resolve conflicts). The workflow stops until resolution.
    *   **Phase 4: Status Updates (Azure DevOps and `.pheromone`).**
        *   UO delegates to `@devops-connector` for ADO.
        *   **onError (ADO Update):** If the ADO update fails, log the error. `.pheromone` will be updated, but a note will indicate the ADO sync failure. The UO may suggest a retry or manual update.
        *   Scribe updates `.pheromone`.

## Phase Details:

### Phase 1: Thorough Pre-Commit Verification (Tests and Linters)
*   **Responsible Agent:** `@developer-agent`
*   **Inputs:** Context from `activeTask`, `activeUserStory` from `.pheromone`. Local code access.
*   **Actions & Tooling:**
    1.  Run linters (.NET, Angular) via `memoryBank.toolingConfigurations.linters`.
    2.  Run relevant unit/integration tests (identified via `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated` and auto-discovered tests).
*   **onError Strategy (for `@developer-agent` to report to UO):**
    1.  If linting errors or test failures:
        *   Compile a detailed failure report (which linters/tests, error messages).
        *   Submit to UO/Scribe: "Pre-commit verification Task Azure#{{activeTask.id}} FAILED. Report: `pre_commit_check_failure_{{activeTask.id}}_{{timestamp}}.md`. Fix before commit." (Report in `03_SPECS/Verification_Failures/`).
        *   Scribe records the link to the report in `memoryBank.tasks.{{activeTask.id}}.lastVerificationFailure`.
        *   UO stops the workflow for this action.
*   **Output (to UO/Scribe if successful):** "Pre-commit verification Task Azure#{{activeTask.id}} SUCCESSFUL. Linters OK. Tests OK. Log: `pre_commit_check_success_{{activeTask.id}}_{{timestamp}}.log`." (Log in `03_SPECS/Verification_Logs/`).

### Phase 2: Commit Message Generation
*   **Responsible Agent:** `@commit-pr-formatter`
*   **Inputs (Injected by UO):**
    *   Confirmation that Phase 1 is OK.
    *   `activeTask.title` and `activeUserStory.title` (from `.pheromone`).
    *   (Optional) Summary of key changes if provided by `@developer-agent`.
*   **Actions & Tooling:**
    1.  **Git Tools MCP** (`get_changed_files_staged`) for the list of files.
    2.  Propose "Conventional Commit" message (type, scope, description, body, footer with `Resolves Azure#...`, `Closes Azure#...`).
*   **Output (to UO):** Formatted commit message.

### Phase 3: User Validation and Commit Execution
*   **Responsible Agent:** `🧐 @uber-orchestrator` (interaction), `@developer-agent` (execution).
*   **Inputs:** Proposed commit message.
*   **Actions & Tooling (UO):**
    1.  `ask_followup_question` for validation of the message by the dev.
*   **If user confirms "yes" (`@developer-agent`):**
    1.  Use **Git Tools MCP**: `add_all_changes`, `commit_files {message: validatedCommitMessage}`.
    2.  **onError (Git Commit):** If `commit_files` fails:
        *   `@developer-agent` reports the exact Git error to the UO.
        *   UO notifies the dev: "Commit failed for Task Azure#{{activeTask.id}}. Git Error: [Error Message]. Please check `git status` and resolve issues (conflicts, hooks?)."
        *   Scribe logs the failure in `memoryBank.tasks.{{activeTask.id}}.commitHistory`.
        *   Workflow stopped until manual resolution or new attempt.
    3.  If commit OK, (Optional) `push_commits`.
*   **Output (`@developer-agent` to Scribe if successful):** NL Summary: "Commit Task Azure#{{activeTask.id}} OK. Hash: `{{commitHash}}`. [Push OK/Not performed]."

### Phase 4: Status Updates (Azure DevOps and `.pheromone`)
*   **Responsible Agent:** `@devops-connector`, Scribe.
*   **Inputs:** Confirmation commit OK (Phase 3). `activeTask.id`, `activeUserStory.id`, `commitHash`.
*   **Actions & Tooling (`@devops-connector`):**
    1.  **Azure DevOps MCP**: `update_work_item_status {id: activeTask.id, status: "Done", comment: "Implementation committed. Commit: {{commitHash}}"}`.
*   **onError Strategy (for UO if `@devops-connector` reports ADO Update failure):**
    1.  Scribe logs the error in `activeWorkflow.lastError` and notes the ADO sync failure in `memoryBank.tasks.{{activeTask.id}}.syncIssues`.
    2.  UO notifies the dev: "Commit {{commitHash}} successful, but updating the status of Task Azure#{{activeTask.id}} in Azure DevOps failed: [MCP Error]. Local status is 'Done'. Please check ADO or retry sync."
    3.  The workflow continues for the `.pheromone` update.
*   **Output (`@devops-connector` to Scribe):** NL Summary: "Task Azure#{{activeTask.id}} status updated to 'Done' in ADO (Commit: {{commitHash}})." or "Failed to update Task Azure#{{activeTask.id}} status in ADO."
*   **Actions & Tooling (Scribe):**
    1.  Update `.pheromone`:
        *   `activeUserStory.tasks` (for the task): `status: "DoneByPheromind"`. `activeTask` -> `null`.
        *   `memoryBank.tasks.{{activeTask.id}}`: `status: "Done"`, `statusHistory` updated, `relatedCommit: "{{commitHash}}"`. If ADO sync failure, add to `syncIssues`.
        *   `memoryBank.commits.{{commitHash}}`: { `message`, `author`, `timestamp`, `relatedTask`, `relatedUS` }.
    2.  **Check US completion:** If all tasks `Done`, update `memoryBank.userStories.{{activeUserStory.id}}.status` -> "ReadyForReview". Notify UO.
*   **Output:** `.pheromone` updated. UO informed to notify dev/next steps.

---