# Workflow: Génération de Documentation Technique (10_Generate_Tech_Docs.md)

**Objectif:** Générer ou mettre à jour la documentation technique pour un module, une fonctionnalité, ou une API spécifique du projet. L'agent analyse le code source, les commentaires, les spécifications (US/tâches), et les conventions de documentation du projet pour produire un document clair, précis et utile pour les développeurs.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@documentation-writer-agent`.

**MCPs Utilisés:** Git Tools MCP (pour lire le code source et les commentaires), Context7 MCP (pour la documentation des librairies et frameworks utilisés dans le code à documenter), Azure DevOps MCP (pour récupérer les descriptions d'US/tâches liées).

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Dev/Tech Lead) demande la génération de documentation pour une cible spécifique (ex: `"AgilePheromind documente le module OrderService"` ou `"AgilePheromind màj doc pour US Azure#12323 après implémentation"`). Le déclenchement peut aussi être automatisé (ex: post-commit sur certaines branches).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Collecte et Analyse des Informations Sources.**
        *   UO délègue à `@documentation-writer-agent`. L'agent récupère le code source, les commentaires, les spécifications de l'US/tâche (via **Azure DevOps MCP** si ID fourni), et les conventions de documentation.
    *   **Phase 2: Structuration du Document Technique.**
        *   UO délègue à `@documentation-writer-agent`. L'agent planifie les sections du document.
    *   **Phase 3: Rédaction et Génération du Document.**
        *   UO délègue à `@documentation-writer-agent`.
    *   **Phase 4: Enregistrement et Notification.**
        *   Scribe enregistre le document et met à jour `.pheromone`.

## Détails des Phases:

