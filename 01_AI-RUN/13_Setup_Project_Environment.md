# Workflow: Interactive Project Environment Setup (.NET/Angular) (13_Setup_Project_Environment.md)

**Objective:** Interactively guide the user to set up their project environment for AgilePheromind. The user can choose to: a) clone an existing Git repository containing backend and/or frontend code, or b) initialize new .NET backend and/or Angular frontend projects locally. The workflow collects necessary paths and Azure DevOps details, initializes Git if new projects are created, creates Dockerfiles/Pipeline stubs (English), and updates `.pheromone`. Error handling and user clarification are integrated. User interaction is in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@project-setup-agent`, `@clarification-agent`, `@devops-connector`.

**MCPs Used:** Git Tools MCP, Azure DevOps MCP, `command_line_tool`.

## Pheromind Workflow Overview:

1.  **Initiation:** User (Tech Lead/Architect) requests project setup (ex: `"AgilePheromind setup project environment"`). This workflow assumes prior successful *user* onboarding (`00_System_Bootstrap.md` Phase 1 & 2 for user details), so `currentUser` info is mostly set. This script focuses on *project* setup. `userLanguage` passed by `🎩 @head-orchestrator`.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO uses `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies user part of `.pheromone.onboardingComplete` is true.
    *   **Phase 1: Determine User's Project Setup Path (Clone or Init New).**
        *   UO, via `@project-setup-agent` (or `@clarification-agent`), asks user their preferred setup method.
    *   **Phase 2: Handle Existing Git Repository (If Chosen).**
        *   UO delegates to `@project-setup-agent`. Collect Git remote URL, local clone path. Clone repo using **Git Tools MCP**. Attempt to identify backend/frontend roots within the cloned repo.
        *   **onError (Git Clone/Access):** Notify, ask for correction.
    *   **Phase 3: Handle New Backend .NET Initialization (If Chosen).**
        *   UO delegates to `@project-setup-agent`. Collect local path for backend. Initialize .NET projects via CLI.
        *   **onError (dotnet CLI):** Notify, ask for correction or to skip.
    *   **Phase 4: Handle New Frontend Angular Initialization (If Chosen).**
        *   UO delegates to `@project-setup-agent`. Collect local path for frontend. Initialize Angular project via CLI.
        *   **onError (ng CLI):** Notify, ask for correction or to skip.
    *   **Phase 5: Azure DevOps Project Connection.**
        *   UO (via `@clarification-agent` if needed) collects ADO Org URL & Project Name.
        *   UO delegates to `@devops-connector` to validate ADO project.
        *   **onError (ADO):** Notify, ask for correction.
    *   **Phase 6: Dockerfiles & Azure Pipeline Stubs (Based on Identified/Created Projects).**
        *   UO delegates to `@project-setup-agent`.
    *   **Phase 7: Git Finalization (Init/Add Remote if new, or confirm existing remote).**
        *   UO delegates to `@project-setup-agent`.
    *   **Phase 8: `.pheromone` Update & Final Report.**
        *   `@project-setup-agent` provides structured data to Scribe. Scribe updates `.agilepherominduserinfo` (for project paths) and `.pheromone`.

## Phase Details:

### Phase 1: Determine User's Project Setup Path (Clone or Init New)
*   **Responsible Agent:** UO (using `ask_followup_question`, possibly with `@project-setup-agent` formulating options).
*   **Inputs:** `currentUser.lastInteractionLanguage`.
*   **Actions (UO):**
    1.  Ask user (in `userLanguage`): "How would you like to set up your project for AgilePheromind?
        1. Clone an existing Git repository that already contains your .NET backend and/or Angular frontend.
        2. Initialize new .NET backend and/or Angular frontend projects locally.
        3. A combination (e.g., clone backend, init new frontend).
        Please choose an option (1, 2, or 3)."
    2.  Based on response, UO will direct subsequent phases. Store choice in `activeWorkflow.setupChoice`.
*   **Output (internal to UO):** User's preferred setup method.

