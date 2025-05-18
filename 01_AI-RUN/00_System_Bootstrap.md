# Workflow: AgilePheromind System Bootstrap and Configuration (00_System_Bootstrap.md)

**Objective:** Initialize or verify the fundamental configuration of the AgilePheromind system. This includes creating/validating the `.pheromone` file, configuring the project's base context in the Memory Bank, initializing convention documents, verifying the Azure DevOps connection, and confirming the presence of essential system agents such as `@clarification-agent`. This workflow is crucial for ensuring the system's robustness and contextual intelligence from the start.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@project-setup-agent`, `@devops-connector`, `@architecture-advisor-agent`.

**MCPs Used:** Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** The user launches this workflow (e.g., `"AgilePheromind bootstrap system for project 'SuperApp' in ADO org 'MySuperOrg'"`). The UO may request clarification if information is incomplete.
2.  **`🧐 @uber-orchestrator`** takes control.
    *   **Phase 1: Initialization/Verification of `.pheromone` and System Agents.**
        *   UO ensures the Scribe is activated (which bootstraps `.pheromone` if needed).
        *   UO verifies the presence of `@clarification-agent` in `.roomodes` (conceptually, via configuration reading or a UO capability).
    *   **Phase 2: Project Context Configuration in the Memory Bank.**
        *   UO delegates to `@project-setup-agent` to collect/confirm project information.
        *   Scribe records this information in `.pheromone.currentProject` and `memoryBank.projectContext`.
    *   **Phase 3: Initialization of Coding and Design Conventions.**
        *   UO delegates to `@architecture-advisor-agent` to create/verify convention files from templates.
    *   **Phase 4: Azure DevOps Connection Verification.**
        *   UO delegates to `@devops-connector`.
    *   **Phase 5: Bootstrap Report and Readiness Status.**
        *   The UO compiles an action summary. Scribe records the report and system status.

## Phase Details:

### Phase 1: Initialization/Verification of `.pheromone` and System Agents
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe` (for `.pheromone`), `🧐 @uber-orchestrator` (for agent verification).
*   **Inputs:** No specific input for the Scribe. The UO has access to the `.roomodes` configuration (conceptually).
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  The UO activates the Scribe.
    2.  The Scribe loads `.pheromone`. If missing/invalid, it initializes it with the basic structure (including `systemVersion`, empty `clarificationContext`, `memoryBank` with its sections, `documentationRegistry`, `currentUser` set to null, `systemHealth` with `lastPheromoneWriteSuccess: true`).
    3.  If `.pheromone` exists, the Scribe validates its basic structure.
*   **Actions & Tooling (`🧐 @uber-orchestrator`):**
    1.  (Conceptual) Verify that the `@clarification-agent` is defined in `.roomodes`. If missing, issue a critical warning in the final report as it is essential for ambiguity management.
*   **Memory Bank Interaction:**
    *   Writing (Scribe): Initialization of the Memory Bank structure if `.pheromone` is new.
*   **Output (internal to UO):** `.pheromone` is ready. Status of `@clarification-agent` presence.

### Phase 2: Project Context Configuration in the Memory Bank
*   **Responsible Agent:** `@project-setup-agent`
*   **Inputs:** Azure DevOps project name and organization URL (provided by the user via UO). The UO injects the default `techStack` from `memoryBank.projectContext` (if already partially filled) or known default values.
*   **Actions & Tooling:**
    1.  Collect/Confirm project information: ADO project name, ADO org URL, main Git repo URL, technical stack (default .NET/Angular/AzureSQL/Docker/AKS, but can be refined here if the user specifies particular versions or components).
    2.  Prepare a JSON object for `.pheromone.currentProject` and for `.pheromone.memoryBank.projectContext`.
*   **Memory Bank Interaction:**
    *   Data preparation.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Project context for '[ProjectName]' collected. Stack: [.NET, Angular, Azure SQL, Docker, AKS]. Azure DevOps Org: '[OrgURL]', Project: '[ProjectName]', Repo: '[RepoURL]'. Ready to update `.pheromone`."

### Phase 3: Initialization of Coding and Design Conventions
*   **Responsible Agent:** `@architecture-advisor-agent`
*   **Inputs:** Paths to templates (`02_AI-DOCS/Conventions/coding_conventions_template.md`, `02_AI-DOCS/Conventions/design_conventions_template.md`). The UO injects the stack context confirmed in Phase 2.
*   **Actions & Tooling:**
    1.  Check if `coding_conventions.md` and `design_conventions.md` exist in `02_AI-DOCS/Conventions/`.
    2.  If not, copy the templates to these locations.
    3.  Make minimal initial adjustments in the copies to reflect the stack (.NET/Angular), and add placeholders for project-specific design decisions.
*   **Memory Bank Interaction:**
    *   Paths will be added to `documentationRegistry` by the Scribe.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Convention files `coding_conventions.md` and `design_conventions.md` initialized in `02_AI-DOCS/Conventions/`. Ready for customization and versioning (v1.0_initial)."

### Phase 4: Azure DevOps Connection Verification
*   **Responsible Agent:** `@devops-connector`
*   **Inputs:** Project information from `.pheromone.currentProject` (after update by the Scribe following Phase 2).
*   **Actions & Tooling:**
    1.  Use **Azure DevOps MCP**:
        *   `get_project_details {projectName: .pheromone.currentProject.name}`.
        *   `get_user_identity`.
    2.  If either command fails, this indicates an MCP configuration issue or ADO project access problem.
*   **Memory Bank Interaction:**
    *   The Scribe will update `.pheromone.systemHealth.mcpStatus.azureDevOpsMCP`.
*   **Output (to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "Azure DevOps connection for project '[ProjectName]' [successful/failed]. MCP Identity: '[MCPIdentity]'." If failure, include the ADO MCP error message.

### Phase 5: Bootstrap Report and Readiness Status
*   **Responsible Agent:** `🧐 @uber-orchestrator` (for compilation), `✍️ @orchestrator-pheromone-scribe` (for final recording).
*   **Inputs:** Summaries from previous phases.
*   **Actions & Tooling (`🧐 @uber-orchestrator`):**
    1.  Compile a global summary. Explicitly note the ADO connection status and the presence of `@clarification-agent`.
*   **Output (`🧐 @uber-orchestrator` to `✍️ @orchestrator-pheromone-scribe`):** NL Summary: "AgilePheromind bootstrap completed for project '[ProjectName]'. `.pheromone` [initialized/verified]. Project context [OK]. Conventions [initialized/existing]. Azure DevOps connection [OK/Failed]. Agent `@clarification-agent` [Present/Missing - ALERT IF MISSING]. System [Ready for operations / Requires attention on points X, Y]. Report: `system_bootstrap_report_[timestamp].md`."
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  Save the report (`system_bootstrap_report_[timestamp].md`) in `documentationRegistry` (e.g., `02_AI-DOCS/System_Admin/`).
    2.  Persist all `.pheromone` updates from previous phases (project context, convention versions, MCP status).
    3.  Update `.pheromone.systemVersion` if necessary.
*   **Memory Bank Interaction:**
    *   Writing (Scribe): Finalization of project context, report recording.
*   **Outcome:** AgilePheromind is configured, its initial state is recorded, and attention points for complete operability are flagged.

---