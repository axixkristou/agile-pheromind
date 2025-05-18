# Workflow: Start a User Story (01_Start_User_Story.md)

**Objective:** Initialize work on a specific Azure DevOps User Story (US). This process involves user identification, retrieving complete US details, injecting relevant context from the `memoryBank`, breaking down the US into estimated technical tasks (with logging of the "chain of thought"), synchronizing with Azure DevOps, handling errors and ambiguities, and preparing the first task for development.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@task-breakdown-estimator`, `@developer-agent`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Git Tools MCP, Sequential Thinking MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The user (Dev) provides the Azure DevOps US ID (e.g., `"AgilePheromind start US Azure#12323"`).
2.  **`🧐 @uber-orchestrator`** (UO) takes control.
    *   **Phase 1: User Identification & Retrieval of Complete US Details.**
        *   UO delegates to `@devops-connector`.
        *   **onError:** If ADO MCP fails, log the error, notify the user, and stop the workflow.
    *   **Phase 2: Initial Update of `.pheromone` and US Clarity Validation.**
        *   Scribe updates `.pheromone` (`currentUser`, `activeUserStory`).
        *   UO evaluates the clarity of the US description. If ambiguous, UO engages `@clarification-agent` to request clarification from the PO/requester via `ask_followup_question`. The workflow waits for the response (processed by `01_AI-RUN/XX_Handle_Clarification_Response.md`).
    *   **Phase 3: Breakdown into Technical Tasks & Estimation (Detailed Analysis and Planning).**
        *   UO evaluates if a (re)decomposition is necessary (based on `.pheromone.activeUserStory.tasks` and `memoryBank.userStories[ID_US].tasks`).
        *   If yes, UO **injects targeted context** (e.g., past similar US, relevant .NET/Angular conventions from `memoryBank`) and delegates to `@task-breakdown-estimator`. The agent uses **Sequential Thinking MCP**, **Context7 MCP**, **MSSQL MCP**, and must **detail its "chain of thought"** in its report.
        *   Tasks/estimates are synchronized with Azure DevOps via `@devops-connector`.
        *   **onError:** If `@task-breakdown-estimator` fails or reports persistent ambiguity, UO logs the error and may re-engage `@clarification-agent` or notify the Tech Lead.
    *   **Phase 4: Preparation of the First Task & Development Environment.**
        *   UO delegates to `@developer-agent` to initialize the task, create Git branch.
        *   **onError:** If branch creation fails, log and notify.

## Phase Details:

### Phase 1: User Identification & Retrieval of Complete US Details
*   **Responsible Agent:** `@devops-connector`
*   **Inputs:** US ID (e.g., `Azure#12323`) provided by the UO.
*   **Actions & Tooling:**
    1.  Use **Azure DevOps MCP**:
        *   `get_user_identity`: Confirm/Identify the user.
        *   `get_work_item_details {id: ID_US}`: Retrieve title, **complete** description, status, priority, ACs (if in a dedicated field), etc.
*   **onError Strategy (for UO if `@devops-connector` reports MCP failure):**
    1.  Scribe logs the error in `activeWorkflow.lastError` and `memoryBank.agentActivityLog`.
    2.  UO notifies the user: "Unable to retrieve details for US Azure#{{usId}} from Azure DevOps. MCP Error: [Error message]. Please check the connection or US ID."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe` if successful):** NL Summary: "User '[AzureDevOpsUsername]' confirmed. Complete details for US 'Azure#{{usId}}' ('{{usTitle}}') retrieved. Description: '{{usDescription}}'. Azure Status: '{{usAzureStatus}}'. Priority: {{usPriority}}. ACs: '{{usAcceptanceCriteria}}'. Log: `azure_wi_{{usId}}_{{timestamp}}.json`."

### Phase 2: Initial Update of `.pheromone` and US Clarity Validation
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`, `🧐 @uber-orchestrator`, `@clarification-agent`
*   **Inputs:** NL Summary from `@devops-connector`.
*   **Actions & Tooling (Scribe):**
    1.  Update `.pheromone` with `currentUser`, `activeUserStory` (including `descriptionFull`, `acceptanceCriteriaFromAzure`), and enrich `memoryBank.userStories.{{usId}}`.
*   **Actions & Tooling (UO):**
    1.  Analyze `activeUserStory.descriptionFull` and `activeUserStory.acceptanceCriteriaFromAzure` to evaluate their clarity and usability for decomposition.
    2.  **If ambiguity detected** (e.g., vague ACs, description lacking critical details):
        *   Store the current workflow state in `.pheromone.activeWorkflow` (e.g., `status: 'PendingClarification'`).
        *   Prepare the context for `@clarification-agent`: the US ID, the problematic description/ACs, and the specific question to ask (e.g., "For US Azure#{{usId}}, the acceptance criterion 'The interface must be intuitive' needs more details. Can you specify 2-3 key aspects of this expected intuitiveness?").
        *   Delegate to `@clarification-agent`.
        *   The `01_Start_User_Story.md` workflow is paused. The user's response will be processed by `01_AI-RUN/XX_Handle_Clarification_Response.md`, which will then reactivate this workflow if clarification is obtained.
