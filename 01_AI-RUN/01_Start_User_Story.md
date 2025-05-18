# Workflow: Démarrer une User Story (01_Start_User_Story.md)

**Objectif:** Initialiser le travail sur une User Story (US) spécifique d'Azure DevOps. Ce processus implique l'identification de l'utilisateur, la récupération des détails complets de l'US, l'injection de contexte pertinent de la `memoryBank`, la décomposition de l'US en tâches techniques estimées (avec journalisation de la "chaîne de pensée"), la synchronisation avec Azure DevOps, la gestion des erreurs et ambiguïtés, et la préparation de la première tâche pour le développement.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@task-breakdown-estimator`, `@developer-agent`, `@clarification-agent`.

**MCPs Utilisés:** Azure DevOps MCP, Git Tools MCP, Sequential Thinking MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Dev) fournit l'ID de l'US Azure DevOps (ex: `"AgilePheromind commence US Azure#12323"`).
2.  **`🧐 @uber-orchestrator`** (UO) prend le contrôle.
    *   **Phase 1: Identification Utilisateur & Récupération des Détails Complets de l'US.**
        *   UO délègue à `@devops-connector`.
        *   **onError:** Si ADO MCP échoue, logguer l'erreur, notifier l'utilisateur, et arrêter le workflow.
    *   **Phase 2: Mise à Jour Initiale de `.pheromone` et Validation de Clarté de l'US.**
        *   Scribe met à jour `.pheromone` (`currentUser`, `activeUserStory`).
        *   UO évalue la clarté de la description de l'US. Si ambiguë, UO engage `@clarification-agent` pour demander des précisions au PO/demandeur via `ask_followup_question`. Le workflow attend la réponse (traitée par `01_AI-RUN/XX_Handle_Clarification_Response.md`).
    *   **Phase 3: Décomposition en Tâches Techniques & Estimation (Analyse et Planification Détaillée).**
        *   UO évalue si une (re)décomposition est nécessaire (basé sur `.pheromone.activeUserStory.tasks` et `memoryBank.userStories[ID_US].tasks`).
        *   Si oui, UO **injecte un contexte ciblé** (ex: US similaires passées, conventions .NET/Angular pertinentes depuis `memoryBank`) et délègue à `@task-breakdown-estimator`. L'agent utilise **Sequential Thinking MCP**, **Context7 MCP**, **MSSQL MCP**, et doit **détailler sa "chaîne de pensée"** dans son rapport.
        *   Les tâches/estimations sont synchronisées avec Azure DevOps via `@devops-connector`.
        *   **onError:** Si `@task-breakdown-estimator` échoue ou signale une ambiguïté persistante, UO loggue l'erreur et peut re-solliciter `@clarification-agent` ou notifier le Tech Lead.
    *   **Phase 4: Préparation de la Première Tâche & Environnement de Développement.**
        *   UO délègue à `@developer-agent` pour initialiser la tâche, créer branche Git.
        *   **onError:** Si création de branche échoue, logguer et notifier.

## Détails des Phases:

### Phase 1: Identification Utilisateur & Récupération des Détails Complets de l'US
*   **Agent Responsable:** `@devops-connector`
*   **Inputs:** ID de l'US (ex: `Azure#12323`) fourni par l'UO.
*   **Actions & Tooling:**
    1.  Utiliser **Azure DevOps MCP**:
        *   `get_user_identity`: Confirmer/Identifier l'utilisateur.
        *   `get_work_item_details {id: ID_US}`: Récupérer titre, description **complète**, état, priorité, ACs (si dans un champ dédié), etc.
