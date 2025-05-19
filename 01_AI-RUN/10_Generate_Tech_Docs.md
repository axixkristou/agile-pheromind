# Workflow: Technical Documentation Generation (10_Generate_Tech_Docs.md)

**Objective:** Generate or update technical documentation (final output in `currentUser.lastInteractionLanguage`) for a specific project module, feature, or API. The agent analyzes source code (with English comments ideally), existing English comments, English specifications (US/tasks), and project documentation conventions. It relies on rich English context injected by the UO, documents its "chain of thought" in English for complex sections, and handles cases where source information is incomplete or ambiguous.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@documentation-writer-agent`, `@devops-connector` (for US/task context), `@clarification-agent`.

**MCPs Used:** Git Tools MCP, Context7 MCP, Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** User (Dev/Tech Lead) requests documentation for a target (e.g., `"AgilePheromind document module OrderService"`). Can be post-commit. `userLanguage` passed by `🎩 @head-orchestrator` to UO.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Scope Definition and In-Depth English Context Injection.**
        *   UO identifies relevant source files.
        *   UO **injects rich English context** to `@documentation-writer-agent`.
        *   UO evaluates if context is sufficient. If ambiguity, UO may initiate clarification via `@clarification-agent` (question translated to `userLanguage`).
        *   **onError:** If source code inaccessible, notify user (in `userLanguage`), stop.
    *   **Phase 2: Source Information Analysis by Agent (English).**
        *   UO delegates to `@documentation-writer-agent`. Agent analyzes code, English comments, English specs. Uses **Context7 MCP**.
    *   **Phase 3: Technical Document Structuring and Writing (English Content, "Chain of Thought", Final Output Localized).**
        *   UO delegates to `@documentation-writer-agent`. Agent plans sections, writes English content, includes examples, Mermaid diagrams. **Must document English "chain of thought"**. Then translates entire document to `currentUser.lastInteractionLanguage`.
        *   **onError/Persistent Ambiguity:** If agent cannot clearly document a section, it reports (English) to UO. UO may restart clarification or ask dev (in `userLanguage`) to comment code.
    *   **Phase 4: Recording and Notification (English Data for Scribe, Localized Report Path).**
        *   Scribe records localized document path and updates `.pheromone`.

## Phase Details:

### Phase 1: Scope Definition and In-Depth English Context Injection
*   **Responsible Agent:** `🧐 @uber-orchestrator`, `@devops-connector`, `@clarification-agent`.
*   **Inputs:** Documentation target. `userLanguage`.
*   **Actions & Tooling (UO):**
    1.  **Identify Source Code:** If module/class name: Locate files. If US/task ID: Use `@devops-connector` (**ADO MCP** `get_work_item_linked_commits`) for commits, then **Git Tools MCP** (`get_commit_changed_files`) for files.
    2.  **onError (Git/ADO MCP):** If failure, Scribe logs English error. UO notifies user (in `userLanguage`), stop.
    3.  **Retrieve Code & Comments (Git Tools MCP `get_file_contents`).**
    4.  **Inject English Context to `@documentation-writer-agent`:** English US/task desc/ACs (`memoryBank`), English `coding_conventions.md`, relevant English `architecturalDecisions_en`, examples of similar English docs from `documentationRegistry`.
    5.  **Clarity Evaluation & Clarification:** If code minimally commented AND English functional specs vague: Pause workflow. Delegate to `@clarification-agent` with English context and English question (to be translated for user). Await response.
    6.  If clear, delegate to `@documentation-writer-agent` with code, injected English context, and `currentUser.lastInteractionLanguage` as `userLanguageForOutput`.
*   **Output:** Task delegated or workflow paused.

### Phase 2: Source Information Analysis by Agent (English)
*   **Responsible Agent:** `@documentation-writer-agent`.
*   **Inputs (Injected by UO):** Source code, English comments, English US/task specs, English conventions, docs of similar libs, English clarifications.
*   **Actions & Tooling:**
    1.  Analyze code (public signatures, main logic).
    2.  Extract/interpret existing English comments.
    3.  Correlate code with English functional specs.
    4.  If code uses external APIs/libraries complexly, use **Context7 MCP** (`get_library_docs`).
*   **Output (internal, English):** In-depth understanding of code to be documented.

### Phase 3: Technical Document Structuring and Writing (English Content, "Chain of Thought", Final Output Localized)
*   **Responsible Agent:** `@documentation-writer-agent`.
*   **Inputs:** English analysis (Phase 2). Expected English doc type. `userLanguageForOutput`.
*   **Actions & Tooling:**
    1.  **Choose Template / Plan Sections (English titles).**
    2.  **Write Content (English):** Clear, precise English. Code examples (English comments). Mermaid diagrams (English text).
    3.  **"Chain of Thought" (English):** For complex logic/design choices, agent includes brief English explanation of *how* it understood this. Integrated into English draft.
    4.  **Translate Content:** Translate entire drafted English content to `userLanguageForOutput`, preserving Markdown.
    5.  **onError/Persistent Ambiguity:** If code part obscure: Document what's possible in English draft. Mark ambiguous section (e.g., `<!-- AMBIGUITY (English): Logic for XYZ unclear, needs dev input -->`). Translate note. Report (English) to UO. UO may re-clarify or notify Tech Lead (in `userLanguage`).
    6.  Name and save localized file in `02_AI-DOCS/` (e.g., `Technical/Modules/[ModuleName_en]_{{userLanguageForOutput}}.md`).
*   **Output (to Scribe, English NL Summary with path to localized doc):** "Technical doc for `{{TargetName_en}}` [generated/updated] at `{{LocalizedFilePath}}` (lang: `{{userLanguageForOutput}}`). Contains [English description summary]. [Optional: Section X ambiguous]. English chain of thought in original draft and translated."

### Phase 4: Recording and Notification (English Data for Scribe, Localized Report Path)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** English NL Summary from `@documentation-writer-agent`.
*   **Actions:**
    1.  Interpret via `.swarmConfig`.
    2.  Update `.pheromone` (English data for `memoryBank`):
        *   `documentationRegistry`: Add/Update `{{LocalizedFilePath}}` (key can be localized path or English alias). Add language metadata.
        *   `memoryBank.tasks.{{taskId_if_contextual}}.relatedDocumentation_localized[]`: Add `{{LocalizedFilePath}}`.
        *   `memoryBank.modules.{{ModuleName_en}}.documentationPath_localized`: Link localized doc.
        *   `memoryBank.modules.{{ModuleName_en}}.reasoningChainLinks_en.documentation`: Link to English draft/summary of reasoning.
*   **Output:** `.pheromone` updated. UO informed (can notify user in `userLanguage`).

---