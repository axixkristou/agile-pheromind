# Workflow: Sprint Planning Assistance (07_Sprint_Planning_Assistant.md)

**Objective:** Assist the Product Owner (PO) and development team in planning a sprint. The system:
1. Retrieves candidate User Stories (US) from Azure DevOps.
2. Ensures US are properly decomposed and estimated in English (injecting English context, requesting clarifications in user's language if needed).
3. Proposes a sprint plan based on team capacity, US priorities, and identified dependencies.
4. Documents the "chain of thought" for selection in English.
5. Presents the final proposed sprint plan summary to the user in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@task-breakdown-estimator`, `@scrum-facilitator-agent`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Sequential Thinking MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User (PO/Scrum Master) provides candidate US IDs and team capacity (e.g., `"AgilePheromind plan sprint. US: Azure#123, Azure#456. Capacity: 40 points."`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Candidate US Detail Retrieval and Initial Clarity Validation.**
        *   UO delegates to `@devops-connector` for US info from ADO (original language).
        *   Scribe/UO ensures English versions (`title_en`, `descriptionFull_en`) in `memoryBank`.
        *   UO evaluates clarity of English US content. If vague, UO engages `@clarification-agent` (question to PO in `userLanguage`).
        *   **onError (ADO MCP):** Log, notify user (in `userLanguage`), stop or continue with partial data.
    *   **Phase 2: Verification/Finalization of Estimates and Task Breakdown (English).**
        *   UO, for each candidate US: **Injects targeted English context** to `@task-breakdown-estimator`. Agent verifies/performs English estimation & decomposition, details English "chain of thought". Syncs with ADO.
        *   **onError:** If estimation impossible or MCP fails, US may be excluded with a note, or clarification re-requested.
    *   **Phase 3: Sprint Plan Proposal with Dependency/Risk Analysis (English).**
        *   UO **injects English context** to `@scrum-facilitator-agent`. Agent selects US/tasks, uses **Sequential Thinking MCP** for English dependency/risk analysis. Details English "chain of thought".
    *   **Phase 4: Plan Recording (English Data) and Reporting (Localized Summary).**
        *   Scribe records proposed plan (English data) and English report in `.pheromone`.
        *   UO presents summary of plan to user (translated to `userLanguage`).

## Phase Details:

### Phase 1: Candidate US Detail Retrieval and Initial Clarity Validation
*   **Responsible Agent:** `@devops-connector`, Scribe, UO, `@clarification-agent`.
*   **Inputs:** List of candidate US IDs, `currentUser`, `userLanguage`.
*   **Actions (`@devops-connector`):** For each US ID, **ADO MCP** `get_work_item_details` (original lang).
*   **onError (ADO MCP):** UO logs (English), notifies user (in `userLanguage`), may stop/continue.
*   **Output (`@devops-connector` -> Scribe, English NL Summary with original lang fields):** "Details for [N] candidate US retrieved: [List IDs/ADO Titles_origLang]. Log: `sprint_planning_us_fetch_{{timestamp}}.json`."
*   **Actions (Scribe):** UO/Scribe conceptually translates ADO content to English. Store/Update `memoryBank.userStories.{{usId}}` with `title_en`, `descriptionFull_en`, `title_originalLang`, etc.
*   **Actions (UO - Clarity Validation):** For each US (using English content): Evaluate clarity. **If major ambiguity** for estimation: Pause. Delegate to `@clarification-agent` (question translated to `userLanguage` for PO). Await response.

### Phase 2: Verification/Finalization of Estimates and Task Breakdown (English)
*   **Responsible Agent:** `@task-breakdown-estimator` (coordinates with `@devops-connector`).
*   **Inputs (Injected by UO, English):** US details (clarified if needed). `memoryBank` context.
*   **Actions (`@task-breakdown-estimator`):**
    1.  Check `memoryBank.userStories.{{usId}}` for reliable English estimation & task breakdown.
    2.  If not/revision needed: Engage English decomposition/estimation. **Document English "Chain of Thought"** in report.
    3.  Sync tasks/estimates with ADO via `@devops-connector`.
*   **onError (Estimation/Decomposition):** Agent reports (English) to UO. UO may exclude US or re-clarify.
*   **Output (`@task-breakdown-estimator` -> Scribe, English NL Summary):** "Estimates/breakdowns finalized for candidate US. [N] US processed. Individual English reports (with chain of thought): `us_{{usId}}_task_breakdown_{{timestamp}}.md`." Scribe updates `memoryBank` (English data).

### Phase 3: Sprint Plan Proposal with Dependency/Risk Analysis (English)
*   **Responsible Agent:** `@scrum-facilitator-agent`.
*   **Inputs (Injected by UO, English):** Candidate US with English estimates. Team capacity. US priorities. `memoryBank.riskRegister_en`.
*   **Actions (`@scrum-facilitator-agent`):**
    1.  Sort US by priority. Iteratively select up to capacity.
    2.  **Dependency/Risk Analysis (Sequential Thinking MCP, English):** Analyze feasibility, dependencies, related risks.
    3.  **Document "Chain of Thought" (English):** Explain selection logic and analysis in final English report.
*   **Output (`@scrum-facilitator-agent` -> Scribe, English NL Summary):** "Sprint plan proposal [ID/Name TBD]: [List US IDs]. Total [Points]/[Capacity] {{estimationUnit}}. Risks/Dependencies (English): [Summary]. English Report (with chain of thought): `sprint_plan_proposal_{{timestamp}}.md`." (Report in `02_AI-DOCS/Sprint_Plans/`).

### Phase 4: Plan Recording (English Data) and Reporting (Localized Summary)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`, UO.
*   **Inputs:** English NL Summary from `@scrum-facilitator-agent`. `currentUser.lastInteractionLanguage`.
*   **Actions (Scribe):** Update `.pheromone` (English data): `currentSprint` (plan info), `documentationRegistry` (English report path), `memoryBank.userStories.{{usId}}.sprintAssignment`, `memoryBank.sprints.{{currentSprint.id}}.reasoningChainLinks_en.planning`.
*   **Actions (UO):**
    1.  Generate English summary of proposed plan. Translate to `currentUser.lastInteractionLanguage`.
    2.  `ask_followup_question` to PO/SM (in `userLanguage`): "Proposed sprint plan: [Translated Summary]. Total points: X/Y. Detailed English report available. Options: 1. View report (English)? 2. Define Sprint Name & Goal? 3. Assign tasks in ADO?" (Scribe handles translation of Name/Goal to English for `memoryBank`).
*   **Output:** `.pheromone` updated with English plan data. UO interacts with user in their language.

---