# Workflow: Suggestion d'Optimisation du Système AgilePheromind (15_Optimize_Workflow_Suggestion.md)

**Objectif:** Analyser les métriques de performance et l'historique des opérations du système AgilePheromind pour identifier les goulots d'étranglement, les inefficacités, ou les zones d'amélioration. Le système doit proposer des optimisations concrètes pour les scripts `01_AI-RUN/*.md`, les définitions d'agents dans `.roomodes`, ou la logique d'interprétation dans `.swarmConfig`.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@workflow-optimizer-agent`, `@swarm-monitor-agent` (peut fournir des données d'entrée).

**MCPs Utilisés:** Aucun MCP externe direct pour cette analyse interne, mais l'agent peut se baser sur des données collectées par d'autres agents utilisant des MCPs (ex: temps de réponse des MCPs externes).

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   **Manuelle:** L'administrateur du système ou un Tech Lead demande une analyse d'optimisation (ex: `"AgilePheromind analyse ses propres workflows pour optimisation"`).
    *   **Automatique:** Déclenchement planifié (ex: fin de chaque sprint, mensuellement) ou après la collecte d'un certain volume de données par `@swarm-monitor-agent`.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Collecte des Données de Performance du Swarm.**
        *   UO délègue à `@workflow-optimizer-agent`. L'agent lit `.pheromone.activeWorkflow.history`, `memoryBank.workflowPerformanceMetrics`, et les rapports de `@swarm-monitor-agent`.
    *   **Phase 2: Analyse des Données et Identification des Axes d'Optimisation.**
        *   UO délègue à `@workflow-optimizer-agent`.
    *   **Phase 3: Génération de Suggestions d'Optimisation Spécifiques.**
        *   UO délègue à `@workflow-optimizer-agent`.
    *   **Phase 4: Rapport des Suggestions et Enregistrement.**
        *   `@workflow-optimizer-agent` génère un rapport.
        *   Scribe enregistre le rapport dans `documentationRegistry`.

## Détails des Phases:

### Phase 1: Collecte des Données de Performance du Swarm
*   **Agent Responsable:** `@workflow-optimizer-agent`
*   **Inputs:** Accès complet à `.pheromone`. (Optionnel) Rapports récents de `@swarm-monitor-agent` (pointés dans `documentationRegistry`).
*   **Actions & Tooling:**
    1.  **Analyser `.pheromone.activeWorkflow.history`:**
        *   Extraire les durées d'exécution pour chaque phase de chaque script `01_AI-RUN/*.md` exécuté.
        *   Identifier les agents qui sont fréquemment impliqués dans les workflows longs ou qui échouent.
        *   Noter la fréquence d'utilisation de chaque script `01_AI-RUN/*.md`.
    2.  **Analyser `memoryBank.workflowPerformanceMetrics`:**
        *   Si cette section est peuplée par le Scribe ou `@swarm-monitor-agent` avec des métriques agrégées (ex: temps moyen de complétion par type de workflow, taux d'erreur par agent), utiliser ces données.
    3.  **Consulter les Rapports de `@swarm-monitor-agent`:**
        *   Lire les derniers rapports de `swarm_health_report_*.md` (via `documentationRegistry`) pour des indicateurs de problèmes systémiques.
    4.  **Analyser l'utilisation des MCPs (indirectement):**
        *   Si les logs d'interaction MCP (via Scribe dans `memoryBank.agentActivityLog` ou logs spécifiques) indiquent des temps de réponse lents ou des échecs fréquents pour certains MCPs, cela peut être un point d'optimisation (ex: revoir la logique d'appel, ou même le MCP lui-même).
*   **Memory Bank Interaction:**
    *   Lecture: `activeWorkflow.history`, `memoryBank.workflowPerformanceMetrics`, `documentationRegistry` (pour les rapports du moniteur).
*   **Output (interne à `@workflow-optimizer-agent`):** Ensemble de données brutes et agrégées sur la performance du système Pheromind.

### Phase 2: Analyse des Données et Identification des Axes d'Optimisation
*   **Agent Responsable:** `@workflow-optimizer-agent`
*   **Inputs:** Données de performance collectées (Phase 1).
*   **Actions & Tooling:**
    1.  **Identifier les Goulots d'Étranglement:**
        *   Quels scripts `01_AI-RUN/*.md` ou quelles phases spécifiques dans ces scripts prennent le plus de temps ?
        *   Quels agents sont les plus lents ou les plus sujets aux erreurs ?
    2.  **Identifier les Inefficac ہم (Inefficiencies):**
        *   Y a-t-il des redondances d'actions entre différents agents ou workflows ?
        *   La logique d'interprétation du Scribe (`.swarmConfig`) est-elle optimale pour les types de résumés produits par les agents ? Des erreurs d'interprétation fréquentes ?
        *   Les prompts des agents (`.roomodes`) sont-ils clairs et non ambigus, ou conduisent-ils à des résultats inattendus ou à de multiples tentatives ?
    3.  **Identifier les Opportunités d'Amélioration de la `MemoryBank`:**
        *   Certaines informations cruciales sont-elles fréquemment recalculées au lieu d'être lues depuis la `memoryBank` ?
        *   La structure de la `memoryBank` est-elle optimale pour les requêtes fréquentes des agents ?
    4.  **Analyser l'Adaptabilité:**
        *   Le système gère-t-il bien les cas d'exception ou les demandes imprévues ? Les workflows sont-ils trop rigides ?
*   **Memory Bank Interaction:**
    *   Aucune lecture directe, mais l'analyse porte sur l'efficacité de son utilisation.
*   **Output (interne à `@workflow-optimizer-agent`):** Liste des problèmes de performance, inefficacités et opportunités d'amélioration identifiés.

### Phase 3: Génération de Suggestions d'Optimisation Spécifiques
*   **Agent Responsable:** `@workflow-optimizer-agent`
*   **Inputs:** Problèmes et opportunités identifiés (Phase 2). Connaissance de la structure des fichiers Pheromind (`01_AI-RUN/`, `.roomodes`, `.swarmConfig`).
*   **Actions & Tooling:**
    1.  Pour chaque problème/opportunité identifié, proposer une ou plusieurs solutions concrètes :
        *   **Pour les scripts `01_AI-RUN/*.md`:**
            *   Suggérer de réorganiser les phases pour une meilleure parallélisation (si possible).
            *   Proposer de fusionner ou de scinder des scripts pour plus de clarté ou d'efficacité.
            *   Suggérer d'ajouter des étapes de validation intermédiaires ou des logiques de gestion d'erreur plus robustes.
        *   **Pour les agents (`.roomodes`):**
            *   Suggérer de réécrire les `customInstructions` d'un agent pour plus de précision ou pour qu'il gère mieux certains cas.
            *   Proposer de scinder un agent complexe en plusieurs agents plus spécialisés.
            *   Suggérer d'ajouter de nouvelles capacités (ex: un agent qui apprend de ses erreurs passées stockées dans la `memoryBank`).
        *   **Pour la configuration du Scribe (`.swarmConfig`):**
            *   Suggérer de nouvelles règles d'`interpretationLogic` pour mieux parser les résumés de certains agents ou pour peupler plus intelligemment la `memoryBank`.
            *   Proposer des `valueExtractors` plus performants.
        *   **Pour la `MemoryBank` (`.pheromone` structure):**
            *   Suggérer d'ajouter de nouvelles sections ou de restructurer des sections existantes pour un accès plus rapide ou plus logique à l'information.
    2.  Prioriser les suggestions en fonction de leur impact potentiel et de leur faisabilité.
*   **Memory Bank Interaction:**
    *   Les suggestions peuvent viser à améliorer l'utilisation de la `memoryBank`.
*   **Output (interne à `@workflow-optimizer-agent`):** Liste de suggestions d'optimisation spécifiques et priorisées.

### Phase 4: Rapport des Suggestions et Enregistrement
*   **Agent Responsable:** `@workflow-optimizer-agent` (pour le rapport), `✍️ @orchestrator-pheromone-scribe` (pour l'enregistrement).
*   **Inputs:** Liste des suggestions d'optimisation (Phase 3).
*   **Actions & Tooling (`@workflow-optimizer-agent`):**
    1.  Générer un rapport Markdown (`system_optimization_suggestions_[timestamp].md`) dans `02_AI-DOCS/System_Optimization/`.
    2.  Le rapport doit inclure :
        *   Date de l'analyse.
        *   Résumé des principales observations de performance.
        *   Liste détaillée des suggestions d'optimisation, classées par type (Scripts `01_AI-RUN/`, Agents `.roomodes`, Scribe `.swarmConfig`, `MemoryBank`) et par priorité.
        *   Pour chaque suggestion : le problème adressé, la solution proposée, et le bénéfice attendu.
*   **Output (`@workflow-optimizer-agent` vers Scribe):** Résumé NL: "Analyse d'optimisation du système AgilePheromind terminée. [N_suggestions] suggestions proposées pour améliorer l'efficacité et la robustesse. Les principaux domaines concernés sont : [liste des domaines]. Rapport détaillé : `system_optimization_suggestions_[timestamp].md`."
*   **Actions & Tooling (Scribe):**
    1.  Interpréter le résumé.
    2.  Enregistrer le rapport `system_optimization_suggestions_[timestamp].md` dans `documentationRegistry`.
    3.  (Optionnel) Ajouter une entrée dans `memoryBank.systemImprovementProposals` avec un lien vers le rapport et un statut "PendingReview".
*   **Memory Bank Interaction:**
    *   Écriture (Scribe): Enregistrement du rapport et potentiellement des propositions.
*   **Output:** `.pheromone` mis à jour. L'administrateur du système ou l'équipe de développement Pheromind est informé des pistes d'amélioration.

---