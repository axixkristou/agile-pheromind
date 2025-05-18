# Workflow: Technical Documentation Generation (10_Generate_Tech_Docs.md)

**Objective:** Generate or update technical documentation for a specific project module, feature, or API. The agent analyzes source code, comments, specifications (US/tasks), documentation conventions, and relies on rich context injected by the UO. It must handle cases where source information is incomplete or ambiguous.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@documentation-writer-agent`, `@devops-connector` (for US/task context), `@clarification-agent`.

**MCPs Used:** Git Tools MCP, Context7 MCP, Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The user (Dev/Tech Lead) requests documentation for a target (e.g., `"AgilePheromind document module OrderService"`). Can be triggered post-commit.
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Scope Definition and In-Depth Context Injection.**
        *   UO identifies relevant source files (via name or commits linked to US/task with **Git Tools MCP** / **Azure DevOps MCP**).
        *   UO **injects rich context** to `@documentation-writer-agent`: source code, comments, US/task specifications (from `.pheromone.memoryBank`), documentation conventions (`documentationRegistry`), documentation of similar libraries (if existing in `memoryBank`).
        *   UO evaluates if the context is sufficient. If ambiguity (e.g., uncommented code and vague specs), UO can initiate clarification via `@clarification-agent`.
        *   **onError:** If source code inaccessible, notify and stop.
    *   **Phase 2: Source Information Analysis by the Agent.**
        *   UO delegates to `@documentation-writer-agent`. The agent analyzes code, comments, specs. Uses **Context7 MCP** for details on external APIs used.
    *   **Phase 3: Technical Document Structuring and Writing (with "Chain of Thought").**
        *   UO delegates to `@documentation-writer-agent`. The agent plans sections, writes content, includes examples, Mermaid diagrams. **Must document the "chain of thought"** for explanations of complex logic or doc structuring choices.
        *   **onError/Persistent Ambiguity:** If the agent cannot clearly document a section, it reports to the UO who can restart clarification or ask the dev to comment the code.
    *   **Phase 4: Recording and Notification.**
        *   Scribe records the document and updates `.pheromone`.

## Phase Details:

### Phase 1: Scope Definition and In-Depth Context Injection
*   **Responsible Agent:** `🧐 @uber-orchestrator`, `@devops-connector`, `@clarification-agent`.
*   **Inputs:** Documentation target (module, US ID, etc.).
*   **Actions & Tooling (UO):**
    1.  **Identify Source Code:**
        *   If module/class name: Locate files.
        *   If US/task ID: Use `@devops-connector` (**Azure DevOps MCP** `get_work_item_linked_commits`) to find commits, then **Git Tools MCP** (`get_commit_changed_files`) to identify files.
        *   **onError (Git/ADO MCP):** If failure, log, notify, stop.
    2.  **Retrieve Code and Comments (Git Tools MCP `get_file_contents`).**
    3.  **Inject Context from `.pheromone`:**
        *   US/task Description/ACs (`memoryBank.userStories/tasks`).
        *   `memoryBank.projectContext.codingConventionsLink` (for doc standards).
        *   (Optional) Excerpts from `memoryBank.architecturalDecisions` or `design_conventions.md` relevant to the module.
        *   (Optional) Examples of documentation for similar modules already in `documentationRegistry`.
    4.  **Clarity Evaluation and Clarification:**
        *   If the code is minimally commented AND functional specs are vague for the target:
            *   Pause workflow (`activeWorkflow.status: 'PendingClarification_TechDoc'`).
            *   Delegate to `@clarification-agent` with context and a question for the developer/PO (e.g., "The `OrderService` module lacks comments and the ACs of US#XYZ are general. Can you describe the main role of methods A, B and their expected interactions for documentation?").
            *   Wait for response via `01_AI-RUN/XX_Handle_Clarification_Response.md`.
    5.  If clear, delegate to `@documentation-writer-agent` with the code and injected context (including clarifications).
*   **Output:** Task delegated to `@documentation-writer-agent` with rich context, or workflow paused.

### Phase 2: Source Information Analysis by the Agent
*   **Responsible Agent:** `@documentation-writer-agent`.
*   **Inputs (Injected by UO):** Source code, comments, US/task specs, conventions, docs of similar libs, clarifications.
*   **Actions & Tooling:**
    1.  Analyze the code in detail (public signatures, main logic).
    2.  Extract and interpret existing comments.
    3.  Correlate code with functional specs to understand intent.
    4.  If the code uses external APIs or .NET/Angular libraries in a complex way, use **Context7 MCP** (`get_library_docs`) to ensure correct understanding of their usage.
*   **Output (internal):** In-depth understanding of the code to document.

### Phase 3: Technical Document Structuring and Writing (with "Chain of Thought")
*   **Responsible Agent:** `@documentation-writer-agent`.
*   **Inputs:** Analysis (Phase 2). Expected doc type.
*   **Actions & Tooling:**
    1.  **Choose Template / Plan Sections:** (API Reference, Module Guide, etc.).
    2.  **Write Content:** Clear language, code examples, Mermaid diagrams if useful.
    3.  **"Chain of Thought":** For sections explaining complex logic or important design choices in the module, the agent must include a brief explanation of *how* it understood this logic from the code and specs (e.g., "The `ProcessOrder` method appears to handle X, Y, Z based on condition A in the code and AC B. The typical flow is..."). This will be integrated into the generated document.
    4.  **Markdown Formatting, Proofreading.**
    5.  **onError/Persistent Ambiguity:** If part of the code remains obscure even after the clarification phase (or if no clarification was requested but proves necessary):
        *   The agent documents what it can and clearly signals the ambiguous section in its report and in the document itself (e.g., `<!-- AMBIGUITY: Logic for XYZ unclear, needs dev input -->`).
        *   It reports this information to the UO. The UO can then request a new targeted clarification or notify the Tech Lead.
    6.  Name and save the file in `02_AI-DOCS/` (e.g., `Technical/Modules/[ModuleName].md`).
*   **Output (to Scribe):** NL Summary: "Technical doc for `{{TargetName}}` [generated/updated] at `{{FilePath}}`. Contains [description]. [Optional: Section X flagged as ambiguous]. Chain of thought for key logic included."

### Phase 4: Recording and Notification
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** NL summary from `@documentation-writer-agent`.
*   **Actions:**
    1.  Interpret via `.swarmConfig`.
    2.  Update `.pheromone`:
        *   `documentationRegistry`: Add/Update `{{FilePath}}`.
        *   `memoryBank.tasks.{{taskId_if_contextual}}.relatedDocumentation[]`: Add `{{FilePath}}`.
        *   `memoryBank.modules.{{ModuleName}}.documentationPath`: Link doc to module.
        *   `memoryBank.modules.{{ModuleName}}.reasoningChainLinks.documentation`: Can point to the document itself if the chain of thought is integrated.
*   **Output:** `.pheromone` updated. UO informed.

---