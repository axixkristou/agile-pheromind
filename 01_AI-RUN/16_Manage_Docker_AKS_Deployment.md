# Workflow: Manage Docker Images (Build and Push to ACR) (16_Manage_Docker_Images.md)

**Objective:** Manage the lifecycle of Docker images for the project's applications (.NET API and Angular Frontend). This includes building Docker images from their respective English-commented Dockerfiles and pushing them to Azure Container Registry (ACR). Error handling for Docker commands is integrated. The final report for the user is provided in `currentUser.lastInteractionLanguage`.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@deployment-agent-aks` (role now focused on images), `@clarification-agent`.

**MCPs Used:**
*   Docker MCP (conceptual, or `command_line_tool` for `docker build`, `docker push`).
*   Git Tools MCP (to retrieve Dockerfiles and build context).
*   Azure CLI MCP (conceptual, or `command_line_tool` for `az acr login` and tag checks).

## Pheromind Workflow Overview:

1.  **Initiation:** User (DevOps/Tech Lead) requests Docker image build/push (e.g., `"AgilePheromind build and push SuperAppAPI version 1.2.0"`). `userLanguage` passed by `🎩 @head-orchestrator`.
2.  **`🧐 @uber-orchestrator`** (UO) takes control. UO updates `currentUser.lastInteractionLanguage`.
    *   **Pre-check:** UO verifies `.pheromone.onboardingComplete`.
    *   **Phase 1: Parameter Validation, Dockerfile Retrieval, and English Context Injection.**
        *   UO validates inputs. **Injects English context** to `@deployment-agent-aks`.
        *   `@deployment-agent-aks` retrieves appropriate English-commented Dockerfile (via **Git Tools MCP**).
        *   **onError (Dockerfile Not Found):** Notify user (in `userLanguage`), ask for clarification (via `@clarification-agent`), or stop.
    *   **Phase 2: Docker Image Build (English logs, error handling).**
        *   UO delegates to `@deployment-agent-aks`. Uses **Docker MCP / CLI**.
        *   **onError (Build):** Log detailed English error, notify user (in `userLanguage`), stop.
    *   **Phase 3: Docker Image Push to ACR (English logs, error handling).**
        *   UO delegates to `@deployment-agent-aks`. Requires ACR login. Uses **Docker MCP / CLI**.
        *   **onError (ACR Login/Push):** Log detailed English error, notify user (in `userLanguage`), stop.
    *   **Phase 4: Report (English Content, Final Output Localized) and `.pheromone` Update (English Data).**
        *   `@deployment-agent-aks` generates English report. Translates summary to `userLanguage` for UO.
        *   Scribe records image details and status in `memoryBank.dockerImages` (English).

## Phase Details:

### Phase 1: Parameter Validation, Dockerfile Retrieval, and English Context Injection
*   **Responsible Agent:** UO, `@deployment-agent-aks`, `@clarification-agent`.
*   **Inputs (from UO):** AppName (e.g., `SuperAppAPI`), Version/Tag. `userLanguage`.
*   **Actions (UO):**
    1.  Validate AppName/Version. If unclear, use `@clarification-agent` (question in `userLanguage`).
    2.  Inject English context to `@deployment-agent-aks`: `memoryBank.projectContext.azureContainerRegistryUrl`, `currentProject.gitRepository.localPath`, Dockerfile location convention.
*   **Actions (`@deployment-agent-aks`):**
    1.  Determine Dockerfile path. Use **Git Tools MCP** (`get_file_contents`) for Dockerfile.
*   **onError (Dockerfile Not Found):** `@deployment-agent-aks` reports (English) to UO. UO uses `@clarification-agent` to ask user (in `userLanguage`). Workflow pauses.
*   **Output (to UO if success, English):** "Prerequisites for Docker image [AppName]:[Version]: Dockerfile retrieved. ACR Target: `{{ACR_URL}}`."

### Phase 2: Docker Image Build (English logs, error handling)
*   **Responsible Agent:** `@deployment-agent-aks`.
*   **Inputs:** Dockerfile content, AppName, Version, BuildContextPath, ACR URL (all English context).
*   **Actions & Tooling:** Use **Docker MCP** (or `command_line_tool` with `docker build`): `docker build -t {{ACR_URL}}/{{AppName}}:{{Version}} ...`.
*   **onError (Build):** If `docker build` fails, capture English Docker error. Signal (English) to UO. UO logs (English) via Scribe, notifies user (in `userLanguage` with key error points translated), stops.
*   **Output (to UO if success, English):** "Docker image `{{ACR_URL}}/{{AppName}}:{{Version}}` built successfully locally."

### Phase 3: Docker Image Push to ACR (English logs, error handling)
*   **Responsible Agent:** `@deployment-agent-aks`.
*   **Inputs:** Full Image Name with Tag (English). ACR Name from `memoryBank.projectContext.azureContainerRegistryUrl`.
*   **Actions & Tooling:**
    1.  **Login to ACR:** (Azure CLI MCP or `az acr login`). **onError (ACR Login):** Capture English error. Signal (English) to UO. UO notifies user (in `userLanguage`), stops.
    2.  **Push Image to ACR:** (Docker MCP or `docker push`). **onError (Docker Push):** Capture English error. Signal (English) to UO. UO notifies user (in `userLanguage`), stops.
*   **Output (to UO if success, English):** "Docker image `{{ACR_URL}}/{{AppName}}:{{Version}}` pushed successfully to ACR."

### Phase 4: Report (English Content, Final Output Localized) and `.pheromone` Update (English Data)
*   **Responsible Agent:** `@deployment-agent-aks` (report), Scribe (recording).
*   **Inputs:** Status of build/push. AppName, Version, ACR URL. `currentUser.lastInteractionLanguage` (as `userLanguageForOutput`).
*   **Actions (`@deployment-agent-aks`):**
    1.  **Generate English Report Content:** Markdown (`docker_image_manage_[AppName]_[Version]_[timestamp].md`). Content: AppName, Version/Tag, ACR Target, Build Status (Success/Fail + English Logs), ACR Login Status, Push Status (Success/Fail + English Logs).
    2.  **Translate Summary for UO:** Provide UO with concise English summary of outcome.
    3.  Save full English report in `03_SPECS/Docker_Images/`.
*   **Output (`@deployment-agent-aks` to Scribe, English NL Summary with path to English report):** "Docker image management for `{{AppName}}:{{Version}}` [completed successfully/failed]. English Report: `docker_image_manage_[AppName]_[Version]_[timestamp].md`."
*   **Actions (Scribe):**
    1.  Interpret English summary. Update `.pheromone` (English data):
        *   `documentationRegistry`: Add English report path.
        *   `memoryBank.dockerImages.{{AppName_en}}.{{Version}}`: Update `status_en`, `acrPath_en`, `timestamp`, `lastActionBy_en`, `reportLink_en`.
        *   If errors, update `activeWorkflow.lastError` (English details).
*   **Actions (UO):** Translate concise summary from `@deployment-agent-aks` to `currentUser.lastInteractionLanguage`. Inform user of final status.
*   **Output:** `.pheromone` updated with English image data. User informed (in their language).

---