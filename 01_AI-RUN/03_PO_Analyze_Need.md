# Workflow: Product Owner Assistance - Customer Need Analysis (03_PO_Analyze_Need.md)

**Objective:** Help the Product Owner (PO) analyze a customer need expressed in natural language. The system must decompose the need in a structured way (using the `Sequential Thinking MCP` and recording the "chain of thought"), propose potential User Stories (US) with initial Acceptance Criteria (ACs), check for similar existing US (via Azure DevOps MCP), manage ambiguities in the need via `@clarification-agent`, and record the complete analysis in the Memory Bank.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@po-assistant`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The PO submits a description of the customer need to AgilePheromind (e.g., `"AgilePheromind analyze need: 'Our users complain that the registration process is too long...'"`).
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Structured Analysis of Customer Need and Identification of Ambiguities.**
        *   UO **injects relevant context** (e.g., existing personas, business glossary from `memoryBank`) to `@po-assistant`.
        *   `@po-assistant` uses **Sequential Thinking MCP** for decomposition. It must **detail its "chain of thought"**.
        *   If major ambiguities are detected in the customer need, `@po-assistant` reports them to the UO. The UO can then engage `@clarification-agent` to request clarification from the PO. The workflow awaits the response.
        *   **onError:** If the initial analysis fails or remains too vague, notify the PO.
    *   **Phase 2: Generation of User Story Proposals and Acceptance Criteria.**
        *   UO delegates to `@po-assistant` (using clarified information if Phase 1b occurred).
    *   **Phase 3: Search for Existing User Stories in the Backlog.**
        *   UO delegates to `@po-assistant`, which queries `@devops-connector`.
        *   **onError:** If ADO MCP fails, inform the PO that duplicate checking could not be done.
    *   **Phase 4: Compilation of Analysis Report and Interaction with the PO.**
        *   `@po-assistant` generates a report (including the analysis "chain of thought").
        *   Scribe records the report and draft US in `.pheromone`.
        *   UO presents a summary to the PO and proposes follow-up actions.

## Phase Details:

### Phase 1: Structured Analysis of Customer Need and Identification of Ambiguities
*   **Responsible Agent:** `@po-assistant` (with UO support for clarification if needed via `@clarification-agent`).
*   **Inputs (Injected by UO):**
    *   Customer need description.
    *   Context from `memoryBank.projectContext` (e.g., `targetAudienceDescription`, `businessGoals`).
    *   List of existing personas (`memoryBank.userPersonas`).
    *   Business glossary (`memoryBank.glossary`).
*   **Actions & Tooling (`@po-assistant`):**
    1.  Use **Sequential Thinking MCP** for detailed analysis:
        *   `set_goal`: "Analyze customer need: '{{customerNeed}}' to identify problems, solutions, and benefits."
        *   `add_step`: "Identify concerned actors/personas (using the provided persona list as reference)."
        *   `add_step`: "Extract specific problems ('pain points')."
        *   `add_step`: "Identify explicit or implicit solutions/desires."
        *   `add_step`: "Deduce expected benefits."
        *   `add_step`: "Identify constraints or assumptions."
        *   `add_step`: "**Evaluate the clarity of the need.** List ambiguous points or missing information that prevent clear decomposition into US."
        *   `run_sequence`.
    2.  **Document the "Chain of Thought":** Preserve the detailed log of this sequential analysis for inclusion in the final report.
    3.  **Report Ambiguities to UO:** If the "Evaluate clarity" step identifies critical ambiguities:
        *   Formulate the ambiguity points.
        *   Suggest specific questions for the PO.
*   **onError / Ambiguity Management (by UO):**
    1.  If `@po-assistant` reports ambiguities:
        *   UO pauses the workflow (`activeWorkflow.status: 'PendingClarification_UserNeed'`).
        *   UO delegates to `@clarification-agent` with the need context and questions suggested by `@po-assistant`.
        *   The response will be processed by `01_AI-RUN/XX_Handle_Clarification_Response.md`, which will update the `memoryBank` and reactivate this workflow.
    2.  If the Sequential Thinking MCP fails, Scribe logs the error, UO notifies the PO.
*   **Output (internal to `@po-assistant` after clarification if necessary):** Structured analysis of the customer need (potentially enriched by PO responses).

### Phase 2: Generation of User Story Proposals and Acceptance Criteria
*   **Responsible Agent:** `@po-assistant`
*   **Inputs:** Structured analysis of the need (clarified if needed in Phase 1).
*   **Actions & Tooling:**
    1.  For each {Problem -> Solution -> Benefit} set:
        *   Formulate US ("As a..., I want..., so that...").
        *   Draft initial ACs (Gherkin or lists).
    2.  **Document the "Chain of Thought":** Briefly explain in the final report why each US was formulated this way in relation to the need analysis.
*   **Memory Bank Interaction (via Scribe in Phase 4):**
    *   Draft US and ACs will be stored.
*   **Output (internal to `@po-assistant`):** List of candidate US with ACs.

### Phase 3: Search for Existing User Stories in the Backlog
*   **Responsible Agent:** `@po-assistant` (coordinating with `@devops-connector`).
*   **Inputs:** Candidate US (Phase 2). `.pheromone.currentProject.name`.
*   **Actions & Tooling:**
    1.  `@po-assistant` identifies keywords for each candidate US.
    2.  `@po-assistant` asks `@devops-connector`: "Search existing US in ADO project `{{currentProject.name}}` for keywords: [list]."
    3.  `@devops-connector` uses **Azure DevOps MCP** (`search_work_items`).
*   **onError Strategy (for UO if `@devops-connector` reports MCP failure):**
    1.  Scribe logs the error.
    2.  UO informs `@po-assistant` that the ADO search failed. The analysis will continue without this information, but the final report will mention it.
*   **Output (`@devops-connector` to `@po-assistant`):** List of potentially similar ADO US IDs/titles.

### Phase 4: Compilation of Analysis Report and Interaction with the PO
*   **Responsible Agent:** `@po-assistant` (report), Scribe (recording), UO (PO interaction).
*   **Inputs:** Results from previous phases.
*   **Actions & Tooling (`@po-assistant`):**
    1.  Compile a Markdown report (`po_need_analysis_[timestamp].md`) in `02_AI-DOCS/PO_Analyses/`. It should include:
        *   Initial customer need.
        *   **Detailed structured analysis (with the "chain of thought" from Phase 1).**
        *   Candidate US with ACs (and the "chain of thought" for their formulation).
        *   ADO search results (or mention of failure if Phase 3 failed).
        *   Recommendations (create, merge, initial priorities).
*   **Output (`@po-assistant` to Scribe):** NL Summary: "Customer need analysis '[summary]' completed. [N_us] US proposed. [N_exist] existing US found. Report (with chain of thought): `po_need_analysis_[timestamp].md`. Recommendations: [keys]."
*   **Actions & Tooling (Scribe):**
    1.  Record report in `documentationRegistry`.
    2.  Store candidate US (with ACs, links to existing US, and a link to the "chain of thought" section of the report) in `memoryBank.draftUserStories` or `memoryBank.userStories.{{usId_draft}}.analysisSummaries[]` (if draft IDs are generated).
*   **Actions & Tooling (UO):**
    1.  Use `ask_followup_question` to present summary and options to the PO: "Need analysis completed. [Summary]. Detailed analysis report available. Options: 1. View report. 2. Create suggested new US in ADO. 3. Discuss a US."
*   **Memory Bank Interaction (via Scribe):**
    *   Archiving of the report, draft US, and link to the "chain of thought".
*   **Outcome:** The PO receives a thorough and traceable analysis, actionable US, and clear options, even if clarifications were necessary.

---