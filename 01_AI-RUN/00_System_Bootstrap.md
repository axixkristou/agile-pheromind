# Workflow: User Onboarding and Initial AgilePheromind Project Configuration (00_System_Bootstrap.md)

**Objective:** Guide the user through the essential initial setup for AgilePheromind. This workflow collects information about the user's Azure DevOps identity, their preferred language, details of the existing Azure DevOps project, and the local path to the Git repository. It creates/updates the local `.agilepherominduserinfo` file and initializes the corresponding sections in `.pheromone`. This script is triggered automatically by the `🧐 @uber-orchestrator` (UO) if critical information is missing, or can be run manually.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@project-setup-agent` (acts as an information gatherer and initial configurator), `@devops-connector`, `@clarification-agent`, `@architecture-advisor-agent`.

**MCPs Used:** Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Automatic:** UO detects `.pheromone.onboardingComplete` is `false` or critical fields in `.pheromone.currentUser` or `.pheromone.currentProject` are null. The UO will have the `userLanguage` from the `🎩 @head-orchestrator`'s initial processing of the user's very first command.
    *   **Manual:** User requests e.g., `"AgilePheromind run onboarding setup"`. UO will use `currentUser.lastInteractionLanguage` or default to English if not set, then proceed to Phase 1.
    The UO sets `activeWorkflow.scriptPath` to this script.

2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Welcome and Preferred Language Confirmation/Collection.**
        *   UO (possibly via `@clarification-agent` if initial `userLanguage` is uncertain) interacts with user to confirm/set `user.preferences.preferredLanguage` and `user.lastKnownInteractionLanguage` (for `.agilepherominduserinfo`).
    *   **Phase 2: Azure DevOps User Information Collection.**
        *   UO (via `@clarification-agent` if needed) prompts user for Azure DevOps email/UPN.
        *   UO delegates to `@devops-connector` to validate and retrieve ADO `displayName` and `userId` (using **Azure DevOps MCP** `get_user_identity`).
        *   **onError:** If invalid identity or MCP fails, re-prompt or guide user. Loop until valid or user aborts.
    *   **Phase 3: Azure DevOps Project and Git Repository Information Collection.**
        *   UO (via `@clarification-agent` if needed) prompts for ADO Org URL, ADO Project Name, and **full local absolute path** to the Git repository.
        *   UO delegates to `@devops-connector` to validate ADO project and get its ID and remote Git URL (using **Azure DevOps MCP** `get_project_details`).
        *   **onError:** If ADO project invalid, re-prompt. If local path seems syntactically incorrect, UO can ask for confirmation.
    *   **Phase 4: Consolidation of Information for Recording.**
        *   UO tasks `@project-setup-agent` to structure all collected information (including generating local Pheromind IDs) for `.agilepherominduserinfo` and for initializing `.pheromone` state.
    *   **Phase 5: Critical Information Recording by Scribe.**
        *   `@project-setup-agent` sends a special English NL summary with structured data to Scribe.
        *   Scribe (guided by specific `.swarmConfig` rule `bootstrap_update_user_and_project_info_v2`) updates/creates `.agilepherominduserinfo` AND populates `currentUser`, `currentProject`, and `memoryBank.projectContext` in `.pheromone`. Sets `.pheromone.onboardingComplete = true`.
        *   **onError:** If Scribe fails to write `.agilepherominduserinfo` or `.pheromone`, this is a critical failure. UO notifies user that setup failed and Pheromind cannot proceed.
    *   **Phase 6: Conventions Initialization (if first time or requested).**
        *   UO checks `documentationRegistry` for `coding_conventions.md` and `design_conventions.md`.
        *   If missing, UO delegates to `@architecture-advisor-agent` to create initial English convention files from templates.
    *   **Phase 7: Bootstrap Completion Report and Workflow Resumption.**
        *   UO compiles a final English summary. Scribe records it.
        *   UO informs user (in `currentUser.preferredLanguage`) of successful setup.
        *   If this bootstrap was triggered by another pending workflow, UO attempts to resume that original workflow.

