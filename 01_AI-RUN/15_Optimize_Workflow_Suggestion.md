# Workflow: Suggestion d'Optimisation du Système AgilePheromind (15_Optimize_Workflow_Suggestion.md)

**Objectif:** Analyser les métriques de performance, l'historique des opérations, et la structure de la `memoryBank` du système AgilePheromind pour identifier les goulots d'étranglement, les inefficacités, ou les zones d'amélioration. Le système doit proposer des optimisations concrètes et justifiées (avec "chaîne de pensée") pour les scripts `01_AI-RUN/*.md`, les définitions d'agents dans `.roomodes`, ou la logique d'interprétation dans `.swarmConfig`.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@workflow-optimizer-agent`, `@swarm-monitor-agent` (peut fournir des données d'entrée).

**MCPs Utilisés:** Sequential Thinking MCP (pour structurer l'analyse d'optimisation).

## Pheromind Workflow Overview:

1.  **Initiation:** Manuelle (Admin/Tech Lead: `"AgilePheromind optimise tes workflows"`) ou automatique (planifiée/déclenchée par `@swarm-monitor-agent` si seuils de performance anormaux).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Collecte et Analyse des Données de Performance du Swarm.**
        *   UO **injecte le contexte pertinent** à `@workflow-optimizer-agent`: `.pheromone.activeWorkflow.history`, `memoryBank.workflowPerformanceMetrics`, rapports de `@swarm-monitor-agent` (via `documentationRegistry`), et la structure actuelle des fichiers `.roomodes` et `.swarmConfig` (conceptuellement, l'agent y a accès ou l'UO lui en fournit des extraits clés).
    *   **Phase 2: Identification Structurée des Axes d'Optimisation (avec "Chaîne de Pensée").**
        *   UO délègue à `@workflow-optimizer-agent`. L'agent utilise **Sequential Thinking MCP** pour identifier les problèmes et les causes racines. **Doit documenter sa "chaîne de pensée".**
        *   **onError:** Si les données de performance sont insuffisantes pour une analyse significative, l'agent le signale.
    *   **Phase 3: Génération de Suggestions d'Optimisation Spécifiques (avec "Chaîne de Pensée").**
        *   UO délègue à `@workflow-optimizer-agent`. **Doit documenter la "chaîne de pensée"** justifiant chaque suggestion.
    *   **Phase 4: Rapport des Suggestions et Enregistrement.**
        *   `@workflow-optimizer-agent` génère un rapport.
        *   Scribe enregistre le rapport et potentiellement les suggestions structurées dans la `memoryBank`.

## Détails des Phases:

### Phase 1: Collecte et Analyse des Données de Performance du Swarm
*   **Agent Responsable:** `@workflow-optimizer-agent`.
*   **Inputs (Injectés par l'UO):**
    *   Accès à `.pheromone` (`activeWorkflow.history`, `memoryBank.workflowPerformanceMetrics`, `systemHealth`).
    *   Chemins vers les rapports de `@swarm-monitor-agent` (via `documentationRegistry`).
    *   (Conceptuel) Accès en lecture aux fichiers `.roomodes` et `.swarmConfig` actuels.
*   **Actions & Tooling:**
    1.  Analyser `activeWorkflow.history`: durées d'exécution/phase, erreurs d'agents, fréquence d'utilisation des scripts.
    2.  Analyser `memoryBank.workflowPerformanceMetrics` (si peuplé).
    3.  Consulter rapports de `@swarm-monitor-agent`.
    4.  Identifier les MCPs fréquemment lents ou en échec (via `systemHealth.mcpStatus` ou `agentActivityLog`).
*   **Output (interne):** Données brutes/agrégées sur la performance de Pheromind.

### Phase 2: Identification Structurée des Axes d'Optimisation (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@workflow-optimizer-agent`.
*   **Inputs:** Données de performance (Phase 1).
*   **Actions & Tooling:**
    1.  Utiliser **Sequential Thinking MCP** pour une analyse méthodique :
        *   `set_goal`: "Identifier les axes d'optimisation pour AgilePheromind basé sur les données de performance."
        *   `add_step`: "Quels sont les 3 workflows `01_AI-RUN/*.md` les plus lents ou les plus sujets aux erreurs ? Pour chacun, quelle phase spécifique est le goulot d'étranglement ?"
        *   `add_step`: "Quels agents (`.roomodes`) sont le plus souvent associés à des échecs, des retries, ou des durées d'exécution excessives ? Quelles sont les causes possibles (prompt ambigu, dépendance MCP lente, logique interne de l'agent) ?"
        *   `add_step`: "La logique du Scribe (`.swarmConfig`) semble-t-elle causer des erreurs d'interprétation ou des mises à jour incorrectes de `.pheromone` (basé sur des patterns d'erreurs ou des logs du Scribe si disponibles) ?"
        *   `add_step`: "L'utilisation de la `memoryBank` semble-t-elle inefficace ? (Ex: agents redemandant des informations déjà calculées, structure de données non optimale pour les requêtes LLM)."
        *   `run_sequence`.
    2.  **"Chaîne de Pensée":** La sortie détaillée de ce Sequential Thinking constituera la "chaîne de pensée" pour l'identification des problèmes.
*   **onError:** Si les données de performance sont jugées insuffisantes par l'agent (ex: trop peu d'historique de workflow), il doit le signaler à l'UO. L'UO peut alors indiquer que l'analyse sera limitée ou reporter le workflow.
*   **Output (interne):** Liste de problèmes de performance, inefficacités, et leurs causes racines potentielles, avec la "chaîne de pensée" de l'analyse.

