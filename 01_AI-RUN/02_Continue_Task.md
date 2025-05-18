# Workflow: Continue Development Task (02_Continue_Task.md)

**Objective:** Enable a developer to efficiently resume work on a specific technical task. The system loads the complete task context (including decision history and previous reasoning stored in the `memoryBank`), prepares the Git environment, and provides contextual assistance during the implementation phase, with error management and clarification mechanisms.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@developer-agent`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Git Tools MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The user (Dev) specifies the Azure DevOps task ID to continue (e.g., `"AgilePheromind continue task Azure#23223"`).
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: User Identification & Azure DevOps Task Context Retrieval/Validation.**
        *   UO delegates to `@devops-connector`.
        *   **onError:** If ADO MCP fails, log, notify, stop.
    *   **Phase 2: Task Activation and Deep Context Loading from `.pheromone`.**
        *   Scribe updates `activeTask`, `activeUserStory` in `.pheromone`.
        *   UO **injects targeted context** from `memoryBank` (notes, previous decisions, snippets, related reasoning) to `@developer-agent`.
    *   **Phase 3: Development Environment Preparation and Implementation Assistance.**
        *   UO delegates to `@developer-agent` to check/change Git branch, open files.
        *   During implementation, if `@developer-agent` encounters ambiguity or a blocker, it reports to the UO who can initiate clarification via `@clarification-agent` or apply an error strategy.
        *   **onError:** Management of MCP failures (Context7, MSSQL) or agent logic errors.

## Phase Details:

### Phase 1: User Identification & Azure DevOps Task Context Retrieval/Validation
*   **Responsible Agent:** `@devops-connector`
*   **Inputs:** Task ID provided by the UO. `currentUser` from `.pheromone`.
*   **Actions & Tooling:**
    1.  Use **Azure DevOps MCP**:
        *   `get_user_identity`.
        *   `get_work_item_details {id: Task_ID}`: Retrieve title, description, ADO status, parent US, ADO assignee.
*   **onError Strategy (for UO if `@devops-connector` reports MCP failure):**
    1.  Scribe logs the error in `activeWorkflow.lastError`.
    2.  UO notifies the user: "Unable to retrieve details for task Azure#{{taskId}} from Azure DevOps. MCP Error: [Message]. Check connection or ID."
    3.  Stop this workflow.
*   **Output (to `✍️ @orchestrator-pheromone-scribe` if successful):** NL Summary: "User '{{currentUser.azureDevOpsUsername}}' confirmed. Task context 'Azure#{{taskId}}' ('{{taskTitle}}'), Parent US: 'Azure#{{parentId}} - {{parentTitle}}', Azure Status: '{{taskAzureStatus}}', Azure Assignee: '{{taskAzureAssignee}}'. Log: `azure_wi_{{taskId}}_{{timestamp}}.json`."

### Phase 2: Task Activation and Deep Context Loading from `.pheromone`
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe` (for active state), `🧐 @uber-orchestrator` (for context injection), `@developer-agent` (for reading).
*   **Inputs:** NL summary from `@devops-connector`. Data from `.pheromone`.
*   **Actions & Tooling (Scribe):**
    1.  Update `.pheromone` (`activeUserStory`, `activeTask`, statuses in `memoryBank.tasks.{{taskId}}` and `activeUserStory.tasks`).
*   **Actions & Tooling (UO prepares context injection for `@developer-agent`):**
    1.  Extract from `memoryBank.tasks.{{activeTask.id}}`: `developerNotes`, `codeSnippets`, `testCasesGeneratedLinks` (links to test generation reports), `relatedCommit`, `reasoningChainLink` (if previous decisions on this task have been logged).
    2.  Extract from `memoryBank.userStories.{{activeTask.parentId}}`: `descriptionFull`, `acceptanceCriteria`, `analysisSummaries`, `keyDecisions`, relevant `reasoningChainLinks`.
    3.  Extract from `memoryBank.projectContext`: `codingConventionsLink`, `designConventionsLink`, `techStack`.
    4.  Extract from `memoryBank.commonIssuesAndSolutions`: Previously solved similar issues.
*   **Actions & Tooling (`@developer-agent` receives this injected context):**
    1.  Integrate all this information to reconstruct the task state.
*   **Memory Bank Interaction:**
    *   Intensive reading by the UO to prepare the context.
*   **Output (for `@developer-agent` during Phase 3):** A rich and targeted context to resume work.

### Phase 3: Development Environment Preparation and Implementation Assistance
*   **Responsible Agent:** `@developer-agent`
*   **Inputs:** Context injected by the UO (Phase 2).
*   **Actions & Tooling:**
    1.  **Git Branch Management (Git Tools MCP):**
        *   `get_current_branch`. Check if it corresponds to `feature/US{{activeUserStory.id}}-description`.
        *   If not, `checkout_branch` or `pull_branch` + `checkout_branch`. Handle Git errors (conflicts, non-existent branch) by informing the UO.
    2.  **File Opening:** (Conceptual) Ask the IDE to open relevant files (`memoryBank.tasks.{{activeTask.id}}.affectedFiles`).
    3.  **Contextual Implementation Assistance:**
        *   While the developer codes, on request:
            *   **Context7 MCP** (`get_library_docs`): For .NET/Angular documentation.
            *   **MSSQL MCP** (`get_schema_details`, `validate_sql_query`): For DB help.
            *   Consult the `memoryBank` (via query to the UO or `@memory-access-agent`) for past decisions/solutions.
        *   **Ambiguity/Blocker Management by `@developer-agent`:**
            *   If a specification is ambiguous or if a dependency blocks:
                *   `@developer-agent` clearly formulates the problem.
                *   It sends a summary to the UO indicating: "Blocker on task Azure#{{taskId}}: [Problem/ambiguity description]. Clarification suggestion: [Precise question for PO/TechLead]."
                *   The UO can then decide to invoke `@clarification-agent` or follow another error strategy from the `01_AI-RUN/*.md` script. Work on this specific task is paused.
    4.  **Work Tracking and Note Taking for the Memory Bank:**
        *   The agent should be instructed to note (for its final summary) technical decisions made, reasons ("chain of thought" for complex solutions), problems solved, and new affected files.
*   **onError Strategy (for UO if `@developer-agent` reports MCP failure or unsolvable blocker):**
    1.  Scribe logs the error/blocker.
    2.  If MCP failure (Context7, MSSQL), UO can suggest to the dev to check the MCPs or manually look for the info for now. The workflow on the task can continue with this missing info if not critical.
    3.  If logical blocker, UO initiates the clarification process or escalates to the Tech Lead.
*   **Output (to `✍️ @orchestrator-pheromone-scribe` at the end of a work session / context save):** Detailed NL Summary: "Session on task Azure#{{taskId}}. Progress: [Description]. Modified files: [List]. Key decisions/reasoning: [For MemoryBank]. Issues/Clarifications needed: [If applicable]." (This summary allows the Scribe to update `developerNotes`, `affectedFiles`, `reasoningChainLink` in `memoryBank.tasks.{{activeTask.id}}`).

---