## Phase Details:

### Phase 1: Welcome and Preferred Language Confirmation/Collection
*   **Responsible Agent:** UO (using `ask_followup_question`) or `@clarification-agent`.
*   **Inputs:** `userLanguage` (initial detection from `🎩 @head-orchestrator`).
*   **Actions (UO):**
    1.  Welcome message (translated to `userLanguage`): "Welcome to AgilePheromind! To effectively assist you, I need to set up some initial information. My responses to you will be in `{{userLanguage}}`. Is this your preferred language for our interactions, or would you like to set a different one? (You can provide a language code like 'en-US', 'fr-FR', etc., or say 'yes' to confirm `{{userLanguage}}`)."
    2.  User responds. UO captures `userPreferredLanguage`. If user confirms, `userPreferredLanguage = userLanguage`.
*   **Output (internal to UO):** `userPreferredLanguage` confirmed/collected.

### Phase 2: Azure DevOps User Information Collection
*   **Responsible Agent:** UO (or `@clarification-agent`), `@devops-connector`.
*   **Inputs:** `userPreferredLanguage`.
*   **Actions (UO):**
    1.  Ask user (in `userPreferredLanguage`): "Please provide your Azure DevOps username (typically your work email/UPN)."
    2.  Delegate to `@devops-connector`: "Validate ADO user and retrieve details for: `{{user_ado_email}}`."
*   **Actions (`@devops-connector`):** Use **ADO MCP** `get_user_identity {identifier: user_ado_email}`.
*   **onError (ADO MCP / User Not Found):** `@devops-connector` signals (English) to UO. UO (via `@clarification-agent` if interactive) re-prompts user (in `userPreferredLanguage`): "ADO user '{{user_ado_email}}' not found or error occurred. Please verify and re-enter." Loop.
*   **Output (`@devops-connector` -> UO if success, English):** `azureDevOpsUsername`, `azureDevOpsDisplayName`, `azureDevOpsUserId`.

### Phase 3: Azure DevOps Project and Git Repository Information Collection
*   **Responsible Agent:** UO (or `@clarification-agent`), `@devops-connector`.
*   **Inputs:** `userPreferredLanguage`.
*   **Actions (UO):**
    1.  Ask (in `userPreferredLanguage`):
        *   "Azure DevOps Organization URL (e.g., `https://dev.azure.com/MyOrg`)?"
        *   "Azure DevOps Project Name within that organization?"
        *   "Full local absolute path to your Git repository clone for this project (e.g., `C:/Projects/MyRepo` or `/home/user/projects/myrepo`)? This is CRITICAL for me to access project files."
        *   (Optional, can be asked later or defaulted) "What is the primary development branch for new features (e.g., `develop`, `main`)?"
    2.  Delegate to `@devops-connector`: "Validate ADO project and get ID/remote URL: Org='...', Project='...'."
*   **Actions (`@devops-connector`):** Use **ADO MCP** `get_project_details {organizationUrl, projectName}`.
*   **onError (ADO MCP / Project Not Found):** `@devops-connector` signals (English) to UO. UO (via `@clarification-agent`) re-prompts user (in `userPreferredLanguage`). Loop.
*   **Warning (Local Path):** UO notes the `gitLocalPath`. If it looks unusual (e.g., very short, relative), UO may ask for confirmation: "The local path `{{gitLocalPath}}` seems [unusual/relative]. Please confirm this is the correct full absolute path."
*   **Output (`@devops-connector` -> UO if ADO success, English):** `adoProjectId`, `gitRemoteUrl`. UO keeps `gitLocalPath` and `defaultBranchPheromind` (if provided, else defaults to 'develop').

