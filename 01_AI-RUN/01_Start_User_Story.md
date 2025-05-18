# Workflow: Démarrer une User Story (01_Start_User_Story.md)

**Objectif:** Initialiser le travail sur une User Story (US) spécifique d'Azure DevOps. Cela inclut l'identification de l'utilisateur, la récupération des détails de l'US, sa décomposition en tâches techniques estimées (si ce n'est pas déjà fait ou si une révision est nécessaire), la mise à jour de la Memory Bank dans `.pheromone`, et la préparation de la première tâche pour le développement.

**Agents IA Clés:** `🧐 @uber-orchestrator`, `✍️ @orchestrator-pheromone-scribe`, `@devops-connector`, `@task-breakdown-estimator`, `@developer-agent`.

**MCPs Utilisés:** Azure DevOps MCP, Git Tools MCP, Sequential Thinking MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Dev) fournit l'ID de l'US Azure DevOps (ex: `"AgilePheromind commence US Azure#12323"`).
2.  **`🧐 @uber-orchestrator`** (UO) prend le contrôle.
    *   **Phase 1: Identification Utilisateur & Récupération des Détails de l'US.**
        *   UO délègue à `@devops-connector`.
    *   **Phase 2: Mise à Jour Initiale de `.pheromone` (État Actif & Memory Bank).**
        *   Le `✍️ @orchestrator-pheromone-scribe` (Scribe) met à jour `.pheromone` avec `currentUser` et `activeUserStory`.
    *   **Phase 3: Décomposition en Tâches Techniques & Estimation (Analyse et Planification).**
        *   UO évalue si une (re)décomposition est nécessaire en consultant `.pheromone.activeUserStory.tasks` et `memoryBank.userStories[ID_US].tasks`.
        *   Si oui, UO délègue à `@task-breakdown-estimator`. Cet agent utilise **Sequential Thinking MCP** pour l'analyse, **Context7 MCP** pour la documentation des librairies .NET/Angular, et **MSSQL MCP** pour l'analyse de schéma si des modifications DB sont envisagées. Les tâches et estimations sont ensuite synchronisées avec Azure DevOps via `@devops-connector`.
    *   **Phase 4: Préparation de la Première Tâche & Environnement de Développement.**
        *   UO délègue à `@developer-agent` pour initialiser la première tâche `ToDo`, créer une branche Git, et mettre à jour `.pheromone`.

## Détails des Phases:

### Phase 1: Identification Utilisateur & Récupération des Détails de l'US
*   **Agent Responsable:** `@devops-connector`
*   **Inputs:** ID de l'US (ex: `Azure#12323`) fourni par l'UO. Contexte de l'utilisateur (depuis `.pheromone.currentUser` si déjà identifié, sinon l'agent tente l'identification).
*   **Actions & Tooling:**
    1.  Utiliser **Azure DevOps MCP**:
        *   `get_user_identity`: Confirmer/Identifier l'utilisateur Azure DevOps. Si l'ID de l'utilisateur Pheromind (`.pheromone.currentUser.id`) n'est pas encore mappé à un `azureDevOpsUsername`, tenter de le faire ou demander clarification à l'UO.
        *   `get_work_item_details {id: ID_US}`: Récupérer titre, description complète, état actuel dans Azure DevOps, priorité, tags, et tout autre champ pertinent de l'US.
*   **Memory Bank Interaction (via Scribe après résumé):**
    *   Le Scribe mettra à jour `.pheromone.currentUser` si une nouvelle identification a eu lieu.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Utilisateur '[NomUtilisateurAzure]' (ID Pheromind: '{{currentUser.id}}') confirmé/identifié. Détails pour US 'Azure#{{usId}}' ('{{usTitle}}') récupérés. Description: '{{usDescription}}'. État Azure: '{{usAzureStatus}}'. Priorité: {{usPriority}}. Détails complets loggés dans `azure_wi_{{usId}}_{{timestamp}}.json`." (Log enregistré dans `03_SPECS/AzureDevOps_Logs/`).