### Phase 1: Collecte et Analyse des Informations Sources
*   **Agent Responsable:** `@documentation-writer-agent`
*   **Inputs:** Cible de la documentation (nom du module, de la classe, de la fonctionnalité, ou ID d'US/tâche) fournie par l'UO. Accès au code source (`memoryBank.currentProject.repositoryUrl`) et à `.pheromone`.
*   **Actions & Tooling:**
    1.  **Identifier le Périmètre du Code:**
        *   Si un nom de module/classe est fourni, localiser les fichiers sources pertinents.
        *   Si un ID d'US/tâche est fourni, utiliser `@devops-connector` (via **Azure DevOps MCP** `get_work_item_details` et potentiellement `get_work_item_linked_commits`) pour identifier les fichiers de code impactés par les récents commits liés à cette US/tâche.
    2.  **Lire le Code Source et les Commentaires:**
        *   Utiliser **Git Tools MCP** (`get_file_contents {filePath}`) pour chaque fichier source identifié.
        *   Analyser les commentaires existants (ex: XML Docs pour C#/.NET, JSDoc/TSDoc pour TypeScript/Angular) pour extraire les descriptions de classes, méthodes, paramètres, retours.
    3.  **Consulter les Spécifications Fonctionnelles:**
        *   Si un ID d'US/tâche est pertinent, lire sa description et ses ACs depuis `.pheromone.memoryBank.userStories` ou `memoryBank.tasks`.
    4.  **Analyser les Dépendances et API Utilisées:**
        *   Identifier les principales librairies ou frameworks .NET/Angular utilisés dans le code cible.
        *   Utiliser **Context7 MCP** (`get_library_docs`) pour obtenir des informations précises sur ces dépendances si nécessaire pour expliquer leur usage dans la documentation.
    5.  **Conventions de Documentation:**
        *   Consulter `memoryBank.projectContext.codingConventionsLink` (qui devrait pointer vers `02_AI-DOCS/Conventions/coding_conventions.md`) pour les standards de documentation du projet.
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.userStories`, `memoryBank.tasks` (pour contexte fonctionnel), `memoryBank.projectContext.codingConventionsLink`, `memoryBank.currentProject.repositoryUrl`.
*   **Output (interne à `@documentation-writer-agent`):** Ensemble d'informations brutes et contextuelles sur le code à documenter.

### Phase 2: Structuration du Document Technique
*   **Agent Responsable:** `@documentation-writer-agent`
*   **Inputs:** Informations collectées en Phase 1. Type de documentation à produire (ex: Référence API, Guide d'utilisation de module, Architecture de fonctionnalité).
*   **Actions & Tooling:**
    1.  **Choisir un Modèle de Document:**
        *   Basé sur le type de documentation, sélectionner ou adapter un modèle (peut être défini dans `02_AI-DOCS/Templates/` ou être une structure standard connue par l'agent).
    2.  **Planifier les Sections Principales:**
        *   **Pour une Référence API (.NET Service / Angular Service):** Introduction, Endpoints/Méthodes Publiques (avec paramètres, retours, exceptions), Types de Données, Exemples d'utilisation, Authentification/Autorisation (si applicable).
        *   **Pour un Module/Composant:** Objectif du module, Architecture interne (si complexe, avec diagramme Mermaid), Interfaces publiques, Configuration, Dépendances, Guide d'utilisation, Exemples.
        *   **Pour une Fonctionnalité (basée sur US):** Objectif de la fonctionnalité (rappel de l'US), Flux utilisateur principal, Composants techniques impliqués, Points d'intégration, Configuration spécifique.
*   **Memory Bank Interaction:**
    *   Aucune lecture directe, mais s'appuie sur le type de cible.
*   **Output (interne à `@documentation-writer-agent`):** Plan détaillé du document à générer.

### Phase 3: Rédaction et Génération du Document
*   **Agent Responsable:** `@documentation-writer-agent`
*   **Inputs:** Plan du document (Phase 2), informations collectées (Phase 1).
*   **Actions & Tooling:**
    1.  **Rédiger Chaque Section:**
        *   Utiliser un langage clair, précis et technique.
        *   Transformer les commentaires de code en descriptions lisibles.
        *   Expliquer la logique métier ou le fonctionnement technique.
        *   Fournir des exemples de code pertinents pour illustrer l'utilisation des APIs ou des composants.
        *   Si des diagrammes (ex: flux, séquence, composants via Mermaid) sont nécessaires, les générer.
    2.  **Mise en Forme:**
        *   Utiliser Markdown.
        *   Respecter les conventions de mise en forme (titres, listes, blocs de code).
    3.  **Relecture et Correction:**
        *   Vérifier la cohérence, l'exactitude et la complétude.
        *   Corriger les erreurs grammaticales ou typographiques.
    4.  **Nommage et Sauvegarde:**
        *   Déterminer le chemin de sauvegarde approprié dans `02_AI-DOCS/` (ex: `02_AI-DOCS/Technical/APIReferences/[ServiceName].md`, `02_AI-DOCS/Technical/Modules/[ModuleName].md`, `02_AI-DOCS/Features/[FeatureNameOrUSID].md`).
        *   Sauvegarder le fichier.
*   **Memory Bank Interaction:**
    *   Le chemin du document sera enregistré par le Scribe.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Documentation technique pour `{{TargetName}}` générée/mise à jour. Le document `{{FilePath}}` contient : [brève description du contenu, ex: référence API, guide d'utilisation du module]. Qualité estimée: [Bonne/Correcte, points à améliorer si connus]."

### Phase 4: Enregistrement et Notification
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@documentation-writer-agent`.
*   **Actions & Tooling:**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter ou mettre à jour l'entrée pour `{{FilePath}}` avec sa description et son type (ex: `APIRefDoc`, `ModuleGuideDoc`).
        *   `memoryBank.tasks.{{activeTask.id_if_contextual}}.relatedDocumentation`: Ajouter `{{FilePath}}`.
        *   `memoryBank.userStories.{{activeUserStory.id_if_contextual}}.relatedDocumentation`: Ajouter `{{FilePath}}`.
*   **Memory Bank Interaction:**
    *   Écriture: Enregistrement du nouveau document et de ses liens avec les tâches/US.
*   **Output:** `.pheromone` mis à jour. L'UO est informé que la documentation est prête.

---