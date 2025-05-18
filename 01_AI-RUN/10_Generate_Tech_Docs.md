# Workflow: Génération de Documentation Technique (10_Generate_Tech_Docs.md)

**Objectif:** Générer ou mettre à jour la documentation technique pour un module, une fonctionnalité, ou une API spécifique du projet. L'agent analyse le code source, les commentaires, les spécifications (US/tâches), les conventions de documentation, et s'appuie sur un contexte riche injecté par l'UO. Il doit gérer les cas où les informations sources sont incomplètes ou ambiguës.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@documentation-writer-agent`, `@devops-connector` (pour contexte US/tâche), `@clarification-agent`.

**MCPs Utilisés:** Git Tools MCP, Context7 MCP, Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Dev/Tech Lead) demande la documentation pour une cible (ex: `"AgilePheromind documente module OrderService"`). Peut être déclenché post-commit.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Définition du Périmètre et Injection de Contexte Approfondi.**
        *   UO identifie les fichiers sources pertinents (via nom ou commits liés à US/tâche avec **Git Tools MCP** / **Azure DevOps MCP**).
        *   UO **injecte un contexte riche** à `@documentation-writer-agent` : code source, commentaires, spécifications de l'US/tâche (depuis `.pheromone.memoryBank`), conventions de documentation (`documentationRegistry`), documentation de librairies similaires (si existante dans `memoryBank`).
        *   UO évalue si le contexte est suffisant. Si ambiguïté (ex: code non commenté et specs vagues), UO peut initier clarification via `@clarification-agent`.
        *   **onError:** Si code source inaccessible, notifier et arrêter.
    *   **Phase 2: Analyse des Informations Sources par l'Agent.**
        *   UO délègue à `@documentation-writer-agent`. L'agent analyse le code, les commentaires, les specs. Utilise **Context7 MCP** pour détails sur APIs externes utilisées.
    *   **Phase 3: Structuration et Rédaction du Document Technique (avec "Chaîne de Pensée").**
        *   UO délègue à `@documentation-writer-agent`. L'agent planifie sections, rédige contenu, inclut exemples, diagrammes Mermaid. **Doit documenter la "chaîne de pensée"** pour les explications de logique complexe ou les choix de structuration de la doc.
        *   **onError/Ambiguïté Persistante:** Si l'agent ne peut documenter clairement une section, il le signale à l'UO qui peut relancer clarification ou demander au dev de commenter le code.
    *   **Phase 4: Enregistrement et Notification.**
        *   Scribe enregistre le document et met à jour `.pheromone`.

## Détails des Phases:

### Phase 1: Définition du Périmètre et Injection de Contexte Approfondi
*   **Agent Responsable:** `🧐 @uber-orchestrator`, `@devops-connector`, `@clarification-agent`.
*   **Inputs:** Cible de documentation (module, US ID, etc.).
*   **Actions & Tooling (UO):**
    1.  **Identifier Code Source:**
        *   Si nom de module/classe: Localiser fichiers.
        *   Si ID US/tâche: Utiliser `@devops-connector` (**Azure DevOps MCP** `get_work_item_linked_commits`) pour trouver commits, puis **Git Tools MCP** (`get_commit_changed_files`) pour identifier fichiers.
        *   **onError (Git/ADO MCP):** Si échec, logguer, notifier, arrêter.
    2.  **Récupérer Code et Commentaires (Git Tools MCP `get_file_contents`).**
    3.  **Injecter Contexte de `.pheromone`:**
        *   Description/ACs de l'US/tâche (`memoryBank.userStories/tasks`).
        *   `memoryBank.projectContext.codingConventionsLink` (pour standards de doc).
        *   (Optionnel) Extraits de `memoryBank.architecturalDecisions` ou `design_conventions.md` pertinents pour le module.
        *   (Optionnel) Exemples de documentation de modules similaires déjà dans `documentationRegistry`.
    4.  **Évaluation de Clarté et Clarification:**
        *   Si le code est minimalement commenté ET les specs fonctionnelles sont vagues pour la cible :
            *   Mettre workflow en pause (`activeWorkflow.status: 'PendingClarification_TechDoc'`).
            *   Déléguer à `@clarification-agent` avec le contexte et une question pour le développeur/PO (ex: "Le module `OrderService` manque de commentaires et les ACs de l'US#XYZ sont généraux. Pouvez-vous décrire le rôle principal des méthodes A, B et leurs interactions attendues pour la documentation ?").
            *   Attendre réponse via `01_AI-RUN/XX_Handle_Clarification_Response.md`.
    5.  Si clair, déléguer à `@documentation-writer-agent` avec le code et le contexte injecté (y compris clarifications).