*   **Output:** `.pheromone` updated. If clarification requested, workflow paused. Otherwise, UO proceeds to Phase 2.5.

### Phase 2.5: Impact and Risk Analysis
*   **Responsible Agent:** `@architecture-advisor-agent`
*   **Inputs:** `activeUserStory` (with clarified description and ACs if clarification occurred).
*   **Actions & Tooling:**
    1.  Use **Sequential Thinking MCP** to methodically analyze potential impacts.
    2.  Consult `memoryBank.architecturalDecisions` and `memoryBank.projectContext` to identify affected components.
    3.  Analyze potential impacts on existing components:
        *   Database schema modifications.
        *   API changes or interface contract modifications.
        *   Performance or security impacts.
    4.  Identify technical risks:
        *   High technical complexity.
        *   Uncontrolled external dependencies.
        *   Potential conflicts with other ongoing developments.
    5.  Evaluate dependencies with other US/features.
    6.  Propose mitigation strategies for each identified risk.
    7.  **Detail the "Chain of Thought":** Explicitly document reasoning for each identified impact and risk.
*   **onError Strategy (for UO if `@architecture-advisor-agent` reports failure):**
    1.  Scribe logs error.
    2.  UO notifies user: "Unable to fully analyze impacts and risks for US Azure#{{usId}}. Error: [Error message]. We will proceed with decomposition with limited visibility on risks."
    3.  UO proceeds to Phase 3 with a note about incomplete risk analysis.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Impact analysis for US 'Azure#{{usId}}' completed. [N] impacts identified, [M] risks detected, [P] mitigation strategies proposed. Report (with chain of thought): `us_{{usId}}_impact_analysis_{{timestamp}}.md`."

### Phase 3: Breakdown into Technical Tasks & Estimation (Detailed Analysis and Planning)
*   **Responsible Agent:** `@task-breakdown-estimator` (coordinating with `@devops-connector`)
*   **Inputs (Injected by UO):**
    *   `activeUserStory` (with clarified description and ACs if Phase 2 occurred).
    *   **Context from `memoryBank`:**
        *   `memoryBank.projectContext` (techStack, conventions, estimationUnit).
        *   (Optional) Examples of past similar US decompositions (`memoryBank.userStories` where `story.type == activeUserStory.type`).
        *   (Optional) Relevant architectural decisions (`memoryBank.architecturalDecisions`).
*   **Actions & Tooling (`@task-breakdown-estimator`):**
    1.  Use **Sequential Thinking MCP** to plan the decomposition.
    2.  **Detail the "Chain of Thought":** Explicitly document in the decomposition report (`us_{{usId}}_task_breakdown_{{timestamp}}.md`):
        *   How ACs were translated into technical needs.
        *   Why certain decomposition choices were made (e.g., creating a separate .NET service vs. modifying an existing one).
        *   Which documentation (from **Context7 MCP**) or schema analyses (**MSSQL MCP**) influenced the proposed tasks.
        *   The basis for each estimate.
    3.  Propose technical tasks, estimates, dependencies.
    4.  Consult `@devops-connector` to synchronize with Azure DevOps (**Azure DevOps MCP**).
*   **onError Strategy (for UO if `@task-breakdown-estimator` reports failure or ambiguity):**
    1.  Scribe logs the error.
    2.  If ambiguity, UO may restart Phase 2 with `@clarification-agent` targeting the ambiguity point raised by `@task-breakdown-estimator`.
    3.  If MCP failure or other, notify the Tech Lead/PO. Stop or propose an alternative.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "US 'Azure#{{usId}}' decomposition [completed/updated]. [N_total] tasks, total [TotalEstimation] {{estimationUnit}}. Synchronized with ADO. Report (with chain of thought): `us_{{usId}}_task_breakdown_{{timestamp}}.md`." (Report in `03_SPECS/Task_Breakdowns/`).

### Phase 4: Preparation of the First Task & Development Environment
*   **Responsible Agent:** `@developer-agent`
*   **Inputs (Injected by UO):**
    *   `activeUserStory` (with populated and estimated tasks).
    *   First `ToDo` task identified.
    *   Context from `memoryBank`: `projectContext.defaultGitBranchingStrategy`.
*   **Actions & Tooling:**
    1.  Identify the first `ToDo` task.
    2.  Use **Git Tools MCP**: `create_branch`, `checkout_branch`.
    3.  Update the task in `.pheromone` (via Scribe): `status: "InProgress"`, `assignee: currentUser.id`.
*   **onError Strategy (for UO if `@developer-agent` reports Git Tools MCP failure):**
    1.  Scribe logs the error.
    2.  UO notifies the developer: "Error during Git branch creation/checkout for task Azure#{{taskId}}. MCP Error: [Message]. Please check your Git configuration."
    3.  The workflow may stop or wait for manual action.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Ready for task 'Azure#{{taskId}}' ('{{taskTitle}}') of US 'Azure#{{usId}}'. Assigned to '{{currentUser.azureDevOpsUsername}}'. Git branch '{{branchName}}' active/created."

---