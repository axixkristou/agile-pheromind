# Workflow: Support au Daily Stand-up Meeting (08_Daily_Standup_Support.md)

**Objectif:** Fournir à l'équipe Agile un résumé concis et pertinent de l'état d'avancement du sprint actuel pour faciliter un Daily Stand-up efficace. Le système agrège les informations d'Azure DevOps et de `.pheromone` (Memory Bank), identifie les progrès, les points d'attention et les potentiels bloquants.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@scrum-facilitator-agent`.

**MCPs Utilisés:** Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Manuelle:** Scrum Master/Tech Lead demande un résumé (ex: `"AgilePheromind prépare résumé Daily pour Sprint {{currentSprint.name}}"`).
    *   **Automatique:** Déclenchement planifié (ex: chaque matin avant le Daily).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Collecte des Mises à Jour du Sprint Actif.**
        *   UO délègue à `@devops-connector` pour les données fraîches d'Azure DevOps (statuts US/tâches, assignés).
        *   UO délègue à `@scrum-facilitator-agent` pour analyser l'activité récente dans `.pheromone.memoryBank` (commits, notes de développeurs, statuts Pheromind).
        *   **onError:** Si ADO MCP échoue, le rapport sera basé uniquement sur les données de `.pheromone`, avec un avertissement.
    *   **Phase 2: Identification des Progrès, Points d'Attention et Bloquants.**
        *   UO délègue à `@scrum-facilitator-agent` pour consolider et analyser.
    *   **Phase 3: Génération du Résumé Structuré pour le Daily.**
        *   UO délègue à `@scrum-facilitator-agent`.
    *   **Phase 4: Enregistrement du Résumé et Diffusion (Optionnelle).**
        *   Scribe enregistre le rapport. UO peut notifier l'équipe.

## Détails des Phases:

### Phase 1: Collecte des Mises à Jour du Sprint Actif
*   **Agent Responsable:** `@devops-connector` (pour ADO), `@scrum-facilitator-agent` (pour `.pheromone`).
*   **Inputs:** `currentSprint.id` et `currentSprint.userStories` depuis `.pheromone`. `memoryBank.lastAdoSyncTimestamp` pour la synchro différentielle.
*   **Actions (`@devops-connector`):**
    1.  Utiliser **Azure DevOps MCP**:
        *   Pour chaque `usId` dans `currentSprint.userStories`, récupérer `get_work_item_details` et `get_child_work_items`.
        *   Pour chaque `taskId` enfant, `get_work_item_details` (état, assigné, travail restant/effectué si tracé).
        *   (Optionnel) Filtrer les requêtes pour ne récupérer que les items modifiés depuis `memoryBank.lastAdoSyncTimestamp` si le MCP le permet.
*   **onError (ADO MCP pour `@devops-connector`):**
    *   Si échec, `@devops-connector` signale à l'UO. L'UO note que le résumé sera basé sur les dernières données Pheromind connues et inclut un avertissement dans le rapport final. Le workflow continue si possible.
*   **Output (`@devops-connector` -> Scribe):** Résumé NL: "MàJ ADO Sprint '{{currentSprint.name}}' OK. [Stats]. Log: `sprint_{{sprintId}}_ado_update_{{timestamp}}.json`." ou "Échec MàJ ADO: [Erreur]."
*   **Actions (Scribe après résumé de `@devops-connector`):**
    1.  Mettre à jour `memoryBank.userStories` et `memoryBank.tasks` avec les `azureStatus`, `azureAssignee`, etc. d'ADO.
    2.  Mettre à jour `memoryBank.lastAdoSyncTimestamp`.
*   **Actions (`@scrum-facilitator-agent`):**
    1.  Lire `.pheromone.memoryBank` pour les tâches du sprint:
        *   `statusHistory`, `developerNotes`, `relatedCommits` (depuis le dernier Daily ou les dernières 24h).
        *   `clarificationHistory` pour voir si des points ont été récemment clarifiés.
*   **Output (interne à `@scrum-facilitator-agent`):** Données consolidées sur l'activité récente.

### Phase 2: Identification des Progrès, Points d'Attention et Bloquants
*   **Agent Responsable:** `@scrum-facilitator-agent`.
*   **Inputs:** Données consolidées (Phase 1). `memoryBank.riskRegister`.
*   **Actions & Tooling:**
    1.  **Analyser Progrès:** Tâches passées à "Done" (dans ADO ou Pheromind). US complétées. Commits significatifs.
    2.  **Analyser Points d'Attention/Bloquants:**
        *   Tâches "InProgress" sans commit/note récente (> X heures/jours).
        *   Tâches dont l'estimation est bientôt/déjà dépassée.
        *   `developerNotes` mentionnant "bloquant", "problème", "attente".
        *   Dépendances entre tâches du sprint (si tracées dans `memoryBank.tasks.{{taskId}}.dependencies`) où une tâche bloquante stagne.
        *   Risques actifs du `memoryBank.riskRegister` liés aux US/tâches du sprint.
        *   Clarifications en attente (`clarificationContext.pendingClarificationId`).
*   **Output (interne à `@scrum-facilitator-agent`):** Listes structurées: Progrès, Points d'Attention.

### Phase 3: Génération du Résumé Structuré pour le Daily
*   **Agent Responsable:** `@scrum-facilitator-agent`.
*   **Inputs:** Listes de la Phase 2. `currentSprint` depuis `.pheromone`.
*   **Actions & Tooling:**
    1.  Formater un rapport Markdown (`daily_standup_summary_{{currentSprint.name}}_{{date}}.md`) dans `03_SPECS/Daily_Summaries/`.
    2.  Structure du rapport :
        *   **Sprint Goal:** `{{currentSprint.goal}}`.
        *   **Avertissement (si synchro ADO échouée):** "Note: Les données Azure DevOps n'ont pas pu être synchronisées. Ce rapport est basé sur le dernier état connu dans Pheromind."
        *   **Terminé Hier:**
            *   Tâche Azure#ID (Titre) - par [Assigné] - (Commit: Hash si dispo)
        *   **En Cours Aujourd'hui (Focus Principal):**
            *   Tâche Azure#ID (Titre) - par [Assigné] - (Dernière activité Pheromind: Note/Commit)
        *   **Points d'Attention / Bloquants Identifiés:**
            *   Tâche Azure#ID: [Raison de l'attention - ex: Pas de progression depuis X temps, Bloquant mentionné: "..."]
            *   Risque Actif: [ID Risque] - [Description] - Impacte US Azure#ID_US.
            *   Clarification en attente: ID `{{clarificationContext.pendingClarificationId}}` pour agent `{{clarificationContext.originalAgent}}`.
*   **Output (vers Scribe et UO):** Résumé NL: "Résumé Daily Sprint '{{currentSprint.name}}' ({date}) prêt. Terminé hier: [N_done]. En cours: [N_inprogress]. Points d'attention: [N_blockers]. Rapport: `daily_standup_summary_{{currentSprint.name}}_{{date}}.md`."

### Phase 4: Enregistrement du Résumé et Diffusion (Optionnelle)
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`, UO.
*   **Inputs:** Résumé NL de `@scrum-facilitator-agent`.
*   **Actions (Scribe):**
    1.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter chemin vers `daily_standup_summary...md`.
        *   `memoryBank.sprints.{{currentSprint.id}}.dailySummaryLinks[]`: Ajouter lien.
*   **Actions (UO - Optionnel):**
    1.  Si MCP de notification configuré, envoyer résumé/lien à l'équipe.
*   **Output:** `.pheromone` mis à jour. Équipe potentiellement notifiée.

---