# Workflow: Gestion des Déploiements Docker sur Azure Kubernetes Service (AKS) (16_Manage_Docker_AKS_Deployment.md)

**Objectif:** Gérer le cycle de vie des applications conteneurisées (.NET API et Angular Frontend) sur AKS. Cela inclut le build des images Docker, leur push vers Azure Container Registry (ACR), et le déploiement ou la mise à jour des applications sur AKS en utilisant des manifestes Kubernetes et potentiellement Azure Pipelines.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@deployment-agent-aks`, `@devops-connector` (pour déclencher Azure Pipelines).

**MCPs Utilisés:**
*   Docker MCP (conceptuel, ou `command_line_tool` pour les commandes Docker CLI: `docker build`, `docker push`).
*   Kubernetes/AKS MCP (conceptuel, ou `command_line_tool` pour `kubectl apply -f <manifest.yaml>`).
*   Azure DevOps MCP (pour `trigger_azure_pipeline`).
*   Git Tools MCP (pour récupérer les Dockerfiles et manifestes Kubernetes du dépôt).

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (DevOps/Tech Lead) demande un déploiement ou une mise à jour (ex: `"AgilePheromind déploie SuperAppAPI version 1.2.0 sur AKS environnement Staging"` ou `"AgilePheromind met à jour SuperAppFrontend sur AKS Prod avec la dernière image de la branche main"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Vérification des Prérequis et Collecte des Artefacts.**
        *   UO délègue à `@deployment-agent-aks`. Vérification de l'existence de l'image Docker dans ACR (ou nécessité de build), récupération des manifestes Kubernetes.
    *   **Phase 2: Build et Push des Images Docker (si nécessaire).**
        *   UO délègue à `@deployment-agent-aks`. Utilisation de Docker MCP / CLI.
    *   **Phase 3: Déploiement sur AKS.**
        *   UO délègue à `@deployment-agent-aks`. Option 1 (préférée): Déclenchement d'un pipeline Azure DevOps via `@devops-connector`. Option 2: Application directe des manifestes via Kubernetes/AKS MCP ou `kubectl`.
    *   **Phase 4: Vérification Post-Déploiement (Basique).**
        *   UO délègue à `@deployment-agent-aks`. Vérification du statut des pods et services.
    *   **Phase 5: Rapport et Mise à Jour de `.pheromone`.**
        *   Scribe enregistre le statut du déploiement.

## Détails des Phases:

### Phase 1: Vérification des Prérequis et Collecte des Artefacts
*   **Agent Responsable:** `@deployment-agent-aks`
*   **Inputs:** Nom de l'application (ex: `SuperAppAPI`, `SuperAppFrontend`), version/tag de l'image, environnement AKS cible (Staging, Prod), fournis par l'UO. `memoryBank.currentProject.repositoryUrl`, `memoryBank.projectContext.azureContainerRegistryUrl`.
*   **Actions & Tooling:**
    1.  **Vérifier l'Image Docker dans ACR:**
        *   (Conceptuel) Utiliser un **Azure CLI MCP** ou `command_line_tool` avec `az acr repository show-tags -n {{ACR_Name}} --repository {{AppName}}` pour vérifier si l'image `{{AppName}}:{{Version}}` existe.
        *   Si l'image n'existe pas, noter qu'un build est nécessaire.
    2.  **Récupérer les Dockerfiles (si build nécessaire):**
        *   Utiliser **Git Tools MCP** (`get_file_contents`) pour récupérer le `Dockerfile` approprié (ex: `backend/{{AppName}}.Api/Dockerfile` ou `frontend/{{AppName}}App/Dockerfile`) de la branche/commit spécifié.
    3.  **Récupérer les Manifestes Kubernetes:**
        *   Utiliser **Git Tools MCP** (`get_file_contents`) pour récupérer les manifestes Kubernetes (ex: `deployment.yaml`, `service.yaml`, `ingress.yaml`) pour l'application et l'environnement cible (ex: `deploy/aks/[AppName]/[Environment]/`).
        *   S'assurer que les placeholders dans les manifestes (ex: `IMAGE_TAG`, `REPLICA_COUNT`) sont identifiés.
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.projectContext` pour URLs ACR, conventions de nommage d'images. `documentationRegistry` pour les chemins des manifestes K8s s'ils sont versionnés et tracés.
*   **Output (interne à `@deployment-agent-aks` ou vers UO):** Statut de l'image Docker, Dockerfile (si build), manifestes K8s. "Prérequis pour déploiement de [AppName] v[Version] sur [Env]: Image Docker [Existe/À Builder]. Manifestes K8s récupérés."

### Phase 2: Build et Push des Images Docker (si nécessaire)
*   **Agent Responsable:** `@deployment-agent-aks`
*   **Inputs:** Dockerfile, nom de l'application, version/tag, contexte de build (chemin vers le code source). `memoryBank.projectContext.azureContainerRegistryUrl`.
*   **Actions & Tooling (si l'image doit être buildée):**
    1.  **Build de l'Image Docker:**
        *   Utiliser **Docker MCP** (ou `command_line_tool` avec `docker build`):
            *   `docker build -t {{ACR_URL}}/{{AppName}}:{{Version}} -f {{PathToDockerfile}} {{BuildContextPath}}`
    2.  **Login à ACR:**
        *   (Conceptuel) Utiliser **Azure CLI MCP** ou `command_line_tool` avec `az acr login -n {{ACR_Name}}` (nécessite des credentials Azure configurés pour l'agent/environnement Pheromind).
    3.  **Push de l'Image vers ACR:**
        *   Utiliser **Docker MCP** (ou `command_line_tool` avec `docker push`):
            *   `docker push {{ACR_URL}}/{{AppName}}:{{Version}}`
*   **Memory Bank Interaction:**
    *   (Via Scribe) Enregistrer l'événement de build et push dans `memoryBank.deployments` ou `agentActivityLog`.
*   **Output (vers UO ou Scribe si c'est une étape distincte):** Résumé NL: "Image Docker `{{ACR_URL}}/{{AppName}}:{{Version}}` buildée et pushée avec succès vers ACR." ou "Échec du build/push de l'image Docker: [Erreur]."

### Phase 3: Déploiement sur AKS
*   **Agent Responsable:** `@deployment-agent-aks` (ou `@devops-connector` si via Azure Pipeline).
*   **Inputs:** Manifestes Kubernetes, nom de l'application, version/tag de l'image, environnement AKS cible. URL de l'image dans ACR.
*   **Option 1: Déclenchement d'Azure Pipeline (Préférée):**
    *   **Agent:** `@devops-connector`
    *   **Actions & Tooling:**
        1.  Identifier le pipeline Azure DevOps pertinent pour le déploiement AKS (nom/ID stocké dans `memoryBank.projectContext.azurePipelines.aksDeploymentPipelineId` ou fourni).
        2.  Utiliser **Azure DevOps MCP** (`trigger_azure_pipeline`):
            *   `trigger_azure_pipeline {pipelineId, branch: 'main' (ou branche de config), parameters: { appName: AppName, imageTag: Version, aksEnvironment: Environment }}`.
*   **Option 2: Déploiement Direct via Kubernetes/AKS MCP ou `kubectl`:**
    *   **Agent:** `@deployment-agent-aks`
    *   **Actions & Tooling:**
        1.  **Mettre à Jour les Manifestes:** Remplacer les placeholders dans les manifestes K8s avec les valeurs actuelles (ex: `IMAGE_TAG = {{ACR_URL}}/{{AppName}}:{{Version}}`).
        2.  **Appliquer les Manifestes:**
            *   Utiliser **Kubernetes/AKS MCP** (ou `command_line_tool` avec `kubectl`):
                *   (Conceptuel) `aks_login {clusterName, resourceGroup}` (si MCP spécifique).
                *   `kubectl apply -f {{PathToUpdatedDeploymentManifest}}`
                *   `kubectl apply -f {{PathToUpdatedServiceManifest}}`
                *   `kubectl apply -f {{PathToUpdatedIngressManifest}}` (si applicable)
*   **Memory Bank Interaction:**
    *   (Via Scribe) Enregistrer l'initiation du déploiement, la méthode utilisée, et le pipeline ID si applicable dans `memoryBank.deployments`.
*   **Output (vers UO ou Scribe):** Résumé NL: "Déploiement de `{{AppName}}:{{Version}}` sur AKS environnement `{{EnvironmentAKS}}` initié [via Pipeline Azure {{PipelineID}}/directement via kubectl]. En attente de confirmation."

### Phase 4: Vérification Post-Déploiement (Basique)
*   **Agent Responsable:** `@deployment-agent-aks`
*   **Inputs:** Nom de l'application, environnement AKS.
*   **Actions & Tooling:**
    1.  Attendre un délai raisonnable pour que les pods démarrent.
    2.  Utiliser **Kubernetes/AKS MCP** (ou `command_line_tool` avec `kubectl`):
        *   `kubectl get pods -l app={{AppName}} -n {{Namespace}}`: Vérifier si les pods sont `Running`.
        *   `kubectl get services -l app={{AppName}} -n {{Namespace}}`: Vérifier si le service a une IP externe/endpoint.
        *   (Optionnel) `kubectl logs deployment/{{AppNameDeployment}} -n {{Namespace}} --tail=50`: Vérifier les logs récents pour des erreurs de démarrage.
    3.  Si Azure Pipeline a été utilisé, consulter son statut final via **Azure DevOps MCP** (`get_pipeline_run_status`).
*   **Memory Bank Interaction:**
    *   (Via Scribe) Mettre à jour le statut du déploiement dans `memoryBank.deployments` avec le résultat de la vérification.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Vérification post-déploiement pour `{{AppName}}` sur `{{EnvironmentAKS}}`: [Statut des Pods (ex: Tous Running / X sur Y Running)]. Service endpoint: [IP/URL si dispo]. Logs récents [OK/Montrent des erreurs]. Statut du Pipeline Azure (si utilisé): [Succès/Échec]."

### Phase 5: Rapport et Mise à Jour de `.pheromone`
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumés des phases précédentes, notamment celui de la vérification post-déploiement.
*   **Actions & Tooling:**
    1.  Interpréter les résumés via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `memoryBank.deployments.{{AppName}}.{{Version}}.{{EnvironmentAKS}}`:
            *   `status`: (ex: "Success", "Failed_Build", "Failed_Push", "Deployed_Pending_Verification", "Verification_OK", "Verification_Failed").
            *   `timestamp`: `{{timestamp}}`.
            *   `deployedBy`: `{{currentUser.id}}` (ou "AzurePipeline:[ID]").
            *   `logsLink`: (Lien vers les logs Azure Pipeline ou un fichier de log généré).
            *   `verificationDetails`: (Résumé de la Phase 4).
        *   `documentationRegistry`: Enregistrer un rapport de déploiement si `@deployment-agent-aks` en génère un (ex: `deployment_report_{{AppName}}_{{Version}}_{{Env}}_{{timestamp}}.md` dans `03_SPECS/Deployments/`).
*   **Memory Bank Interaction:**
    *   Écriture: Enregistrement détaillé de l'historique et du statut du déploiement.
*   **Output:** `.pheromone` mis à jour. L'UO est informé du résultat final du déploiement et peut notifier l'utilisateur.

---