### Phase 3: Génération de Suggestions d'Optimisation Spécifiques (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@workflow-optimizer-agent`.
*   **Inputs:** Problèmes identifiés et leur analyse (Phase 2).
*   **Actions & Tooling:**
    1.  Pour chaque problème identifié, utiliser **Sequential Thinking MCP** pour brainstormer des solutions :
        *   `set_goal`: "Proposer des optimisations pour le problème : [Description du problème]."
        *   `add_step`: "Si le problème est dans un script `01_AI-RUN/*.md` (lenteur de phase, erreur fréquente): envisager de réorganiser/paralléliser les phases, ajouter une étape de clarification préalable, améliorer les instructions pour l'agent responsable, ou ajouter une meilleure gestion d'erreur."
        *   `add_step`: "Si le problème vient d'un agent (`.roomodes`): envisager de réécrire ses `customInstructions` (plus de clarté, gestion d'exceptions, meilleure utilisation du contexte injecté), de le scinder, ou de lui ajouter une capacité (ex: appeler `@clarification-agent`)."
        *   `add_step`: "Si le problème est lié au Scribe (`.swarmConfig`): envisager de nouvelles règles d'`interpretationLogic` plus spécifiques, ou des `valueExtractors` plus robustes."
        *   `add_step`: "Si le problème est lié à la `memoryBank`: suggérer des changements de structure, l'ajout d'index (conceptuels), ou des patterns d'accès plus efficaces pour les agents."
        *   `run_sequence`.
    2.  **"Chaîne de Pensée":** La sortie de ce MCP (et toute élaboration par l'agent) constituera la "chaîne de pensée" justifiant chaque suggestion.
    3.  Prioriser les suggestions (impact vs faisabilité).
*   **Output (interne):** Liste de suggestions d'optimisation spécifiques, priorisées, avec leur justification ("chaîne de pensée").

### Phase 4: Rapport des Suggestions et Enregistrement
*   **Agent Responsable:** `@workflow-optimizer-agent` (rapport), Scribe (enregistrement).
*   **Inputs:** Suggestions d'optimisation (Phase 3).
*   **Actions (`@workflow-optimizer-agent`):**
    1.  Générer rapport MD (`system_optimization_suggestions_[timestamp].md`) dans `02_AI-DOCS/System_Optimization/`.
    2.  Contenu: Date, Résumé observations performance, Liste détaillée des suggestions (problème, solution proposée, **justification/chaîne de pensée**, impact attendu, priorité). Mentionner les analyses limitées si données insuffisantes.
*   **Output (`@workflow-optimizer-agent` -> Scribe):** Résumé NL: "Analyse optimisation AgilePheromind terminée. [N_sugg] suggestions (avec chaîne de pensée) pour améliorer scripts/agents/Scribe/MemoryBank. Rapport: `system_optimization_suggestions_[timestamp].md`."
*   **Actions (Scribe):**
    1.  Enregistrer rapport dans `documentationRegistry`.
    2.  Ajouter entrée dans `memoryBank.systemImprovementProposals`: `{ reportPath, timestamp, status: "PendingReview", summary: "[Extrait du résumé]" }`.
    3.  Stocker le lien vers le rapport (`reasoningChainLinks.systemOptimization`) dans cette entrée.
*   **Output:** `.pheromone` à jour. Administrateur Pheromind informé.

---