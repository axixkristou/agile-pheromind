# Workflow: Continuer une Tâche de Développement (02_Continue_Task.md)

**Objectif:** Permettre à un développeur de reprendre efficacement le travail sur une tâche technique spécifique. Le système charge le contexte complet de la tâche (incluant l'historique des décisions et le raisonnement précédent stocké dans la `memoryBank`), prépare l'environnement Git, et fournit une assistance contextuelle pendant la phase d'implémentation, avec des mécanismes de gestion d'erreur et de clarification.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@developer-agent`, `@clarification-agent`.

**MCPs Utilisés:** Azure DevOps MCP, Git Tools MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Dev) spécifie l'ID de la tâche Azure DevOps à continuer (ex: `"AgilePheromind continue tâche Azure#23223"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Identification Utilisateur & Récupération/Validation Contexte Tâche Azure DevOps.**
        *   UO délègue à `@devops-connector`.
        *   **onError:** Si ADO MCP échoue, logguer, notifier, arrêter.
    *   **Phase 2: Activation de la Tâche et Chargement du Contexte Approfondi depuis `.pheromone`.**
        *   Scribe met à jour `activeTask`, `activeUserStory` dans `.pheromone`.
        *   UO **injecte un contexte ciblé** de la `memoryBank` (notes, décisions antérieures, snippets, raisonnements liés) à `@developer-agent`.
    *   **Phase 3: Préparation de l'Environnement de Développement et Assistance à l'Implémentation.**
        *   UO délègue à `@developer-agent` pour vérifier/changer branche Git, ouvrir fichiers.
        *   Pendant l'implémentation, si `@developer-agent` rencontre une ambiguïté ou un blocage, il le signale à l'UO qui peut initier une clarification via `@clarification-agent` ou appliquer une stratégie d'erreur.
        *   **onError:** Gestion des échecs MCP (Context7, MSSQL) ou des erreurs de logique de l'agent.

## Détails des Phases:

### Phase 1: Identification Utilisateur & Récupération/Validation Contexte Tâche Azure DevOps
*   **Agent Responsable:** `@devops-connector`
*   **Inputs:** ID de la tâche fourni par l'UO. `currentUser` depuis `.pheromone`.
*   **Actions & Tooling:**
    1.  Utiliser **Azure DevOps MCP**:
        *   `get_user_identity`.
        *   `get_work_item_details {id: ID_Tache}`: Récupérer titre, description, état ADO, US parente, assigné ADO.
*   **onError Strategy (pour l'UO si `@devops-connector` signale échec MCP):**
    1.  Scribe loggue l'erreur dans `activeWorkflow.lastError`.
    2.  UO notifie l'utilisateur: "Impossible de récupérer les détails de la tâche Azure#{{taskId}} depuis Azure DevOps. Erreur MCP: [Message]. Vérifiez la connexion ou l'ID."
    3.  Arrêter ce workflow.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe` si succès):** Résumé NL: "Utilisateur '{{currentUser.azureDevOpsUsername}}' confirmé. Contexte tâche 'Azure#{{taskId}}' ('{{taskTitle}}'), US Parente: 'Azure#{{parentId}} - {{parentTitle}}', État Azure: '{{taskAzureStatus}}', Assigné Azure: '{{taskAzureAssignee}}'. Log: `azure_wi_{{taskId}}_{{timestamp}}.json`."

### Phase 2: Activation de la Tâche et Chargement du Contexte Approfondi depuis `.pheromone`
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe` (pour état actif), `🧐 @uber-orchestrator` (pour injection de contexte), `@developer-agent` (pour lecture).
*   **Inputs:** Résumé NL de `@devops-connector`. Données de `.pheromone`.
*   **Actions & Tooling (Scribe):**
    1.  Mettre à jour `.pheromone` (`activeUserStory`, `activeTask`, statuts dans `memoryBank.tasks.{{taskId}}` et `activeUserStory.tasks`).
*   **Actions & Tooling (UO prépare l'injection de contexte pour `@developer-agent`):**
    1.  Extraire de `memoryBank.tasks.{{activeTask.id}}`: `developerNotes`, `codeSnippets`, `testCasesGeneratedLinks` (liens vers rapports de génération de tests), `relatedCommit`, `reasoningChainLink` (si des décisions précédentes sur cette tâche ont été loggées).
    2.  Extraire de `memoryBank.userStories.{{activeTask.parentId}}`: `descriptionFull`, `acceptanceCriteria`, `analysisSummaries`, `keyDecisions`, `reasoningChainLinks` pertinents.
    3.  Extraire de `memoryBank.projectContext`: `codingConventionsLink`, `designConventionsLink`, `techStack`.
    4.  Extraire de `memoryBank.commonIssuesAndSolutions`: Problèmes similaires résolus précédemment.
*   **Actions & Tooling (`@developer-agent` reçoit ce contexte injecté):**
    1.  Intégrer toutes ces informations pour reconstituer l'état de la tâche.
*   **Memory Bank Interaction:**
    *   Lecture intensive par l'UO pour préparer le contexte.
*   **Output (pour `@developer-agent` lors de la Phase 3):** Un contexte riche et ciblé pour reprendre le travail.

### Phase 3: Préparation de l'Environnement de Développement et Assistance à l'Implémentation
*   **Agent Responsable:** `@developer-agent`
*   **Inputs:** Contexte injecté par l'UO (Phase 2).
*   **Actions & Tooling:**
    1.  **Gestion de Branche Git (Git Tools MCP):**
        *   `get_current_branch`. Vérifier si correspond à `feature/US{{activeUserStory.id}}-description`.
        *   Si non, `checkout_branch` ou `pull_branch` + `checkout_branch`. Gérer les erreurs de Git (conflits, branche inexistante) en informant l'UO.
    2.  **Ouverture des Fichiers:** (Conceptuel) Demander à l'IDE d'ouvrir les fichiers pertinents (`memoryBank.tasks.{{activeTask.id}}.affectedFiles`).
    3.  **Assistance Contextuelle à l'Implémentation:**
        *   Pendant que le développeur code, sur demande :
            *   **Context7 MCP** (`get_library_docs`): Pour documentation .NET/Angular.
            *   **MSSQL MCP** (`get_schema_details`, `validate_sql_query`): Pour aide DB.
            *   Consulter la `memoryBank` (via requête à l'UO ou `@memory-access-agent`) pour des décisions/solutions passées.
        *   **Gestion d'Ambiguïté/Blocage par `@developer-agent`:**
            *   Si une spécification est ambiguë ou si une dépendance bloque :
                *   `@developer-agent` formule le problème clairement.
                *   Il envoie un résumé à l'UO indiquant: "Blocage sur tâche Azure#{{taskId}}: [Description du problème/ambiguïté]. Suggestion de clarification: [Question précise pour le PO/TechLead]."
                *   L'UO peut alors décider d'invoquer `@clarification-agent` ou de suivre une autre stratégie d'erreur du script `01_AI-RUN/*.md`. Le travail sur cette tâche spécifique est mis en pause.
    4.  **Suivi du Travail et Prise de Notes pour la Memory Bank:**
        *   L'agent doit être instruit de noter (pour son résumé final) les décisions techniques prises, les raisons ("chaîne de pensée" pour les solutions complexes), les problèmes résolus, et les nouveaux fichiers affectés.
*   **onError Strategy (pour l'UO si `@developer-agent` signale échec MCP ou blocage insoluble):**
    1.  Scribe loggue l'erreur/le blocage.
    2.  Si échec MCP (Context7, MSSQL), UO peut suggérer au dev de vérifier les MCPs ou de chercher l'info manuellement pour l'instant. Le workflow sur la tâche peut continuer avec cette info manquante si non critique.
    3.  Si blocage logique, UO initie le processus de clarification ou escalade au Tech Lead.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe` à la fin d'une session de travail / sauvegarde de contexte):** Résumé NL détaillé: "Session sur tâche Azure#{{taskId}}. Avancement: [Description]. Fichiers modifiés: [Liste]. Décisions/Raisonnements clés: [Pour MemoryBank]. Problèmes/Clarifications nécessaires: [Si applicable]." (Ce résumé permet au Scribe de mettre à jour `developerNotes`, `affectedFiles`, `reasoningChainLink` dans `memoryBank.tasks.{{activeTask.id}}`).

---