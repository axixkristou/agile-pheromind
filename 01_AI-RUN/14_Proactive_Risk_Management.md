# Workflow: Gestion Proactive des Risques du Projet (14_Proactive_Risk_Management.md)

**Objectif:** Identifier, évaluer, et suivre les risques potentiels du projet de manière proactive et traçable. Ce workflow utilise l'analyse de `.pheromone` et des données d'Azure DevOps pour maintenir un registre des risques à jour, documenter la "chaîne de pensée" pour l'évaluation des risques, et proposer des mitigations. Il intègre la gestion des erreurs et des clarifications si besoin.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@risk-manager-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Utilisés:** Azure DevOps MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Manuelle (Scrum Master/Tech Lead: `"AgilePheromind analyse risques projet"`) ou automatique (planifiée).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Collecte de Données et Identification des Indicateurs de Risque.**
        *   UO **injecte contexte** à `@risk-manager-agent`: `currentSprint.id`, `currentProject.id`, `memoryBank.projectContext.riskMatrix` (si définie), historique des risques passés.
        *   `@risk-manager-agent` scanne `.pheromone` (Memory Bank, état tâches/US, historique workflows).
        *   `@risk-manager-agent` demande à `@devops-connector` de récupérer les items "Risk"/"Impediment" d'Azure DevOps (via **ADO MCP**).
        *   **onError (ADO MCP):** Si échec, notifier l'utilisateur, l'analyse se basera sur `.pheromone` uniquement avec un avertissement.
    *   **Phase 2: Évaluation et Priorisation des Risques (avec "Chaîne de Pensée").**
        *   UO délègue à `@risk-manager-agent`. Attribution d'impact, probabilité. Utilisation de **Sequential Thinking MCP** pour structurer l'évaluation. **Doit documenter la "chaîne de pensée"** pour l'évaluation de chaque risque majeur.
        *   Si un indicateur est ambigu, `@risk-manager-agent` peut le signaler à l'UO pour clarification via `@clarification-agent` (ex: demander plus de détails sur un impediment).
    *   **Phase 3: Mise à Jour du Registre des Risques (`memoryBank.riskRegister`).**
        *   UO délègue à `@risk-manager-agent` pour préparer les mises à jour.
        *   Scribe met à jour `.pheromone`.
    *   **Phase 4: Proposition de Plans de Mitigation (avec "Chaîne de Pensée", Optionnel).**
        *   Si risques critiques, UO demande à `@risk-manager-agent` de suggérer des mitigations (via **Sequential Thinking MCP**), en **documentant la "chaîne de pensée"**.
    *   **Phase 5: Rapport et Notification.**
        *   `@risk-manager-agent` génère rapport (incluant "chaînes de pensée").
        *   Scribe enregistre. UO notifie.

## Détails des Phases:

### Phase 1: Collecte de Données et Identification des Indicateurs de Risque
*   **Agent Responsable:** `@risk-manager-agent` (coordonne avec `@devops-connector`).
*   **Inputs (Injectés par l'UO):** `currentSprint.id`, `currentProject.id`. Contexte de `memoryBank` (matrice de risques, historique des risques).
*   **Actions (`@risk-manager-agent`):**
    1.  **Scanner `.pheromone.memoryBank`:** `tasks` (Blocked, Delayed, estimations dépassées), `technicalDebtItems` (critiques), `sprintRetrospectivesSummaries` (impediments récurrents), `architecturalDecisions` (risquées).
    2.  **Scanner `.pheromone.activeWorkflow.history`:** Agents/scripts échouant fréquemment.
    3.  **Demander à `@devops-connector` pour Azure DevOps:**
        *   `get_work_items_with_state {projectName, state:'Impediment'}` ou `search_work_items {query:"[System.Tags] CONTAINS 'Risk' OR [System.WorkItemType] = 'Risk'"}`.
*   **onError (ADO MCP via `@devops-connector`):**
    *   `@devops-connector` signale l'échec à `@risk-manager-agent`.
    *   `@risk-manager-agent` inclut un avertissement dans son résumé final: "La synchronisation avec Azure DevOps pour les risques/impediments a échoué. L'analyse est basée sur les données Pheromind uniquement."
*   **Output (interne à `@risk-manager-agent`):** Liste d'observations et indicateurs de risque.

### Phase 2: Évaluation et Priorisation des Risques (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@risk-manager-agent`, UO, `@clarification-agent`.
*   **Inputs:** Indicateurs de risque (Phase 1). `memoryBank.projectContext.riskMatrix` (injectée par UO).
*   **Actions (`@risk-manager-agent`):**
    1.  Utiliser **Sequential Thinking MCP** pour chaque indicateur majeur afin de formaliser le risque :
        *   `set_goal`: "Évaluer l'indicateur de risque : [Indicateur]."
        *   `add_step`: "Décrire le risque (si l'indicateur se confirme)."
        *   `add_step`: "Identifier la catégorie (Technique, Planning, etc.)."
        *   `add_step`: "Analyser les causes potentielles."
        *   `add_step`: "Évaluer l'impact potentiel (Faible, Moyen, Élevé, Critique) sur le projet, en justifiant."
        *   `add_step`: "Évaluer la probabilité d'occurrence (Faible, Moyenne, Élevée, Très Élevée), en justifiant."
        *   `add_step`: "Calculer le score de risque (si matrice fournie)."
        *   `run_sequence`. **Conserver la sortie de ce MCP comme "chaîne de pensée" pour ce risque.**
    2.  Prioriser les risques.
    3.  **Gestion d'Ambiguïté:** Si l'évaluation d'un indicateur est bloquée par manque d'information :
        *   Signaler à l'UO: "Impossible d'évaluer pleinement l'indicateur '[Indicateur]' car [information manquante]. Suggestion de question pour [PO/TechLead/Dev]: '[Question précise]'".
        *   L'UO peut initier clarification via `@clarification-agent`. L'évaluation de cet indicateur est mise en pause.
*   **Output (interne):** Liste de risques formalisés, évalués (avec "chaîne de pensée" pour chacun), priorisés.

### Phase 3: Mise à Jour du Registre des Risques (`memoryBank.riskRegister`)
*   **Agent Responsable:** `@risk-manager-agent` (préparation), Scribe (écriture).
*   **Inputs:** Risques évalués (Phase 2). `memoryBank.riskRegister` existant (injecté par UO).
*   **Actions (`@risk-manager-agent`):**
    1.  Comparer, préparer ajouts/mises à jour pour `memoryBank.riskRegister`.
    2.  Chaque entrée: `id`, `description`, `category`, `impactLevel`, `probabilityLevel`, `riskScore`, `status`, `dateIdentified`, `lastAssessed`, `owner`, `mitigationPlanLink`, `reasoningChainSummary` (résumé de la chaîne de pensée de l'évaluation, ou lien vers la section du rapport).
*   **Output (`@risk-manager-agent` -> Scribe):** Données structurées pour `memoryBank.riskRegister`. Résumé NL des changements.
*   **Actions (Scribe):** Mettre à jour `memoryBank.riskRegister` dans `.pheromone`.

### Phase 4: Proposition de Plans de Mitigation (avec "Chaîne de Pensée", Optionnel)
*   **Agent Responsable:** `@risk-manager-agent`.
*   **Inputs:** Risques critiques/élevés de `memoryBank.riskRegister` (injectés par UO).
*   **Actions:**
    1.  Pour les risques ciblés, utiliser **Sequential Thinking MCP** pour brainstormer mitigations (réduire probabilité, réduire impact, plans de contingence).
    2.  **"Chaîne de Pensée":** Documenter le raisonnement derrière chaque plan de mitigation proposé.
*   **Output (interne, pour rapport Phase 5):** Suggestions de plans de mitigation avec justifications.

### Phase 5: Rapport et Notification
*   **Agent Responsable:** `@risk-manager-agent` (rapport), Scribe (enregistrement), UO (notification).
*   **Inputs:** Registre des risques à jour, propositions de mitigation.
*   **Actions (`@risk-manager-agent`):**
    1.  Générer rapport MD (`risk_assessment_report_[timestamp].md`) dans `03_SPECS/Risk_Management/`. Inclure: Résumé risques critiques, registre des risques (ou changements), **détails de la "chaîne de pensée" pour l'évaluation des risques majeurs**, plans de mitigation (avec leur "chaîne de pensée"). Mentionner les échecs de collecte de données (ex: ADO).
*   **Output (`@risk-manager-agent` -> Scribe):** Résumé NL: "Analyse risques terminée. [N_open] risques ouverts ([N_crit] critiques). [N_new] identifiés. [N_mitigations] proposées. Rapport (avec chaînes de pensée): `risk_assessment_report_[timestamp].md`."
*   **Actions (Scribe):** Enregistrer rapport dans `documentationRegistry`. Mettre à jour `memoryBank.riskRegister` (si des liens vers le rapport doivent être ajoutés aux items de risque pour leur `reasoningChainLink`).
*   **Actions (UO):** Notifier parties prenantes. Utiliser `ask_followup_question` pour actions sur risques critiques.
*   **Output:** `.pheromone` à jour. Parties prenantes informées.

---