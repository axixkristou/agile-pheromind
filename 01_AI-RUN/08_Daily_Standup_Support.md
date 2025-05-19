# Workflow: Daily Stand-up Meeting Support (08_Daily_Standup_Support.md)

**Objective:** Provide the Agile team with a concise and relevant English summary of the current sprint's progress to facilitate an effective Daily Stand-up. The system aggregates information from Azure DevOps and `.pheromone` (English `memoryBank`), identifies progress, attention points, and potential blockers in English. The final summary for the team can be translated by the UO if the workflow was manually triggered by a user with a specific language preference.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@scrum-facilitator-agent`.

**MCPs Used:** Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Manual:** Scrum Master/Tech Lead requests summary (e.g., `"AgilePheromind prepare Daily summary for Sprint {{currentSprint.name}}"`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
    *   **Automatic:** Scheduled trigger. (UO uses default English for report, or a pre-configured language for notifications).
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage` if manually triggered.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Active Sprint Updates Collection.**
        *   UO delegates to `@devops-connector` for fresh Azure DevOps data (statuses, assignees - original language from ADO).
        *   Scribe (with UO help if needed for translation) updates `memoryBank` with English versions of titles/descriptions from ADO.
        *   UO delegates to `@scrum-facilitator-agent` to analyze recent activity in `.pheromone.memoryBank` (English data).
        *   **onError:** If ADO MCP fails, report will be based solely on `.pheromone` data, with an English warning in the report.
    *   **Phase 2: Identification of Progress, Attention Points, and Blockers (English Analysis).**
        *   UO delegates to `@scrum-facilitator-agent` to consolidate and analyze English data.
    *   **Phase 3: Generation of Structured Summary for the Daily (English).**
        *   UO delegates to `@scrum-facilitator-agent` to create an English report.
    *   **Phase 4: Summary Recording (English Data) and Distribution (Localized if manual trigger).**
        *   Scribe records the English report. UO can notify team (translating summary if manually triggered with a `userLanguage`).

## Phase Details:

### Phase 1: Active Sprint Updates Collection
*   **Responsible Agent:** `@devops-connector`, Scribe, `@scrum-facilitator-agent`.
*   **Inputs:** `currentSprint.id` and `currentSprint.userStories` from `.pheromone`. `memoryBank.lastAdoSyncTimestamp`.
*   **Actions (`@devops-connector`):**
    1.  Use **Azure DevOps MCP**: Retrieve US/task details (status, assignee, work remaining - in ADO's original language).
*   **onError (ADO MCP):** If failure, `@devops-connector` reports (English) to UO. UO notes summary will be based on last Pheromind data and includes English warning in final report.
*   **Output (`@devops-connector` -> Scribe, English NL Summary with original language fields):** "ADO updates for Sprint '{{currentSprint.name}}' retrieved. [Stats]. Log: `sprint_{{sprintId}}_ado_update_{{timestamp}}.json`." or "ADO Update Failed: [Error]."
*   **Actions (Scribe after `@devops-connector` summary):**
    1.  For each US/task, if ADO title/desc not English, UO/Scribe conceptually translates to English.
    2.  Update `memoryBank.userStories` and `memoryBank.tasks` with English `title_en`, `description_en` (if applicable), `azureStatus`, `azureAssignee`.
    3.  Update `memoryBank.lastAdoSyncTimestamp`.
*   **Actions (`@scrum-facilitator-agent`):**
    1.  Read `.pheromone.memoryBank` (English data) for current sprint tasks: `statusHistory_en`, `developerNotes_en`, `relatedCommits`. Read `clarificationHistory`.
*   **Output (internal to `@scrum-facilitator-agent`, English):** Consolidated data on recent activity.

### Phase 2: Identification of Progress, Attention Points, and Blockers (English Analysis)
*   **Responsible Agent:** `@scrum-facilitator-agent`.
*   **Inputs:** Consolidated English data (Phase 1). `memoryBank.riskRegister` (English).
*   **Actions & Tooling:**
    1.  **Analyze Progress (English):** Tasks "Done". US completed. Commits.
    2.  **Analyze Attention Points/Blockers (English):** "InProgress" tasks with no recent English activity. Overdue tasks. English `developerNotes_en` mentioning "blocker", "issue". Task dependencies. Active risks. Pending clarifications.
*   **Output (internal, English):** Structured lists: Progress, Attention Points.

### Phase 3: Generation of Structured Summary for the Daily (English)
*   **Responsible Agent:** `@scrum-facilitator-agent`.
*   **Inputs:** English lists from Phase 2. `currentSprint` (with `goal_en`) from `.pheromone`.
*   **Actions & Tooling:**
    1.  Format English Markdown report (`daily_standup_summary_{{currentSprint.name}}_{{date}}.md`) in `03_SPECS/Daily_Summaries/`.
    2.  English Report Structure:
        *   **Sprint Goal:** `{{currentSprint.goal_en}}`.
        *   **Warning (if ADO sync failed, English):** "Note: Azure DevOps data could not be synchronized..."
        *   **Completed Yesterday (English Titles):** Task Azure#ID (`title_en`) - by [Assignee]
        *   **In Progress Today (English Titles):** Task Azure#ID (`title_en`) - by [Assignee]
        *   **Identified Attention Points / Blockers (English):** Task Azure#ID: [English reason]. Risk: [ID] - [English Desc]. Pending clarification: `{{clarificationContext.pendingClarificationId}}`.
*   **Output (to Scribe and UO, English NL Summary):** "Daily Stand-up summary for Sprint '{{currentSprint.name}}' ({date}) is ready. [Stats English]. Detailed English report: `daily_standup_summary_{{currentSprint.name}}_{{date}}.md`."

### Phase 4: Summary Recording (English Data) and Distribution (Localized if manual trigger)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`, UO.
*   **Inputs:** English NL Summary from `@scrum-facilitator-agent`. `currentUser.lastInteractionLanguage` (if manually triggered).
*   **Actions (Scribe):**
    1.  Update `.pheromone` (English data):
        *   `documentationRegistry`: Add path to English `daily_standup_summary...md`.
        *   `memoryBank.sprints.{{currentSprint.id}}.dailySummaryLinks_en[]`: Add link to English report.
*   **Actions (UO - Optional Distribution):**
    1.  If manually triggered by user AND notification MCP configured:
        *   Take key points from English summary.
        *   Conceptually translate to `currentUser.lastInteractionLanguage`.
        *   Use notification MCP to send translated summary/link.
    2.  If automated, distribution defaults to English or pre-configured channel/language.
*   **Output:** `.pheromone` updated. Team potentially notified.

---