# Workflow: Daily Stand-up Meeting Support (08_Daily_Standup_Support.md)

**Objective:** Provide the Agile team with a concise and relevant summary of the current sprint's progress to facilitate an effective Daily Stand-up. The system aggregates information from Azure DevOps and `.pheromone` (Memory Bank), identifies progress, attention points, and potential blockers.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@scrum-facilitator-agent`.

**MCPs Used:** Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Manual:** Scrum Master/Tech Lead requests a summary (e.g., `"AgilePheromind prepare Daily summary for Sprint {{currentSprint.name}}"`).
    *   **Automatic:** Scheduled trigger (e.g., every morning before the Daily).
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Active Sprint Updates Collection.**
        *   UO delegates to `@devops-connector` for fresh Azure DevOps data (US/task statuses, assignees).
        *   UO delegates to `@scrum-facilitator-agent` to analyze recent activity in `.pheromone.memoryBank` (commits, developer notes, Pheromind statuses).
        *   **onError:** If ADO MCP fails, the report will be based solely on `.pheromone` data, with a warning.
    *   **Phase 2: Progress, Attention Points, and Blockers Identification.**
        *   UO delegates to `@scrum-facilitator-agent` to consolidate and analyze.
    *   **Phase 3: Structured Summary Generation for the Daily.**
        *   UO delegates to `@scrum-facilitator-agent`.
    *   **Phase 4: Summary Recording and Distribution (Optional).**
        *   Scribe records the report. UO can notify the team.

## Phase Details:

### Phase 1: Active Sprint Updates Collection
*   **Responsible Agent:** `@devops-connector` (for ADO), `@scrum-facilitator-agent` (for `.pheromone`).
*   **Inputs:** `currentSprint.id` and `currentSprint.userStories` from `.pheromone`. `memoryBank.lastAdoSyncTimestamp` for differential sync.
*   **Actions (`@devops-connector`):**
    1.  Use **Azure DevOps MCP**:
        *   For each `usId` in `currentSprint.userStories`, retrieve `get_work_item_details` and `get_child_work_items`.
        *   For each child `taskId`, `get_work_item_details` (status, assignee, remaining/completed work if tracked).
        *   (Optional) Filter queries to retrieve only items modified since `memoryBank.lastAdoSyncTimestamp` if the MCP allows it.
*   **onError (ADO MCP for `@devops-connector`):**
    *   If failure, `@devops-connector` reports to the UO. The UO notes that the summary will be based on the latest known Pheromind data and includes a warning in the final report. The workflow continues if possible.
*   **Output (`@devops-connector` -> Scribe):** NL Summary: "ADO Update Sprint '{{currentSprint.name}}' OK. [Stats]. Log: `sprint_{{sprintId}}_ado_update_{{timestamp}}.json`." or "ADO Update Failed: [Error]."
*   **Actions (Scribe after `@devops-connector` summary):**
    1.  Update `memoryBank.userStories` and `memoryBank.tasks` with `azureStatus`, `azureAssignee`, etc. from ADO.
    2.  Update `memoryBank.lastAdoSyncTimestamp`.
*   **Actions (`@scrum-facilitator-agent`):**
    1.  Read `.pheromone.memoryBank` for sprint tasks:
        *   `statusHistory`, `developerNotes`, `relatedCommits` (since the last Daily or the last 24h).
        *   `clarificationHistory` to see if points have been recently clarified.
*   **Output (internal to `@scrum-facilitator-agent`):** Consolidated data on recent activity.

### Phase 2: Progress, Attention Points, and Blockers Identification
*   **Responsible Agent:** `@scrum-facilitator-agent`.
*   **Inputs:** Consolidated data (Phase 1). `memoryBank.riskRegister`.
*   **Actions & Tooling:**
    1.  **Analyze Progress:** Tasks moved to "Done" (in ADO or Pheromind). Completed US. Significant commits.
    2.  **Analyze Attention Points/Blockers:**
        *   "InProgress" tasks without recent commit/note (> X hours/days).
        *   Tasks whose estimate is soon/already exceeded.
        *   `developerNotes` mentioning "blocker", "issue", "waiting".
        *   Dependencies between sprint tasks (if tracked in `memoryBank.tasks.{{taskId}}.dependencies`) where a blocking task is stagnating.
        *   Active risks from `memoryBank.riskRegister` related to sprint US/tasks.
        *   Pending clarifications (`clarificationContext.pendingClarificationId`).
*   **Output (internal to `@scrum-facilitator-agent`):** Structured lists: Progress, Attention Points.

### Phase 3: Structured Summary Generation for the Daily
*   **Responsible Agent:** `@scrum-facilitator-agent`.
*   **Inputs:** Lists from Phase 2. `currentSprint` from `.pheromone`.
*   **Actions & Tooling:**
    1.  Format a Markdown report (`daily_standup_summary_{{currentSprint.name}}_{{date}}.md`) in `03_SPECS/Daily_Summaries/`.
    2.  Report structure:
        *   **Sprint Goal:** `{{currentSprint.goal}}`.
        *   **Warning (if ADO sync failed):** "Note: Azure DevOps data could not be synchronized. This report is based on the last known state in Pheromind."
        *   **Completed Yesterday:**
            *   Task Azure#ID (Title) - by [Assignee] - (Commit: Hash if available)
        *   **In Progress Today (Main Focus):**
            *   Task Azure#ID (Title) - by [Assignee] - (Last Pheromind activity: Note/Commit)
        *   **Identified Attention Points / Blockers:**
            *   Task Azure#ID: [Reason for attention - e.g., No progress for X time, Blocker mentioned: "..."]
            *   Active Risk: [Risk ID] - [Description] - Impacts US Azure#ID_US.
            *   Pending clarification: ID `{{clarificationContext.pendingClarificationId}}` for agent `{{clarificationContext.originalAgent}}`.
*   **Output (to Scribe and UO):** NL Summary: "Daily summary Sprint '{{currentSprint.name}}' ({date}) ready. Completed yesterday: [N_done]. In progress: [N_inprogress]. Attention points: [N_blockers]. Report: `daily_standup_summary_{{currentSprint.name}}_{{date}}.md`."

### Phase 4: Summary Recording and Distribution (Optional)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`, UO.
*   **Inputs:** NL summary from `@scrum-facilitator-agent`.
*   **Actions (Scribe):**
    1.  Update `.pheromone`:
        *   `documentationRegistry`: Add path to `daily_standup_summary...md`.
        *   `memoryBank.sprints.{{currentSprint.id}}.dailySummaryLinks[]`: Add link.
*   **Actions (UO - Optional):**
    1.  If notification MCP configured, send summary/link to the team.
*   **Output:** `.pheromone` updated. Team potentially notified.

---