*   **onError Strategy (pour l'UO si `@devops-connector` signale échec MCP):**
    1.  Scribe loggue l'erreur dans `activeWorkflow.lastError` et `memoryBank.agentActivityLog`.
    2.  UO notifie l'utilisateur: "Impossible de récupérer les détails de l'US Azure#{{usId}} depuis Azure DevOps. Erreur MCP: [Message d'erreur]. Veuillez vérifier la connexion ou l'ID de l'US."
    3.  Arrêter ce workflow.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe` si succès):** Résumé NL: "Utilisateur '[NomUtilisateurAzure]' confirmé. Détails complets pour US 'Azure#{{usId}}' ('{{usTitle}}') récupérés. Description: '{{usDescription}}'. État Azure: '{{usAzureStatus}}'. Priorité: {{usPriority}}. ACs: '{{usAcceptanceCriteria}}'. Log: `azure_wi_{{usId}}_{{timestamp}}.json`."

### Phase 2: Mise à Jour Initiale de `.pheromone` et Validation de Clarté de l'US
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`, `🧐 @uber-orchestrator`, `@clarification-agent`
*   **Inputs:** Résumé NL de `@devops-connector`.
*   **Actions & Tooling (Scribe):**
    1.  Mettre à jour `.pheromone` avec `currentUser`, `activeUserStory` (incluant `descriptionFull`, `acceptanceCriteriaFromAzure`), et enrichir `memoryBank.userStories.{{usId}}`.
*   **Actions & Tooling (UO):**
    1.  Analyser `activeUserStory.descriptionFull` et `activeUserStory.acceptanceCriteriaFromAzure` pour évaluer leur clarté et leur exploitabilité pour la décomposition.
    2.  **Si ambiguïté détectée** (ex: ACs vagues, description manquant de détails critiques):
        *   Stocker l'état actuel du workflow dans `.pheromone.activeWorkflow` (ex: `status: 'PendingClarification'`).
        *   Préparer le contexte pour `@clarification-agent`: l'US ID, la description/ACs problématiques, et la question spécifique à poser (ex: "Pour l'US Azure#{{usId}}, le critère d'acceptation 'L'interface doit être intuitive' nécessite plus de détails. Pouvez-vous spécifier 2-3 aspects clés de cette intuitivité attendue ?").
        *   Déléguer à `@clarification-agent`.
        *   Le workflow `01_Start_User_Story.md` est mis en pause. La réponse de l'utilisateur sera traitée par `01_AI-RUN/XX_Handle_Clarification_Response.md`, qui réactivera ensuite ce workflow si la clarification est obtenue.
*   **Output:** `.pheromone` mis à jour. Si clarification demandée, workflow en pause. Sinon, UO passe à Phase 3.

### Phase 3: Décomposition en Tâches Techniques & Estimation (Analyse et Planification Détaillée)
*   **Agent Responsable:** `@task-breakdown-estimator` (coordonnant avec `@devops-connector`)
*   **Inputs (Injectés par l'UO):**
    *   `activeUserStory` (avec description et ACs clarifiés si Phase 2 a eu lieu).
    *   **Contexte de la `memoryBank`:**
        *   `memoryBank.projectContext` (techStack, conventions, estimationUnit).
        *   (Optionnel) Exemples de décompositions d'US similaires passées (`memoryBank.userStories` où `story.type == activeUserStory.type`).
        *   (Optionnel) Décisions architecturales pertinentes (`memoryBank.architecturalDecisions`).
*   **Actions & Tooling (`@task-breakdown-estimator`):**
    1.  Utiliser **Sequential Thinking MCP** pour planifier la décomposition.
    2.  **Détailler la "Chaîne de Pensée":** Documenter explicitement dans le rapport de décomposition (`us_{{usId}}_task_breakdown_{{timestamp}}.md`) :
        *   Comment les ACs ont été traduits en besoins techniques.
        *   Pourquoi certains choix de décomposition ont été faits (ex: création d'un service .NET séparé vs modification d'un existant).
        *   Quelles documentations (issues de **Context7 MCP**) ou analyses de schéma (**MSSQL MCP**) ont influencé les tâches proposées.
        *   La base de chaque estimation.
    3.  Proposer tâches techniques, estimations, dépendances.
    4.  Consulter `@devops-connector` pour synchroniser avec Azure DevOps (**Azure DevOps MCP**).
*   **onError Strategy (pour l'UO si `@task-breakdown-estimator` signale échec ou ambiguïté):**
    1.  Scribe loggue l'erreur.
    2.  Si ambiguïté, UO peut relancer Phase 2 avec `@clarification-agent` en ciblant le point d'ambiguïté soulevé par `@task-breakdown-estimator`.
    3.  Si échec MCP ou autre, notifier le Tech Lead/PO. Arrêter ou proposer une alternative.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Décomposition US 'Azure#{{usId}}' [terminée/mise à jour]. [N_total] tâches, total [TotalEstimation] {{estimationUnit}}. Synchronisé avec ADO. Rapport (avec chaîne de pensée): `us_{{usId}}_task_breakdown_{{timestamp}}.md`." (Rapport dans `03_SPECS/Task_Breakdowns/`).

### Phase 4: Préparation de la Première Tâche & Environnement de Développement
*   **Agent Responsable:** `@developer-agent`
*   **Inputs (Injectés par l'UO):**
    *   `activeUserStory` (avec tâches peuplées et estimées).
    *   Première tâche `ToDo` identifiée.
    *   Contexte de la `memoryBank`: `projectContext.defaultGitBranchingStrategy`.
*   **Actions & Tooling:**
    1.  Identifier la première tâche `ToDo`.
    2.  Utiliser **Git Tools MCP**: `create_branch`, `checkout_branch`.
    3.  Mettre à jour la tâche dans `.pheromone` (via Scribe): `status: "InProgress"`, `assignee: currentUser.id`.
*   **onError Strategy (pour l'UO si `@developer-agent` signale échec Git Tools MCP):**
    1.  Scribe loggue l'erreur.
    2.  UO notifie le développeur: "Erreur lors de la création/checkout de la branche Git pour la tâche Azure#{{taskId}}. Erreur MCP: [Message]. Veuillez vérifier votre configuration Git."
    3.  Le workflow peut s'arrêter ou attendre une action manuelle.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Prêt pour tâche 'Azure#{{taskId}}' ('{{taskTitle}}') de l'US 'Azure#{{usId}}'. Assignée à '{{currentUser.azureDevOpsUsername}}'. Branche Git '{{branchName}}' active/créée."

---