### Phase 2: Handle Existing Git Repository (If Chosen in Phase 1)
*   **Responsible Agent:** `@project-setup-agent`, UO, `@clarification-agent`.
*   **Inputs:** User choice from Phase 1. `userLanguage`.
*   **Actions (`@project-setup-agent` / UO via `ask_followup_question`):**
    1.  Ask user (in `userLanguage`):
        *   "Please provide the Git remote URL of your existing repository."
        *   "Please provide the full local absolute path where you want to clone this repository (or where it's already cloned)."
    2.  (`@project-setup-agent`) Use **Git Tools MCP** (`clone_repository {remoteUrl, localPath}`). If repo already exists at `localPath`, confirm it's the correct one and `git_pull` latest.
    3.  (`@project-setup-agent`) Attempt to identify backend/frontend root paths within the cloned repo (e.g., look for `.sln` files, `angular.json`). This might be heuristic.
    4.  Ask user to confirm/provide paths if auto-detection is unsure: "I've cloned/found the repo at `{{localPath}}`. Is the .NET backend root at `{{detected_backend_path_or_ask}}`? Is the Angular frontend root at `{{detected_frontend_path_or_ask}}`?"
*   **onError (Git Clone/Access):** If `clone_repository` fails or path issues: `@project-setup-agent` reports (English) to UO. UO (via `@clarification-agent`) asks user (in `userLanguage`) to verify URL/path or permissions. Loop.
*   **Output (to UO/Scribe):** `gitRemoteUrl`, `gitLocalPath`, confirmed `backendRootPath` (if any), `frontendRootPath` (if any).

### Phase 3: Handle New Backend .NET Initialization (If Chosen in Phase 1 or as part of combination)
*   **Responsible Agent:** `@project-setup-agent`, UO, `@clarification-agent`.
*   **Inputs:** User choice. `userLanguage`. `gitLocalPath` (if part of a larger new project structure).
*   **Actions (`@project-setup-agent` / UO):**
    1.  If not already defined by a cloned repo, ask user (in `userLanguage`): "Please provide the full local absolute path where the new .NET backend should be initialized (e.g., `{{gitLocalPath}}/backend`)." Let this be `newBackendPath`.
    2.  Ask user for .NET Solution & Project App Name (e.g., `MySolution`, `MyProjectName.Api`).
    3.  (`@project-setup-agent`) Execute `dotnet new` commands (as in previous version of this script) within `newBackendPath`.
*   **onError (dotnet CLI):** `@project-setup-agent` reports (English) to UO. UO notifies user (in `userLanguage`). Loop or option to skip.
*   **Output (to UO/Scribe):** `backendRootPath = newBackendPath`. Status of .NET init.

### Phase 4: Handle New Frontend Angular Initialization (If Chosen in Phase 1 or as part of combination)
*   **Responsible Agent:** `@project-setup-agent`, UO, `@clarification-agent`.
*   **Inputs:** User choice. `userLanguage`. `gitLocalPath`.
*   **Actions (`@project-setup-agent` / UO):**
    1.  If not already defined, ask user (in `userLanguage`): "Please provide the full local absolute path for the new Angular frontend (e.g., `{{gitLocalPath}}/frontend`)." Let this be `newFrontendPath`.
    2.  Ask user for Angular Project App Name (e.g., `my-angular-app`).
    3.  (`@project-setup-agent`) Execute `ng new` (as in previous script) within `newFrontendPath`.
*   **onError (ng CLI):** `@project-setup-agent` reports (English) to UO. UO handles as in Phase 3.
*   **Output (to UO/Scribe):** `frontendRootPath = newFrontendPath`. Status of Angular init.

### Phase 5: Azure DevOps Project Connection
*   **Responsible Agent:** UO (or `@clarification-agent`), `@devops-connector`.
*   **Inputs:** `userLanguage`.
*   **Actions (UO):**
    1.  Ask user (in `userLanguage`, if not already in `.agilepherominduserinfo`): "Azure DevOps Organization URL?" and "Azure DevOps Project Name?"
    2.  Delegate to `@devops-connector`: "Validate ADO project and get ID/remote repo URL (if not set from cloned repo): Org='...', Project='...'."
