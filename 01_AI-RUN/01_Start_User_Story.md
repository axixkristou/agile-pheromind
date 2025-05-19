# Workflow: Start a User Story (01_Start_User_Story.md)

**Objective:** Initialize work on a specific Azure DevOps User Story (US). This process involves user identification, retrieving full US details, injecting relevant English context from the `memoryBank`, decomposing the US into estimated technical tasks (logging the English "chain of thought"), synchronizing with Azure DevOps, handling errors and ambiguities, and preparing the first task for development. All internal system operations and `memoryBank` storage are in English. User interaction will be in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@task-breakdown-estimator`, `@developer-agent`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Git Tools MCP, Sequential Thinking MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User (Dev) provides the Azure DevOps US ID (e.g., `"AgilePheromind start US Azure#12323"`). The `userLanguage` is passed by `🎩 @head-orchestrator` to the UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage` via Scribe.
    *   **Pre-check (CRITICAL):** UO verifies `.pheromone.onboardingComplete` is true. If not, UO HALTS this workflow and INITIATES `01_AI-RUN/00_System_Bootstrap.md`, passing the current `userLanguage`. This workflow can only resume after successful bootstrap.
    *   **Phase 1: User Identification & Full US Detail Retrieval.**
        *   UO delegates to `@devops-connector`.
        *   **onError:** If ADO MCP fails, log error, notify user (in `userLanguage`), and stop workflow.
    *   **Phase 2: Initial `.pheromone` Update & US Clarity Validation (English Internals).**
        *   Scribe updates `.pheromone` (`currentUser`, `activeUserStory` with English titles/descriptions like `title_en`, `descriptionFull_en`).
        *   UO evaluates clarity of US description (English `descriptionFull_en`). If ambiguous, UO engages `@clarification-agent` to ask PO/requester for clarification (question posed in their language, based on `userLanguage` of PO if known, or default project language). Workflow pauses pending response (processed by `01_AI-RUN/XX_Handle_Clarification_Response.md`).
    *   **Phase 3: Technical Task Breakdown & Estimation (Detailed English Analysis and Planning).**
        *   UO evaluates if (re)decomposition is needed based on `.pheromone.activeUserStory.tasks` and `memoryBank.userStories[ID_US].tasks`.
        *   If yes, UO **injects targeted English context** and delegates to `@task-breakdown-estimator`. Agent uses **Sequential Thinking MCP**, **Context7 MCP**, **MSSQL MCP** (all with English logic), and must **detail its English "chain of thought"** in its English report.
        *   Tasks/estimations (English) are synced with Azure DevOps by `@devops-connector` (ADO items can be in project's ADO language).
        *   **onError:** If `@task-breakdown-estimator` fails or reports persistent ambiguity, UO logs error and may re-engage `@clarification-agent` or notify Tech Lead (in their language).
    *   **Phase 4: First Task Preparation & Development Environment Setup.**
        *   UO delegates to `@developer-agent` to initialize task, create Git branch.
        *   **onError:** If branch creation fails, log and notify (in `userLanguage`).

## Phase Details:

### Phase 1: User Identification & Full US Detail Retrieval
*   **Responsible Agent:** `@devops-connector`.
*   **Inputs:** US ID (e.g., `Azure#12323`) from UO. Current user info from `.pheromone.currentUser`.
*   **Actions & Tooling:**
    1.  Use **Azure DevOps MCP**:
        *   `get_user_identity {identifier: .pheromone.currentUser.azureDevOpsUsername}` (to ensure current Pheromind user matches ADO context).
        *   `get_work_item_details {id: US_ID}`: Retrieve title, **full** description, ADO status, priority, ACs, etc. (These will be in ADO's original language).
*   **onError Strategy (for UO if `@devops-connector` reports MCP failure):**
    1.  Scribe logs English error in `activeWorkflow.lastError` and `memoryBank.agentActivityLog`.
    2.  UO notifies user (in `currentUser.lastInteractionLanguage`): "Unable to retrieve details for US Azure#{{usId}} from Azure DevOps. MCP Error: [Error message]. Please check connection or US ID."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`, English NL Summary with original language fields):** "User '{{currentUser.azureDevOpsUsername}}' confirmed. Full details for US 'Azure#{{usId}}' ('{{usTitleFromADO_origLang}}') retrieved. Description (orig lang): '{{usDescriptionFromADO_origLang}}'. Azure Status: '{{usAzureStatus}}'. Priority: {{usPriority}}. ACs (orig lang): '{{usACsFromADO_origLang}}'. Log: `azure_wi_{{usId}}_{{timestamp}}.json`."

### Phase 2: Initial `.pheromone` Update & US Clarity Validation (English Internals)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`, `🧐 @uber-orchestrator`, `@clarification-agent`.
*   **Inputs:** English NL Summary from `@devops-connector` (containing original language fields). `currentUser.lastInteractionLanguage`.
*   **Actions & Tooling (Scribe):**
    1.  Interpret summary.
    2.  **Translate to English (Conceptual Task for Scribe or UO before Scribe acts):** If `usTitleFromADO_origLang`, `usDescriptionFromADO_origLang`, `usACsFromADO_origLang` are not English, they MUST be translated to English. Store these as `title_en`, `descriptionFull_en`, `acceptanceCriteria_en`.
    3.  Update `.pheromone`: `currentUser`. `activeUserStory`: { `id`, `title_en`, `status`: "InProgressByPheromind", `descriptionFull_en`, `acceptanceCriteria_en`, `azureStatus`, `priority`, `tasks`: [] }.
    4.  Update `memoryBank.userStories.{{usId}}`: Create/update with `title_en`, `descriptionFull_en`, `acceptanceCriteria_en`, `azureStatus`, `priority`, and `statusHistory_en` (add "InProgressByPheromind" entry).
*   **Actions & Tooling (UO):**
    1.  Analyze `activeUserStory.descriptionFull_en` and `activeUserStory.acceptanceCriteria_en` for clarity.
    2.  **If ambiguity detected:** Pause workflow. Prepare English context and English question for `@clarification-agent`. Delegate to `@clarification-agent`, providing `userLanguage` for question delivery to PO/requester. Await response via `01_AI-RUN/XX_Handle_Clarification_Response.md`.
*   **Output:** `.pheromone` updated with English core data. Workflow paused if clarification needed.

### Phase 3: Technical Task Breakdown & Estimation (Detailed English Analysis and Planning)
*   **Responsible Agent:** `@task-breakdown-estimator` (coordinates with `@devops-connector`).
*   **Inputs (Injected by UO, all English):** `activeUserStory` (clarified English desc/ACs). `memoryBank` context.
*   **Actions (`@task-breakdown-estimator`):**
    1.  Use **Sequential Thinking MCP** (English) for decomposition plan.
    2.  **Detail "Chain of Thought" (English):** In report (`us_{{usId}}_task_breakdown_{{timestamp}}.md`), document: How English ACs -> tech needs. Decomposition choices rationale. Influence of Context7/MSSQL MCP. Estimation basis.
    3.  Propose English technical tasks, estimates, dependencies.
    4.  Sync with Azure DevOps via `@devops-connector` (task titles/desc can be English or translated by `@devops-connector` or UO if ADO project uses another language).
*   **onError Strategy (for UO):** If failure/persistent ambiguity, log (English), may re-engage `@clarification-agent`, or notify Tech Lead (in their language).
*   **Output (to Scribe, English NL Summary):** "US 'Azure#{{usId}}' decomposition [completed/updated]. [N_total] English tasks, total [TotalEstimation] {{estimationUnit}}. Synced with ADO. English Report (with reasoning): `us_{{usId}}_task_breakdown_{{timestamp}}.md`." (Report in `03_SPECS/Task_Breakdowns/`). Scribe updates `memoryBank` with English task details (`title_en`, `description_en`).

### Phase 4: First Task Preparation & Development Environment Setup
*   **Responsible Agent:** `@developer-agent`.
*   **Inputs (Injected by UO, English context):** `activeUserStory` (with English task details). First `ToDo` task. `memoryBank.projectContext.defaultGitBranchingStrategy`.
*   **Actions & Tooling:**
    1.  Identify first `ToDo` task (based on English `title_en`).
    2.  Use **Git Tools MCP**: `create_branch {branchName: feature/US{{usId}}-{{taskShortName_en}}}`, `checkout_branch`.
    3.  Update task in `.pheromone` (via Scribe): `status: "InProgress"`, `assignee: currentUser.id`.
*   **onError Strategy (for UO):** If Git MCP fails, Scribe logs English error. UO notifies user (in `userLanguage`): "Error creating/checking out Git branch for task Azure#{{taskId}}. MCP Error: [Message]. Please check Git config."
*   **Output (to Scribe, English NL Summary):** "Ready for task 'Azure#{{taskId}}' ('{{taskTitle_en}}') of US 'Azure#{{usId}}'. Assigned to '{{currentUser.azureDevOpsUsername}}'. Git branch '{{branchName}}' active/created."

---