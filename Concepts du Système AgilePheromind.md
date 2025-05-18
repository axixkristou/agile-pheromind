# Concepts du Système d'Assistance IA Agile "AgilePheromind"

**Version:** 1.0
**Date de Dernière Mise à Jour:** {{TIMESTAMP_PAR_PHEROMIND}}

## 1. Introduction à AgilePheromind

AgilePheromind est un système d'intelligence artificielle (IA) avancé, conçu comme un **essaim d'agents IA collaboratifs**, propulsé par le framework **Pheromind**. Son but principal est d'assister et d'augmenter les capacités des équipes de développement logiciel qui suivent des méthodologies Agiles, en particulier celles utilisant des technologies comme .NET, Angular, Azure DevOps, Docker et Azure Kubernetes Service (AKS).

Contrairement à un outil IA monolithique, AgilePheromind fonctionne comme un écosystème où des agents spécialisés interagissent indirectement pour accomplir des tâches complexes, apprendre du contexte du projet, et s'adapter aux besoins de l'équipe.

## 2. Principes Fondamentaux de Pheromind Appliqués

AgilePheromind hérite et applique les principes clés du framework Pheromind :

*   **Intelligence en Essaim (Swarm Intelligence) via Stigmergie:** Les agents ne communiquent pas directement entre eux de manière extensive. Ils interagissent en modifiant un environnement partagé, le fichier d'état `.pheromone`. Les actions d'un agent laissent des "traces" (informations) dans `.pheromone` qui influencent le comportement des agents suivants.
*   **Orchestration Hiérarchique et Pilotée par Scripts:**
    *   Le `🎩 @head-orchestrator` reçoit les directives initiales de l'utilisateur.
    *   Il active le `🧐 @uber-orchestrator`, qui interprète des **scripts de workflow** (`01_AI-RUN/*.md`). Ces scripts définissent la séquence logique des opérations pour une tâche donnée (ex: démarrer une User Story).
    *   L'`🧐 @uber-orchestrator` délègue ensuite des phases spécifiques de ces workflows à des **agents IA spécialisés**.
*   **Gestion Centrale de l'État et de la Mémoire (`.pheromone`):** Ce fichier JSON est le cœur du système.
    *   Il contient l'**état opérationnel actuel** (utilisateur actif, tâche en cours, etc.).
    *   Il héberge le `documentationRegistry`, un index de tous les artefacts et documents importants générés ou référencés.
    *   Il maintient la `memoryBank`, une base de connaissances persistante et évolutive du projet (décisions, analyses, historique des US/tâches, conventions, risques, etc.).
