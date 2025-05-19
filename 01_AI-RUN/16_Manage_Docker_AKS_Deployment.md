# Workflow: Manage Docker Images (Build and Push to ACR) (16_Manage_Docker_Images.md)

**Objective:** Manage the lifecycle of Docker images for the project's applications (.NET API and Angular Frontend). This includes building Docker images from their respective Dockerfiles and pushing them to Azure Container Registry (ACR). Error handling for Docker commands is integrated. User interaction is in `currentUser.lastInteractionLanguage`. AKS deployment and Azure Pipelines for AKS are out of scope for this specific workflow.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@deployment-agent-aks` (name kept for now, but role is focused on images), `@clarification-agent`.

**MCPs Used:**
*   Docker MCP (conceptuel, ou `command_line_tool` pour `docker build`, `docker push`).
*   Git Tools MCP (pour récupérer Dockerfiles et le contexte de build).
*   Azure CLI MCP (conceptuel, ou `command_line_tool` pour `az acr login` et vérifier tags).

## Pheromind Workflow Overview:

1.  **Initiation:** User (DevOps/Tech Lead) requests a Docker image build and push (e.g., `"AgilePheromind build and push SuperAppAPI version 1.2.0"` or `"AgilePheromind build SuperAppFrontend from branch feature/new-ui tag as latest"`). `userLanguage` passed by `🎩 @head-orchestrator`.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Parameter Validation, Dockerfile Retrieval, and Context Injection.**
        *   UO validates inputs (AppName, Version/Tag).
        *   UO **injects English context** (`memoryBank.projectContext` for ACR URL, Git repo path) to `@deployment-agent-aks`.
        *   `@deployment-agent-aks` retrieves the appropriate Dockerfile (via **Git Tools MCP**).
        *   **onError (Dockerfile Not Found):** Notify user (in `userLanguage`), ask for clarification on path/branch via `@clarification-agent`, or stop.
    *   **Phase 2: Docker Image Build (with error handling).**
        *   UO delegates to `@deployment-agent-aks`. Utilizes **Docker MCP / CLI**.
        *   **onError (Build):** If build fails, log detailed English error, notify user (in `userLanguage`), stop.
    *   **Phase 3: Docker Image Push to ACR (with error handling).**
        *   UO delegates to `@deployment-agent-aks`. Requires ACR login (via **Azure CLI MCP** or CLI). Utilizes **Docker MCP / CLI**.
        *   **onError (ACR Login/Push):** If fails, log detailed English error, notify user (in `userLanguage`), stop.
    *   **Phase 4: Report (English) and `.pheromone` Update (English Data).**
        *   `@deployment-agent-aks` generates an English report of build/push actions.
        *   Scribe records the image details and status in `memoryBank.deployments` (renamed to `memoryBank.dockerImages` or similar).

## Phase Details:

### Phase 1: Parameter Validation, Dockerfile Retrieval, and Context Injection
*   **Responsible Agent:** UO, `@deployment-agent-aks`, `@clarification-agent`.
*   **Inputs (from UO):** AppName (e.g., `SuperAppAPI`, `SuperAppFrontend`), Version/Tag. `userLanguage`.
*   **Actions (UO):**
    1.  Validate AppName and Version/Tag format. If unclear, use `@clarification-agent` to ask user (in `userLanguage`).
    2.  Inject English context to `@deployment-agent-aks`:
        *   `memoryBank.projectContext.azureContainerRegistryUrl`.
        *   `currentProject.gitRepository.localPath` (or remote URL and branch/commit to checkout for build context).
        *   Expected Dockerfile location convention (e.g., `backend/[AppName].Api/Dockerfile` or `frontend/[AppName]App/Dockerfile`).
*   **Actions (`@deployment-agent-aks`):**
    1.  Determine Dockerfile path based on AppName and conventions.
    2.  Use **Git Tools MCP** (`get_file_contents`) to retrieve the Dockerfile from the specified branch/commit (or local path).
*   **onError (Dockerfile Not Found):**
    *   `@deployment-agent-aks` reports (English) to UO: "Dockerfile for [AppName] not found at expected path/branch. Path checked: [path]".
    *   UO uses `@clarification-agent` to ask user (in `userLanguage`): "Dockerfile for [AppName] not found. Please provide correct path or branch, or confirm it exists." Workflow pauses.
*   **Output (to UO if success):** "Prerequisites for Docker image [AppName]:[Version]: Dockerfile retrieved. ACR Target: `{{ACR_URL}}`."

