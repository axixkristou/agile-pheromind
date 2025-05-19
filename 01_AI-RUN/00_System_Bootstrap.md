# Workflow: User Onboarding and Initial AgilePheromind Project Configuration (00_System_Bootstrap.md)

**Objective:** Guide the user through the essential initial setup for AgilePheromind. This workflow collects information about the user's Azure DevOps identity, their preferred language, details of the existing Azure DevOps project, and the local path to the Git repository. It creates/updates the local `.agilepherominduserinfo` file and initializes the corresponding sections in `.pheromone`. This script is triggered automatically by the `🧐 @uber-orchestrator` (UO) if critical information is missing, or can be run manually. User interaction is in `userLanguage` provided by HO or detected.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@project-setup-agent` (acts as an information gatherer and initial configurator), `@devops-connector`, `@clarification-agent`, `@architecture-advisor-agent`.

**MCPs Used:** Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Automatic:** UO detects `.pheromone.onboardingComplete` is `false` OR critical fields in `.pheromone.currentUser.azureDevOpsUsername` OR `.pheromone.currentProject.gitRepository.localPath` are null. UO uses `userLanguage` passed from `🎩 @head-orchestrator`.
    *   **Manual:** User requests e.g., `"AgilePheromind run onboarding setup"`. UO uses `userLanguage` from HO.
    UO sets `activeWorkflow.scriptPath` to this script. UO updates `currentUser.lastInteractionLanguage`.

2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Welcome and Preferred Language Confirmation/Collection.**
        *   UO (possibly via `@clarification-agent`) interacts with user (in `userLanguage`) to confirm/set `user.preferences.preferredLanguage` and `user.lastKnownInteractionLanguage` (for `.agilepherominduserinfo`).
    *   **Phase 2: Azure DevOps User Information Collection.**
        *   UO (via `@clarification-agent` if needed) prompts user (in `userLanguage`) for Azure DevOps email/UPN.
        *   UO delegates to `@devops-connector` to validate and retrieve ADO `displayName` and `userId` (using **Azure DevOps MCP** `get_user_identity`).
        *   **onError:** If identity invalid or MCP fails, re-prompt (in `userLanguage`) or guide user. Loop.
    *   **Phase 3: Azure DevOps Project and Git Repository Information Collection.**
        *   UO (via `@clarification-agent` if needed) prompts (in `userLanguage`) for ADO Org URL, ADO Project Name, and **full local absolute path** to Git repository.
        *   UO delegates to `@devops-connector` to validate ADO project and get its ID and remote Git URL.
        *   **onError:** If ADO project invalid, re-prompt. If local path seems syntactically incorrect, UO asks for confirmation.
    *   **Phase 4: Consolidation of Information for Recording (English structure).**
        *   UO tasks `@project-setup-agent` to structure all collected information (English keys, user-provided values) for `.agilepherominduserinfo` and `.pheromone`.
    *   **Phase 5: Critical Information Recording by Scribe.**
        *   `@project-setup-agent` sends a special English NL summary with structured data to Scribe.
        *   Scribe (guided by `.swarmConfig` rule `bootstrap_update_user_and_project_info_v3`) updates/creates `.agilepherominduserinfo` AND populates `currentUser`, `currentProject` (English fields), and `memoryBank.projectContext` in `.pheromone`. Sets `.pheromone.onboardingComplete = true`.
        *   **onError:** Critical failure if Scribe fails. UO notifies user (in `userLanguage`).
    *   **Phase 6: Conventions Initialization (English templates, if first time).**
        *   UO checks `documentationRegistry` for English convention files. If missing, delegates to `@architecture-advisor-agent` to create from English templates.
    *   **Phase 7: Bootstrap Completion Report and Workflow Resumption.**
        *   UO compiles final English summary. Scribe records.
        *   UO informs user (in `currentUser.preferredLanguage`) of successful setup.
        *   If bootstrap was triggered by another pending workflow, UO attempts to resume it.

## Phase Details:

### Phase 1: Welcome and Preferred Language Confirmation/Collection
*   **Responsible Agent:** UO (using `ask_followup_question`) or `@clarification-agent`.
*   **Inputs:** `userLanguage` (initial detection from `🎩 @head-orchestrator`).
*   **Actions (UO):**
    1.  Welcome message (translated to `userLanguage`): "Welcome to AgilePheromind! To get started, we need to configure a few details. My responses to you will be in `{{userLanguage}}`. Is this your preferred language for our interactions (e.g., 'yes'), or would you like to set a different one (e.g., 'fr-FR', 'en-US')?"
    2.  User responds. UO captures `userPreferredLanguage`. If user confirms, `userPreferredLanguage = userLanguage`.
*   **Output (internal to UO):** `userPreferredLanguage` confirmed/collected.

### Phase 2: Azure DevOps User Information Collection
*   **Responsible Agent:** UO (or `@clarification-agent`), `@devops-connector`.
*   **Inputs:** `userPreferredLanguage`.
*   **Actions (UO):**
    1.  Ask user (in `userPreferredLanguage`): "Please provide your Azure DevOps username (typically your work email/UPN)."
    2.  Delegate to `@devops-connector` (English instruction): "Validate ADO user and retrieve details for: `{{user_ado_email}}`."
