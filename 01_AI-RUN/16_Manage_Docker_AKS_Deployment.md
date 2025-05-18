# Workflow: Gestion des Déploiements Docker sur Azure Kubernetes Service (AKS) (16_Manage_Docker_AKS_Deployment.md)

**Objectif:** Gérer le cycle de vie des applications conteneurisées (.NET API et Angular Frontend) sur AKS de manière robuste et traçable. Cela inclut le build conditionnel des images Docker, leur push vers Azure Container Registry (ACR), le déclenchement de pipelines Azure DevOps pour le déploiement sur AKS (ou le déploiement direct si nécessaire), et des vérifications post-déploiement. Les erreurs à chaque étape sont gérées et journalisées.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@deployment-agent-aks`, `@devops-connector`, `@clarification-agent`.

**MCPs Utilisés:**
*   Docker MCP (conceptuel, ou `command_line_tool` pour `docker build`, `docker push`).
*   Kubernetes/AKS MCP (conceptuel, ou `command_line_tool` pour `kubectl apply`, `kubectl get`).
*   Azure DevOps MCP (pour `trigger_azure_pipeline`, `get_pipeline_run_status`).
*   Git Tools MCP (pour récupérer Dockerfiles, manifestes K8s).
*   Azure CLI MCP (conceptuel, ou `command_line_tool` pour `az acr`).

## Pheromind Workflow Overview:

1.  **Initiation:** Utilisateur (DevOps/Tech Lead) demande déploiement/mise à jour (ex: `"AgilePheromind déploie SuperAppAPI v1.2.0 sur AKS Staging"`). L'UO s'assure d'avoir toutes les informations (App, Version, Env).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Vérification Prérequis, Collecte Artefacts, et Injection de Contexte.**
        *   UO **injecte contexte** (`memoryBank.projectContext` pour ACR URL, pipeline IDs, K8s conventions) à `@deployment-agent-aks`.
        *   `@deployment-agent-aks` vérifie image Docker dans ACR (via **Azure CLI MCP** ou CLI), récupère Dockerfile et manifestes K8s (via **Git Tools MCP**).
        *   **onError (Accès Artefacts/ACR):** Si échec, notifier, demander clarification (via `@clarification-agent` pour chemin/tag), ou arrêter.
    *   **Phase 2: Build et Push des Images Docker (si nécessaire, avec gestion d'erreur).**
        *   UO délègue à `@deployment-agent-aks`. Utilisation de **Docker MCP / CLI**.
        *   **onError (Build/Push):** Si échec, logguer erreur détaillée, notifier, arrêter.
    *   **Phase 3: Déploiement sur AKS (avec gestion d'erreur).**
        *   UO délègue à `@deployment-agent-aks`. Option 1 (préférée): Déclencher Azure Pipeline via `@devops-connector` (**ADO MCP**). Option 2: `kubectl apply` (via **K8s/AKS MCP** ou CLI).
        *   **onError (Trigger Pipeline/kubectl):** Si échec, logguer erreur, notifier, potentiellement suggérer vérification manuelle des configs pipeline/K8s.
    *   **Phase 4: Vérification Post-Déploiement (avec gestion d'erreur).**
        *   UO délègue à `@deployment-agent-aks`. Vérifier statut pods/services (via **K8s/AKS MCP** ou CLI), statut pipeline ADO (via **ADO MCP**).
        *   **onError (Vérification):** Si pods non `Running` ou service non accessible, logguer, notifier pour investigation.
    *   **Phase 5: Rapport Détaillé (incluant "Chaîne de Décision") et Mise à Jour `.pheromone`.**
        *   `@deployment-agent-aks` génère rapport. Scribe enregistre.

## Détails des Phases:

### Phase 1: Vérification Prérequis, Collecte Artefacts, et Injection de Contexte
*   **Agent Responsable:** `@deployment-agent-aks`, UO, `@clarification-agent`.
*   **Inputs (de l'UO):** Nom App, Version/Tag, Env AKS. Contexte `memoryBank` (ACR URL, pipeline IDs, conventions K8s).
*   **Actions (`@deployment-agent-aks`):**
    1.  **Vérifier Image ACR:** (Azure CLI MCP ou `az acr...`).
    2.  **Récupérer Dockerfile (si build requis):** Git Tools MCP (`get_file_contents`).
    3.  **Récupérer Manifestes K8s:** Git Tools MCP (`get_file_contents` pour `deploy/aks/[AppName]/[Env]/`).
*   **onError (Accès Artefacts/ACR par `@deployment-agent-aks`):**
    *   Si image introuvable et Dockerfile non trouvé, ou manifestes K8s absents:
        *   `@deployment-agent-aks` signale à l'UO: "Prérequis manquants pour déploiement [App]: [Détail].".
        *   UO met workflow en pause, délègue à `@clarification-agent` pour demander à l'utilisateur de vérifier/fournir les chemins corrects ou de confirmer si un build est nécessaire.
*   **Output (vers UO):** "Prérequis [App] v[Version] sur [Env]: Image Docker [Existe/À Builder]. Dockerfile [OK/Manquant]. Manifestes K8s [OK/Manquants]."

### Phase 2: Build et Push des Images Docker (si nécessaire, avec gestion d'erreur)
*   **Agent Responsable:** `@deployment-agent-aks`.
*   **Inputs:** Dockerfile, AppName, Version, BuildContextPath, ACR URL.
*   **Actions (si build nécessaire):**
    1.  Build Image (Docker MCP / `docker build`).
    2.  Login ACR (Azure CLI MCP / `az acr login`).
    3.  Push Image (Docker MCP / `docker push`).
*   **onError (Build/Push par `@deployment-agent-aks`):**
    *   Si `docker build` ou `docker push` échoue, capturer l'erreur Docker.
    *   Signaler à l'UO: "Échec [Build/Push] image `{{ACR_URL}}/{{AppName}}:{{Version}}`. Erreur Docker: [MsgErreurDocker].".
    *   UO loggue via Scribe (`activeWorkflow.lastError`), notifie utilisateur, arrête workflow.
*   **Output (vers UO):** "Image Docker `{{ACR_URL}}/{{AppName}}:{{Version}}` [buildée et pushée OK / échec build / échec push]."

### Phase 3: Déploiement sur AKS (avec gestion d'erreur)
*   **Agent Responsable:** `@deployment-agent-aks` (ou `@devops-connector`).
*   **Inputs:** Manifestes K8s, AppName, ImageTag (URL complète ACR), Env AKS.
*   **Option 1: Trigger Azure Pipeline (`@devops-connector`):**
    1.  **ADO MCP** `trigger_azure_pipeline` (avec params AppName, ImageTag, Env).
    2.  **onError (Trigger Pipeline):** Si échec, `@devops-connector` signale à l'UO. UO loggue, notifie "Échec déclenchement pipeline déploiement AKS: [Erreur MCP]. Vérifiez config pipeline et permissions.", arrête ou propose Option 2.
*   **Option 2: Déploiement Direct (`@deployment-agent-aks`):**
    1.  Mettre à jour manifestes (IMAGE_TAG).
    2.  Appliquer manifestes (K8s/AKS MCP ou `kubectl apply`).
    3.  **onError (`kubectl apply`):** Si échec, `@deployment-agent-aks` capture erreur kubectl. Signale à l'UO. UO loggue, notifie "Échec `kubectl apply` pour [App] sur [Env]: [ErreurKubectl]. Vérifiez manifestes et connexion AKS.", arrête.
*   **"Chaîne de Décision" pour le rapport:** L'agent doit noter quelle option a été choisie et pourquoi (ex: "Pipeline préféré utilisé", ou "Pipeline échec, fallback sur kubectl direct").
*   **Output (vers UO):** "Déploiement [App]:[Version] sur AKS [Env] initié [via Pipeline/kubectl]. Attente vérification."

### Phase 4: Vérification Post-Déploiement (avec gestion d'erreur)
*   **Agent Responsable:** `@deployment-agent-aks`.
*   **Inputs:** AppName, Env AKS, Namespace K8s.
*   **Actions:**
    1.  Attendre X secondes/minutes.
    2.  Si Pipeline ADO: vérifier statut final (**ADO MCP** `get_pipeline_run_status`). **onError:** Si échec récupération statut, le noter.
    3.  Vérifier pods/services (**K8s/AKS MCP** ou `kubectl get pods/services -l app={{AppName}} -n {{Namespace}}`). (Optionnel) `kubectl logs deployment/...`.
*   **onError (Vérification K8s):**
    *   Si pods non `Running`, service sans endpoint, ou erreurs dans les logs:
        *   `@deployment-agent-aks` signale à l'UO: "Vérification post-déploiement [App] sur [Env] ÉCHOUÉE. Statut Pods: [Détail]. Erreurs logs: [Extraits].".
        *   UO loggue, notifie utilisateur pour investigation. Le statut du déploiement dans `.pheromone` sera marqué comme "Deployed_With_Errors".
*   **Output (vers Scribe):** Résumé NL: "Vérif post-déploiement [App] sur [Env]: Pipeline [StatutADO]. Pods [StatutK8s]. Service [Endpoint]. Logs [OK/Erreurs]."

### Phase 5: Rapport Détaillé (incluant "Chaîne de Décision") et Mise à Jour `.pheromone`
*   **Agent Responsable:** `@deployment-agent-aks` (rapport), Scribe (enregistrement).
*   **Inputs:** Résumés des phases.
*   **Actions (`@deployment-agent-aks`):**
    1.  Générer rapport MD (`aks_deployment_[App]_[Ver]_[Env]_[timestamp].md`) dans `03_SPECS/Deployments/`.
    2.  Contenu: App/Version/Env, Statut image build/push, **Méthode de déploiement choisie (et pourquoi si fallback)**, Statut pipeline ADO (si utilisé) avec lien vers logs, Résultat vérifications K8s (pods, services, extraits de logs pertinents), Erreurs rencontrées et actions prises.
*   **Output (`@deployment-agent-aks` -> Scribe):** Résumé NL: "Déploiement [App] v[Version] sur AKS [Env] [Réussi/Réussi avec avertissements/Échoué]. Rapport (avec chaîne de décision): `aks_deployment...md`."
*   **Actions (Scribe):**
    1.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter rapport.
        *   `memoryBank.deployments.{{AppName}}.{{Version}}.{{EnvAKS}}`: Mettre à jour `status`, `timestamp`, `deployedBy`, `logsLink`, `verificationDetails`, `reasoningChainLink` (vers le rapport).
        *   Si erreurs, mettre à jour `activeWorkflow.lastError`.
*   **Output:** `.pheromone` à jour. UO informe utilisateur du statut final.

---