*   **Interprétation et Mise à Jour de l'État par le Scribe (`✍️ @orchestrator-pheromone-scribe`):** Cet agent unique est responsable de la lecture des résumés en langage naturel produits par les autres agents et de la mise à jour structurée et intelligente de `.pheromone` en utilisant une logique définie dans `.swarmConfig`.
*   **Modularité et Spécialisation des Agents (`.roomodes`):** Chaque agent IA a un rôle bien défini, des compétences spécifiques (outils MCP qu'il peut utiliser) et des instructions personnalisées (`customInstructions`) stockées dans le fichier `.roomodes`.
*   **Interaction avec l'Environnement Externe (MCPs):** Les agents utilisent des **Model Context Protocol (MCP) servers** pour interagir avec des outils et services externes (Azure DevOps, Git, bases de données MSSQL, APIs de documentation comme Context7, outils de navigation web, etc.).

```mermaid
graph LR
    User["Utilisateur (PO, Dev, Lead)"] -- Commande Initiale --> HO["🎩 @head-orchestrator"]
    HO -- Déclenche Workflow + Params --> UO["🧐 @uber-orchestrator"]

    subgraph Pheromind Core Engine
        UO -- Lit --> AIRUN["Scripts 01_AI-RUN/*.md"]
        AIRUN -- Définit Phases & Logique --> UO
        UO -- Consulte/Informe via Scribe --> PheromoneDB[(".pheromone (État + MemoryBank + DocRegistry)")]
        PheromoneDB -- Fournit Contexte --> UO
        UO -- Délègue Tâche de Phase + Contexte --> SA["Agent Spécialisé (ex: @developer-agent)"]
        SA -- Utilise --> MCPs["Model Context Protocol Servers (Azure DevOps, Git, Context7, MSSQL, etc.)"]
        MCPs -- Interagit avec --> ExternalTools["Outils Externes (Azure DevOps, Repo Git, SQL DB, APIs Docs)"]
        SA -- Produit Résultat + Résumé NL --> Scribe["✍️ @orchestrator-pheromone-scribe"]
        Scribe -- Lit --> SwarmConfig[".swarmConfig (InterpretationLogic)"]
        Scribe -- Met à jour --> PheromoneDB
    end

    style PheromoneDB fill:#f9f,stroke:#333,stroke-width:2px
    style AIRUN fill:#ccf,stroke:#333,stroke-width:2px
    style SwarmConfig fill:#e6e6fa,stroke:#333,stroke-width:1px
    style MCPs fill:#add8e6,stroke:#333,stroke-width:1px
    style ExternalTools fill:#fafad2,stroke:#333,stroke-width:1px
```

## 3. Composants Clés du Système

### 3.1. Le Fichier d'État `.pheromone`

C'est le "cerveau" et la mémoire collective du Swarm. Sa structure bien définie est cruciale.

*   **`systemVersion`, `lastUpdated`:** Méta-informations sur le système.
*   **`currentUser`:** Contexte de l'utilisateur interagissant avec Pheromind (ID, nom d'utilisateur Azure DevOps, rôles, préférences, contexte de tâche actuel).
*   **`currentProject`:** Détails du projet Azure DevOps et du dépôt Git actifs.
*   **`currentSprint`:** Informations sur le sprint Agile en cours (ID, objectif, US sélectionnées).
*   **`activeWorkflow`:** Informations sur le script `01_AI-RUN/*.md` en cours d'exécution.
*   **`activeUserStory`, `activeTask`:** Pointeurs vers les éléments de travail actuellement focus par le système.
*   **`documentationRegistry`:** Un dictionnaire clé-valeur où les clés sont des identifiants de document (ex: `us_Azure#123_analysis.md`) et les valeurs sont leurs chemins relatifs ou absolus. Permet aux agents de retrouver facilement les rapports, spécifications, etc.
*   **`systemHealth`:** Informations sur l'état de Pheromind et de ses connexions MCP.
*   **`memoryBank`:** La section la plus dynamique et la plus riche. Elle contient :
    *   `projectContext`: Stack technologique, liens vers les conventions, nom du projet ADO.
    *   `userProfiles`: Informations agrégées sur les utilisateurs du système Pheromind.
    *   `userStories`: Historique détaillé, statuts, tâches liées, analyses, décisions, commits pour chaque US traitée.
    *   `tasks`: Détails similaires pour chaque tâche technique.
    *   `pullRequests`, `commits`: Informations sur les PRs et les commits.
    *   `legacyCodeAnalyses`, `sprintRetrospectivesSummaries`, `technicalDebtItems`, `architecturalDecisions`, `commonIssuesAndSolutions`, `toolingConfigurations`, `riskRegister`, `workflowPerformanceMetrics`: Autant de "bases de connaissances" spécifiques alimentées et consultées par les agents.
*   **`notifications`:** Une file d'attente pour les notifications à destination des utilisateurs.

L'intégrité et la richesse de `.pheromone`, en particulier de la `memoryBank`, conditionnent l'intelligence et l'efficacité d'AgilePheromind.

### 3.2. Les Fichiers de Configuration

*   **`.roomodes`:** Ce fichier JSON définit chaque agent IA spécialisé. Pour chaque agent, il spécifie :
    *   `slug`: Un identifiant unique (ex: `developer-agent`).
    *   `name`: Un nom lisible (ex: `💻 @developer-agent`).
    *   `roleDefinition`: Une description générale de son rôle et de ses responsabilités primaires.
    *   `customInstructions`: Des instructions détaillées et spécifiques sur la manière dont l'agent doit accomplir ses tâches, quels MCPs utiliser, comment interagir avec `.pheromone` (via le Scribe), quels formats de sortie produire, et comment gérer les erreurs. Ces instructions sont le "code source comportemental" de l'agent.
    *   `groups`: Permissions d'accès aux fichiers (lecture, écriture, exécution de commandes, utilisation de MCPs).
*   **`.swarmConfig`:** Ce fichier JSON contient principalement la `interpretationLogic` pour le `✍️ @orchestrator-pheromone-scribe`. Cette logique est une série de règles (conditions et actions) qui permettent au Scribe de :
    *   Parser les résumés en langage naturel (NL) des agents.
    *   Extraire des informations structurées (IDs, statuts, chemins de fichiers, décisions).
    *   Savoir quelles sections de `.pheromone` mettre à jour et comment.
    *   Par exemple, une règle peut stipuler : "Si le résumé de `@developer-agent` contient 'Commit effectué avec succès. Hash: {{commitHash}}.', alors ajouter `{{commitHash}}` à `memoryBank.userStories.{{activeUserStory.id}}.relatedCommits`."

### 3.3. Les Scripts de Workflow `01_AI-RUN/*.md`

Ces fichiers Markdown sont les "programmes" que l'`🧐 @uber-orchestrator` exécute. Chaque script correspond à une fonctionnalité de haut niveau d'AgilePheromind (ex: analyser un besoin, générer des tests, réviser une PR).

