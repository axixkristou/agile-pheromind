# Workflow: Sprint Planning Assistance (07_Sprint_Planning_Assistant.md)

**Objective:** Help the Product Owner (PO) and development team plan a sprint. The system retrieves candidate User Stories (US), ensures they are properly decomposed and estimated (by injecting context and requesting clarifications if needed), and proposes a sprint plan based on team capacity, priorities, and identified dependencies. The "chain of thought" for selection is documented.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@task-breakdown-estimator`, `@scrum-facilitator-agent`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Sequential Thinking MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The user (PO/Scrum Master) provides a list of candidate US IDs and team capacity (e.g., `"AgilePheromind plan sprint. US: Azure#123, Azure#456. Capacity: 40 points."`).
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Retrieval and Initial Validation of Candidate US Details.**
        *   UO delegates to `@devops-connector` to get info for each US from Azure DevOps.
        *   UO evaluates the clarity of retrieved US. If a candidate US is too vague for reliable estimation, UO can engage `@clarification-agent` to request clarification from the PO.
        *   **onError:** If ADO MCP fails, log, notify, and potentially stop or continue with the US for which info was retrieved.
    *   **Phase 2: Verification/Finalization of Estimates and Task Breakdown.**
        *   UO, for each candidate US (clarified if needed):
            *   **Injects targeted context** (info from `memoryBank` on past estimates, complexity of similar modules, technical conventions) to `@task-breakdown-estimator`.
            *   `@task-breakdown-estimator` verifies/performs estimation and decomposition. Must **detail its "chain of thought"**.
            *   Synchronizes with Azure DevOps via `@devops-connector`.
        *   **onError:** If estimation impossible or MCP fails, the US can be excluded from planning scope with a note, or clarification requested.
    *   **Phase 3: Sprint Plan Proposal with Dependency/Risk Analysis.**
        *   UO **injects context** (list of estimated US, team capacity, priorities, `memoryBank.riskRegister`) to `@scrum-facilitator-agent`.
        *   `@scrum-facilitator-agent` selects US/tasks. Uses **Sequential Thinking MCP** to analyze dependencies and risks of the plan. **Details its "chain of thought"** for selection.
    *   **Phase 4: Plan Recording and Reporting.**
        *   Scribe records the proposed plan and report (including chain of thought) in `.pheromone`.

## Phase Details:

### Phase 1: Retrieval and Initial Validation of Candidate US Details
*   **Responsible Agent:** `@devops-connector`, UO, `@clarification-agent`.
*   **Inputs:** List of candidate US IDs, `currentUser`.
*   **Actions (`@devops-connector`):** For each US ID, **Azure DevOps MCP** `get_work_item_details` (title, desc, priority, status, ADO estimate).
*   **onError (ADO MCP):** UO logs via Scribe, notifies user, may decide to stop or continue with retrieved US.
*   **Output (`@devops-connector` -> Scribe):** NL Summary: "Details for [N] candidate US retrieved: [List of IDs/Titles]. Log: `sprint_planning_us_fetch_{{timestamp}}.json`." Scribe updates `memoryBank.userStories`.
*   **Actions (UO):** For each retrieved US, evaluate clarity of description and ACs (if present).
    *   **If major ambiguity** preventing estimation:
        *   UO pauses workflow (`activeWorkflow.status: 'PendingClarification_SprintPlanUS'`).
        *   UO delegates to `@clarification-agent` with US ID, ambiguous text, and a question for the PO (e.g., "US Azure#{{usId}} '[Title]' has a vague description regarding [aspect]. To estimate it, can you specify [specific question]?").
        *   Wait for response via `01_AI-RUN/XX_Handle_Clarification_Response.md`.

### Phase 2: Verification/Finalization of Estimates and Task Breakdown
*   **Responsible Agent:** `@task-breakdown-estimator` (coordinates with `@devops-connector`).
*   **Inputs (Injected by UO for each US):**
    *   US details (clarified if needed).
    *   `memoryBank` context: `projectContext` (stack, estimationUnit), past similar US estimates/breakdowns, `technicalDebtItems` or `architecturalDecisions` that may impact effort.
*   **Actions (`@task-breakdown-estimator`):**
    1.  Check if `memoryBank.userStories.{{usId}}` has a reliable estimate and up-to-date task breakdown.
    2.  If not, or if revision requested:
        *   Engage decomposition/estimation process (as in `01_Start_User_Story.md` Phase 3), using **Sequential Thinking MCP**, **Context7 MCP**, **MSSQL MCP**.
        *   **Document the "Chain of Thought"** in its report: explain decomposition logic, estimation assumptions, impact of injected contextual information.
    3.  Synchronize tasks/estimates with ADO via `@devops-connector` (**Azure DevOps MCP**).
*   **onError (Estimation/Decomposition):** If the agent cannot estimate (even after clarification), it reports to the UO. The UO may exclude the US from this planning cycle, noting the reason, or request further clarification. If MCP fails, error handling similar to Phase 1.
*   **Output (`@task-breakdown-estimator` -> Scribe):** NL Summary: "Estimates/breakdowns finalized for candidate US. [N] US processed. Individual reports (with chain of thought): `us_{{usId}}_task_breakdown_{{timestamp}}.md`." Scribe updates `memoryBank.userStories` and `memoryBank.tasks`.

### Phase 3: Sprint Plan Proposal with Dependency/Risk Analysis
*   **Responsible Agent:** `@scrum-facilitator-agent`.
*   **Inputs (Injected by UO):**
    *   List of candidate US with final estimates (from `memoryBank.userStories`).
    *   Team capacity for the sprint.
    *   US priorities.
    *   Content of `memoryBank.riskRegister`.
*   **Actions (`@scrum-facilitator-agent`):**
    1.  Sort US by priority.
    2.  Iteratively select US until capacity is reached.
    3.  **Dependency/Risk Analysis (Sequential Thinking MCP):**
        *   `set_goal`: "Analyze the feasibility and risks of the proposed sprint plan."
        *   `add_step`: "For selected US, identify internal dependencies (between US tasks) and external dependencies (other US, teams - if info available in `memoryBank.userStories.{{usId}}.dependencies`)."
        *   `add_step`: "Check if risks from `memoryBank.riskRegister` are directly related to selected US."
        *   `add_step`: "Evaluate if high-priority US were excluded and why (capacity, dependencies)."
        *   `run_sequence`.
    4.  **Document the "Chain of Thought":** Explain selection logic and conclusions of risk/dependency analysis in the final report.
*   **Output (`@scrum-facilitator-agent` -> Scribe):** NL Summary: "Sprint plan proposal [ID/Name to be defined]: [List of US IDs]. Total [Points]/[Capacity] {{estimationUnit}}. Risks/Dependencies: [Summary]. Report (with chain of thought): `sprint_plan_proposal_{{timestamp}}.md`." (Report in `02_AI-DOCS/Sprint_Plans/`).

### Phase 4: Plan Recording and Reporting
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** NL summary from `@scrum-facilitator-agent`.
*   **Actions:**
    1.  Update `.pheromone`:
        *   `currentSprint`: plan info (US IDs, planned points, capacity). Sprint name/ID and goal can be requested from PO/SM by UO via `ask_followup_question` before this update.
        *   `documentationRegistry`: Add path to `sprint_plan_proposal_{{timestamp}}.md`.
        *   `memoryBank.userStories.{{usId}}.sprintAssignment = currentSprint.id` for included US.
        *   `memoryBank.sprints.{{currentSprint.id}}.reasoningChainLink.planning`: Link to report.
*   **Output:** `.pheromone` updated. UO presents the plan to the team.

---