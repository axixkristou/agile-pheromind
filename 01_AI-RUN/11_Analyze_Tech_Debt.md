# Workflow: Analyse de la Dette Technique et des "Code Smells" (11_Analyze_Tech_Debt.md)

**Objectif:** Effectuer une analyse approfondie du codebase du projet (ou d'un module spécifique) pour identifier la dette technique, les "code smells", et les zones nécessitant un refactoring. Le système doit produire un rapport détaillé avec des suggestions priorisées et la "chaîne de pensée" justifiant les principales conclusions. Une gestion des erreurs d'accès au code ou aux outils d'analyse est prévue.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@code-reviewer-assistant`, `@clarification-agent`.

**MCPs Utilisés:** Git Tools MCP, potentiellement un MCP d'analyse statique dédié (ex: SonarQube), Context7 MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Tech Lead/Architecte) demande une analyse (ex: `"AgilePheromind analyse dette technique projet"`). Peut être planifié.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Définition du Périmètre, Collecte du Code, et Injection de Contexte.**
        *   UO délègue à `@code-reviewer-assistant` pour identifier et récupérer les fichiers cibles via **Git Tools MCP**.
        *   UO **injecte un contexte pertinent** de `memoryBank` (conventions, historique de dette, décisions architecturales) à `@code-reviewer-assistant`.
        *   **onError (Accès Code):** Si code inaccessible, logguer, notifier, arrêter.
    *   **Phase 2: Planification de l'Analyse et Analyse Statique/Heuristique (avec "Chaîne de Pensée").**
        *   UO délègue à `@code-reviewer-assistant`. L'agent utilise **Sequential Thinking MCP** pour planifier son analyse.
        *   Exécute linters, analyseurs, et heuristiques pour "code smells". Utilise **Context7 MCP** pour bonnes pratiques librairies.
        *   **Doit documenter la "chaîne de pensée"** pour l'identification des principaux "smells" ou zones de dette.
        *   **onError (Outils d'Analyse):** Si un outil d'analyse (ex: MCP SonarQube) échoue, noter et continuer avec les autres méthodes si possible.
    *   **Phase 3: Identification des Zones de Dette Technique, Priorisation et Suggestions (avec "Chaîne de Pensée").**
        *   UO délègue à `@code-reviewer-assistant`. Évalue impact, priorise.
        *   **Doit documenter la "chaîne de pensée"** pour la priorisation et les suggestions de refactoring.
        *   Si des zones de code sont trop obscures pour une analyse de dette pertinente, l'agent peut le signaler à l'UO pour une clarification potentielle via `@clarification-agent` (demandant au dev d'expliquer la section).
    *   **Phase 4: Génération du Rapport Détaillé.**
        *   UO délègue à `@code-reviewer-assistant`.
    *   **Phase 5: Enregistrement dans `.pheromone`.**
        *   Scribe enregistre rapport et met à jour `memoryBank.technicalDebtItems`.

## Détails des Phases:

### Phase 1: Définition du Périmètre, Collecte du Code, et Injection de Contexte
*   **Agent Responsable:** `@code-reviewer-assistant`, UO.
*   **Inputs:** Périmètre d'analyse. `memoryBank.currentProject.repositoryUrl` et `defaultBranch`.
*   **Actions & Tooling (UO et `@code-reviewer-assistant`):**
    1.  Définir fichiers cibles.
    2.  `@code-reviewer-assistant` récupère code via **Git Tools MCP**.
    3.  **onError (Git Tools MCP):** Si échec, UO loggue via Scribe, notifie utilisateur, arrête workflow.
    4.  UO injecte contexte de `memoryBank` à `@code-reviewer-assistant`:
        *   `projectContext.codingConventionsLink`, `designConventionsLink`.
        *   `technicalDebtItems` existants (pour éviter doublons ou voir évolution).
        *   `architecturalDecisions` pertinentes.
        *   (Optionnel) Configuration spécifique pour outils d'analyse statique (si stockée).
*   **Output:** Code source et contexte injecté prêts pour `@code-reviewer-assistant`.

### Phase 2: Planification de l'Analyse et Analyse Statique/Heuristique (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@code-reviewer-assistant`.
*   **Inputs:** Code source, contexte injecté.
*   **Actions & Tooling:**
    1.  **Planification (Sequential Thinking MCP):**
        *   `set_goal`: "Analyser le code [périmètre] pour la dette technique."
        *   `add_step`: "Exécuter linters et analyseurs standards."
        *   `add_step`: "Rechercher duplications de code."
        *   `add_step`: "Analyser complexité cyclomatique (méthodes/fonctions)."
        *   `add_step`: "Identifier longues méthodes/classes/composants."
        *   `add_step`: "Évaluer couplage/cohésion."
        *   `add_step`: "Rechercher 'dead code'."
        *   `add_step`: "Vérifier TODO/FIXME."
        *   `add_step`: "Consulter Context7 pour bonnes pratiques sur librairies utilisées dans zones suspectes."
        *   `run_sequence`. **Conserver ce plan comme partie de la "chaîne de pensée".**
    2.  Exécuter les étapes du plan.
    3.  **"Chaîne de Pensée":** Pour chaque "code smell" ou zone de dette majeure identifiée, documenter dans le rapport comment elle a été détectée et pourquoi elle est considérée comme de la dette (ex: "La méthode X dépasse 100 lignes et a une complexité de Y, indiquant une violation de la convention Z et un risque de maintenabilité." ).
*   **onError (Outils d'Analyse):**
    *   Si un outil externe (ex: MCP SonarQube) échoue, `@code-reviewer-assistant` le note, informe l'UO, et continue l'analyse avec les autres méthodes. Le rapport final mentionnera l'outil défaillant.
*   **Output (interne):** Liste de "code smells", problèmes de qualité, avec localisation et début de "chaîne de pensée".

### Phase 3: Identification des Zones de Dette Technique, Priorisation et Suggestions (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@code-reviewer-assistant`.
*   **Inputs:** Liste des "code smells" (Phase 2).
*   **Actions & Tooling:**
    1.  Regrouper problèmes, évaluer impact (maintenabilité, risque bugs, perf, extensibilité).
    2.  Prioriser la dette (Critique, Élevée, Moyenne, Faible).
    3.  Pour les items prioritaires, suggérer des refactorings ou actions.
    4.  **"Chaîne de Pensée":** Documenter dans le rapport la justification de la priorisation pour les items critiques/élevés et la logique derrière les suggestions de refactoring (ex: "Refactoriser X en utilisant le pattern Y améliorera la lisibilité et réduira le couplage, comme suggéré par la bonne pratique Z de Context7 pour cette librairie.").
    5.  **Gestion d'Ambiguïté:** Si une zone de code est si obscure qu'il est impossible d'évaluer la dette ou de suggérer un refactoring pertinent:
        *   Signaler à l'UO: "Impossible d'analyser la dette pour [fichier Z ligne A]. Code obscur. Suggestion de question pour dev: 'Pouvez-vous expliquer l'objectif et la structure de cette section pour évaluer son refactoring ?'".
        *   L'UO peut initier clarification via `@clarification-agent`. L'analyse de cette section spécifique est mise en pause.
*   **Output (interne):** Liste priorisée d'items de dette technique avec suggestions et justifications.

### Phase 4: Génération du Rapport Détaillé
*   **Agent Responsable:** `@code-reviewer-assistant`.
*   **Inputs:** Liste priorisée de dette technique, justifications ("chaînes de pensée").
*   **Actions & Tooling:**
    1.  Rédiger rapport MD (`tech_debt_analysis_[scope]_[timestamp].md`) dans `03_SPECS/Tech_Debt_Reports/`.
    2.  Structure: Périmètre, Résumé zones de dette, Détail items prioritaires (description, localisation, impact, sévérité, **raisonnement/chaîne de pensée**, suggestion refactoring), Métriques (optionnel).
    3.  Si des sections n'ont pu être analysées faute de clarté (et que la clarification n'a pas eu lieu ou n'a pas résolu), le mentionner clairement.
*   **Output (vers Scribe):** Résumé NL: "Analyse dette technique '[Périmètre]' terminée. [N_total] problèmes, dont [N_crit] critiques. Suggestions refactoring incluses. Rapport (avec chaîne de pensée): `tech_debt_analysis_[scope]_[timestamp].md`."

### Phase 5: Enregistrement dans `.pheromone`
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** Résumé NL de `@code-reviewer-assistant`.
*   **Actions:**
    1.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter chemin vers `tech_debt_analysis...md`.
        *   `memoryBank.technicalDebtItems`: Ajouter/Mettre à jour des objets structurés pour chaque item de dette critique/élevé identifié, incluant un lien vers la section pertinente du rapport (`reasoningChainLink`).
        *   `memoryBank.projectContext.lastTechDebtAnalysisTimestamp = "{{timestamp}}"`.
*   **Output:** `.pheromone` mis à jour. UO informé.

---