### Phase 2: Mise à Jour Initiale de `.pheromone` (État Actif & Memory Bank)
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@devops-connector`.
*   **Actions & Tooling:**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `currentUser`: Mettre à jour `id` et `azureDevOpsUsername` si modifié/identifié.
        *   `activeUserStory`: { `id`: "Azure#{{usId}}", `title`: "{{usTitle}}", `status`: "InProgressByPheromind", `descriptionFromAzure`: "{{usDescription}}", `azureStatus`: "{{usAzureStatus}}", `priority`: {{usPriority}}, `tasks`: [] (sera peuplé plus tard) }.
        *   `memoryBank.userStories.{{usId}}`:
            *   Créer ou mettre à jour l'entrée pour l'US.
            *   `title`: "{{usTitle}}".
            *   `descriptionFull`: "{{usDescription}}".
            *   `azureStatus`: "{{usAzureStatus}}".
            *   `priority`: {{usPriority}}.
            *   `statusHistory`: Ajouter { `status`: "InProgressByPheromind", `timestamp`: "{{timestamp}}", `actor`: "AgilePheromindSystem", `developer`: "{{currentUser.id}}" }.
            *   `tasks`: Initialiser à `[]` si nouvelle.
        *   `documentationRegistry`: Ajouter une entrée pour `azure_wi_{{usId}}_{{timestamp}}.json`.
*   **Memory Bank Interaction:**
    *   Écriture: Création/Mise à jour extensive de l'entrée de l'US dans `memoryBank.userStories`.
*   **Output:** `.pheromone` mis à jour. UO est informé pour passer à la phase suivante.

### Phase 3: Décomposition en Tâches Techniques & Estimation (Analyse et Planification)
*   **Agent Responsable:** `@task-breakdown-estimator`
*   **Inputs:** `activeUserStory` depuis `.pheromone`. Accès à `memoryBank.projectContext.techStack`.
*   **Actions & Tooling:**
    1.  Analyser `activeUserStory.descriptionFromAzure` et les ACs (si disponibles dans la description ou une section liée).
    2.  Utiliser **Sequential Thinking MCP** pour structurer l'analyse de décomposition:
        *   Identifier les modules/couches impactés (API .NET, services .NET, composants Angular, base de données MSSQL, etc.).
        *   Pour chaque module, lister les actions de haut niveau.
        *   Affiner chaque action en tâches techniques spécifiques et granulaires.
    3.  Pour chaque tâche potentielle nécessitant des connaissances techniques spécifiques:
        *   Consulter **Context7 MCP** (`get_library_docs`) pour la documentation des librairies .NET/Angular pertinentes afin d'évaluer la complexité ou les approches d'implémentation.
        *   Si des modifications de schéma DB ou des requêtes complexes sont envisagées, utiliser **MSSQL MCP** (`get_schema_details`, `get_stored_procedure_definition`) pour analyser les tables/procs impactées et s'assurer de la faisabilité.
    4.  Pour chaque tâche technique définie:
        *   Rédiger une description claire.
        *   Estimer l'effort (en points ou heures, selon la convention du projet dans `memoryBank.projectContext.estimationUnit`).
        *   Identifier les dépendances entre tâches.
    5.  Consulter `@devops-connector` pour utiliser **Azure DevOps MCP**:
        *   `get_child_work_items {parentId: activeUserStory.id}`: Récupérer les tâches existantes pour cette US.
        *   Comparer les tâches générées avec celles existantes. Proposer des créations, mises à jour (d'estimation, description) ou suppressions (si une tâche existante n'est plus pertinente).
        *   Utiliser `create_work_item` ou `update_work_item` pour synchroniser les tâches dans Azure DevOps, en les liant à l'US parente.
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.projectContext.techStack`, `memoryBank.projectContext.estimationUnit`.
    *   Écriture (via Scribe): Les tâches (avec ID Azure, titre, description, estimation, statut `ToDo`) sont ajoutées à `memoryBank.tasks` et leurs IDs sont listés dans `memoryBank.userStories.{{usId}}.tasks`. L'historique d'estimation est aussi stocké.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Analyse et décomposition de l'US 'Azure#{{usId}}' terminées. [N_total] tâches techniques identifiées/mises à jour, pour un total estimé de [TotalEstimation] {{estimationUnit}}. [N_new] nouvelles tâches créées dans Azure DevOps, [N_updated] tâches mises à jour. Rapport détaillé de décomposition : `us_{{usId}}_task_breakdown_{{timestamp}}.md`." (Rapport enregistré dans `03_SPECS/Task_Breakdowns/`).

### Phase 4: Préparation de la Première Tâche & Environnement de Développement
*   **Agent Responsable:** `@developer-agent`
*   **Inputs:** `activeUserStory` (avec ses tâches synchronisées et estimées) depuis `.pheromone`. `currentUser.id` pour l'assignation.
*   **Actions & Tooling:**
    1.  Identifier la première tâche avec le statut `ToDo` (ou la plus prioritaire non bloquée) dans `activeUserStory.tasks`. Si aucune tâche `ToDo` n'est disponible, le signaler à l'UO.
    2.  Utiliser **Git Tools MCP**:
        *   `get_current_branch`.
        *   `create_branch {branchName: feature/US{{usId}}-{{taskShortNameOrId}}}` (si pas déjà sur une branche appropriée pour l'US).
        *   `checkout_branch {branchName}`.
    3.  Mettre à jour la tâche identifiée dans `.pheromone.activeUserStory.tasks` et `memoryBank.tasks.[ID_Tache]` :
        *   `status`: "InProgress"
        *   `assignee`: `currentUser.id` (ou `currentUser.azureDevOpsUsername`)
        *   `startTime`: `{{timestamp}}`
*   **Memory Bank Interaction:**
    *   Écriture (via Scribe): Mettre à jour le statut, l'assigné, et l'heure de début de la tâche dans `memoryBank.tasks`. Ajouter une entrée dans `statusHistory` de la tâche.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Préparation pour la tâche 'Azure#{{taskId}}' ('{{taskTitle}}') de l'US 'Azure#{{usId}}' terminée. Assignée à '{{currentUser.azureDevOpsUsername}}'. Branche Git '{{branchName}}' est active/créée. Prêt à commencer le développement."

---