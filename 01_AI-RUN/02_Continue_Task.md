# Workflow: Continue a Development Task (02_Continue_Task.md)

**Objective:** Enable a developer to efficiently resume work on a specific technical task. The system loads the complete English task context (including decision history and previous English reasoning stored in the `memoryBank`), prepares the Git environment, and provides contextual assistance (potentially requiring English to `userLanguage` translation for display during interaction) during implementation. Error management and clarification mechanisms are integrated.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@developer-agent`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Git Tools MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User (Dev) specifies the Azure DevOps task ID to continue (e.g., `"AgilePheromind continue task Azure#23223"`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: User Identification & Azure DevOps Task Context Retrieval/Validation.**
        *   UO delegates to `@devops-connector`. Task details from ADO are retrieved in their original language.
        *   **onError:** If ADO MCP fails, log English error, notify user (in `userLanguage`), stop.
    *   **Phase 2: Task Activation and Deep English Context Loading from `.pheromone`.**
        *   Scribe updates `activeTask` (with `title_en`), `activeUserStory` (with `title_en`) in `.pheromone` after conceptual translation of ADO content to English if necessary by UO/Scribe.
        *   UO **injects targeted English context** from `memoryBank` to `@developer-agent`.
    *   **Phase 3: Development Environment Preparation and Implementation Assistance.**
        *   UO delegates to `@developer-agent` to check/change Git branch, open files.
        *   During implementation, if `@developer-agent` encounters ambiguity (in understanding English context or code), it reports (English) to UO. UO can initiate clarification via `@clarification-agent` (question translated to `userLanguage`).
        *   **onError:** Management of MCP failures or agent logic errors.

## Phase Details:

### Phase 1: User Identification & Azure DevOps Task Context Retrieval/Validation
*   **Responsible Agent:** `@devops-connector`.
*   **Inputs:** Task ID from UO. `currentUser` from `.pheromone`.
*   **Actions & Tooling:**
    1.  Use **Azure DevOps MCP**:
        *   `get_user_identity {identifier: .pheromone.currentUser.azureDevOpsUsername}`.
        *   `get_work_item_details {id: Task_ID}`: Retrieve title, description, ADO status, parent US (ID and title), ADO assignee (all in ADO's original language).
*   **onError Strategy (for UO):** If ADO MCP fails, Scribe logs English error. UO notifies user (in `currentUser.lastInteractionLanguage`): "Unable to retrieve task Azure#{{taskId}} details. MCP Error: [Msg].", stop workflow.
*   **Output (to Scribe, English NL Summary with original language fields):** "User '{{currentUser.azureDevOpsUsername}}' confirmed. Task 'Azure#{{taskId}}' ('{{taskTitleFromADO_origLang}}') context retrieved. Parent US: 'Azure#{{parentIdFromADO}} - {{parentTitleFromADO_origLang}}'. Azure Status: '{{taskAzureStatus}}'. Azure Assignee: '{{taskAzureAssignee}}'. Log: `azure_wi_{{taskId}}_{{timestamp}}.json`."

### Phase 2: Task Activation and Deep English Context Loading from `.pheromone`
*   **Responsible Agent:** Scribe (active state update), UO (context injection), `@developer-agent` (reading).
*   **Inputs:** English NL Summary from `@devops-connector`. `.pheromone` data.
*   **Actions & Tooling (Scribe):**
    1.  Interpret summary.
    2.  **Translate to English (Conceptual for Scribe/UO):** If `taskTitleFromADO_origLang`, `parentTitleFromADO_origLang` not English, translate for internal English storage.
    3.  Update `.pheromone`:
        *   `activeUserStory`: If parent US differs, load its English details from `memoryBank.userStories` into `activeUserStory` (e.g., `activeUserStory.title_en`).
        *   `activeTask`: { `id`, `title_en` (translated), `status`: "InProgressByPheromind", `parentId` }.
        *   Update specific task in `activeUserStory.tasks` (English `title_en`, status).
        *   `memoryBank.tasks.{{taskId}}`: Update `title_en`, `azureStatus`, `azureAssignee`. Add to `statusHistory_en`.
*   **Actions & Tooling (UO prepares English context injection for `@developer-agent`):**
    1.  Extract from `memoryBank.tasks.{{activeTask.id}}`: `developerNotes_en`, `codeSnippets` (paths), `testCasesGeneratedLinks_en`, `relatedCommit`, `reasoningChainLink_en`.
    2.  Extract from `memoryBank.userStories.{{activeTask.parentId}}`: `descriptionFull_en`, `acceptanceCriteria_en`, `analysisSummaries_en`, `keyDecisions_en`.
    3.  Extract from `memoryBank.projectContext`: `codingConventionsLink`, `designConventionsLink`, `techStack`.
    4.  Extract from `memoryBank.commonIssuesAndSolutions_en`.
*   **Actions (`@developer-agent` receives this English context).**
*   **Output (for `@developer-agent` in Phase 3):** Rich, targeted English context.

### Phase 3: Development Environment Preparation and Implementation Assistance
*   **Responsible Agent:** `@developer-agent`.
*   **Inputs:** English context injected by UO (Phase 2). `currentUser.lastInteractionLanguage`.
*   **Actions & Tooling:**
    1.  **Git Branch Management (Git Tools MCP):** `get_current_branch`. Check vs expected `feature/US{{activeUserStory.id}}-{{english_us_short_title}}`. If needed, `checkout_branch` or `pull_branch` + `checkout_branch`. Report Git errors (English) to UO.
    2.  **File Opening:** (Conceptual) IDE opens relevant files.
    3.  **Contextual Implementation Assistance (Developer codes, Agent assists):**
        *   On request (from Dev in `userLanguage`, agent processes in English):
            *   **Context7 MCP** (`get_library_docs`): For .NET/Angular docs.
            *   **MSSQL MCP** (`get_schema_details`, `validate_sql_query`): For DB help.
            *   Consult English `memoryBank` (via UO/@memory-access-agent) for past decisions/solutions.
        *   **Ambiguity/Blocker Management by `@developer-agent`:**
            *   If English specs/context unclear or dependency blocks: Formulate problem (English). Send English summary to UO: "Blocker task Azure#{{taskId}}: [English problem]. Suggest clarification: [Precise English question]."
            *   UO initiates clarification via `@clarification-agent` (question translated to `userLanguage`). Task work paused.
    4.  **Work Tracking & Note Taking (for English `memoryBank`):** Agent notes (for its final English summary) key English technical decisions, "chain of thought" for complex solutions, problems solved, new affected files.
*   **onError Strategy (for UO):** If `@developer-agent` reports MCP failure or insoluble blocker (English): Scribe logs English error. UO may suggest Dev checks MCPs or finds info manually. If logical blocker, UO initiates clarification or escalates (user notified in `userLanguage`).
*   **Output (to Scribe at end of session, English NL Summary):** "Session on task Azure#{{taskId}}. Progress: [English description]. Files modified: [List]. Key English decisions/reasoning: [For MemoryBank]. Issues/Clarifications needed (English): [If applicable]." (Scribe updates `developerNotes_en`, `affectedFiles`, `reasoningChainLink_en` in `memoryBank.tasks.{{activeTask.id}}`).

---