### Phase 4: Consolidation of Information for Recording
*   **Responsible Agent:** `@project-setup-agent`.
*   **Inputs (from UO):** All collected info (English where applicable): `userPreferredLanguage`, `lastInteractionLanguage` (initially same as preferred), ADO user details, ADO project details, Git paths.
*   **Actions (`@project-setup-agent`):**
    1.  Generate local `pheromindId` (UUID) for user, `pheromindProjectId` (UUID).
    2.  Assemble `structuredData` object (as per `.agilepherominduserinfo` structure) including all user and project connection details.
*   **Output (to Scribe, English):** Special NL summary: "AgilePheromind User and Project Info for Bootstrap. StructuredData attached. Update '.agilepherominduserinfo' and relevant '.pheromone' sections. Set onboardingComplete flag." + `structuredData`.

### Phase 5: Critical Information Recording by Scribe
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** Special English NL summary and `structuredData` from `@project-setup-agent`.
*   **Actions (Scribe, per `.swarmConfig` rule `bootstrap_update_user_and_project_info_v2`):**
    1.  **Write/Update `.agilepherominduserinfo`** with `structuredData.user` and `structuredData.projectConnection`.
    2.  **Update `.pheromone` (English data):** Populate `currentUser`, `currentProject`. Initialize `memoryBank.projectContext` with `azureDevOpsProject`, `gitRepositoryLocalPath`, etc. Set `currentUser.lastInteractionLanguage` and `currentUser.preferredLanguage`. Set `.pheromone.onboardingComplete = true`.
*   **onError (Scribe write failure):** This is critical. Scribe reports to UO. UO informs user (in `userPreferredLanguage`): "Critical error saving setup information. AgilePheromind cannot proceed. Please check file permissions for '.agilepherominduserinfo' and '.pheromone' or contact support." Workflow HALTS.
*   **Output:** `.agilepherominduserinfo` and `.pheromone` updated.

### Phase 6: Conventions Initialization (if first time or requested)
*   **Responsible Agent:** `@architecture-advisor-agent`.
*   **Inputs (from UO):** Status of `documentationRegistry` regarding convention files. `userLanguage` (for the final Scribe summary regarding this step).
*   **Actions (UO):**
    1.  Check if `coding_conventions.md` and `design_conventions.md` are in `documentationRegistry`.
    2.  If missing, delegate to `@architecture-advisor-agent`: "Initialize English convention documents from templates. Provide summary about the language of the created documents."
*   **Actions (`@architecture-advisor-agent`):** Copy English templates to `02_AI-DOCS/Conventions/`, make minimal tech stack adjustments.
*   **Output (`@architecture-advisor-agent` -> Scribe, English NL Summary):** "English convention documents initialized: `coding_conventions.md` and `design_conventions.md` created from templates. Version: 1.0_initial." (Scribe updates `documentationRegistry` and `memoryBank.projectContext` versions).

### Phase 7: Bootstrap Completion Report and Workflow Resumption
*   **Responsible Agent:** UO, Scribe.
*   **Actions (UO):** Compile final English summary of bootstrap.
*   **Output (UO to Scribe, English NL Summary):** "AgilePheromind Bootstrap/Onboarding complete for user '{{currentUser.azureDevOpsDisplayName}}' and project '{{currentProject.azureDevOps.projectName}}'. User and project info recorded. Onboarding flag set. System ready."
*   **Actions (Scribe):** Record summary in `memoryBank.agentActivityLog`. Record bootstrap report in `documentationRegistry`.
*   **Actions (UO):**
    1.  Inform user (in `currentUser.preferredLanguage`): "AgilePheromind setup is complete for project '{{currentProject.azureDevOps.projectName}}'! I will interact with you in `{{currentUser.preferredLanguage}}`. I am ready to assist."
    2.  If this bootstrap was triggered by another pending workflow, UO attempts to resume that original workflow.
*   **Outcome:** AgilePheromind fully configured. Original workflow (if any) resumes.

---