*   **Actions (`@devops-connector`):** **ADO MCP** `get_project_details`.
*   **onError (ADO):** `@devops-connector` reports (English) to UO. UO (via `@clarification-agent`) re-prompts.
*   **Output (to UO/Scribe):** `adoOrganizationUrl`, `adoProjectName`, `adoProjectId`. `gitRemoteUrl` (if not set from cloned repo, ADO might provide a default).

### Phase 6: Dockerfiles & Azure Pipeline Stubs (Based on Identified/Created Projects)
*   **Responsible Agent:** `@project-setup-agent`.
*   **Inputs:** `backendRootPath` (if any), `frontendRootPath` (if any).
*   **Actions:**
    1.  If `backendRootPath` exists, create `{{backendRootPath}}/Dockerfile` (English comments).
    2.  If `frontendRootPath` exists, create `{{frontendRootPath}}/Dockerfile` (English comments).
    3.  If both, create `{{gitLocalPath}}/docker-compose.yml` stub.
    4.  Create `{{gitLocalPath}}/.azuredevops/azure-pipelines.yml` stub (English comments, stages for identified backend/frontend).
*   **Output (to UO/Scribe):** Paths to created Dockerfiles/pipeline stub.

### Phase 7: Git Finalization (Init/Add Remote if new, or confirm existing remote)
*   **Responsible Agent:** `@project-setup-agent`.
*   **Inputs:** `gitLocalPath`, `gitRemoteUrl` (from ADO or user if cloned existing).
*   **Actions (Git Tools MCP):**
    1.  If projects were newly initialized (not cloned): `git_init` at `gitLocalPath`. `create_gitignore`. `add_all_changes`. `commit_files {message: "feat: initial project setup by AgilePheromind"}`.
    2.  If `gitRemoteUrl` is set (either from cloned repo or ADO project) and local repo has no remote 'origin', or if a new repo was init'd: `add_remote {remoteName: "origin", remoteUrl}`.
    3.  If new repo was init'd and remote added: `push_commits {remoteName: "origin", branchName: "main", setUpstream: true}`.
*   **onError (Git):** Report (English) to UO. UO notifies user (in `userLanguage`).
*   **Output (to UO/Scribe):** Status of Git operations.

### Phase 8: `.pheromone` Update & Final Report
*   **Responsible Agent:** `@project-setup-agent` (data prep), Scribe (recording).
*   **Inputs:** All collected/confirmed paths and project details.
*   **Actions (`@project-setup-agent`):**
    1.  Assemble `structuredData` for `.agilepherominduserinfo` (with `gitLocalPath`, `gitRemoteUrl`, ADO details) and for `.pheromone` (`currentProject` section, `memoryBank.projectContext`).
*   **Output (`@project-setup-agent` to Scribe, English):** Special NL summary "AgilePheromind Project Connection Info for Bootstrap. StructuredData attached..." + `structuredData`.
*   **Actions (Scribe, per `.swarmConfig` rule `bootstrap_update_user_and_project_info_v2`):**
    1.  Update/Create `.agilepherominduserinfo` section `projectConnection`.
    2.  Update `.pheromone`: `currentProject` (ADO info, Git info including `localPath`, `remoteUrl`). `memoryBank.projectContext` updated. Add Dockerfiles/pipeline to `documentationRegistry`. Mark project part of `onboardingComplete` flag if distinct from user part.
*   **Actions (UO):** Inform user (in `userLanguage`): "Project environment setup/connection complete for '{{currentProject.azureDevOps.projectName}}'. AgilePheromind is now configured to use the repository at `{{currentProject.gitRepository.localPath}}`." Resume original workflow if any.
*   **Outcome:** AgilePheromind is configured for the user's chosen project setup.

---