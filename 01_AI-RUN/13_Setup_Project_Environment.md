# Workflow: Interactive Project Environment Setup (.NET/Angular) (13_Setup_Project_Environment.md)

**Objective:** Interactively guide the user to set up their project environment for AgilePheromind. The user can choose to: a) clone an existing Git repository, or b) initialize new .NET backend and/or Angular frontend projects locally. The workflow collects necessary paths and Azure DevOps details, initializes Git if new projects are created, creates English Dockerfiles/Pipeline stubs, and updates `.pheromone`. Error handling and user clarification (in `currentUser.lastInteractionLanguage`) are integrated. All internal Pheromind data (e.g., in `memoryBank`) and generated file names/comments will be in English.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@project-setup-agent`, `@clarification-agent`, `@devops-connector`.

**MCPs Used:** Git Tools MCP, Azure DevOps MCP, `command_line_tool`.

## Pheromind Workflow Overview:

1.  **Initiation:** User (Tech Lead/Architect) requests project setup (e.g., `"AgilePheromind setup project environment"`). This workflow assumes prior successful *user* onboarding part of `00_System_Bootstrap.md`. `userLanguage` passed by `🎩 @head-orchestrator`.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies user part of `.pheromone.onboardingComplete`.
    *   **Phase 1: Determine User's Project Setup Path (Clone or Init New).**
        *   UO (via `@project-setup-agent` or `@clarification-agent`) asks user (in `userLanguage`) their preferred setup.
    *   **Phase 2: Handle Existing Git Repository (If Chosen).**
        *   UO delegates to `@project-setup-agent`. Collect Git remote URL, local clone path. Clone repo (**Git Tools MCP**). Identify backend/frontend roots.
        *   **onError (Git):** Notify (in `userLanguage`), ask for correction.
    *   **Phase 3: Handle New .NET Backend Initialization (If Chosen, English configs).**
        *   UO delegates to `@project-setup-agent`. Collect local path. Initialize .NET projects (CLI).
        *   **onError (dotnet CLI):** Notify (in `userLanguage`), ask to correct/skip.
    *   **Phase 4: Handle New Angular Frontend Initialization (If Chosen, English configs).**
        *   UO delegates to `@project-setup-agent`. Collect local path. Initialize Angular project (CLI).
        *   **onError (ng CLI):** Notify (in `userLanguage`), ask to correct/skip.
    *   **Phase 5: Azure DevOps Project Connection.**
        *   UO (via `@clarification-agent` if needed) collects ADO Org URL & Project Name (in `userLanguage`).
        *   UO delegates to `@devops-connector` to validate ADO project.
        *   **onError (ADO):** Notify (in `userLanguage`), ask for correction.
    *   **Phase 6: Dockerfiles & Azure Pipeline Stubs (English Comments).**
        *   UO delegates to `@project-setup-agent`. Based on identified/created projects.
    *   **Phase 7: Git Finalization (English commit message).**
        *   UO delegates to `@project-setup-agent`.
    *   **Phase 8: `.pheromone` Update & Final Report (English Data for Scribe).**
        *   `@project-setup-agent` provides structured English data to Scribe. Scribe updates `.agilepherominduserinfo` (project paths) and `.pheromone`. UO informs user (in `userLanguage`).

## Phase Details:

### Phase 1: Determine User's Project Setup Path (Clone or Init New)
*   **Responsible Agent:** UO (using `ask_followup_question`).
*   **Inputs:** `currentUser.lastInteractionLanguage`.
*   **Actions (UO):** Ask user (in `userLanguage`): "Project setup options: 1. Clone existing Git repo. 2. Initialize new .NET/Angular projects. 3. Combination. Choose (1, 2, or 3)." Store choice in `activeWorkflow.setupChoice`.
*   **Output (internal to UO):** User's preferred setup method.

### Phase 2: Handle Existing Git Repository (If Chosen)
*   **Responsible Agent:** `@project-setup-agent`, UO, `@clarification-agent`.
*   **Inputs:** User choice. `userLanguage`.
*   **Actions (`@project-setup-agent` / UO):**
    1.  Ask user (in `userLanguage`): "Git remote URL?" and "Full local absolute path to clone/find repository?"
    2.  (`@project-setup-agent`) Use **Git Tools MCP** (`clone_repository` or `git_pull` if exists).
    3.  Attempt to identify backend/frontend roots (heuristic, English names). Ask user (in `userLanguage`) to confirm/provide paths.
*   **onError (Git):** `@project-setup-agent` reports (English). UO (via `@clarification-agent`) asks user (in `userLanguage`) to verify. Loop.
*   **Output (to UO/Scribe, English):** `gitRemoteUrl`, `gitLocalPath`, confirmed `backendRootPath_en`, `frontendRootPath_en`.

### Phase 3: Handle New .NET Backend Initialization (If Chosen, English configs)
*   **Responsible Agent:** `@project-setup-agent`, UO, `@clarification-agent`.
*   **Inputs:** User choice. `userLanguage`. `gitLocalPath`.
*   **Actions (`@project-setup-agent` / UO):**
    1.  If not defined by clone, ask user (in `userLanguage`) for new backend local path. Let it be `newBackendPath_en`.
    2.  Ask for .NET Solution & Project App Name (user provides in their lang, UO/agent make it a valid English var name like `MySolution`, `MyProjectApi`).
    3.  (`@project-setup-agent`) Execute `dotnet new` (English project names, English comments in stubs) within `newBackendPath_en`.
*   **onError (dotnet CLI):** `@project-setup-agent` reports (English). UO notifies user (in `userLanguage`). Loop or skip.
*   **Output (to UO/Scribe, English):** `backendRootPath_en = newBackendPath_en`. Status of .NET init.

### Phase 4: Handle New Angular Frontend Initialization (If Chosen, English configs)
*   **Responsible Agent:** `@project-setup-agent`, UO, `@clarification-agent`.
*   **Inputs:** User choice. `userLanguage`. `gitLocalPath`.
*   **Actions (`@project-setup-agent` / UO):**
    1.  If not defined, ask user (in `userLanguage`) for new frontend local path. Let it be `newFrontendPath_en`.
    2.  Ask for Angular Project App Name (user provides, UO/agent make valid English var name `my-angular-app-en`).
    3.  (`@project-setup-agent`) Execute `ng new` (English project name, English comments) within `newFrontendPath_en`.
*   **onError (ng CLI):** `@project-setup-agent` reports (English). UO handles as Phase 3.
*   **Output (to UO/Scribe, English):** `frontendRootPath_en = newFrontendPath_en`. Status of Angular init.

### Phase 5: Azure DevOps Project Connection
*   **Responsible Agent:** UO (or `@clarification-agent`), `@devops-connector`.
*   **Inputs:** `userLanguage`. (Info might be in `.agilepherominduserinfo` already).
*   **Actions (UO):** If not in `.agilepherominduserinfo` (or if user wants to change): Ask (in `userLanguage`): "ADO Org URL?" & "ADO Project Name?". Delegate (English) to `@devops-connector`.
*   **Actions (`@devops-connector`):** **ADO MCP** `get_project_details`.
*   **onError (ADO):** `@devops-connector` reports (English). UO (via `@clarification-agent`) re-prompts.
*   **Output (to UO/Scribe, English):** `adoOrganizationUrl`, `adoProjectName`, `adoProjectId`. `gitRemoteUrl` (if from ADO).

### Phase 6: Dockerfiles & Azure Pipeline Stubs (English Comments)
*   **Responsible Agent:** `@project-setup-agent`.
*   **Inputs:** `backendRootPath_en`, `frontendRootPath_en`.
*   **Actions:** Create `Dockerfile` (API .NET, English comments). Create `Dockerfile` (App Angular, English comments). Create `docker-compose.yml` stub (English). Create `.azuredevops/azure-pipelines.yml` stub (English).
*   **Output (to UO/Scribe, English):** Paths to created config files.

### Phase 7: Git Finalization (English commit message)
*   **Responsible Agent:** `@project-setup-agent`.
*   **Inputs:** `gitLocalPath`, `gitRemoteUrl`.
*   **Actions (Git Tools MCP):** If new projects initialized: `git_init`, `create_gitignore`, `add_all_changes`, `commit_files {message: "feat: initial project setup by AgilePheromind with user choices"}`. If `gitRemoteUrl` set and no 'origin', or new repo: `add_remote`. If new repo & remote added: `push_commits`.
*   **onError (Git):** Report (English) to UO. UO notifies user (in `userLanguage`).
*   **Output (to UO/Scribe, English):** Status of Git ops.

### Phase 8: `.pheromone` Update & Final Report
*   **Responsible Agent:** `@project-setup-agent` (data prep), Scribe (recording).
*   **Inputs:** All collected/confirmed English paths and project details.
*   **Actions (`@project-setup-agent`):** Assemble `structuredData` for `.agilepherominduserinfo` (project paths) and for `.pheromone` (`currentProject` (English), `memoryBank.projectContext`).
*   **Output (`@project-setup-agent` to Scribe, English):** Special NL summary "AgilePheromind Project Connection Info for Bootstrap. StructuredData attached..." + `structuredData`.
*   **Actions (Scribe):** Update/Create `.agilepherominduserinfo` `projectConnection`. Update `.pheromone`: `currentProject` (English Git info incl. `localPath`), `memoryBank.projectContext`. Add Dockerfiles/pipeline to `documentationRegistry`. Mark project part of `onboardingComplete` flag.
*   **Actions (UO):** Inform user (in `userLanguage`): "Project environment setup/connection complete for '{{currentProject.azureDevOps.projectName}}'. AgilePheromind configured for repository at `{{currentProject.gitRepository.localPath}}`." Resume original workflow if any.
*   **Outcome:** AgilePheromind configured for chosen project setup.

---