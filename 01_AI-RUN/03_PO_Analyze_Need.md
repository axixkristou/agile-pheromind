# Workflow: Product Owner Assistance - Customer Need Analysis (03_PO_Analyze_Need.md)

**Objective:** Assist the Product Owner (PO) in analyzing a customer need expressed in their natural language. The system will:
1. Understand the need (conceptually translating to English if input is in another language).
2. Structurally decompose the need (using `Sequential Thinking MCP` and recording the English "chain of thought").
3. Propose potential User Stories (US) with initial Acceptance Criteria (ACs) (all in English for internal storage).
4. Check for similar existing US (via Azure DevOps MCP, using English search terms).
5. Manage ambiguities in the need via `@clarification-agent` (questions to PO in PO's language).
6. Record the complete English analysis in the Memory Bank.
7. Provide the final analysis report to the PO in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@po-assistant`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** PO submits customer need description (e.g., `"AgilePheromind analyze need: 'Our users complain that the registration process is too long...'"`). `userLanguage` (of PO's input) passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Structured Analysis of Customer Need (Internal English Processing) & Ambiguity Identification.**
        *   UO **injects relevant English context** to `@po-assistant`. PO's input need (in `userLanguage`) is passed.
        *   `@po-assistant` (internal logic in English) understands the need. Uses **Sequential Thinking MCP** for English decomposition. Must **detail its English "chain of thought"**.
        *   If major ambiguities, `@po-assistant` reports (English) to UO. UO engages `@clarification-agent` (question to PO in `userLanguage`). Workflow pauses.
        *   **onError:** If analysis fails/vague, notify PO (in `userLanguage`).
    *   **Phase 2: Generation of User Story Proposals and Acceptance Criteria (All English Internally).**
        *   UO delegates to `@po-assistant` (using clarified info). All draft US/ACs are English.
    *   **Phase 3: Search for Existing User Stories in Backlog.**
        *   UO delegates to `@po-assistant`, who queries `@devops-connector` (English keywords).
        *   **onError:** If ADO MCP fails, inform PO (in `userLanguage`) that duplicate check failed.
    *   **Phase 4: Compilation of Analysis Report (English Content, Final Output Localized) and Interaction with PO.**
        *   `@po-assistant` generates English report content. Translates to `userLanguage`.
        *   Scribe records localized report path and English draft US in `.pheromone`.
        *   UO presents summary (translated to `userLanguage`) to PO.

## Phase Details:

### Phase 1: Structured Analysis of Customer Need (Internal English Processing) & Ambiguity Identification
*   **Responsible Agent:** `@po-assistant` (with UO support for clarification via `@clarification-agent`).
*   **Inputs (Injected by UO):** Customer need description (in `userLanguageOfInput`). `userLanguageOfInput` code. English context from `memoryBank`.
*   **Actions & Tooling (`@po-assistant`):**
    1.  **Understand Need:** If `userLanguageOfInput` not English, internally translate/understand to English. All internal processing & "chain of thought" MUST be English.
    2.  Use **Sequential Thinking MCP** for detailed analysis (English steps): Goal: "Analyze customer need ('{{translatedNeed_en}}') for problems, solutions, benefits." Steps: Actors/personas (ref English persona list), English problems, English solutions/desires, English benefits, English constraints, **Evaluate clarity (English).** Run.
    3.  **Document "Chain of Thought" (English):** Preserve detailed log for final report.
    4.  **Report Ambiguities to UO (English):** If "Evaluate clarity" finds critical ambiguities: Formulate English ambiguity points. Suggest specific English questions for PO.
*   **onError / Ambiguity Management (by UO):** If ambiguities: Pause workflow. Delegate to `@clarification-agent` (English context/questions). `@clarification-agent` translates questions to `currentUser.lastInteractionLanguage` for PO. Await response via `01_AI-RUN/XX_Handle_Clarification_Response.md`. If SeqThink MCP fails, Scribe logs English error, UO notifies PO (in `userLanguage`).
*   **Output (internal to `@po-assistant` after clarification, English):** Structured English analysis.

### Phase 2: Generation of User Story Proposals and Acceptance Criteria (All English Internally)
*   **Responsible Agent:** `@po-assistant`.
*   **Inputs:** Structured English analysis of need (clarified).
*   **Actions & Tooling:** For each English {Problem -> Solution -> Benefit} set: Formulate English US ("As a..., I want..., so that..."). Draft initial English ACs (Gherkin/lists). **Document "Chain of Thought" (English):** Briefly explain in final English report why each English US was formulated.
*   **Output (internal, English):** List of candidate English US with English ACs.

### Phase 3: Search for Existing User Stories in Backlog
*   **Responsible Agent:** `@po-assistant` (coordinating with `@devops-connector`).
*   **Inputs:** Candidate English US (Phase 2). `.pheromone.currentProject.azureDevOps.projectName`.
*   **Actions & Tooling:** `@po-assistant` IDs English keywords. Asks `@devops-connector`: "Search ADO project `{{currentProject.name}}` for US with English keywords: [list]." `@devops-connector` uses **ADO MCP** (`search_work_items`).
*   **onError Strategy (for UO):** If ADO MCP fails, Scribe logs English error. UO informs `@po-assistant` (English). Report notes failed search.
*   **Output (`@devops-connector` to `@po-assistant`):** List of ADO US IDs/titles (titles in original ADO language).

### Phase 4: Compilation of Analysis Report (English Content, Final Output Localized) and Interaction with PO
*   **Agent Responsable:** `@po-assistant` (report), Scribe (recording), UO (PO interaction).
*   **Inputs:** Results of previous phases. `currentUser.lastInteractionLanguage`.
*   **Actions (`@po-assistant`):**
    1.  **Compile English Report Content:** Markdown. Include: Initial need (original & English summary), **Detailed structured English analysis (with "chain of thought")**, Candidate English US with English ACs (and "chain of thought"), ADO search results, English Recommendations.
    2.  **Translate Report:** Translate entire English report to `currentUser.lastInteractionLanguage`. Save as `po_need_analysis_[timestamp]_{{currentUser.lastInteractionLanguage}}.md` in `02_AI-DOCS/PO_Analyses/`.
*   **Output (`@po-assistant` to Scribe, English NL Summary with path to localized report):** "Customer need '{{english_need_summary}}' analysis complete. [N_us] English US proposed. [N_exist] existing US found. Report (in `{{currentUser.lastInteractionLanguage}}`, English reasoning available) at `{{localized_report_path}}`. Key English recommendations: [1-2]."
*   **Actions (Scribe):** Record localized report path in `documentationRegistry`. Store candidate English US (with English ACs, links to existing US, link to English reasoning chain) in `memoryBank.draftUserStories_en`.
*   **Actions (UO):**
    1.  Generate English summary of findings/recommendations (from `@po-assistant`'s internal English report).
    2.  Translate summary to `currentUser.lastInteractionLanguage`.
    3.  `ask_followup_question` to PO (in `userLanguage`): "Need analysis complete. [Translated Summary]. Detailed report (in `userLanguage`) available. Options: 1. View report? 2. Create suggested new US in ADO (will use English titles/desc by default, editable in ADO)? 3. Discuss US?"
*   **Output:** PO receives insights/report in their language. `memoryBank` remains English.

---