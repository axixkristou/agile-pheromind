# Workflow: Assistance au Product Owner - Analyse d'un Besoin Client (03_PO_Analyze_Need.md)

**Objectif:** Aider le Product Owner (PO) à analyser un besoin client exprimé en langage naturel. Le système doit décomposer le besoin, proposer des User Stories (US) potentielles avec des Critères d'Acceptation (ACs) initiaux, vérifier si des US similaires existent déjà dans le backlog Azure DevOps, et enregistrer cette analyse dans la Memory Bank pour référence future.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@po-assistant`, `@devops-connector`.

**MCPs Utilisés:** Azure DevOps MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Le PO soumet une description du besoin client à AgilePheromind (ex: `"AgilePheromind analyse besoin : 'Nos utilisateurs se plaignent que le processus d'inscription est trop long...'"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Analyse Structurée du Besoin Client.**
        *   UO délègue à `@po-assistant`, qui utilisera **Sequential Thinking MCP** pour une décomposition méthodique.
    *   **Phase 2: Génération de Propositions d'User Stories et Critères d'Acceptation.**
        *   UO délègue à `@po-assistant`.
    *   **Phase 3: Recherche d'User Stories Existantes dans le Backlog.**
        *   UO délègue à `@po-assistant`, qui interrogera `@devops-connector` (utilisant **Azure DevOps MCP**).
    *   **Phase 4: Compilation du Rapport d'Analyse et Interaction avec le PO.**
        *   `@po-assistant` génère un rapport.
        *   Scribe enregistre le rapport et les US brouillons dans `.pheromone`.
        *   UO présente un résumé au PO et propose des actions de suivi (ex: créer les US dans Azure DevOps).

## Détails des Phases:

### Phase 1: Analyse Structurée du Besoin Client
*   **Agent Responsable:** `@po-assistant`
*   **Inputs:** Description du besoin client fournie par l'UO (provenant de l'input du PO). Contexte du projet (`.pheromone.currentProject`, `memoryBank.projectContext`).
*   **Actions & Tooling:**
    1.  Utiliser **Sequential Thinking MCP** pour analyser le besoin client:
        *   `set_goal`: "Comprendre en profondeur le besoin client concernant [sujet principal du besoin]."
        *   `add_step`: "Identifier les acteurs principaux (Personas) concernés par ce besoin." -> Résultat attendu: Liste des personas.
        *   `add_step`: "Extraire les problèmes spécifiques ou 'pain points' rencontrés par ces acteurs." -> Résultat attendu: Liste des problèmes.
        *   `add_step`: "Identifier les solutions, désirs, ou suggestions explicites mentionnés par le client." -> Résultat attendu: Liste des solutions/désirs.
        *   `add_step`: "Déduire les bénéfices attendus ou les 'gains' pour les utilisateurs si le besoin est satisfait." -> Résultat attendu: Liste des bénéfices.
        *   `add_step`: "Identifier toute contrainte ou information contextuelle importante mentionnée." -> Résultat attendu: Liste des contraintes.
        *   `run_sequence`: Exécuter l'analyse.
    2.  Conserver les résultats structurés de cette analyse pour la phase suivante.
*   **Memory Bank Interaction (via Scribe après résumé final):**
    *   Le rapport final de `@po-assistant` contiendra cette analyse structurée, qui sera archivée.
*   **Output (interne à `@po-assistant` pour la Phase 2):** Une décomposition structurée du besoin client (acteurs, problèmes, solutions, bénéfices, contraintes).

### Phase 2: Génération de Propositions d'User Stories et Critères d'Acceptation
*   **Agent Responsable:** `@po-assistant`
*   **Inputs:** L'analyse structurée du besoin de la Phase 1. Connaissance des bonnes pratiques de rédaction d'US et d'ACs.
*   **Actions & Tooling:**
    1.  Pour chaque ensemble {Problème -> Solution Souhaitée -> Bénéfice Attendu} identifié en Phase 1:
        *   Formuler une ou plusieurs User Stories claires et concises au format: "En tant que `<Persona identifiée ou type d'utilisateur pertinent>`, je veux `<objectif/action lié à la solution>` afin de `<bénéfice attendu>`."
        *   S'assurer que l'US est "INVEST" (Indépendante, Négociable, Valeur, Estimable, Petite, Testable) autant que possible à ce stade.
    2.  Pour chaque User Story proposée:
        *   Rédiger 3 à 5 Critères d'Acceptation (ACs) initiaux. Utiliser un format clair, de préférence Gherkin (Given/When/Then) ou des listes à puces précises.
        *   Les ACs doivent définir les conditions de satisfaction de l'US du point de vue de l'utilisateur et être testables.
*   **Memory Bank Interaction (via Scribe après résumé final):**
    *   Les US brouillons et leurs ACs seront stockés dans la `memoryBank` (ex: `memoryBank.draftUserStories`) ou dans le rapport d'analyse.
*   **Output (interne à `@po-assistant` pour la Phase 3 et 4):** Une liste d'User Stories candidates, chacune avec ses ACs initiaux.

### Phase 3: Recherche d'User Stories Existantes dans le Backlog
*   **Agent Responsable:** `@po-assistant` (coordonnant avec `@devops-connector`)
*   **Inputs:** La liste des User Stories candidates de la Phase 2.
*   **Actions & Tooling:**
    1.  Pour chaque US candidate, `@po-assistant` identifie des mots-clés pertinents.
    2.  `@po-assistant` demande à `@devops-connector`: "Recherche dans le backlog Azure DevOps du projet `{{currentProject.name}}` des User Stories existantes avec les mots-clés suivants: [liste de mots-clés pour US1], puis [liste de mots-clés pour US2], etc."
    3.  `@devops-connector` utilise **Azure DevOps MCP**:
        *   `search_work_items {projectName: currentProject.name, queryText: "SELECT [System.Id], [System.Title] FROM WorkItems WHERE [System.WorkItemType] = 'User Story' AND ([System.Title] CONTAINS 'motclé1' OR [System.Description] CONTAINS 'motclé1' OR ...)"}` pour chaque ensemble de mots-clés.
        *   Retourne les IDs et titres des US trouvées à `@po-assistant`.
*   **Memory Bank Interaction:**
    *   Lecture: `.pheromone.currentProject.name`.
*   **Output (`@devops-connector` vers `@po-assistant`):** Pour chaque US candidate, une liste d'IDs et de titres d'US Azure DevOps potentiellement similaires ou dupliquées.

### Phase 4: Compilation du Rapport d'Analyse et Interaction avec le PO
*   **Agent Responsable:** `@po-assistant` (pour la synthèse du rapport), `✍️ @orchestrator-pheromone-scribe` (pour l'enregistrement), `🧐 @uber-orchestrator` (pour l'interaction PO).
*   **Inputs:** Résultats de l'analyse structurée (Phase 1), US candidates & ACs (Phase 2), US existantes trouvées (Phase 3).
*   **Actions & Tooling (`@po-assistant`):**
    1.  Compiler un rapport Markdown structuré (`po_need_analysis_[timestamp].md`) dans `02_AI-DOCS/PO_Analyses/` contenant:
        *   Le besoin client initial.
        *   L'analyse structurée (acteurs, problèmes, solutions, bénéfices, contraintes).
        *   Chaque US candidate avec ses ACs.
        *   Pour chaque US candidate, la liste des US Azure DevOps existantes potentiellement similaires.
        *   Une section "Recommandations" :
            *   Suggérer quelles US candidates semblent nouvelles et devraient être créées.
            *   Suggérer si certaines US candidates pourraient être fusionnées avec des US existantes ou si elles sont des doublons.
            *   Proposer des priorités initiales (High, Medium, Low) pour les nouvelles US (basées sur l'impact perçu du problème/bénéfice).
*   **Output (`@po-assistant` vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Analyse du besoin client '[résumé du besoin]' terminée. [N_us_candidates] US candidates proposées avec ACs. [N_existing_found] US existantes potentiellement pertinentes identifiées dans Azure DevOps. Rapport complet avec recommandations disponible à `po_need_analysis_[timestamp].md`. Recommandations principales: [1-2 recommandations clés]."
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  Enregistrer le rapport `po_need_analysis_[timestamp].md` dans `documentationRegistry`.
    2.  Stocker les US candidates (avec leurs ACs et liens vers les US existantes trouvées) dans `memoryBank.draftUserStories` ou une section dédiée de la `memoryBank` liée à l'analyse du besoin, avec un lien vers le rapport complet.
*   **Actions & Tooling (`🧐 @uber-orchestrator`):**
    1.  Utiliser `ask_followup_question` pour présenter le résumé des découvertes et recommandations au PO: "L'analyse du besoin '[résumé du besoin]' est terminée. J'ai identifié [N_us_candidates] nouvelles User Stories potentielles et trouvé [N_existing_found] US existantes qui pourraient être liées. Un rapport détaillé avec des recommandations est disponible. Souhaitez-vous: \n1. Visualiser le rapport détaillé ? \n2. Que je procède à la création des nouvelles US suggérées dans Azure DevOps ? \n3. Discuter d'une US spécifique ?"
*   **Memory Bank Interaction (via Scribe):**
    *   Archivage du rapport et des US brouillons. L'état des US candidates (ex: `status: 'DraftPendingPOReview'`) est mis à jour.
*   **Outcome:** Le PO reçoit une analyse complète du besoin client, des propositions d'US et d'ACs exploitables, une vérification des doublons potentiels, et des recommandations claires pour les prochaines étapes de gestion du backlog. Le tout est archivé pour référence.

---