# Workflow: Assistance à la Planification de Sprint (07_Sprint_Planning_Assistant.md)

**Objectif:** Aider le Product Owner (PO) et l'équipe de développement à planifier un sprint. Le système récupère les User Stories (US) candidates, aide à leur estimation (si nécessaire), et propose un plan de sprint basé sur la capacité de l'équipe et les priorités.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@task-breakdown-estimator`, `@scrum-facilitator-agent`.

**MCPs Utilisés:** Azure DevOps MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (PO/Scrum Master) fournit une liste d'IDs d'US candidates pour le sprint et la capacité de l'équipe (ex: `"AgilePheromind planifie sprint. US candidates: Azure#123, Azure#456, Azure#789. Capacité: 40 points."`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Récupération des Détails des US Candidates.**
        *   UO délègue à `@devops-connector` pour obtenir les informations de chaque US candidate depuis Azure DevOps.
    *   **Phase 2: Vérification des Estimations et Décomposition (si nécessaire).**
        *   UO délègue à `@task-breakdown-estimator`. Pour chaque US, l'agent vérifie si une estimation existe et si une décomposition en tâches est présente et à jour (dans `.pheromone.memoryBank` ou Azure DevOps). Si non, il procède à l'estimation/décomposition.
    *   **Phase 3: Proposition du Plan de Sprint.**
        *   UO délègue à `@scrum-facilitator-agent` pour sélectionner les US/tâches en fonction de la capacité, des priorités et des dépendances.
    *   **Phase 4: Enregistrement du Plan et Rapport.**
        *   Scribe enregistre le plan proposé dans `.pheromone` et `documentationRegistry`.

## Détails des Phases:

### Phase 1: Récupération des Détails des US Candidates
*   **Agent Responsable:** `@devops-connector`
*   **Inputs:** Liste des IDs d'US candidates fournie par l'UO.
*   **Actions & Tooling:**
    1.  Pour chaque ID d'US candidate:
        *   Utiliser **Azure DevOps MCP** (`get_work_item_details {id: ID_US_Candidate}`): Récupérer titre, description, priorité actuelle dans Azure DevOps, état, et estimation existante (si champ d'estimation est utilisé dans Azure DevOps).
*   **Memory Bank Interaction (via Scribe après résumé):**
    *   Le Scribe mettra à jour/créera des entrées pour ces US dans `memoryBank.userStories` avec les informations récupérées d'Azure DevOps.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe` et UO):** Résumé NL: "Détails récupérés pour [Nombre] US candidates: [Liste des IDs et Titres]. Informations enregistrées pour analyse d'estimation." Pour chaque US, un log `azure_wi_{{usId}}_{{timestamp}}.json` peut être créé.

### Phase 2: Vérification des Estimations et Décomposition (si nécessaire)
*   **Agent Responsable:** `@task-breakdown-estimator`
*   **Inputs:** Liste des US candidates (avec leurs détails chargés dans `.pheromone.memoryBank.userStories` par le Scribe). `memoryBank.projectContext.estimationUnit`.
*   **Actions & Tooling:**
    1.  Pour chaque US candidate (lue depuis `memoryBank.userStories`):
        *   **Vérifier Estimation Existante:** Contrôler si `memoryBank.userStories.{{usId}}.estimationPoints` (ou un champ équivalent) est déjà renseigné et considéré comme fiable/récent.
        *   **Vérifier Décomposition en Tâches:** Contrôler si `memoryBank.userStories.{{usId}}.tasks` contient une liste de tâches techniques avec leurs propres estimations.
        *   **Si Estimation ou Décomposition Manquante/Obsolète:**
            *   Engager le processus de décomposition et d'estimation comme décrit dans `01_AI-RUN/01_Start_User_Story.md` (Phase 3), utilisant **Sequential Thinking MCP**, **Context7 MCP**, **MSSQL MCP** si nécessaire.
            *   Consulter `@devops-connector` pour synchroniser les nouvelles tâches/estimations avec Azure DevOps (via **Azure DevOps MCP** `create_work_item` / `update_work_item`).
            *   Assurer que chaque tâche a une estimation. L'estimation de l'US sera la somme des estimations de ses tâches.
    2.  Compiler une liste des US candidates avec leurs estimations finales (en points ou heures).
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.userStories` pour les détails des US. `memoryBank.projectContext.estimationUnit`.
    *   Écriture (via Scribe): Mettre à jour `memoryBank.userStories.{{usId}}` avec `estimationPoints`, `tasks` (liste d'IDs de tâches), et `memoryBank.tasks` avec les détails et estimations de chaque tâche.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe` et UO):** Résumé NL: "Estimations et décompositions vérifiées/effectuées pour toutes les US candidates. [Nombre] US ont été (ré)estimées. Détails mis à jour dans la Memory Bank et synchronisés avec Azure DevOps. Prêt pour la sélection du sprint."

