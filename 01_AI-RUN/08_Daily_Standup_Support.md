# Workflow: Support au Daily Stand-up Meeting (08_Daily_Standup_Support.md)

**Objectif:** Fournir à l'équipe Agile un résumé concis de l'état d'avancement du sprint actuel pour faciliter le Daily Stand-up. Le système collecte les mises à jour des tâches depuis Azure DevOps et `.pheromone`, identifie les potentiels bloquants, et génère un rapport.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@scrum-facilitator-agent`.

**MCPs Utilisés:** Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Ce workflow peut être déclenché manuellement par le Scrum Master (ex: `"AgilePheromind prépare résumé Daily pour Sprint {{currentSprint.name}}"`) ou de manière planifiée.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Collecte des Mises à Jour du Sprint Actif.**
        *   UO délègue à `@devops-connector` pour récupérer les statuts récents des US et tâches du sprint actuel depuis Azure DevOps.
        *   UO délègue à `@scrum-facilitator-agent` pour analyser `.pheromone.memoryBank` (commits, notes, statuts Pheromind des tâches).
    *   **Phase 2: Identification des Progrès et des Bloquants Potentiels.**
        *   UO délègue à `@scrum-facilitator-agent` pour consolider les informations et identifier les points d'attention.
    *   **Phase 3: Génération du Résumé pour le Daily Stand-up.**
        *   UO délègue à `@scrum-facilitator-agent` pour formater le rapport.
    *   **Phase 4: Enregistrement et Diffusion (Optionnelle).**
        *   Scribe enregistre le rapport dans `documentationRegistry`.
        *   (Optionnel) UO pourrait utiliser un MCP de notification pour diffuser le résumé.

## Détails des Phases:

### Phase 1: Collecte des Mises à Jour du Sprint Actif
*   **Agent Responsable:** `@devops-connector` (pour Azure DevOps) et `@scrum-facilitator-agent` (pour `.pheromone`).
*   **Inputs:** `currentSprint.id` et `currentSprint.userStories` (liste d'IDs) depuis `.pheromone`.
*   **Actions & Tooling (`@devops-connector`):**
    1.  Utiliser **Azure DevOps MCP**:
        *   Pour chaque `usId` dans `currentSprint.userStories`:
            *   `get_work_item_details {id: usId}` (pour l'état de l'US).
            *   `get_child_work_items {parentId: usId}` (pour obtenir les IDs de toutes les tâches liées).
        *   Pour chaque `taskId` enfant récupéré:
            *   `get_work_item_details {id: taskId}` (pour l'état, l'assigné, les heures restantes/complétées si tracées dans Azure DevOps).
*   **Output (`@devops-connector` vers Scribe):** Résumé NL: "Mises à jour Azure DevOps pour Sprint '{{currentSprint.name}}' récupérées. [N_US] US, [N_Tasks] tâches avec leurs statuts et assignés actuels. Détails loggés: `sprint_{{currentSprint.id}}_ado_update_{{timestamp}}.json`." (Log dans `03_SPECS/AzureDevOps_Logs/`).
*   **Actions & Tooling (`@scrum-facilitator-agent`):**
    1.  Lire `.pheromone.memoryBank`:
        *   Pour chaque tâche du sprint actuel (`memoryBank.tasks` filtré par `sprintAssignment` ou US parentes):
            *   Analyser `statusHistory`, `developerNotes`, `relatedCommits` depuis la dernière synchronisation ou le dernier Daily.
*   **Memory Bank Interaction (via Scribe après résumé de `@devops-connector`):**
    *   Scribe met à jour `memoryBank.userStories` et `memoryBank.tasks` avec les `azureStatus` et `azureAssignee` frais d'Azure DevOps.
*   **Output (`@scrum-facilitator-agent` interne):** Données consolidées de `.pheromone` sur l'activité récente.

### Phase 2: Identification des Progrès et des Bloquants Potentiels
*   **Agent Responsable:** `@scrum-facilitator-agent`
*   **Inputs:** Données d'Azure DevOps (via Scribe dans `memoryBank`) et analyse de `.pheromone` (Phase 1). `memoryBank.riskRegister`.
*   **Actions & Tooling:**
    1.  **Analyse des Progrès:**
        *   Identifier les tâches passées à "Done" (ou équivalent) depuis le dernier Daily.
        *   Noter les US où toutes les tâches sont "Done".
        *   Identifier les tâches qui ont eu des commits récents.
    2.  **Analyse des Bloquants/Points d'Attention:**
        *   Tâches "InProgress" sans activité récente (pas de commits, pas de notes dans `developerNotes` depuis > X temps).
        *   Tâches dont l'estimation est dépassée (si heures tracées).
        *   Commentaires dans `developerNotes` mentionnant explicitement des "bloquants", "problèmes", "attente".
        *   Dépendances entre tâches du sprint où une tâche bloquante n'avance pas.
        *   Comparer avec le `memoryBank.riskRegister` pour voir si des risques identifiés se matérialisent.
    3.  (Optionnel) Croiser avec les `currentUser.currentContext` pour voir qui travaille activement sur quoi selon Pheromind.
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.tasks`, `memoryBank.userStories`, `memoryBank.commits`, `memoryBank.riskRegister`.
*   **Output (interne à `@scrum-facilitator-agent`):** Listes structurées des progrès, des tâches "à risque" ou bloquées.

### Phase 3: Génération du Résumé pour le Daily Stand-up
*   **Agent Responsable:** `@scrum-facilitator-agent`
*   **Inputs:** Progrès et bloquants identifiés (Phase 2). `currentSprint` depuis `.pheromone`.
*   **Actions & Tooling:**
    1.  Formater un rapport Markdown (`daily_standup_summary_{{currentSprint.name}}_{{date}}.md`) dans `03_SPECS/Daily_Summaries/`.
    2.  Le rapport doit contenir des sections claires :
        *   **Objectif du Sprint:** Rappel de `currentSprint.goal`.
        *   **Progrès d'Hier (Tâches Terminées):**
            *   Liste des tâches passées à "Done".
            *   Par qui (si disponible).
        *   **En Cours Aujourd'hui (Tâches InProgress Actives):**
            *   Liste des tâches marquées "InProgress" avec activité récente.
            *   Assigné et brève description.
        *   **Points d'Attention / Bloquants Potentiels:**
            *   Tâches sans progression notable.
            *   Bloquants explicitement mentionnés.
            *   Dépendances critiques.
            *   Risques du `riskRegister` pertinents pour le sprint.
        *   **(Optionnel) Burndown Chart simplifié** (si les données d'estimation et de travail restant sont disponibles et fiables).
*   **Memory Bank Interaction:**
    *   Aucune écriture directe, mais le rapport est basé sur son contenu.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe` et UO):** Résumé NL: "Résumé pour le Daily Stand-up du Sprint '{{currentSprint.name}}' ({date}) est prêt. [N_done] tâches terminées hier. [N_inprogress] tâches activement en cours. [N_blockers] points d'attention identifiés. Rapport détaillé : `daily_standup_summary_{{currentSprint.name}}_{{date}}.md`."

### Phase 4: Enregistrement et Diffusion (Optionnelle)
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`, et `🧐 @uber-orchestrator` pour diffusion.
*   **Inputs:** Résumé NL de `@scrum-facilitator-agent`.
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  Interpréter le résumé.
    2.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter le chemin vers `daily_standup_summary_{{currentSprint.name}}_{{date}}.md`.
        *   `memoryBank.sprintRetrospectivesSummaries` (ou une nouvelle section `dailySummaries`): Ajouter un lien vers le résumé du jour.
*   **Actions & Tooling (`🧐 @uber-orchestrator` - Optionnel):**
    1.  Si un MCP de notification est configuré (ex: email, Slack, Teams):
        *   Utiliser le MCP pour envoyer le résumé (ou un lien vers le rapport) aux membres de l'équipe (liste de diffusion à configurer).
*   **Memory Bank Interaction:**
    *   Écriture: Enregistrement du rapport.
*   **Outcome:** L'équipe dispose d'un résumé structuré et basé sur les données pour commencer son Daily Stand-up, améliorant la focalisation et la transparence.

---