*   **Output:** Tâche déléguée à `@documentation-writer-agent` avec contexte riche, ou workflow en pause.

### Phase 2: Analyse des Informations Sources par l'Agent
*   **Agent Responsable:** `@documentation-writer-agent`.
*   **Inputs (Injectés par l'UO):** Code source, commentaires, specs US/tâche, conventions, docs de libs similaires, clarifications.
*   **Actions & Tooling:**
    1.  Analyser en détail le code (signatures publiques, logique principale).
    2.  Extraire et interpréter les commentaires existants.
    3.  Corréler le code avec les specs fonctionnelles pour comprendre l'intention.
    4.  Si le code utilise des APIs externes ou des librairies .NET/Angular de manière complexe, utiliser **Context7 MCP** (`get_library_docs`) pour s'assurer de la compréhension correcte de leur usage.
*   **Output (interne):** Compréhension approfondie du code à documenter.

### Phase 3: Structuration et Rédaction du Document Technique (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@documentation-writer-agent`.
*   **Inputs:** Analyse (Phase 2). Type de doc attendu.
*   **Actions & Tooling:**
    1.  **Choisir Modèle / Planifier Sections:** (Référence API, Guide Module, etc.).
    2.  **Rédiger Contenu:** Langage clair, exemples de code, diagrammes Mermaid si utiles.
    3.  **"Chaîne de Pensée":** Pour les sections expliquant une logique complexe ou des choix de design importants dans le module, l'agent doit inclure une brève explication de *comment* il a compris cette logique à partir du code et des specs (ex: "La méthode `ProcessOrder` semble gérer X, Y, Z basé sur la condition A dans le code et l'AC B. Le flux typique est..."). Ceci sera intégré dans le document généré.
    4.  **Mise en Forme Markdown, Relecture.**
    5.  **onError/Ambiguïté Persistante:** Si une partie du code reste obscure même après la phase de clarification (ou si aucune clarification n'a été demandée mais s'avère nécessaire):
        *   L'agent documente ce qu'il peut et signale clairement la section ambiguë dans son rapport et dans le document lui-même (ex: `<!-- AMBIGUITY: Logic for XYZ unclear, needs dev input -->`).
        *   Il remonte cette information à l'UO. L'UO peut alors demander une nouvelle clarification ciblée ou notifier le Tech Lead.
    6.  Nommer et sauvegarder le fichier dans `02_AI-DOCS/` (ex: `Technical/Modules/[ModuleName].md`).
*   **Output (vers Scribe):** Résumé NL: "Doc technique pour `{{TargetName}}` [générée/màj] à `{{FilePath}}`. Contient [description]. [Optionnel: Section X signalée comme ambiguë]. Chaîne de pensée pour les logiques clés incluse."

### Phase 4: Enregistrement et Notification
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** Résumé NL de `@documentation-writer-agent`.
*   **Actions:**
    1.  Interpréter via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter/MàJ `{{FilePath}}`.
        *   `memoryBank.tasks.{{taskId_if_contextual}}.relatedDocumentation[]`: Ajouter `{{FilePath}}`.
        *   `memoryBank.modules.{{ModuleName}}.documentationPath`: Lier la doc au module.
        *   `memoryBank.modules.{{ModuleName}}.reasoningChainLinks.documentation`: Peut pointer vers le document lui-même si la chaîne de pensée y est intégrée.
*   **Output:** `.pheromone` mis à jour. UO informé.

---