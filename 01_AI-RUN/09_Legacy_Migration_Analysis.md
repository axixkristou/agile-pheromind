# Workflow: Analyse de Code Legacy pour Projet de Migration (09_Legacy_Migration_Analysis.md)

**Objectif:** Fournir une analyse détaillée et traçable d'un codebase legacy (VB6, anciens .NET, COM+, SPs MSSQL) en vue d'une migration vers .NET Core/Angular. L'analyse doit identifier composants clés, dépendances, logique métier, interactions DB, et proposer complexité, risques, et stratégies de migration, avec une documentation claire de la "chaîne de pensée" de l'agent. Gérer les erreurs d'accès aux sources ou aux MCPs.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@migration-analyst-agent`, `@clarification-agent`.

**MCPs Utilisés:** Git Tools MCP, Context7 MCP, Fetch MCP, MSSQL MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Tech Lead/Architecte) spécifie le chemin/repo du code legacy et la stack cible (ex: `"AgilePheromind analyse VB6 à [chemin] pour migration vers .NET Core/Angular."`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Planification de l'Analyse, Ingestion du Code, et Injection de Contexte.**
        *   UO **injecte contexte** (stack cible depuis `memoryBank.projectContext`, analyses de migration similaires passées depuis `memoryBank.legacyCodeAnalyses`) à `@migration-analyst-agent`.
        *   `@migration-analyst-agent` utilise **Sequential Thinking MCP** pour structurer son approche. Tente d'ingérer le code.
        *   **onError (Ingestion Code):** Si accès/lecture du code legacy échoue, l'agent le signale à l'UO. L'UO peut demander à l'utilisateur de vérifier le chemin/accès ou d'uploader le code, potentiellement via `@clarification-agent`. Workflow en pause.
    *   **Phase 2: Analyse Détaillée des Composants Legacy (avec "Chaîne de Pensée").**
        *   UO délègue à `@migration-analyst-agent`. Analyse syntaxique, modules, UI, logique métier, accès DB (avec **MSSQL MCP**), dépendances (avec **Fetch MCP**). **Doit documenter la "chaîne de pensée"** pour l'identification et l'interprétation des composants.
        *   **onError (MCP):** Si un MCP (MSSQL, Fetch) échoue, l'agent note l'information manquante, continue si possible, et le signale dans son rapport.
    *   **Phase 3: Mapping vers Stack Moderne et Stratégies (avec "Chaîne de Pensée").**
        *   UO délègue à `@migration-analyst-agent`. Proposition d'équivalents, défis, stratégies. Utilisation de **Context7 MCP**. **Doit documenter la "chaîne de pensée"** pour les propositions de mapping.
    *   **Phase 4: Estimation Complexité et Risques (avec "Chaîne de Pensée").**
        *   UO délègue à `@migration-analyst-agent`. **Doit documenter la "chaîne de pensée"** pour les évaluations.
    *   **Phase 5: Génération du Rapport d'Analyse (incluant "Chaîne de Pensée").**
        *   UO délègue à `@migration-analyst-agent`.
    *   **Phase 6: Enregistrement du Rapport.**
        *   Scribe enregistre.

## Détails des Phases:

### Phase 1: Planification de l'Analyse, Ingestion du Code, et Injection de Contexte
*   **Agent Responsable:** `@migration-analyst-agent`.
*   **Inputs (Injectés par l'UO):** Chemin/repo code legacy, stack cible. Contexte `memoryBank` (analyses migrations passées, tech stack cible détaillée).
*   **Actions & Tooling:**
    1.  **Sequential Thinking MCP** pour planifier l'analyse (étapes: ingestion, identification modules, dépendances, DB, mapping, stratégie, complexité/risques). **La sortie de ce MCP constituera la première partie de la "chaîne de pensée".**
    2.  **Ingestion Code:** Si repo Git, **Git Tools MCP** (`clone_repository`). Sinon, accès fichiers.
*   **onError (Ingestion Code):**
    *   Si échec, `@migration-analyst-agent` signale à l'UO: "Impossible d'accéder au code legacy à [chemin/repo]: [Erreur].".
    *   UO met workflow en pause, délègue à `@clarification-agent` pour demander à l'utilisateur de vérifier/fournir un accès valide.
*   **Output (interne si succès):** Plan d'analyse, code legacy accessible.

### Phase 2: Analyse Détaillée des Composants Legacy (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@migration-analyst-agent`.
*   **Inputs:** Code legacy, plan d'analyse.
*   **Actions & Tooling:**
    1.  Analyse statique (VB6, .NET Framework etc.): modules, UI, logique, accès données.
    2.  Analyse SPs (si applicable): **MSSQL MCP** (`get_database_schema`, `list_stored_procedures`, `get_stored_procedure_definition`). **onError (MSSQL MCP):** Si échec, noter l'impossibilité d'analyser les SPs et continuer si possible. Le signaler dans le rapport.
    3.  Analyse Dépendances: DLLs, COM+, libs tierces. Pour les obscures, **Fetch MCP** (web scraping) ou **Context7 MCP**. **onError (Fetch/Context7):** Si doc introuvable, noter la dépendance comme "inconnue/à risque".
    4.  **"Chaîne de Pensée":** Documenter dans le rapport (section dédiée ou en annexe) comment les principaux composants ont été identifiés, quelle a été leur interprétation fonctionnelle, et comment les dépendances ont été tracées.
*   **Output (interne):** Inventaire détaillé des composants, logique, dépendances, avec justification de l'analyse.

### Phase 3: Mapping vers Stack Moderne et Stratégies (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@migration-analyst-agent`.
*   **Inputs:** Analyse composants (Phase 2). Stack cible.
*   **Actions & Tooling:**
    1.  **Mapping Fonctionnel:** Proposer réécriture/remplacement (Logique VB6/.NET -> Services .NET Core; UI VB6/WinForms -> Composants Angular; SPs -> API .NET Core/EF Core).
    2.  **Documentation Moderne (Context7 MCP):** `get_library_docs` pour .NET Core/Angular pour guider les propositions.
    3.  **Stratégies de Migration:** Proposer (Big Bang, Strangler Fig, Phasée). Suggérer ordre, PoCs.
    4.  **"Chaîne de Pensée":** Documenter dans le rapport pourquoi certaines approches de mapping sont préférées, les alternatives envisagées et écartées, et la logique derrière la stratégie de migration proposée.
*   **Output (interne):** Propositions de mapping technique, stratégies de migration, avec justifications.

### Phase 4: Estimation Complexité et Risques (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@migration-analyst-agent`.
*   **Inputs:** Analyse composants, propositions de mapping.
*   **Actions & Tooling:**
    1.  **Estimation Complexité:** Par composant/groupe de fonctionnalités (Faible, Moyen, Élevé, Très Élevé). Justifier.
    2.  **Identification Risques:** Techniques (logique non doc, dépendances irremplaçables), données, coexistence.
    3.  **"Chaîne de Pensée":** Documenter dans le rapport comment la complexité a été évaluée pour chaque composant (quels facteurs ont le plus pesé) et comment les risques ont été identifiés et évalués.
*   **Output (interne):** Évaluations de complexité, liste des risques, avec justifications.

### Phase 5: Génération du Rapport d'Analyse de Migration (incluant "Chaîne de Pensée")
*   **Agent Responsable:** `@migration-analyst-agent`.
*   **Inputs:** Toutes les informations des phases précédentes.
*   **Actions & Tooling:**
    1.  Rédiger rapport MD (`legacy_analysis_[nom_projet_legacy]_[timestamp].md`) dans `02_AI-DOCS/Migration_Analyses/`.
    2.  Structure: Intro, Aperçu Legacy, Inventaire Composants, Analyse Détaillée, Propositions Mapping, Stratégie(s) Migration, Estimations Complexité, Risques, Recommandations.
    3.  **Intégrer explicitement les sections "Chaîne de Pensée"** ou les références aux annexes documentant le raisonnement pour chaque étape majeure d'analyse et de proposition.
*   **Output (vers Scribe):** Résumé NL: "Analyse migration '[NomProjetLegacy]' vers .NET Core/Angular terminée. Complexité: [Globale]. [N_risques] majeurs. Rapport (avec chaîne de pensée détaillée): `legacy_analysis_[nom_projet_legacy]_[timestamp].md`. Recommandation: [Clé]."

### Phase 6: Enregistrement du Rapport
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** Résumé NL de `@migration-analyst-agent`.
*   **Actions:**
    1.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter chemin vers `legacy_analysis...md`.
        *   `memoryBank.legacyCodeAnalyses.{{nom_projet_legacy}}`: Ajouter `{ summary, linkToReport, timestamp, overallComplexity, keyRisksCount, reasoningChainLink: reportPath }`.
        *   Si des risques spécifiques sont identifiés et peuvent être ajoutés au `riskRegister`, créer des entrées préliminaires.
*   **Output:** `.pheromone` mis à jour. UO informé.

---