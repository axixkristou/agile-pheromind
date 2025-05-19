# Workflow: Maintain Available Workflows Registry (18_Maintain_Available_Workflows.md)

**Objective:** Update the `memoryBank.availableWorkflows` list in `.pheromone`. This list helps the `🎩 @head-orchestrator` suggest relevant actions when a user's command is ambiguous. This workflow can be triggered manually by a system administrator or potentially by a file watcher detecting changes in the `01_AI-RUN/` directory. All data processing and storage for `availableWorkflows` is in English.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@workflow-registry-manager` (new agent).

**MCPs Used:** File System Access (conceptual, for listing/reading files in `01_AI-RUN/` - might be a capability of `@workflow-registry-manager` or UO via `command_line_tool` if Pheromind runs with local file access).

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Manual:** System Admin requests update (e.g., `"AgilePheromind update available workflows registry"`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
    *   **Automatic (Advanced):** Triggered by detection of new/modified `.md` files in `01_AI-RUN/`.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage` if manually triggered.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Scan `01_AI-RUN/` Directory and Parse Script Metadata (English).**
        *   UO delegates to `@workflow-registry-manager`.
        *   Agent lists all `.md` files in `01_AI-RUN/` (excluding this script itself).
        *   For each script, agent attempts to parse metadata (ID from filename, English name from H1 title, English description from 'Objective', typical roles, English example command template).
        *   **onError (File Access/Parsing):** If directory unreadable or script unparsable, log English error, skip problematic script, continue with others.
    *   **Phase 2: Format Workflow Data for Scribe (English).**
        *   `@workflow-registry-manager` prepares a structured list of English workflow objects.
    *   **Phase 3: Update `memoryBank.availableWorkflows` via Scribe.**
        *   `@workflow-registry-manager` sends English NL summary and structured data to Scribe.
        *   Scribe updates `.pheromone.memoryBank.availableWorkflows` (overwriting or merging).
    *   **Phase 4: Report Completion.**
        *   UO informs user/admin (in `userLanguage`) that the registry has been updated.

## Phase Details:

### Phase 1: Scan `01_AI-RUN/` Directory and Parse Script Metadata (English)
*   **Responsible Agent:** `@workflow-registry-manager`.
*   **Inputs:** Path to `01_AI-RUN/` directory (from `memoryBank.projectContext.paths.aiRunDir` or a default).
*   **Actions & Tooling:**
    1.  **List Scripts:** (Conceptual File System Access) List all `.md` files in the `01_AI-RUN/` directory, excluding `18_Maintain_Available_Workflows.md` and potentially `XX_Handle_Clarification_Response.md`.
    2.  **For each script file:**
        *   Read its content.
        *   **Parse Metadata (English):**
            *   `id`: Derive from filename (e.g., `01_start_user_story.md` -> `start_user_story`).
            *   `name_en`: Extract from the H1 Markdown title (e.g., "# Workflow: Start a User Story (...)").
            *   `description_en`: Extract from the "Objective:" section.
            *   `scriptPath`: Full path to the script file.
            *   `typicalUserRoles_en`: Extract from "Key AI Agents" or a new dedicated metadata field in scripts if added (e.g., "Target Roles: PO, Dev"). If not found, can be generic like ["User"].
            *   `exampleCommand_template_en`: Extract from a dedicated "Example Command Template (English):" section within each script, or agent makes a best guess based on `name_en` and objective. (E.g. `AgilePheromind {{name_en_slugified}} {{param_placeholder}}`). This requires each workflow script to have this metadata.
        *   Create an English workflow object with these fields.
*   **onError (File Access/Parsing):** If unable to list directory or read/parse a script: `@workflow-registry-manager` logs an English error for that script (e.g., "Could not parse metadata from script X.md, skipping."), and continues with other scripts. These errors should be part of the final summary.
*   **Output (internal, English):** A list of structured English workflow definition objects.

### Phase 2: Format Workflow Data for Scribe (English)
*   **Responsible Agent:** `@workflow-registry-manager`.
*   **Inputs:** List of English workflow definition objects from Phase 1.
*   **Actions & Tooling:**
    1.  Ensure the list is correctly formatted as an array of objects, matching the expected structure for `.pheromone.memoryBank.availableWorkflows`.
*   **Output (to Scribe, English):**
    *   NL Summary: "Available workflows registry scan complete. Found [N] valid workflow scripts. Data prepared to update `memoryBank.availableWorkflows`. [M] scripts could not be parsed. See log `workflow_registry_update_[timestamp].log` for details." (Log in `03_SPECS/System_Config_Logs/`).
    *   Structured Data: The array of English workflow definition objects.

### Phase 3: Update `memoryBank.availableWorkflows` via Scribe
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** English NL Summary and structured data (the array of workflow objects) from `@workflow-registry-manager`.
*   **Actions & Tooling (Scribe, guided by a new `.swarmConfig` rule `update_available_workflows_registry`):**
    1.  Interpret summary and structured data.
    2.  Overwrite the entire `.pheromone.memoryBank.availableWorkflows` array with the new list provided in `structuredData`.
    3.  Record the log file path (`workflow_registry_update_[timestamp].log`) in `documentationRegistry`.
*   **Output:** `.pheromone.memoryBank.availableWorkflows` updated with the latest English definitions.

### Phase 4: Report Completion
*   **Responsible Agent:** `🧐 @uber-orchestrator`.
*   **Inputs:** Confirmation from Scribe that `.pheromone` is updated. `currentUser.lastInteractionLanguage`.
*   **Actions & Tooling (UO):**
    1.  Formulate a completion message in English (e.g., "AgilePheromind's available workflows registry has been successfully updated with [N] workflows. The Head Orchestrator can now use this list to provide better suggestions for ambiguous commands.").
    2.  Translate the message to `currentUser.lastInteractionLanguage`.
    3.  Inform the user/admin (via `ask_followup_question` or a simple notification if no further interaction is needed).
*   **Output:** User/Admin informed of the update.

---
**Metadata required in each `01_AI-RUN/*.md` script for this to work well:**

To facilitate parsing by `@workflow-registry-manager`, each `01_AI-RUN/*.md` script (except `18_*` and `XX_*`) should ideally contain clearly marked sections for:

*   **H1 Title:** Used for `name_en`.
    ```markdown
    # Workflow: Start a User Story (01_Start_User_Story.md)
    ```
*   **Objective:** Used for `description_en`.
    ```markdown
    **Objective:** Initialize work on a specific Azure DevOps User Story...
    ```
*   **Target Roles (New Suggested Metadata):**
    ```markdown
    **Target User Roles:** Developer, PO
    ```
*   **Example Command Template (English - New Suggested Metadata):**
    ```markdown
    **Example Command Template (English):** AgilePheromind start US Azure#{{US_ID}}
    ```

If this metadata is not present, `@workflow-registry-manager` will have to make more "best guesses", which might be less accurate.

---