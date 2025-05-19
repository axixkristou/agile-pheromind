# Workflow: Maintain Project Coding and Design Conventions (17_Maintain_Project_Conventions.md)

**Objective:** Ensure the project's coding (`coding_conventions.md`) and design (`design_conventions.md`) documents (stored in English) are up-to-date, reflect current best practices, team decisions, and .NET/Angular stack standards. This workflow can be triggered by retrospectives, major tech changes, or Tech Lead/Architect request. The agent proposes English changes with "chain of thought" justification, and after human validation (interaction in `currentUser.lastInteractionLanguage`), applies and commits them.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@architecture-advisor-agent`, `@code-reviewer-assistant` (as source of feedback on deviations), `@clarification-agent`.

**MCPs Used:** Context7 MCP, Git Tools MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   User (Tech Lead/Architect) requests convention update (e.g., `"AgilePheromind update .NET coding conventions"`). `userLanguage` passed by `🎩 @head-orchestrator`.
    *   Triggered by `@workflow-optimizer-agent` if recurrent convention-related issues detected.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Information Collection and Update Needs Identification (English Context).**
        *   UO **injects English context** to `@architecture-advisor-agent`: current English convention files (via `documentationRegistry` & **Git Tools MCP**), `memoryBank` insights (retrospectives, tech debt related to conventions, architectural decisions).
        *   `@architecture-advisor-agent` researches current best practices (via **Context7 MCP**).
        *   **onError:** If convention files not found or Context7 MCP fails, report (English).
    *   **Phase 2: Structured Proposal of Convention Changes (English, with "Chain of Thought").**
        *   UO delegates to `@architecture-advisor-agent`. Agent uses **Sequential Thinking MCP** (English) to structure proposals. **Must document English "chain of thought"** justifying each major change.
        *   If info is missing to justify a change, agent reports (English) to UO for potential clarification with team via `@clarification-agent` (question in `userLanguage`).
    *   **Phase 3: Human Validation of Proposed Changes.**
        *   UO presents English proposals (or a translated summary if extensive) to Tech Lead/Architect (in `userLanguage`) via `ask_followup_question`.
    *   **Phase 4: Application of Changes and Versioning (English Documents).**
        *   If validated, UO delegates to `@architecture-advisor-agent` to update English `.md` files and commit (English message) via **Git Tools MCP**.
        *   **onError (Git Commit):** If fails, log (English), notify (in `userLanguage`).
    *   **Phase 5: `.pheromone` Update and Communication (English Data for Scribe).**
        *   Scribe updates English convention versions in `memoryBank.projectContext` and `documentationRegistry`. UO can notify team (in `userLanguage`).

## Phase Details:

### Phase 1: Information Collection and Update Needs Identification (English Context)
*   **Responsible Agent:** `@architecture-advisor-agent`.
*   **Inputs (Injected by UO, English context):** Type of convention to review. Content of current English `coding_conventions.md` / `design_conventions.md`. Relevant English extracts from `memoryBank`.
*   **Actions & Tooling (`@architecture-advisor-agent`):**
    1.  Analyze current English conventions and `memoryBank` inputs.
    2.  Research via **Context7 MCP** (`get_library_docs`): official .NET/C#, Angular/TypeScript style guides, Tailwind/UI lib best practices, design system trends.
    3.  Identify outdated, missing, or unclear sections in English conventions.
*   **onError (Git Tools / Context7 MCP):** If files not found or Context7 fails: `@architecture-advisor-agent` signals (English) to UO. UO may continue with limited analysis or stop.
*   **Output (internal, English):** List of convention points needing update, with justification and references.

### Phase 2: Structured Proposal of Convention Changes (English, with "Chain of Thought")
*   **Responsible Agent:** `@architecture-advisor-agent`, UO, `@clarification-agent`.
*   **Inputs:** English analysis (Phase 1). Current English convention files.
*   **Actions (`@architecture-advisor-agent`):**
    1.  Use **Sequential Thinking MCP** (English) for each major section to modify:
        *   Goal: "Propose update for section '[EnglishSection]' of [Coding/Design] conventions."
        *   Steps: Current convention (English), Identified problem, Best practice ref, Proposed new/modified convention (English), Justification / English "Chain of Thought".
        *   Run.
    2.  Compile all English proposals (with "chain of thought") into a "proposed changes" document.
    3.  **Ambiguity/Information Gaps:** If impact of a new rule unclear or team consensus needed: Formulate English point/question. Signal (English) to UO for potential clarification via `@clarification-agent`.
*   **Output (to UO, English):** Document of proposed English changes (with justifications/"chains of thought").

### Phase 3: Human Validation of Proposed Changes
*   **Responsible Agent:** `🧐 @uber-orchestrator`.
*   **Inputs:** English proposals from `@architecture-advisor-agent`. `currentUser.lastInteractionLanguage`.
*   **Actions (UO):**
    1.  Prepare a summary of major proposed changes (translate to `userLanguage` if needed for the prompt).
    2.  `ask_followup_question` to Tech Lead/Architect (in `userLanguage`): "Proposed updates for [Coding/Design] conventions (English, with justifications). Summary: [Translated Summary]. Full English proposal doc: `[link_to_english_proposal_doc]`. Approve? (approve / reject / request_changes)".
    3.  Handle response (iterate with `@architecture-advisor-agent` if "request_changes", stop if "reject").
*   **Output (to `@architecture-advisor-agent` if approved):** Approval confirmation.

### Phase 4: Application of Changes and Versioning (English Documents)
*   **Responsible Agent:** `@architecture-advisor-agent`.
*   **Inputs:** Approval from UO. Validated English proposals.
*   **Actions:**
    1.  Apply approved changes to English `coding_conventions.md` and/or `design_conventions.md` in `02_AI-DOCS/Conventions/`.
    2.  Update version and `last_updated_date` in the English documents.
    3.  **Git Tools MCP**: `add_files`, `commit_files {message: "docs(conventions): update [coding/design] conventions v[NewEnglishVersion]\n\n[English summary of key changes]"}`, (optional) `push_commits`.
*   **onError (Git Commit):** If fails, `@architecture-advisor-agent` signals (English) to UO. UO logs (English) via Scribe, notifies user (in `userLanguage`).
*   **Output (to Scribe, English NL Summary):** "Project conventions updated. `coding_conventions.md` (vX.Y) and/or `design_conventions.md` (vZ.A) (English) modified and committed. Commit: `{{commitHash}}`. Main changes (English): [List]."

### Phase 5: `.pheromone` Update and Communication (English Data for Scribe)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** English NL Summary from `@architecture-advisor-agent`.
*   **Actions:**
    1.  Update `.pheromone` (English data):
        *   `memoryBank.projectContext.codingConventionsVersion_en` / `designConventionsVersion_en`.
        *   `documentationRegistry`: Ensure entries for English convention files are correct, update `lastModified`.
        *   `memoryBank.architecturalDecisions_en` or `conventionUpdateHistory_en`: Record update (commit, English summary, link to diff/proposal doc containing English "chain of thought").
    2.  (Optional) UO notifies team (in `userLanguage` or project default) about updated conventions.
*   **Output:** `.pheromone` updated with English convention versions. Team informed.

---