Un script type contient :

*   **Objectif du Workflow:** Ce que le script accomplit.
*   **Agents IA Clés Impliqués:** Quels agents spécialisés seront probablement sollicités.
*   **MCPs Potentiellement Utilisés.**
*   **Aperçu du Workflow Pheromind:** Les grandes phases logiques.
*   **Détails des Phases:** Pour chaque phase :
    *   L'agent responsable.
    *   Les inputs requis (souvent tirés de `.pheromone`).
    *   Les actions et outils (y compris les appels MCPs spécifiques).
    *   Comment la `memoryBank` ou l'état actif est impacté.
    *   L'output attendu (typiquement un résumé NL pour le Scribe et potentiellement des artefacts).

```mermaid
graph TD
    subgraph Workflow Execution
        UO["🧐 @uber-orchestrator"] -- Lit Instructions de --> ScriptMD["01_AI-RUN/script_X.md"]
        ScriptMD -- Décrit Phase A --> UO
        UO -- Délègue Phase A à --> AgentX["@agent-X"]
        AgentX -- Exécute Tâches de Phase A --> ResultA["Résultat & Résumé NL (Phase A)"]
        ResultA -- Envoyé à --> Scribe["✍️ @orchestrator-pheromone-scribe"]
        Scribe -- Met à jour --> PheromoneDB[".pheromone"]
        ScriptMD -- Décrit Phase B (après Phase A) --> UO
        UO -- Lit état mis à jour --> PheromoneDB
        UO -- Délègue Phase B à --> AgentY["@agent-Y"]
        AgentY -- Exécute Tâches de Phase B --> ResultB["Résultat & Résumé NL (Phase B)"]
        ResultB -- Envoyé à --> Scribe
        Scribe -- Met à jour --> PheromoneDB
    end
```

### 3.4. Les Agents IA Spécialisés

Chaque agent est un LLM configuré avec un rôle et des instructions spécifiques dans `.roomodes`. Ils sont conçus pour exceller dans un domaine particulier. Exemples :

*   `@po-assistant`: Spécialisé dans l'analyse des besoins, la rédaction d'US et d'ACs. Utilise le `Sequential Thinking MCP`.
*   `@developer-agent`: Spécialisé dans l'écriture de code .NET et Angular, la création de tests unitaires, l'utilisation de Git. Utilise `Context7 MCP`, `MSSQL MCP`, `Git Tools MCP`.
*   `@code-reviewer-assistant`: Spécialisé dans l'analyse de code pour la qualité, les conventions, et la coordination de l'analyse de sécurité.
*   `@deployment-agent-aks`: Spécialisé dans la gestion des déploiements sur AKS. Utilise des MCPs Docker, Kubernetes (conceptuels) et Azure DevOps.

La force du système réside dans la capacité de l'`🧐 @uber-orchestrator` à sélectionner et à coordonner le bon agent pour la bonne tâche, en fournissant le contexte nécessaire depuis `.pheromone`.

### 3.5. Les Model Context Protocol (MCP) Servers

Ce sont des services externes qui exposent des fonctionnalités spécifiques via une API structurée que les agents Pheromind peuvent appeler. AgilePheromind s'appuie sur :

*   **Azure DevOps MCP:** Pour toute interaction avec Azure Boards (US, tâches), Azure Repos (PRs), Azure Pipelines.
*   **Git Tools MCP:** Pour les opérations Git locales (commit, branch, diff, etc.).
*   **Context7 MCP:** Pour obtenir la documentation à jour des librairies et frameworks (.NET, Angular, etc.).
*   **MSSQL MCP:** Pour interagir avec la base de données SQL Server (lire schémas, valider/exécuter des requêtes).
*   **Browser Tools MCP (Puppeteer/Playwright):** Pour l'automatisation des tests UI et la validation visuelle.
*   **Sequential Thinking MCP:** Pour aider les agents à structurer leur pensée et leur analyse pour des tâches complexes (décomposition, planification).
*   **Fetch MCP:** En fallback pour scraper des documentations ou exemples si Context7 ne suffit pas.
*   **(Conceptuels/Futurs) Docker MCP, Kubernetes/AKS MCP, SonarQube MCP.**

L'utilisation de MCPs permet aux agents de manipuler directement les outils de l'équipe de développement.

## 4. Flux d'Information et Cycle de Vie d'une Tâche

Le diagramme suivant illustre un cycle de vie typique d'une commande utilisateur traitée par AgilePheromind :

