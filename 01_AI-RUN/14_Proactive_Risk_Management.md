# Workflow: Gestion Proactive des Risques du Projet (14_Proactive_Risk_Management.md)

**Objectif:** Identifier, évaluer, et suivre les risques potentiels du projet de manière proactive. Ce workflow peut être déclenché périodiquement (ex: début de sprint, hebdomadairement) ou sur demande pour maintenir un registre des risques à jour et s'assurer que des plans de mitigation sont envisagés.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@risk-manager-agent`, `@devops-connector` (pour le contexte des tâches/US).

**MCPs Utilisés:** Azure DevOps MCP (pour obtenir des détails sur les éléments de travail liés aux risques).

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Manuelle:** L'utilisateur (Scrum Master/Tech Lead) demande une session de gestion des risques (ex: `"AgilePheromind lance analyse des risques du projet"`).
    *   **Automatique:** Déclenchement planifié (ex: tous les lundis matin).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Collecte de Données et Identification des Indicateurs de Risque.**
        *   UO délègue à `@risk-manager-agent`. L'agent scanne `.pheromone` (Memory Bank, état des tâches/US, historique des workflows) et potentiellement Azure DevOps pour des signaux d'alerte.
    *   **Phase 2: Évaluation et Priorisation des Risques Identifiés.**
        *   UO délègue à `@risk-manager-agent`. Attribution d'impact, probabilité, et score de risque.
    *   **Phase 3: Mise à Jour du Registre des Risques.**
        *   UO délègue à `@risk-manager-agent` pour la préparation des mises à jour.
        *   Scribe met à jour `memoryBank.riskRegister` dans `.pheromone`.
    *   **Phase 4: Proposition de Plans de Mitigation (Optionnel).**
        *   Si des risques critiques sont identifiés, UO peut demander à `@risk-manager-agent` de suggérer des actions de mitigation.
    *   **Phase 5: Rapport et Notification.**
        *   `@risk-manager-agent` génère un rapport.
        *   Scribe enregistre le rapport. L'UO notifie les parties prenantes.

## Détails des Phases:

### Phase 1: Collecte de Données et Identification des Indicateurs de Risque
*   **Agent Responsable:** `@risk-manager-agent`
*   **Inputs:** Accès complet à `.pheromone`. `currentSprint.id` et `currentProject.id` depuis `.pheromone`.
*   **Actions & Tooling:**
    1.  **Scanner `.pheromone.memoryBank`:**
        *   `memoryBank.tasks` et `memoryBank.userStories`: Rechercher les éléments avec des statuts "Blocked", "Delayed", des estimations fréquemment dépassées, des taux élevés de réouverture après "Done", de nombreux commentaires indiquant des difficultés.
        *   `memoryBank.technicalDebtItems`: Identifier les items critiques ou élevés non adressés.
        *   `memoryBank.sprintRetrospectivesSummaries`: Rechercher les impediments récurrents ou les points d'amélioration non suivis.
        *   `memoryBank.architecturalDecisions`: Noter les décisions récentes qui pourraient introduire des risques (ex: adoption d'une nouvelle technologie non maîtrisée).
        *   `memoryBank.legacyCodeAnalyses`: Si une migration est en cours, revoir les risques identifiés.
    2.  **Scanner `.pheromone.activeWorkflow.history`:**
        *   Identifier les agents ou les scripts `01_AI-RUN/` qui échouent fréquemment ou prennent beaucoup de temps.
    3.  **Consulter Azure DevOps (via `@devops-connector`):**
        *   Demander à `@devops-connector` de récupérer via **Azure DevOps MCP** (`get_work_items_with_state {projectName, state:'Impediment'}` ou `search_work_items {query:"[System.Tags] CONTAINS 'Risk'"}`) les éléments explicitement marqués comme risques ou impediments.
        *   Vérifier les dépendances entre équipes si visibles dans Azure DevOps.
    4.  **Compiler une liste d'indicateurs de risque potentiels.**
*   **Memory Bank Interaction:**
    *   Lecture extensive de multiples sections de la `memoryBank`.
*   **Output (interne à `@risk-manager-agent`):** Liste d'observations et d'indicateurs de risque.

### Phase 2: Évaluation et Priorisation des Risques Identifiés
*   **Agent Responsable:** `@risk-manager-agent`
*   **Inputs:** Liste d'indicateurs de risque (Phase 1). (Optionnel) Grille de notation des risques (peut être définie dans `memoryBank.projectContext.riskMatrix`).
*   **Actions & Tooling:**
    1.  Pour chaque indicateur, formaliser un risque :
        *   **Description du Risque:** Clair, concis.
        *   **Catégorie:** Technique, Planning/Échéancier, Ressources Humaines, Dépendances Externes, Scope/Exigences, Qualité.
        *   **Cause(s) Potentielle(s).**
        *   **Impact Potentiel sur le Projet:** (ex: Retard de livraison, Dépassement de budget, Qualité réduite, Insatisfaction client). Évaluer l'impact sur une échelle (ex: Faible, Moyen, Élevé, Critique).
        *   **Probabilité d'Occurrence:** (ex: Faible, Moyenne, Élevée, Très Élevée).
    2.  **Calculer un Score de Risque (Optionnel):** Si une matrice est définie (Impact x Probabilité).
    3.  **Prioriser les Risques:** Classer les risques du plus critique au moins critique.
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.projectContext.riskMatrix` (si existante).
*   **Output (interne à `@risk-manager-agent`):** Liste de risques formalisés, évalués et priorisés.

### Phase 3: Mise à Jour du Registre des Risques
*   **Agent Responsable:** `@risk-manager-agent` (pour la préparation), `✍️ @orchestrator-pheromone-scribe` (pour l'écriture).
*   **Inputs:** Liste des risques évalués (Phase 2). `memoryBank.riskRegister` existant.
*   **Actions & Tooling (`@risk-manager-agent`):**
    1.  Comparer les risques nouvellement identifiés/réévalués avec ceux déjà présents dans `memoryBank.riskRegister`.
    2.  Pour les nouveaux risques, préparer des objets JSON à ajouter.
    3.  Pour les risques existants, préparer des mises à jour (ex: changement de probabilité/impact, ajout de notes).
    4.  Marquer les risques résolus ou non pertinents comme "Closed" ou les archiver.
    5.  Chaque entrée de risque doit avoir au minimum : `id` (unique, ex: `RISK_UUID`), `description`, `category`, `impactLevel`, `probabilityLevel`, `riskScore` (optionnel), `status` ("Open", "Mitigating", "Monitoring", "Closed"), `dateIdentified`, `lastAssessed`, `owner` (optionnel, ex: TechLead), `mitigationPlanLink` (optionnel).
*   **Output (`@risk-manager-agent` vers Scribe):** Un ensemble de données structurées représentant les ajouts, mises à jour, et suppressions pour `memoryBank.riskRegister`. Et un résumé NL de ces changements.
*   **Actions & Tooling (Scribe):**
    1.  Interpréter les données/résumé.
    2.  Mettre à jour `memoryBank.riskRegister` dans `.pheromone`.
*   **Memory Bank Interaction:**
    *   Lecture/Écriture: `memoryBank.riskRegister`.
*   **Output (Scribe):** `.pheromone` avec `memoryBank.riskRegister` mis à jour.

### Phase 4: Proposition de Plans de Mitigation (Optionnel)
*   **Agent Responsable:** `@risk-manager-agent`
*   **Inputs:** Risques critiques/élevés identifiés dans `memoryBank.riskRegister` (après mise à jour Phase 3).
*   **Actions & Tooling:**
    1.  Pour les 2-3 risques les plus prioritaires (ou ceux spécifiés par l'UO):
        *   Utiliser **Sequential Thinking MCP** pour brainstormer des actions de mitigation potentielles.
            *   `set_goal`: "Proposer des actions de mitigation pour le risque : [Description du risque]."
            *   `add_step`: "Identifier des actions pour réduire la probabilité."
            *   `add_step`: "Identifier des actions pour réduire l'impact."
            *   `add_step`: "Identifier des plans de contingence si le risque se matérialise."
        *   Proposer des actions concrètes et assignables (ex: "Allouer du temps pour refactoriser le module X", "Former l'équipe sur la technologie Y", "Mettre en place une surveillance accrue pour Z").
    2.  Les propositions de mitigation peuvent être ajoutées aux descriptions des risques dans le rapport ou suggérées comme de nouvelles tâches techniques.
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.riskRegister`.
    *   Les plans de mitigation seront dans le rapport, et si des tâches sont créées, elles iront dans `memoryBank.tasks`.
*   **Output (interne ou vers `@code-reviewer-assistant` pour le rapport):** Suggestions de plans de mitigation.

### Phase 5: Rapport et Notification
*   **Agent Responsable:** `@risk-manager-agent` (pour le rapport), `✍️ @orchestrator-pheromone-scribe` (pour l'enregistrement), `🧐 @uber-orchestrator` (pour la notification).
*   **Inputs:** Registre des risques mis à jour, propositions de mitigation (si Phase 4 exécutée).
*   **Actions & Tooling (`@risk-manager-agent`):**
    1.  Générer un rapport Markdown (`risk_assessment_report_[timestamp].md`) dans `03_SPECS/Risk_Management/`.
    2.  Le rapport doit inclure :
        *   Date de l'analyse.
        *   Résumé des risques les plus critiques.
        *   Le registre des risques complet (ou les changements significatifs).
        *   Les plans de mitigation proposés (si applicable).
*   **Output (`@risk-manager-agent` vers Scribe):** Résumé NL: "Analyse proactive des risques terminée. [N_total_open] risques ouverts, dont [N_high_critical] critiques/élevés. [N_new] nouveaux risques identifiés. [N_mitigations] plans de mitigation proposés. Rapport détaillé : `risk_assessment_report_[timestamp].md`."
*   **Actions & Tooling (Scribe):**
    1.  Enregistrer le rapport dans `documentationRegistry`.
    2.  Assurer que `memoryBank.riskRegister` est à jour.
*   **Actions & Tooling (UO):**
    1.  Notifier les parties prenantes (Scrum Master, PO, Tech Lead) de la disponibilité du rapport. Peut utiliser `ask_followup_question` pour demander une revue ou une action sur les risques critiques.
*   **Memory Bank Interaction:**
    *   Écriture (Scribe): Enregistrement du rapport.
*   **Outcome:** Le registre des risques du projet est à jour, les risques majeurs sont identifiés, et des plans de mitigation peuvent être discutés et mis en œuvre par l'équipe.

---