### Phase 3: Proposition du Plan de Sprint
*   **Agent Responsable:** `@scrum-facilitator-agent`
*   **Inputs:** Liste des US candidates avec leurs estimations (depuis `.pheromone.memoryBank.userStories`). Capacité de l'équipe pour le sprint (fournie par l'UO). Priorités des US (depuis `memoryBank.userStories.{{usId}}.priority` ou Azure DevOps).
*   **Actions & Tooling:**
    1.  **Trier les US:** Ordonner les US candidates par priorité (la plus haute en premier).
    2.  **Sélection Itérative:**
        *   Initialiser `pointsChargesDuSprint = 0`.
        *   Pour chaque US triée :
            *   Si `pointsChargesDuSprint + us.estimationPoints <= capaciteEquipe`:
                *   Ajouter l'US au plan de sprint proposé.
                *   `pointsChargesDuSprint += us.estimationPoints`.
            *   Sinon, passer à l'US suivante (ou arrêter si une option "ne pas découper les US" est active).
    3.  **Identifier Dépendances et Risques:**
        *   Pour les US sélectionnées, vérifier `memoryBank.userStories.{{usId}}.dependencies` (si cette information est tracée).
        *   Signaler si des US sélectionnées ont des dépendances non satisfaites ou si des US de haute priorité n'ont pas pu être incluses.
        *   Utiliser **Sequential Thinking MCP** pour analyser les risques potentiels du plan proposé (ex: trop d'US dépendantes d'un seul développeur, estimations trop optimistes sur des US complexes).
    4.  Formuler le plan de sprint proposé (liste d'IDs d'US, total de points).
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.userStories` (estimations, priorités, dépendances). `memoryBank.riskRegister` (pour voir si des risques existants impactent les US).
*   **Output (vers `✍️ @orchestrator-pheromone-scribe` et UO):** Résumé NL: "Proposition de plan pour Sprint [Nom/ID Sprint à définir]: [Liste des IDs d'US sélectionnées]. Total estimé: [TotalPoints] / [CapaciteEquipe] {{estimationUnit}}. Risques/Dépendances notables: [Liste]. Rapport détaillé: `sprint_plan_proposal_{{timestamp}}.md`." (Rapport enregistré dans `02_AI-DOCS/Sprint_Plans/`).

### Phase 4: Enregistrement du Plan et Rapport
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@scrum-facilitator-agent`.
*   **Actions & Tooling:**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `currentSprint`: { `id` (généré ou fourni), `name` (généré ou fourni), `goal` (à définir par le PO/équipe), `userStories`: [Liste des IDs d'US du plan proposé], `plannedPoints`: TotalPoints, `capacityPoints`: CapaciteEquipe }.
        *   `documentationRegistry`: Ajouter le chemin vers `sprint_plan_proposal_{{timestamp}}.md`.
        *   Pour chaque US incluse dans le sprint, mettre à jour `memoryBank.userStories.{{usId}}.sprintAssignment = currentSprint.id`.
*   **Memory Bank Interaction:**
    *   Écriture: Stockage des informations du sprint planifié, mise à jour des US assignées.
*   **Output:** `.pheromone` mis à jour. L'UO peut ensuite présenter ce plan à l'équipe pour validation finale et engagement. L'UO peut utiliser `ask_followup_question` pour demander au PO/Scrum Master de nommer le sprint et de définir son objectif.

---