```mermaid
sequenceDiagram
    participant User
    participant HO as 🎩 @head-orchestrator
    participant UO as 🧐 @uber-orchestrator
    participant SA as Agent Spécialisé
    participant Scribe as ✍️ @scribe
    participant Pheromone as .pheromone
    participant MCP

    User->>+HO: Commande (ex: "Démarre US Azure#123")
    HO->>+UO: Délègue (Script: 01_Start_User_Story.md, Params: Azure#123)
    UO->>Pheromone: Lit État Actuel
    Pheromone-->>UO: État Actuel
    UO->>ScriptMD: Lit Phase 1 du Script
    UO->>+SA: Délègue Tâche Phase 1 (ex: @devops-connector lit US)
    SA->>+MCP: Appel MCP (ex: Azure DevOps MCP get_work_item_details)
    MCP-->>-SA: Réponse MCP
    SA->>-UO: (Indirectement via Scribe) Résumé NL de la Phase 1
    Note right of SA: Agent envoie résumé au Scribe
    UO->>+Scribe: (Activation implicite du Scribe par le résumé de SA)
    Scribe->>Pheromone: Lit État Actuel
    Pheromone-->>Scribe: État Actuel
    Scribe->>SwarmConfig: Lit InterpretationLogic
    SwarmConfig-->>Scribe: Logique
    Scribe->>Pheromone: Met à Jour État (ex: activeUserStory, memoryBank)
    Pheromone-->>Scribe: Confirmation Mise à Jour
    Scribe->>-UO: (Indirectement, en activant HO qui réactive UO avec état frais)
    Note right of Scribe: Scribe active HO, qui réactive UO
    UO->>Pheromone: Lit Nouvel État
    Pheromone-->>UO: Nouvel État
    UO->>ScriptMD: Lit Phase 2 du Script
    UO->>+SA: Délègue Tâche Phase 2 (ex: @task-estimator décompose)
    %% ... Cycle continue pour les autres phases ...
    SA-->>-UO: (Indirectement) Résumé NL Final
    UO->>+Scribe: (Activation implicite)
    Scribe->>Pheromone: Mise à Jour Finale de l'État
    Scribe->>-UO: (Indirectement)
    UO-->>-HO: Fin du Workflow Script
    HO-->>-User: (Optionnel) Confirmation/Résultat Final
```

## 5. La Memory Bank : Le Cœur Apprenant

La `memoryBank` dans `.pheromone` est plus qu'un simple log. C'est une base de connaissances structurée qui permet au système de :

*   **Conserver le Contexte à Long Terme:** Se souvenir des décisions architecturales passées, des estimations de tâches similaires, des solutions à des problèmes récurrents.
*   **Apprendre des Interactions:** Par exemple, `@task-breakdown-estimator` peut potentiellement affiner ses estimations futures en comparant les estimations passées (stockées) avec le temps réel passé (si cette donnée est remontée).
*   **Fournir un Contexte Riche aux Agents:** Avant de démarrer une tâche, un agent peut consulter la `memoryBank` pour des informations pertinentes sur l'US, les tâches sœurs, les conventions, etc.
*   **Assurer la Cohérence:** Les conventions de codage et de design y sont référencées, garantissant que tous les agents (et les humains) travaillent à partir des mêmes standards.
*   **Suivre l'Évolution:** Historique des statuts, des reviews, des déploiements.

La qualité des informations stockées dans la `memoryBank` et la capacité du `✍️ @orchestrator-pheromone-scribe` à l'alimenter et à l'exploiter correctement sont déterminantes pour l'intelligence globale d'AgilePheromind.

## 6. Personnalisation et Évolution

AgilePheromind est conçu pour être adaptable :

*   **`.roomodes`:** Les instructions des agents peuvent être affinées, et de nouveaux agents peuvent être ajoutés pour couvrir de nouveaux besoins.
*   **`01_AI-RUN/*.md`:** De nouveaux workflows peuvent être créés pour automatiser ou assister de nouvelles tâches. Les workflows existants peuvent être modifiés.
*   **`.swarmConfig`:** La logique d'interprétation du Scribe peut être étendue pour comprendre de nouveaux types de résumés ou pour enrichir la `memoryBank` de manière plus fine.
*   **MCPs:** De nouveaux MCPs peuvent être intégrés pour connecter AgilePheromind à d'autres outils ou services.

Le système peut ainsi évoluer avec les processus et les technologies de l'équipe. Des agents comme `@workflow-optimizer-agent` et `@swarm-monitor-agent` sont spécifiquement conçus pour aider à cette évolution en analysant les performances du système et en suggérant des améliorations.

## Conclusion

AgilePheromind représente une approche sophistiquée de l'assistance IA pour les équipes Agiles. En combinant une architecture d'essaim flexible, une gestion d'état centralisée et intelligente, et une intégration profonde avec les outils existants, il a le potentiel de transformer la manière dont les équipes développent des logiciels. Son succès repose sur la clarté des workflows définis, la précision des instructions des agents, et l'intelligence du mécanisme d'interprétation et de mise à jour de la `memoryBank`.