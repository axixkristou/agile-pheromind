# Workflow: Analyse de la Dette Technique et des "Code Smells" (11_Analyze_Tech_Debt.md)

**Objectif:** Effectuer une analyse du codebase du projet pour identifier la dette technique, les "code smells", et les zones potentielles de refactoring. Le système doit produire un rapport avec des suggestions priorisées pour améliorer la maintenabilité, la lisibilité et la robustesse du code.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@code-reviewer-assistant`.

**MCPs Utilisés:** Git Tools MCP (pour accéder au code), potentiellement un MCP d'analyse statique dédié (ex: SonarQube si un MCP existe ou si l'agent peut interagir avec son API/CLI), Context7 MCP (pour comprendre les bonnes pratiques des librairies utilisées dans les zones analysées).

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Tech Lead/Architecte) demande une analyse de la dette technique, potentiellement pour tout le projet ou un module spécifique (ex: `"AgilePheromind analyse dette technique du projet"` ou `"AgilePheromind analyse dette technique module AuthenticationService"`). Peut aussi être une tâche planifiée.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Définition du Périmètre et Collecte du Code.**
        *   UO délègue à `@code-reviewer-assistant` pour identifier les fichiers/modules à analyser et les récupérer.
    *   **Phase 2: Analyse Statique Approfondie et Identification des "Code Smells".**
        *   UO délègue à `@code-reviewer-assistant`. Utilisation d'outils d'analyse (linters avancés, MCP SonarQube conceptuel) et d'heuristiques.
    *   **Phase 3: Identification des Zones de Dette Technique et Priorisation.**
        *   UO délègue à `@code-reviewer-assistant`.
    *   **Phase 4: Génération du Rapport et des Suggestions de Refactoring.**
        *   UO délègue à `@code-reviewer-assistant`.
    *   **Phase 5: Enregistrement dans `.pheromone`.**
        *   Scribe enregistre le rapport et met à jour la section `technicalDebtItems` de la Memory Bank.

## Détails des Phases:

### Phase 1: Définition du Périmètre et Collecte du Code
*   **Agent Responsable:** `@code-reviewer-assistant`
*   **Inputs:** Périmètre d'analyse (projet entier ou module spécifique) fourni par l'UO. `memoryBank.currentProject.repositoryUrl`.
*   **Actions & Tooling:**
    1.  **Définir les Fichiers Cibles:**
        *   Si "projet entier": Identifier tous les répertoires de code source .NET et Angular.
        *   Si "module spécifique": Identifier les fichiers appartenant à ce module.
    2.  **Récupérer le Code Source:**
        *   Utiliser **Git Tools MCP** (`clone_repository` ou `get_file_contents` pour les fichiers ciblés) pour obtenir la version la plus récente de la branche par défaut (ex: `develop` ou `main` depuis `.pheromone.currentProject.defaultBranch`).
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.currentProject.repositoryUrl`, `memoryBank.currentProject.defaultBranch`.
*   **Output (interne à `@code-reviewer-assistant`):** Code source prêt pour analyse.

