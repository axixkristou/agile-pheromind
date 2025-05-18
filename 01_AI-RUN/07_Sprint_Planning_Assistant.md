# Workflow: Assistance à la Planification de Sprint (07_Sprint_Planning_Assistant.md)

**Objectif:** Aider le Product Owner (PO) et l'équipe de développement à planifier un sprint. Le système récupère les User Stories (US) candidates, s'assure qu'elles sont correctement décomposées et estimées (en injectant du contexte et en demandant des clarifications si besoin), et propose un plan de sprint en fonction de la capacité de l'équipe, des priorités, et des dépendances identifiées. La "chaîne de pensée" pour la sélection est documentée.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@task-breakdown-estimator`, `@scrum-facilitator-agent`, `@clarification-agent`.

**MCPs Utilisés:** Azure DevOps MCP, Sequential Thinking MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (PO/Scrum Master) fournit une liste d'IDs d'US candidates et la capacité de l'équipe (ex: `"AgilePheromind planifie sprint. US: Azure#123, Azure#456. Capacité: 40 points."`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Récupération et Validation Initiale des Détails des US Candidates.**
        *   UO délègue à `@devops-connector` pour obtenir les infos de chaque US depuis Azure DevOps.
        *   UO évalue la clarté des US récupérées. Si une US candidate est trop vague pour une estimation fiable, UO peut engager `@clarification-agent` pour demander des précisions au PO.
        *   **onError:** Si ADO MCP échoue, logguer, notifier, et potentiellement arrêter ou continuer avec les US pour lesquelles les infos ont été récupérées.
    *   **Phase 2: Vérification/Finalisation des Estimations et Décomposition en Tâches.**
        *   UO, pour chaque US candidate (clarifiée si besoin):
            *   **Injecte un contexte ciblé** (infos de `memoryBank` sur estimations passées, complexité de modules similaires, conventions techniques) à `@task-breakdown-estimator`.
            *   `@task-breakdown-estimator` vérifie/effectue l'estimation et la décomposition. Doit **détailler sa "chaîne de pensée"**.
            *   Synchronise avec Azure DevOps via `@devops-connector`.
        *   **onError:** Si estimation impossible ou MCP échoue, l'US peut être exclue du scope de planification avec une note, ou une clarification demandée.
    *   **Phase 3: Proposition du Plan de Sprint avec Analyse de Dépendances/Risques.**
        *   UO **injecte contexte** (liste des US estimées, capacité équipe, priorités, `memoryBank.riskRegister`) à `@scrum-facilitator-agent`.
        *   `@scrum-facilitator-agent` sélectionne les US/tâches. Utilise **Sequential Thinking MCP** pour analyser les dépendances et les risques du plan. **Détaille sa "chaîne de pensée"** pour la sélection.
    *   **Phase 4: Enregistrement du Plan et Rapport.**
        *   Scribe enregistre le plan proposé et le rapport (incluant la chaîne de pensée) dans `.pheromone`.

## Détails des Phases:

### Phase 1: Récupération et Validation Initiale des Détails des US Candidates
*   **Agent Responsable:** `@devops-connector`, UO, `@clarification-agent`.
*   **Inputs:** Liste IDs US candidates, `currentUser`.
*   **Actions (`@devops-connector`):** Pour chaque ID d'US, **Azure DevOps MCP** `get_work_item_details` (titre, desc, priorité, état, estimation ADO).
*   **onError (ADO MCP):** UO loggue via Scribe, notifie utilisateur, peut décider d'arrêter ou de continuer avec les US récupérées.
*   **Output (`@devops-connector` -> Scribe):** Résumé NL: "Détails pour [N] US candidates récupérés: [Liste IDs/Titres]. Log: `sprint_planning_us_fetch_{{timestamp}}.json`." Scribe met à jour `memoryBank.userStories`.
*   **Actions (UO):** Pour chaque US récupérée, évaluer la clarté de la description et des ACs (si présents).
    *   **Si ambiguïté majeure** empêchant l'estimation:
        *   UO met workflow en pause (`activeWorkflow.status: 'PendingClarification_SprintPlanUS'`).
        *   UO délègue à `@clarification-agent` avec l'ID de l'US, le texte ambigu, et une question pour le PO (ex: "L'US Azure#{{usId}} '[Titre]' a une description vague concernant [aspect]. Pour l'estimer, pouvez-vous préciser [question spécifique] ?").
        *   Attendre réponse via `01_AI-RUN/XX_Handle_Clarification_Response.md`.

### Phase 2: Vérification/Finalisation des Estimations et Décomposition en Tâches
*   **Agent Responsable:** `@task-breakdown-estimator` (coordonne avec `@devops-connector`).
*   **Inputs (Injectés par l'UO pour chaque US):**
    *   Détails de l'US (clarifiée si besoin).
    *   Contexte `memoryBank`: `projectContext` (stack, estimationUnit), estimations/décompositions d'US similaires passées, `technicalDebtItems` ou `architecturalDecisions` pouvant impacter l'effort.
*   **Actions (`@task-breakdown-estimator`):**
    1.  Vérifier si `memoryBank.userStories.{{usId}}` a une estimation fiable et une décomposition en tâches à jour.
    2.  Si non, ou si révision demandée:
        *   Engager processus de décomposition/estimation (comme dans `01_Start_User_Story.md` Phase 3), utilisant **Sequential Thinking MCP**, **Context7 MCP**, **MSSQL MCP**.
        *   **Documenter la "Chaîne de Pensée"** dans son rapport : expliquer la logique de décomposition, les hypothèses pour l'estimation, l'impact des infos contextuelles injectées.
    3.  Synchroniser tâches/estimations avec ADO via `@devops-connector` (**Azure DevOps MCP**).
*   **onError (Estimation/Décomposition):** Si l'agent ne peut estimer (même après clarification), il le signale à l'UO. L'UO peut exclure l'US de ce cycle de planification, en notant la raison, ou demander une nouvelle clarification. Si MCP échoue, gestion d'erreur similaire à Phase 1.
*   **Output (`@task-breakdown-estimator` -> Scribe):** Résumé NL: "Estimations/décompositions finalisées pour US candidates. [N] US traitées. Rapports individuels (avec chaîne de pensée): `us_{{usId}}_task_breakdown_{{timestamp}}.md`." Scribe met à jour `memoryBank.userStories` et `memoryBank.tasks`.

### Phase 3: Proposition du Plan de Sprint avec Analyse de Dépendances/Risques
*   **Agent Responsable:** `@scrum-facilitator-agent`.
*   **Inputs (Injectés par l'UO):**
    *   Liste des US candidates avec estimations finales (depuis `memoryBank.userStories`).
    *   Capacité de l'équipe pour le sprint.
    *   Priorités des US.
    *   Contenu de `memoryBank.riskRegister`.
*   **Actions (`@scrum-facilitator-agent`):**
    1.  Trier US par priorité.
    2.  Sélectionner itérativement les US jusqu'à atteindre la capacité.
    3.  **Analyse de Dépendances/Risques (Sequential Thinking MCP):**
        *   `set_goal`: "Analyser la faisabilité et les risques du plan de sprint proposé."
        *   `add_step`: "Pour les US sélectionnées, identifier les dépendances internes (entre tâches de l'US) et externes (autres US, équipes - si info dispo dans `memoryBank.userStories.{{usId}}.dependencies`)."
        *   `add_step`: "Vérifier si des risques du `memoryBank.riskRegister` sont directement liés aux US sélectionnées."
        *   `add_step`: "Évaluer si des US hautement prioritaires ont été exclues et pourquoi (capacité, dépendances)."
        *   `run_sequence`.
    4.  **Documenter la "Chaîne de Pensée":** Expliquer la logique de sélection et les conclusions de l'analyse de risques/dépendances dans le rapport final.
*   **Output (`@scrum-facilitator-agent` -> Scribe):** Résumé NL: "Proposition plan Sprint [ID/Nom à définir]: [Liste IDs US]. Total [Points]/[Capacité] {{estimationUnit}}. Risques/Dépendances: [Résumé]. Rapport (avec chaîne de pensée): `sprint_plan_proposal_{{timestamp}}.md`." (Rapport dans `02_AI-DOCS/Sprint_Plans/`).

### Phase 4: Enregistrement du Plan et Rapport
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** Résumé NL de `@scrum-facilitator-agent`.
*   **Actions:**
    1.  Mettre à jour `.pheromone`:
        *   `currentSprint`: infos du plan (IDs US, points planifiés, capacité). Le nom/ID et l'objectif du sprint peuvent être demandés au PO/SM par l'UO via `ask_followup_question` avant cette mise à jour.
        *   `documentationRegistry`: Ajouter chemin vers `sprint_plan_proposal_{{timestamp}}.md`.
        *   `memoryBank.userStories.{{usId}}.sprintAssignment = currentSprint.id` pour les US incluses.
        *   `memoryBank.sprints.{{currentSprint.id}}.reasoningChainLink.planning`: Lier au rapport.
*   **Output:** `.pheromone` mis à jour. UO présente le plan à l'équipe.

---