*   **Actions (`@devops-connector`):** Use **ADO MCP** `get_user_identity {identifier: user_ado_email}`.
*   **onError (ADO MCP / User Not Found):** `@devops-connector` signals (English) to UO. UO (via `@clarification-agent` if interactive) re-prompts user (in `userPreferredLanguage`): "The ADO user '{{user_ado_email}}' was not found or an error occurred. Please verify and provide your Azure DevOps email/UPN again." Loop.
*   **Output (`@devops-connector` -> UO if success, English):** `azureDevOpsUsername`, `azureDevOpsDisplayName`, `azureDevOpsUserId`.

### Phase 3: Azure DevOps Project and Git Repository Information Collection
*   **Responsible Agent:** UO (or `@clarification-agent`), `@devops-connector`.
*   **Inputs:** `userPreferredLanguage`.
*   **Actions (UO):**
    1.  Ask (in `userPreferredLanguage`): "Azure DevOps Org URL?", "Azure DevOps Project Name?", "Full local absolute path to your Git repository clone?" (provide examples). Optionally, "Default development branch (e.g., `develop`, `main`)?".
    2.  Delegate to `@devops-connector` (English instruction): "Validate ADO project, get ID/remote URL: Org='...', Project='...'."
*   **Actions (`@devops-connector`):** Use **ADO MCP** `get_project_details {organizationUrl, projectName}`.
*   **onError (ADO MCP / Project Not Found):** `@devops-connector` signals (English) to UO. UO (via `@clarification-agent`) re-prompts user (in `userPreferredLanguage`). Loop.
*   **Warning (Local Path by UO):** UO notes `gitLocalPath`. If unusual, may ask user (in `userPreferredLanguage`) for confirmation.
*   **Output (`@devops-connector` -> UO if ADO success, English):** `adoProjectId`, `gitRemoteUrl`. UO keeps `gitLocalPath` and `defaultBranchPheromind`.

### Phase 4: Consolidation of Information for Recording (English structure)
*   **Responsible Agent:** `@project-setup-agent`.
*   **Inputs (from UO):** All collected info (English where applicable): `userPreferredLanguage`, `lastInteractionLanguage`, ADO user details, ADO project details, Git paths.
*   **Actions (`@project-setup-agent`):**
    1.  Generate local `pheromindId` (UUID) for user, `pheromindProjectId` (UUID).
    2.  Assemble `structuredData` object (matching `.agilepherominduserinfo` structure) with English keys and collected values. Git username/email can be collected here too if desired for `.agilepherominduserinfo`.
*   **Output (to Scribe, English):** Special NL summary: "AgilePheromind User and Project Info for Bootstrap. StructuredData attached..." + `structuredData`.

### Phase 5: Critical Information Recording by Scribe
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** Special English NL summary and `structuredData` from `@project-setup-agent`.
*   **Actions (Scribe, per `.swarmConfig` rule `bootstrap_update_user_and_project_info_v3`):**
    1.  **Write/Update `.agilepherominduserinfo`** with `structuredData.user` and `structuredData.projectConnection`.
    2.  **Update `.pheromone` (English data):** Populate `currentUser`, `currentProject`. Initialize `memoryBank.projectContext`. Set `currentUser.lastInteractionLanguage`, `currentUser.preferredLanguage`. Set `.pheromone.onboardingComplete = true`.
*   **onError (Scribe write failure):** Critical. Scribe reports (English) to UO. UO informs user (in `userPreferredLanguage`): "Critical error saving setup. AgilePheromind cannot proceed. Check file permissions for '.agilepherominduserinfo' & '.pheromone'." HALT.
*   **Output:** `.agilepherominduserinfo` & `.pheromone` updated.

### Phase 6: Conventions Initialization (English templates, if first time or requested)
*   **Responsible Agent:** `@architecture-advisor-agent`.
*   **Inputs (from UO):** Status of `documentationRegistry`.
*   **Actions (UO):** If English convention files (`coding_conventions.md`, `design_conventions.md`) not in `documentationRegistry`, delegate to `@architecture-advisor-agent`: "Initialize English convention documents from templates."
*   **Actions (`@architecture-advisor-agent`):** Copy English templates to `02_AI-DOCS/Conventions/`.
*   **Output (`@architecture-advisor-agent` -> Scribe, English NL Summary):** "English convention documents initialized. Version: 1.0_initial." (Scribe updates `documentationRegistry` & `memoryBank.projectContext` versions).

### Phase 7: Bootstrap Completion Report and Workflow Resumption
*   **Responsible Agent:** UO, Scribe.
*   **Actions (UO):** Compile final English summary of bootstrap.
*   **Output (UO to Scribe, English NL Summary):** "AgilePheromind Bootstrap/Onboarding complete for user '{{currentUser.azureDevOpsDisplayName}}' and project '{{currentProject.azureDevOps.projectName}}'. Info recorded. Onboarding flag set. System ready."
*   **Actions (Scribe):** Record summary in `memoryBank.agentActivityLog`. Record bootstrap report in `documentationRegistry`.
*   **Actions (UO):**
    1.  Inform user (in `currentUser.preferredLanguage`): "AgilePheromind setup is complete for project '{{currentProject.azureDevOps.projectName}}'! I will interact with you in `{{currentUser.preferredLanguage}}`. Ready to assist."
    2.  If bootstrap was triggered by another pending workflow, UO attempts to resume it.
*   **Outcome:** AgilePheromind configured. Original workflow (if any) resumes.

---