# Workflow: Sprint Planning Assistance (07_Sprint_Planning_Assistant.md)

**Objective:** Assist the Product Owner (PO) and development team in planning a sprint. The system retrieves candidate User Stories (US), ensures they are properly decomposed and estimated in English (injecting English context and requesting clarifications if needed via user's language), and proposes a sprint plan based on team capacity, US priorities, and identified dependencies. The "chain of thought" for selection is documented in English. The final proposed sprint plan summary for the user is presented in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@task-breakdown-estimator`, `@scrum-facilitator-agent`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Sequential Thinking MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User (PO/Scrum Master) provides a list of candidate US IDs and team capacity (e.g., `"AgilePheromind plan sprint. US: Azure#123, Azure#456. Capacity: 40 points."`). `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Candidate US Detail Retrieval and Initial Clarity Validation.**
        *   UO delegates to `@devops-connector` for US info from Azure DevOps (original language).
        *   Scribe (with UO help if needed for translation) stores English versions (`title_en`, `descriptionFull_en`) in `memoryBank`.
        *   UO evaluates clarity of English US content. If vague for estimation, UO engages `@clarification-agent` (question to PO in `userLanguage`).
        *   **onError:** If ADO MCP fails, log, notify user (in `userLanguage`), stop or continue with partial data.
    *   **Phase 2: Verification/Finalization of Estimates and Task Breakdown (English).**
        *   UO, for each candidate US (clarified if needed): **Injects targeted English context** to `@task-breakdown-estimator`. Agent verifies/performs English estimation & decomposition, details English "chain of thought". Syncs with ADO.
        *   **onError:** If estimation impossible or MCP fails, US may be excluded from planning with a note, or clarification re-requested.
    *   **Phase 3: Sprint Plan Proposal with Dependency/Risk Analysis (English).**
        *   UO **injects English context** to `@scrum-facilitator-agent`. Agent selects US/tasks, uses **Sequential Thinking MCP** for English dependency/risk analysis. Details English "chain of thought".
    *   **Phase 4: Plan Recording (English Data) and Reporting (Localized Summary).**
        *   Scribe records proposed plan (English data) and English report in `.pheromone`.
        *   UO presents summary of the plan to user (translated to `userLanguage`).

## Phase Details:

### Phase 1: Candidate US Detail Retrieval and Initial Clarity Validation
*   **Responsible Agent:** `@devops-connector`, Scribe, UO, `@clarification-agent`.
*   **Inputs:** List of candidate US IDs, `currentUser`, `userLanguage`.
*   **Actions (`@devops-connector`):** For each US ID, **ADO MCP** `get_work_item_details` (title, desc, ADO priority, ADO status, ADO estimate - all in ADO's original language).
*   **onError (ADO MCP):** UO logs (English) via Scribe. UO notifies user (in `userLanguage`), may stop or continue.
*   **Output (`@devops-connector` -> Scribe, English NL Summary with original language fields):** "Details for [N] candidate US retrieved: [List IDs/ADO Titles_origLang]. Log: `sprint_planning_us_fetch_{{timestamp}}.json`."
*   **Actions (Scribe):**
    1.  For each US: If ADO title/desc not English, UO/Scribe conceptually translates to English.
    2.  Store/Update `memoryBank.userStories.{{usId}}` with `title_en`, `descriptionFull_en`, `title_originalLang`, `description_originalLang`, and other ADO details.
*   **Actions (UO - Clarity Validation):** For each US (using English content from `memoryBank`):
    *   Evaluate clarity of `descriptionFull_en` and `acceptanceCriteria_en`.
    *   **If major ambiguity** for estimation: Pause workflow. Delegate to `@clarification-agent` with US ID, ambiguous English text, English question (to be translated to `userLanguage` for PO). Await response.

### Phase 2: Verification/Finalization of Estimates and Task Breakdown (English)
*   **Responsible Agent:** `@task-breakdown-estimator` (coordinates with `@devops-connector`).
*   **Inputs (Injected by UO for each US, English context):** US details (clarified if needed, English desc/ACs). `memoryBank` context.
*   **Actions (`@task-breakdown-estimator`):**
    1.  Check `memoryBank.userStories.{{usId}}` for reliable English estimation & task breakdown.
    2.  If not/revision needed: Engage English decomposition/estimation (as in `01_Start_User_Story.md` Phase 3).
    3.  **Document English "Chain of Thought"** in its report.
    4.  Sync tasks/estimates with ADO via `@devops-connector` (task details to ADO can be English or translated).
*   **onError (Estimation/Decomposition):** If agent cannot estimate, report (English) to UO. UO may exclude US or re-clarify.
*   **Output (`@task-breakdown-estimator` -> Scribe, English NL Summary):** "Estimates/breakdowns finalized for candidate US. [N] US processed. Individual English reports (with chain of thought): `us_{{usId}}_task_breakdown_{{timestamp}}.md`." Scribe updates `memoryBank` with English data.

### Phase 3: Sprint Plan Proposal with Dependency/Risk Analysis (English)
*   **Responsible Agent:** `@scrum-facilitator-agent`.
*   **Inputs (Injected by UO, English context):** List of candidate US with final English estimates. Team capacity. US priorities (English). `memoryBank.riskRegister` (English).
*   **Actions (`@scrum-facilitator-agent`):**
    1.  Sort US by priority. Iteratively select US up to capacity.
    2.  **Dependency/Risk Analysis (Sequential Thinking MCP, English):** Analyze feasibility, dependencies (from `memoryBank.userStories.{{usId}}.dependencies_en`), related risks.
    3.  **Document "Chain of Thought" (English):** Explain selection logic and conclusions in final English report.
*   **Output (`@scrum-facilitator-agent` -> Scribe, English NL Summary):** "Sprint plan proposal [ID/Name TBD]: [List US IDs]. Total [Points]/[Capacity] {{estimationUnit}}. Risks/Dependencies (English): [Summary]. English Report (with chain of thought): `sprint_plan_proposal_{{timestamp}}.md`." (Report in `02_AI-DOCS/Sprint_Plans/`).

### Phase 4: Plan Recording (English Data) and Reporting (Localized Summary)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`, UO.
*   **Inputs:** English NL Summary from `@scrum-facilitator-agent`. `currentUser.lastInteractionLanguage`.
*   **Actions (Scribe):**
    1.  Update `.pheromone` (English data):
        *   `currentSprint`: plan info (`userStories` (IDs), `plannedPoints_en`, `capacityPoints`).
        *   `documentationRegistry`: Add path to English `sprint_plan_proposal_{{timestamp}}.md`.
        *   `memoryBank.userStories.{{usId}}.sprintAssignment = currentSprint.id`.
        *   `memoryBank.sprints.{{currentSprint.id}}.reasoningChainLinks_en.planning`: Link to English report.
*   **Actions (UO):**
    1.  Generate a summary of the proposed plan in English (from Scribe-updated `.pheromone` or `@scrum-facilitator-agent`'s report).
    2.  Translate this summary to `currentUser.lastInteractionLanguage`.
    3.  Use `ask_followup_question` to present translated summary to PO/SM: "Proposed sprint plan is ready: [Translated Summary]. Total points: X/Y. Detailed English report available. Options (in `currentUser.lastInteractionLanguage`): 1. View detailed report (English)? 2. Define Sprint Name and Goal (in `currentUser.lastInteractionLanguage` or English)? 3. Proceed to assign tasks in Azure DevOps?" (Scribe will handle translation of Sprint Name/Goal to English for `memoryBank` if provided in another language).
*   **Output:** `.pheromone` updated with English plan data. UO interacts with user in their language.

---