# Workflow: Analyse de Code Legacy pour Projet de Migration (09_Legacy_Migration_Analysis.md)

**Objectif:** Fournir une analyse détaillée d'un codebase legacy (VB6, anciens composants .NET, COM+, procédures stockées MSSQL) en vue d'une migration vers une stack moderne (.NET Core/Angular). L'analyse doit identifier les composants clés, les dépendances, la logique métier encapsulée, les interactions avec la base de données, et proposer une estimation de la complexité ainsi que des stratégies de migration potentielles.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@migration-analyst-agent`.

**MCPs Utilisés:** Git Tools MCP (si le code legacy est dans un repo Git), Context7 MCP (pour la documentation sur les technologies legacy et modernes), Fetch MCP (pour documentation de dépendances obscures), MSSQL MCP (pour analyser schémas et procédures stockées), Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Tech Lead/Architecte) spécifie le chemin vers le code source legacy et la stack cible (ex: `"AgilePheromind analyse VB6 à [chemin/repo] pour migration vers .NET Core/Angular."`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Planification de l'Analyse et Ingestion du Code.**
        *   UO délègue à `@migration-analyst-agent`. L'agent utilise **Sequential Thinking MCP** pour structurer son approche d'analyse.
    *   **Phase 2: Analyse Détaillée des Composants Legacy.**
        *   UO délègue à `@migration-analyst-agent`. Analyse syntaxique, identification des modules, UI, logique métier, accès aux données (utilisant **MSSQL MCP** pour les SPs), et dépendances (utilisant **Fetch MCP** si besoin).
    *   **Phase 3: Mapping vers la Stack Moderne et Stratégies de Migration.**
        *   UO délègue à `@migration-analyst-agent`. Proposition d'équivalents modernes, identification des défis, suggestion de stratégies (ex: réécriture, strangler fig). Utilisation de **Context7 MCP** pour les technologies cibles.
    *   **Phase 4: Estimation de la Complexité et Identification des Risques.**
        *   UO délègue à `@migration-analyst-agent`.
    *   **Phase 5: Génération du Rapport d'Analyse de Migration.**
        *   UO délègue à `@migration-analyst-agent`.
    *   **Phase 6: Enregistrement du Rapport.**
        *   Scribe enregistre le rapport dans `.pheromone`.

## Détails des Phases:

### Phase 1: Planification de l'Analyse et Ingestion du Code
*   **Agent Responsable:** `@migration-analyst-agent`
*   **Inputs:** Chemin vers le code source legacy, informations sur la stack cible (depuis UO et `memoryBank.projectContext.techStack`).
*   **Actions & Tooling:**
    1.  Utiliser **Sequential Thinking MCP** pour planifier l'analyse:
        *   `set_goal`: "Analyser le codebase legacy [type de code, ex: VB6] pour migration vers .NET Core/Angular."
        *   `add_step`: "Ingérer et structurer le code source legacy."
        *   `add_step`: "Identifier les principaux modules applicatifs et leurs responsabilités (UI, métier, données)."
        *   `add_step`: "Analyser les dépendances externes et internes."
        *   `add_step`: "Analyser les interactions avec la base de données (schéma, SPs)."
        *   `add_step`: "Mapper les fonctionnalités legacy vers la stack moderne."
        *   `add_step`: "Proposer des stratégies de migration."
        *   `add_step`: "Estimer la complexité et identifier les risques."
    2.  **Ingestion du Code:**
        *   Si c'est un repo Git, utiliser **Git Tools MCP** (`clone_repository {repoUrl, localPath}`).
        *   Sinon, accéder aux fichiers via le chemin fourni (supposant un accès au système de fichiers via l'environnement d'exécution de l'agent).
        *   Organiser une copie locale du code pour analyse.
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.projectContext.techStack` (pour la cible).
*   **Output (interne à `@migration-analyst-agent`):** Plan d'analyse et code legacy accessible localement.

### Phase 2: Analyse Détaillée des Composants Legacy
*   **Agent Responsable:** `@migration-analyst-agent`
*   **Inputs:** Code legacy localisé, plan d'analyse.
*   **Actions & Tooling:**
    1.  **Analyse Statique du Code (VB6, .NET Framework, etc.):**
        *   Identifier les fichiers de projet, modules (`.bas`, `.cls`, `.frm` en VB6), classes, assemblys.
        *   Lister les formulaires UI, contrôles personnalisés, bibliothèques graphiques utilisées.
        *   Identifier les modules de logique métier, les fonctions/méthodes clés.
        *   Repérer les sections d'accès aux données (ADO, DAO, etc.).
    2.  **Analyse des Procédures Stockées (si applicable):**
        *   Utiliser **MSSQL MCP** (`get_database_schema`, `list_stored_procedures`, `get_stored_procedure_definition {procName}`) pour examiner la base de données MSSQL liée au code legacy.
        *   Analyser la logique des SPs complexes, leurs paramètres, et les tables qu'elles manipulent.
    3.  **Analyse des Dépendances:**
        *   Lister les DLLs externes, composants COM/COM+, librairies tierces.
        *   Pour les dépendances obscures ou non standard, tenter de trouver leur documentation ou des alternatives via **Fetch MCP** (scraping web) ou **Context7 MCP** (si ce sont des librairies connues).
    4.  Documenter les interconnexions entre les composants.
*   **Memory Bank Interaction:**
    *   Aucune écriture directe, mais les résultats alimentent le rapport final.
*   **Output (interne à `@migration-analyst-agent`):** Inventaire détaillé des composants legacy, de leur logique et de leurs dépendances.

### Phase 3: Mapping vers la Stack Moderne et Stratégies de Migration
*   **Agent Responsable:** `@migration-analyst-agent`
*   **Inputs:** Analyse des composants legacy (Phase 2). Connaissance de la stack cible .NET Core/Angular.
*   **Actions & Tooling:**
    1.  **Mapping Fonctionnel:**
        *   Pour chaque composant/fonctionnalité legacy, proposer une approche de réécriture ou de remplacement dans la stack moderne:
            *   Logique métier VB6/ancien .NET -> Services .NET Core (C#).
            *   UI VB6/WinForms/ASP.NET WebForms -> Composants Angular.
            *   Logique des SPs -> API .NET Core utilisant Entity Framework Core ou Dapper + requêtes SQL optimisées, ou refactoring en logique service.
            *   Dépendances COM+/DLLs -> Alternatives .NET Core/NuGet ou réécriture.
    2.  **Consultation de Documentation Moderne:**
        *   Utiliser **Context7 MCP** (`get_library_docs`) pour la documentation sur .NET Core (ASP.NET Core, EF Core, etc.) et Angular (modules, services, composants, RxJS) afin de guider les propositions de mapping.
    3.  **Stratégies de Migration:**
        *   Proposer des stratégies globales (ex: Big Bang, Strangler Fig, Approche Modulaire/Phasée).
        *   Suggérer un ordre de migration pour les modules, en commençant potentiellement par des PoCs (Proofs of Concept) sur des parties critiques ou représentatives.
*   **Memory Bank Interaction:**
    *   Les stratégies et mappings seront documentés dans le rapport final.
*   **Output (interne à `@migration-analyst-agent`):** Propositions de mapping technique et de stratégies de migration.

### Phase 4: Estimation de la Complexité et Identification des Risques
*   **Agent Responsable:** `@migration-analyst-agent`
*   **Inputs:** Analyse des composants (Phase 2), propositions de mapping (Phase 3).
*   **Actions & Tooling:**
    1.  **Estimation de la Complexité:**
        *   Pour chaque composant majeur ou groupe de fonctionnalités, attribuer un niveau de complexité de migration (ex: Faible, Moyen, Élevé, Très Élevé) basé sur la quantité de code, la complexité logique, les dépendances, et la difficulté du mapping.
        *   Fournir une justification pour les estimations élevées.
    2.  **Identification des Risques:**
        *   Lister les risques techniques spécifiques à la migration (ex: logique métier non documentée ou mal comprise, dépendances impossibles à remplacer, performances des nouvelles solutions, nécessité de formation de l'équipe sur la nouvelle stack).
        *   Identifier les risques liés aux données (migration de données, intégrité).
        *   Identifier les risques liés à la coexistence si une migration phasée est envisagée.
*   **Memory Bank Interaction:**
    *   Les estimations et risques seront dans le rapport final.
*   **Output (interne à `@migration-analyst-agent`):** Évaluations de complexité et liste des risques.

### Phase 5: Génération du Rapport d'Analyse de Migration
*   **Agent Responsable:** `@migration-analyst-agent`
*   **Inputs:** Toutes les informations collectées et analysées dans les phases précédentes.
*   **Actions & Tooling:**
    1.  Rédiger un rapport Markdown complet et structuré (ex: `legacy_analysis_[nom_projet_legacy]_[timestamp].md`) dans `02_AI-DOCS/Migration_Analyses/`.
    2.  Le rapport doit inclure :
        *   **Introduction:** Objectif de l'analyse, périmètre du code legacy.
        *   **Aperçu du Système Legacy:** Architecture générale, technologies clés.
        *   **Inventaire des Composants:** Liste des modules, UI, SPs, dépendances, avec description de leur rôle.
        *   **Analyse Détaillée:** Points saillants de la logique métier, complexité des SPs, problèmes de dépendances.
        *   **Propositions de Mapping vers .NET Core/Angular:** Équivalences suggérées, approches de réécriture.
        *   **Stratégie(s) de Migration Recommandée(s):** Justification.
        *   **Estimations de Complexité:** Par module/composant et global.
        *   **Risques Identifiés et Suggestions de Mitigation.**
        *   **Recommandations pour les Prochaines Étapes** (ex: PoCs, priorisation des modules).
    3.  S'assurer que le rapport est clair, factuel et actionnable pour l'équipe technique.
*   **Memory Bank Interaction:**
    *   Le chemin du rapport sera enregistré par le Scribe. Un résumé sera stocké dans `memoryBank.legacyCodeAnalyses`.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Analyse de migration du codebase legacy '[NomProjetLegacy]' terminée. Stack cible: .NET Core/Angular. [N_composants] composants majeurs analysés. Complexité globale estimée: [Faible/Moyen/Élevé]. [N_risques] risques majeurs identifiés. Rapport détaillé: `legacy_analysis_[nom_projet_legacy]_[timestamp].md`. Recommandation principale: [1-2 recommandations clés]."

### Phase 6: Enregistrement du Rapport
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@migration-analyst-agent`.
*   **Actions & Tooling:**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter le chemin vers `legacy_analysis_[nom_projet_legacy]_[timestamp].md`.
        *   `memoryBank.legacyCodeAnalyses.[nom_projet_legacy]`: Ajouter une entrée avec `{ summary: "[Extrait du résumé de l'agent]", linkToReport: "[Chemin du rapport]", timestamp: "{{timestamp}}", overallComplexity: "[Faible/Moyen/Élevé]", keyRisksCount: N_risques }`.
*   **Memory Bank Interaction:**
    *   Écriture: Archivage des informations clés de l'analyse de migration.
*   **Output:** `.pheromone` mis à jour. L'UO est informé de la disponibilité du rapport d'analyse.

---