### Phase 2: Analyse Statique Approfondie et Identification des "Code Smells"
*   **Agent Responsable:** `@code-reviewer-assistant`
*   **Inputs:** Code source à analyser. `memoryBank.projectContext.codingConventionsLink`.
*   **Actions & Tooling:**
    1.  **Exécuter les Linters et Analyseurs Statiques Standards:**
        *   Appliquer les linters .NET (Roslyn Analyzers + StyleCop) et Angular (ESLint) configurés dans `memoryBank.toolingConfigurations.linters`.
    2.  **Analyse Heuristique des "Code Smells":**
        *   **Duplication de Code:** Rechercher des blocs de code identiques ou très similaires.
        *   **Complexité Cyclomatique Élevée:** Identifier les méthodes/fonctions avec de nombreux chemins d'exécution.
        *   **Longues Méthodes/Classes/Composants:** Détecter les unités de code excessivement volumineuses.
        *   **Couplage Fort / Faible Cohésion:** Analyser les dépendances entre classes/modules.
        *   **Variables/Méthodes Inutilisées ("Dead Code").**
        *   **Commentaires Obsolètes ou TODO/FIXME Anciens.**
        *   **Non-Respect des Principes SOLID (pour .NET notamment).**
        *   **Utilisation Inappropriée des API de Frameworks/Librairies:** Consulter **Context7 MCP** pour les bonnes pratiques des librairies .NET/Angular utilisées dans les sections de code suspectes.
    3.  **(Conceptuel) Utilisation d'un MCP d'Analyse Statique Avancée:**
        *   Si un **MCP SonarQube** (ou équivalent) est disponible et configuré, l'utiliser pour une analyse plus approfondie. L'agent interpréterait les résultats de ce MCP.
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.toolingConfigurations.linters`, `memoryBank.projectContext.codingConventionsLink`.
*   **Output (interne à `@code-reviewer-assistant`):** Liste des "code smells" et problèmes de qualité identifiés, avec leur localisation.

### Phase 3: Identification des Zones de Dette Technique et Priorisation
*   **Agent Responsable:** `@code-reviewer-assistant`
*   **Inputs:** Liste des "code smells" et problèmes (Phase 2).
*   **Actions & Tooling:**
    1.  **Regrouper les Problèmes:** Associer les "code smells" à des zones spécifiques du code (fichiers, modules, classes).
    2.  **Évaluer l'Impact de Chaque Zone de Dette:**
        *   **Maintenabilité:** Complexité pour comprendre et modifier.
        *   **Risque de Bugs:** Probabilité d'introduire des erreurs.
        *   **Performance:** Impact potentiel sur les performances (si applicable).
        *   **Extensibilité:** Difficulté à ajouter de nouvelles fonctionnalités.
    3.  **Prioriser la Dette Technique:**
        *   Attribuer une sévérité (Critique, Élevée, Moyenne, Faible) à chaque élément de dette technique identifié.
        *   Se concentrer sur la dette qui a le plus d'impact négatif ou qui bloque des évolutions futures.
*   **Memory Bank Interaction:**
    *   Aucune lecture directe.
*   **Output (interne à `@code-reviewer-assistant`):** Liste priorisée d'éléments de dette technique.

### Phase 4: Génération du Rapport et des Suggestions de Refactoring
*   **Agent Responsable:** `@code-reviewer-assistant`
*   **Inputs:** Liste priorisée de la dette technique (Phase 3).
*   **Actions & Tooling:**
    1.  Rédiger un rapport Markdown (`tech_debt_analysis_[scope]_[timestamp].md`) dans `03_SPECS/Tech_Debt_Reports/`.
    2.  Le rapport doit inclure:
        *   **Périmètre de l'Analyse.**
        *   **Résumé des Principales Zones de Dette Technique.**
        *   **Détail de Chaque Élément de Dette Prioritaire:**
            *   Description du problème ("code smell").
            *   Localisation (fichier(s), ligne(s)).
            *   Impact évalué.
            *   Sévérité.
            *   Suggestion(s) de refactoring ou d'action corrective (ex: "Extraire la méthode X", "Simplifier la condition Y", "Créer une classe Z pour encapsuler cette logique").
        *   **(Optionnel) Métriques Globales:** (ex: % de duplication de code, complexité moyenne).
    3.  Pour les suggestions de refactoring les plus critiques, envisager de créer des ébauches de tâches techniques.
*   **Memory Bank Interaction (via Scribe):**
    *   Le chemin du rapport sera stocké, et les items de dette seront ajoutés à `memoryBank.technicalDebtItems`.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Analyse de la dette technique pour '[Périmètre]' terminée. [N_total] problèmes identifiés, dont [N_critiques] critiques et [N_eleves] élevés. Des suggestions de refactoring sont incluses. Rapport détaillé: `tech_debt_analysis_[scope]_[timestamp].md`."

### Phase 5: Enregistrement dans `.pheromone`
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@code-reviewer-assistant`.
*   **Actions & Tooling:**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter le chemin vers `tech_debt_analysis_[scope]_[timestamp].md`.
        *   `memoryBank.technicalDebtItems`: Pour chaque item de dette critique/élevé identifié dans le rapport (le Scribe devrait pouvoir extraire une liste structurée du rapport ou du résumé de l'agent):
            *   Ajouter un nouvel objet: `{ id: "TD_UUID_Example", description: "[Description du problème]", severity: "[Critique/Élevée]", location: "[Fichier:Ligne]", status: "Identified", suggestedAction: "[Brève suggestion]", linkToReportSection: "[CheminDuRapport#SectionID]", dateIdentified: "{{timestamp}}" }`.
        *   Mettre à jour `memoryBank.projectContext.lastTechDebtAnalysisTimestamp = "{{timestamp}}"`.
*   **Memory Bank Interaction:**
    *   Écriture: Ajout structuré des éléments de dette technique.
*   **Output:** `.pheromone` mis à jour. L'UO est informé de la disponibilité du rapport d'analyse de la dette technique.

---