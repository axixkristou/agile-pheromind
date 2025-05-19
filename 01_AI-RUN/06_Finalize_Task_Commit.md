# Workflow: Finalize Task and Prepare Commit (06_Finalize_Task_Commit.md)

**Objective:** Guide the developer through task finalization. This includes rigorous pre-commit verification (tests and linters pass), preparing a "Conventional Commits" compliant message (in English), executing the commit (with the English message), and updating statuses in `.pheromone` (English data) and Azure DevOps. Error handling is integrated. User interaction (validation of commit message) is in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@developer-agent`, `@commit-pr-formatter`, `@devops-connector`.

**MCPs Used:** Git Tools MCP, Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Developer (Dev) signals task completion and associated US (e.g., `"AgilePheromind: Task Azure#23223 is complete. Prepare commit for US Azure#12323."`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Thorough Pre-Commit Verification (Tests and Linters).**
        *   UO delegates to `@developer-agent`.
        *   **onError:** If failures, notify dev (in `userLanguage`), store English failure details in `memoryBank.tasks.{{activeTask.id}}.lastVerificationFailure_en`, stop workflow.
    *   **Phase 2: Commit Message Generation (English).**
        *   UO **injects English context** (US/task English titles from `.pheromone`) to `@commit-pr-formatter`.
    *   **Phase 3: User Validation and Commit Execution.**
        *   UO presents the English commit message to dev (translated to `userLanguage` for display if needed by `ask_followup_question`).
        *   If validated, UO instructs `@developer-agent` to use **Git Tools MCP** for commit (with the original **English** message).
        *   **onError (Git Commit):** If `commit_files` fails, log English error, notify dev (in `userLanguage`), suggest actions. Workflow stops.
    *   **Phase 4: Status Updates (Azure DevOps and `.pheromone` - English Data).**
        *   UO delegates to `@devops-connector` for ADO update (comment can be English or localized).
        *   **onError (ADO Update):** If ADO update fails, log English error. `.pheromone` updated, but with note of ADO sync failure. UO may suggest retry or manual update (in `userLanguage`).
        *   Scribe updates `.pheromone` (English data).

## Phase Details:

### Phase 1: Thorough Pre-Commit Verification (Tests and Linters)
*   **Responsible Agent:** `@developer-agent`.
*   **Inputs:** Context of `activeTask`, `activeUserStory` from `.pheromone`. Local source code.
*   **Actions & Tooling:**
    1.  Run linters (.NET, Angular) via `memoryBank.toolingConfigurations.linters`.
    2.  Run relevant unit/integration tests (identified via `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated_en` and auto-discovered tests).
*   **onError Strategy (for `@developer-agent` to report to UO in English):**
    1.  If linting errors or test failures: Compile detailed English failure report. Submit to UO/Scribe (English summary): "Pre-commit verification Task Azure#{{activeTask.id}} FAILED. English Report: `pre_commit_check_failure_{{activeTask.id}}_{{timestamp}}.md`. Please fix." (Report in `03_SPECS/Verification_Failures/`).
    2.  Scribe records link to English report in `memoryBank.tasks.{{activeTask.id}}.lastVerificationFailure_en`.
    3.  UO stops workflow for this action.
*   **Output (to UO/Scribe if success, English NL Summary):** "Pre-commit verification Task Azure#{{activeTask.id}} SUCCESSFUL. Linters OK. Tests OK. English Log: `pre_commit_check_success_{{activeTask.id}}_{{timestamp}}.log`." (Log in `03_SPECS/Verification_Logs/`).

### Phase 2: Commit Message Generation (English)
*   **Responsible Agent:** `@commit-pr-formatter`.
*   **Inputs (Injected by UO, English context):** Confirmation Phase 1 OK. `activeTask.title_en`, `activeUserStory.title_en` (from `.pheromone`).
*   **Actions & Tooling:**
    1.  **Git Tools MCP** (`get_changed_files_staged`).
    2.  Consult `memoryBank.tasks.{{activeTask.id}}.title_en` and `memoryBank.userStories.{{activeUserStory.id}}.title_en`.
    3.  Propose English commit message using "Conventional Commits".
*   **Output (to UO, English):** Formatted English commit message.

### Phase 3: User Validation and Commit Execution
*   **Responsible Agent:** `🧐 @uber-orchestrator` (interaction), `@developer-agent` (execution).
*   **Inputs:** Proposed English commit message. `currentUser.lastInteractionLanguage`.
*   **Actions & Tooling (UO):**
    1.  Translate (conceptually) proposed English commit message to `currentUser.lastInteractionLanguage` if different.
    2.  Use `ask_followup_question` to present translated message to dev: "Pre-commit checks OK. Proposed commit message for Task Azure#{{activeTask.id}}:\n```\n{{translatedCommitMessage}}\n```\nConfirm to commit? (yes/no/edit)".
*   **If user confirms "yes" (`@developer-agent`):**
    1.  Use the original **English** commit message (`validatedEnglishCommitMessage`).
    2.  Use **Git Tools MCP**: `add_all_changes`, `commit_files {message: validatedEnglishCommitMessage}`.
    3.  **onError (Git Commit):** If `commit_files` fails: `@developer-agent` reports exact English Git error to UO. UO notifies dev (in `currentUser.lastInteractionLanguage`): "Commit failed for Task Azure#{{activeTask.id}}. Git Error: [Error Message]. Check `git status` and resolve." Scribe logs English failure in `memoryBank.tasks.{{activeTask.id}}.commitHistory_en`. Workflow stopped.
    4.  If commit OK, (Optional) `push_commits`.
*   **If user requests "edit":** UO asks dev for modified message (in `userLanguage`), UO conceptually translates to English, then re-triggers commit action with new English message.
*   **Output (`@developer-agent` to Scribe if success, English NL Summary):** "Commit for Task Azure#{{activeTask.id}} successful. Commit Hash: `{{commitHash}}`. [Push OK/Not performed]."

### Phase 4: Status Updates (Azure DevOps and `.pheromone` - English Data)
*   **Responsible Agent:** `@devops-connector`, Scribe.
*   **Inputs:** Confirmation commit OK. `activeTask.id`, `activeUserStory.id`, `commitHash`. `currentUser.lastInteractionLanguage`.
*   **Actions & Tooling (`@devops-connector`):**
    1.  Prepare ADO comment (English or translated to project's ADO language by UO/agent): `"Implementation committed. Commit: {{commitHash}}"`.
    2.  **Azure DevOps MCP**: `update_work_item_status {id: activeTask.id, status: "Done", comment: adoComment}`.
*   **onError Strategy (for UO if ADO Update failure):**
    1.  Scribe logs English error. UO notifies dev (in `currentUser.lastInteractionLanguage`): "Commit {{commitHash}} successful, but updating Task Azure#{{activeTask.id}} status in ADO failed: [MCP Error]. Local status 'Done'. Check ADO or retry sync."
    2.  Workflow continues for `.pheromone` update.
*   **Output (`@devops-connector` to Scribe, English NL Summary):** "Task Azure#{{activeTask.id}} status updated to 'Done' in ADO (Commit: {{commitHash}})." or "Failed to update Task Azure#{{activeTask.id}} status in ADO."
*   **Actions & Tooling (Scribe):**
    1.  Interpret English summaries.
    2.  Update `.pheromone` (all English data):
        *   `activeUserStory.tasks` (for task): `status: "DoneByPheromind"`. `activeTask` -> `null`.
        *   `memoryBank.tasks.{{activeTask.id}}`: `status: "Done"`, `statusHistory_en` updated, `relatedCommit: "{{commitHash}}"`. If ADO sync fail, add to `syncIssues_en`.
        *   `memoryBank.commits.{{commitHash}}`: { `message_en`: "{{validatedEnglishCommitMessage}}", `author`: "{{currentUser.azureDevOpsUsername}}", `timestamp`, `relatedTask`, `relatedUS` }.
    3.  **Check US completion (English logic):** If all tasks "Done", update `memoryBank.userStories.{{activeUserStory.id}}.status_en` to "ReadyForReview". Notify UO.
*   **Output:** `.pheromone` updated with English data. UO informed for next steps (notification to dev in `currentUser.lastInteractionLanguage`).

---