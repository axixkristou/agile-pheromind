# Workflow: Product Owner Assistance - Customer Need Analysis (03_PO_Analyze_Need.md)

**Objective:** Assist the Product Owner (PO) in analyzing a customer need expressed in their natural language. The system will structurally decompose the need (using `Sequential Thinking MCP` and recording the English "chain of thought"), propose potential User Stories (US) with initial Acceptance Criteria (ACs) (all in English for internal storage), check for similar existing US (via Azure DevOps MCP), manage ambiguities in the need via `@clarification-agent`, and record the complete English analysis in the Memory Bank. The final analysis report provided to the PO will be in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@po-assistant`, `@devops-connector`, `@clarification-agent`.

**MCPs Used:** Azure DevOps MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** PO submits a customer need description to AgilePheromind (e.g., `"AgilePheromind analyze need: 'Our users complain that the registration process is too long...'"`). `userLanguage` (of the PO's input) passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Structured Analysis of Customer Need (Internal English Processing) and Ambiguity Identification.**
        *   UO **injects relevant English context** (e.g., existing English personas, English business glossary from `memoryBank`) to `@po-assistant`. The PO's input need (in `userLanguage`) is also passed.
        *   `@po-assistant` (whose internal logic is English) understands the need (conceptually translating if `userLanguage` is not English). Uses **Sequential Thinking MCP** for decomposition (English). Must **detail its English "chain of thought"**.
        *   If major ambiguities are detected in the need, `@po-assistant` reports them (English) to UO. UO engages `@clarification-agent` to ask PO for clarification (question translated to `userLanguage`). Workflow pauses.
        *   **onError:** If initial analysis fails or remains too vague, notify PO (in `userLanguage`).
    *   **Phase 2: Generation of User Story Proposals and Acceptance Criteria (All English Internally).**
        *   UO delegates to `@po-assistant` (using clarified information if Phase 1b occurred). All generated US/ACs drafts are in English.
    *   **Phase 3: Search for Existing User Stories in Backlog.**
        *   UO delegates to `@po-assistant`, who queries `@devops-connector` (using English keywords for search against ADO content which may be in various languages).
        *   **onError:** If ADO MCP fails, inform PO (in `userLanguage`) that duplicate check failed.
    *   **Phase 4: Compilation of Analysis Report (English Content, Final Output Localized) and Interaction with PO.**
        *   `@po-assistant` generates an English report (including "chain of thought"). Then translates this report to `userLanguage`.
        *   Scribe records the path to the localized report and stores English draft US in `.pheromone`.
        *   UO presents a summary (translated to `userLanguage`) to PO and proposes follow-up actions.

## Phase Details:

### Phase 1: Structured Analysis of Customer Need (Internal English Processing) and Ambiguity Identification
*   **Responsible Agent:** `@po-assistant` (with UO support for clarification via `@clarification-agent`).
*   **Inputs (Injected by UO):**
    *   Customer need description (in original `userLanguageOfInput`).
    *   `userLanguageOfInput` code (for `@po-assistant` to know if translation needed for its understanding).
    *   English context from `memoryBank.projectContext` (e.g., `targetAudienceDescription_en`, `businessGoals_en`).
    *   List of existing English personas (`memoryBank.userPersonas_en`).
    *   English business glossary (`memoryBank.glossary_en`).
*   **Actions & Tooling (`@po-assistant`):**
    1.  **Understand Need:** If `userLanguageOfInput` is not English, internally translate/understand the input need to English. All subsequent internal processing and "chain of thought" MUST be in English.
    2.  Use **Sequential Thinking MCP** for detailed analysis (English steps):
        *   `set_goal`: "Analyze customer need (English version: '{{translatedNeed_en}}') to identify problems, solutions, benefits."
        *   `add_step`: "Identify concerned English actors/personas (using provided English persona list)."
        *   `add_step`: "Extract specific English problems ('pain points')."
        *   `add_step`: "Identify explicit or implicit English solutions/desires."
        *   `add_step`: "Deduce expected English benefits."
        *   `add_step`: "Identify English constraints or assumptions."
        *   `add_step`: "**Evaluate clarity of the need (English).** List ambiguous points or missing information."
        *   `run_sequence`.
    3.  **Document "Chain of Thought" (English):** Preserve detailed log of this sequential analysis for final report.
    4.  **Report Ambiguities to UO (English):** If "Evaluate clarity" identifies critical ambiguities: Formulate ambiguity points (English). Suggest specific English questions for PO.
*   **onError / Ambiguity Management (by UO):**
    1.  If `@po-assistant` reports ambiguities: Pause workflow. Delegate to `@clarification-agent` with English context and English questions. `@clarification-agent` will translate questions to `currentUser.lastInteractionLanguage` for PO. Await response via `01_AI-RUN/XX_Handle_Clarification_Response.md`.
    2.  If Sequential Thinking MCP fails, Scribe logs English error. UO notifies PO (in `currentUser.lastInteractionLanguage`).
*   **Output (internal to `@po-assistant` after clarification if necessary, English):** Structured English analysis of customer need.

### Phase 2: Generation of User Story Proposals and Acceptance Criteria (All English Internally)
*   **Responsible Agent:** `@po-assistant`.
*   **Inputs:** Structured English analysis of need (clarified if needed in Phase 1).
*   **Actions & Tooling:**
    1.  For each English {Problem -> Solution -> Benefit} set: Formulate English US ("As a..., I want..., so that..."). Draft initial English ACs (Gherkin or lists).
    2.  **Document "Chain of Thought" (English):** Briefly explain in final English report why each English US was formulated.
*   **Output (internal, English):** List of candidate English US with English ACs.

### Phase 3: Search for Existing User Stories in Backlog
*   **Responsible Agent:** `@po-assistant` (coordinating with `@devops-connector`).
*   **Inputs:** Candidate English US (Phase 2). `.pheromone.currentProject.azureDevOps.projectName`.
*   **Actions & Tooling:**
    1.  `@po-assistant` identifies relevant English keywords from candidate US.
    2.  `@po-assistant` requests `@devops-connector`: "Search ADO project `{{currentProject.name}}` for existing US with English keywords: [list]."
    3.  `@devops-connector` uses **Azure DevOps MCP** (`search_work_items` with English query terms. ADO search itself might be language-aware depending on ADO project settings).
*   **onError Strategy (for UO):** If ADO MCP fails, Scribe logs English error. UO informs `@po-assistant` (English). Report will note failed search.
*   **Output (`@devops-connector` to `@po-assistant`):** List of ADO US IDs/titles (titles in original ADO language).

### Phase 4: Compilation of Analysis Report (English Content, Final Output Localized) and Interaction with PO
*   **Agent Responsable:** `@po-assistant` (report), Scribe (recording), UO (PO interaction).
*   **Inputs:** Results of previous phases. `currentUser.lastInteractionLanguage`.
*   **Actions (`@po-assistant`):**
    1.  **Compile English Report Content:** Create full English Markdown report content. Include: Initial customer need (original and English summary), **Detailed structured English analysis (with "chain of thought")**, Candidate English US with English ACs (and "chain of thought"), ADO search results, English Recommendations.
    2.  **Translate Report:** Translate the entire English report content to `currentUser.lastInteractionLanguage`, preserving Markdown. Save as `po_need_analysis_[timestamp]_{{currentUser.lastInteractionLanguage}}.md` in `02_AI-DOCS/PO_Analyses/`.
*   **Output (`@po-assistant` to Scribe, English NL Summary with path to localized report):** "Customer need '{{english_need_summary}}' analysis complete. [N_us] English US proposed. [N_exist] existing US found. Report (in `{{currentUser.lastInteractionLanguage}}`, English reasoning available) at `{{localized_report_path}}`. Key English recommendations: [1-2]."
*   **Actions (Scribe):**
    1.  Record localized report path in `documentationRegistry`.
    2.  Store candidate English US (with English ACs, links to existing US, link to English reasoning chain from the untranslated report version or a specific section) in `memoryBank.draftUserStories_en`.
*   **Actions (UO):**
    1.  Generate a summary of findings and recommendations in English (from `@po-assistant`'s internal English report).
    2.  Translate this summary to `currentUser.lastInteractionLanguage`.
    3.  Use `ask_followup_question` to present translated summary and options to PO: "Need analysis complete. [Translated Summary]. Detailed report (in `{{currentUser.lastInteractionLanguage}}`) available. Options (in `currentUser.lastInteractionLanguage`): 1. View detailed report? 2. Create suggested new US in ADO (will use English titles/desc by default, can be edited in ADO)? 3. Discuss a specific US?"
*   **Output:** PO receives actionable insights and a report in their language, while `memoryBank` and core analyses remain consistently English.

---