### Phase 2: Docker Image Build (with error handling)
*   **Responsible Agent:** `@deployment-agent-aks`.
*   **Inputs:** Dockerfile content, AppName, Version/Tag, BuildContextPath (e.g., `{{currentProject.gitRepository.localPath}}/backend/{{AppName}}.Api` or a checked-out temporary path), ACR URL.
*   **Actions & Tooling:**
    1.  Use **Docker MCP** (or `command_line_tool` with `docker build`):
        *   `docker build -t {{ACR_URL}}/{{AppName}}:{{Version}} -f {{PathToDockerfileInContext}} {{BuildContextPath}}`
        *   The `PathToDockerfileInContext` would be the name of the Dockerfile if the build context is its parent directory.
*   **onError (Build by `@deployment-agent-aks`):**
    *   If `docker build` fails, capture the detailed English Docker error output.
    *   Signal (English) to UO: "Failed to build Docker image `{{ACR_URL}}/{{AppName}}:{{Version}}`. Docker Error: [FullDockerErrorMsg]."
    *   UO logs via Scribe (`activeWorkflow.lastError`), notifies user (in `userLanguage` with key error points translated), and stops this workflow.
*   **Output (to UO if success, English):** "Docker image `{{ACR_URL}}/{{AppName}}:{{Version}}` built successfully locally."

### Phase 3: Docker Image Push to ACR (with error handling)
*   **Responsible Agent:** `@deployment-agent-aks`.
*   **Inputs:** Full Image Name with Tag (e.g., `myacr.azurecr.io/SuperAppAPI:v1.2.0`). ACR Name from `memoryBank.projectContext.azureContainerRegistryUrl`.
*   **Actions & Tooling:**
    1.  **Login to ACR:**
        *   Use **Azure CLI MCP** (conceptual, if it handles `az acr login` non-interactively) or `command_line_tool` with `az acr login --name {{ACR_Name}}`. This requires prior Azure authentication in the Pheromind execution environment.
        *   **onError (ACR Login):** If `az acr login` fails, capture English error. Signal (English) to UO: "Failed to login to ACR '{{ACR_Name}}'. Azure CLI Error: [MsgError]. Ensure Pheromind environment is logged into Azure and has ACR permissions." UO notifies user (in `userLanguage`), stops.
    2.  **Push Image to ACR:**
        *   Use **Docker MCP** (or `command_line_tool` with `docker push`):
            *   `docker push {{ACR_URL}}/{{AppName}}:{{Version}}`
        *   **onError (Docker Push):** If `docker push` fails, capture English error. Signal (English) to UO: "Failed to push Docker image `{{ACR_URL}}/{{AppName}}:{{Version}}` to ACR. Docker Error: [MsgErrorDocker]. Check ACR permissions or image tag." UO notifies user (in `userLanguage`), stops.
*   **Output (to UO if success, English):** "Docker image `{{ACR_URL}}/{{AppName}}:{{Version}}` pushed successfully to ACR."

### Phase 4: Report (English) and `.pheromone` Update (English Data)
*   **Responsible Agent:** `@deployment-agent-aks` (for report content), Scribe (for recording).
*   **Inputs:** Status of build and push operations. AppName, Version, ACR URL.
*   **Actions (`@deployment-agent-aks`):**
    1.  Generate English Markdown report (`docker_image_manage_[AppName]_[Version]_[timestamp].md`) in `03_SPECS/Docker_Images/`.
    2.  Content: AppName, Version/Tag, ACR Target, Status of Build (Success/Fail + Logs if fail), Status of ACR Login (Success/Fail), Status of Push (Success/Fail + Logs if fail).
*   **Output (`@deployment-agent-aks` to Scribe, English NL Summary):** "Docker image management for `{{AppName}}:{{Version}}` [completed successfully/failed at build/failed at push]. Report: `docker_image_manage_[AppName]_[Version]_[timestamp].md`."
*   **Actions (Scribe):**
    1.  Interpret English summary.
    2.  Update `.pheromone` (English data):
        *   `documentationRegistry`: Add English report path.
        *   `memoryBank.dockerImages.{{AppName}}.{{Version}}`: (New section or adapt `deployments`)
            *   `status_en`: (e.g., "Built_And_Pushed_To_ACR", "Build_Failed", "Push_Failed").
            *   `acrPath_en`: `{{ACR_URL}}/{{AppName}}:{{Version}}`.
            *   `timestamp`: `{{timestamp}}`.
            *   `lastActionBy_en`: `{{currentUser.pheromindId}}`.
            *   `reportLink_en`: Path to the English MD report.
        *   If errors occurred, update `activeWorkflow.lastError` with English details.
*   **Output:** `.pheromone` updated with English image data. UO informs user (in `currentUser.lastInteractionLanguage`) of the final status.

---