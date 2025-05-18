# Workflow: Continuer une Tâche de Développement (02_Continue_Task.md)

**Objectif:** Permettre à un développeur de reprendre efficacement le travail sur une tâche technique spécifique. Le système charge le contexte de la tâche depuis `.pheromone` (Memory Bank et état actif), prépare l'environnement Git, et fournit les outils et informations nécessaires (documentation via Context7, schéma DB via MSSQL MCP) pour la poursuite de l'implémentation.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@devops-connector`, `@developer-agent`.

**MCPs Utilisés:** Azure DevOps MCP, Git Tools MCP, Context7 MCP, MSSQL MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Dev) spécifie l'ID de la tâche Azure DevOps à continuer (ex: `"AgilePheromind continue tâche Azure#23223"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Identification Utilisateur & Récupération/Validation Contexte Tâche.**
        *   UO délègue à `@devops-connector` pour confirmer l'utilisateur et récupérer/valider les détails de la tâche depuis Azure DevOps.
    *   **Phase 2: Activation de la Tâche et Chargement du Contexte Approfondi depuis `.pheromone`.**
        *   Scribe met à jour `activeTask` et `activeUserStory` dans `.pheromone`.
        *   `@developer-agent` lit les informations contextuelles (notes précédentes, décisions, snippets) de la `memoryBank`.
    *   **Phase 3: Préparation de l'Environnement de Développement et Assistance à l'Implémentation.**
        *   UO délègue à `@developer-agent` pour vérifier/changer la branche Git, ouvrir les fichiers, et fournir une assistance contextuelle pendant que le développeur code.

## Détails des Phases:

### Phase 1: Identification Utilisateur & Récupération/Validation Contexte Tâche
*   **Agent Responsable:** `@devops-connector`
*   **Inputs:** ID de la tâche (ex: `Azure#23223`) fourni par l'UO. `currentUser` depuis `.pheromone`.
*   **Actions & Tooling:**
    1.  Utiliser **Azure DevOps MCP**:
        *   `get_user_identity`: Confirmer l'identité de `currentUser.azureDevOpsUsername`.
        *   `get_work_item_details {id: ID_Tache}`: Récupérer le titre, la description complète, l'état actuel dans Azure DevOps, l'US parente (ID et titre), l'assigné actuel.
*   **Memory Bank Interaction (via Scribe après résumé):**
    *   Le Scribe vérifiera si les informations d'Azure DevOps correspondent à celles de la `memoryBank` pour cette tâche et son US parente. Des divergences pourraient signaler des mises à jour manuelles dans Azure DevOps.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Utilisateur '{{currentUser.azureDevOpsUsername}}' confirmé. Détails pour tâche 'Azure#{{taskId}}' ('{{taskTitle}}') récupérés. US Parente: 'Azure#{{parentId}} - {{parentTitle}}'. État Azure: '{{taskAzureStatus}}'. Assigné Azure: '{{taskAzureAssignee}}'. Détails loggés: `azure_wi_{{taskId}}_{{timestamp}}.json`." (Log dans `03_SPECS/AzureDevOps_Logs/`).

### Phase 2: Activation de la Tâche et Chargement du Contexte Approfondi depuis `.pheromone`
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe` (pour la mise à jour de l'état actif), suivi de `@developer-agent` (pour la lecture de la Memory Bank).
*   **Inputs:** Résumé NL de `@devops-connector`. `currentUser` et `currentProject` depuis `.pheromone`.
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `activeUserStory`: Si la `parentId` de la tâche diffère de `activeUserStory.id`, charger les informations de l'US parente depuis `memoryBank.userStories` dans `activeUserStory`.
        *   `activeTask`: { `id`: "Azure#{{taskId}}", `title`: "{{taskTitle}}", `status`: "InProgressByPheromind", `parentId`: "Azure#{{parentId}}" }.
        *   Mettre à jour la tâche spécifique dans `activeUserStory.tasks` avec `status: "InProgressByPheromind"` et `assignee: currentUser.id`.
        *   `memoryBank.tasks.{{taskId}}`:
            *   S'assurer que l'entrée existe.
            *   `azureStatus`: "{{taskAzureStatus}}".
            *   `azureAssignee`: "{{taskAzureAssignee}}".
            *   `statusHistory`: Ajouter { `status`: "InProgressByPheromind", `timestamp`: "{{timestamp}}", `actor`: "AgilePheromindSystem", `developer`: "{{currentUser.id}}" }.
        *   `documentationRegistry`: Ajouter `azure_wi_{{taskId}}_{{timestamp}}.json`.
*   **Actions & Tooling (`@developer-agent`):**
    1.  Lire `.pheromone.activeTask` pour les informations de base.
    2.  Lire `memoryBank.tasks.{{activeTask.id}}` pour récupérer:
        *   `developerNotes`: Toutes notes ou décisions antérieures.
        *   `codeSnippets`: Tous snippets de code sauvegardés.
        *   `testCasesGenerated`: Chemins vers les fichiers de tests liés.
        *   `relatedCommit`: Le dernier commit lié à cette tâche (s'il y en a un).
    3.  Lire `memoryBank.userStories.{{activeTask.parentId}}` pour le contexte de l'US (ACs, description complète).
    4.  Lire `memoryBank.projectContext` pour les conventions de codage, la stack technique.
*   **Memory Bank Interaction:**
    *   Lecture extensive par `@developer-agent` des sections `tasks`, `userStories`, et `projectContext`.
*   **Output (`@developer-agent` vers `✍️ @orchestrator-pheromone-scribe` *après une session de travail*):** Résumé NL: "Session de travail sur tâche 'Azure#{{taskId}}'. Avancement: [Description du travail effectué, défis rencontrés, solutions explorées]. Fichiers modifiés: [Liste des chemins]. Points de décision pour Memory Bank: [décision technique X prise pour raison Y]. Problèmes nécessitant une discussion: [Liste]."

### Phase 3: Préparation de l'Environnement de Développement et Assistance à l'Implémentation
*   **Agent Responsable:** `@developer-agent`
*   **Inputs:** Contexte chargé depuis `.pheromone` (Memory Bank et état actif).
*   **Actions & Tooling:**
    1.  **Gestion de Branche Git:**
        *   Utiliser **Git Tools MCP** (`get_current_branch`).
        *   Vérifier si la branche actuelle correspond à la branche attendue pour `activeUserStory.id` (ex: `feature/US{{activeUserStory.id}}-description`).
        *   Si non, utiliser `checkout_branch` pour changer vers la branche correcte. Si la branche n'existe pas localement mais existe sur le remote, `pull_branch` puis `checkout_branch`. Si elle n'existe nulle part, la créer (`create_branch`) et informer l'utilisateur.
    2.  **Ouverture des Fichiers:**
        *   Identifier les fichiers de code principaux liés à `activeTask.id` (basé sur `memoryBank.tasks.{{activeTask.id}}.affectedFiles` ou analyse du titre/description de la tâche).
        *   (Conceptuel) Demander à l'IDE (via intégration Roo Code) d'ouvrir ces fichiers.
    3.  **Assistance Contextuelle à l'Implémentation:**
        *   Pendant que le développeur code :
            *   Sur demande, utiliser **Context7 MCP** (`get_library_docs {libraryName, topic}`) pour fournir de la documentation ciblée sur des librairies .NET (ex: Entity Framework, ASP.NET Core Identity) ou Angular (ex: RxJS, NgModules, HttpClient).
            *   Sur demande, si la tâche implique des interactions DB : utiliser **MSSQL MCP** (`get_schema_details {tableName}`, `validate_sql_query {query}`, `get_stored_procedure_definition {procName}`) pour aider à écrire/valider des requêtes ou comprendre le schéma.
            *   Sur demande, interagir avec `@test-generator-agent` pour créer/mettre à jour des tests unitaires.
            *   Répondre à des questions techniques en se basant sur `memoryBank.projectContext`, `memoryBank.commonIssuesAndSolutions`, et la documentation récupérée.
    4.  **Suivi du Travail et Prise de Notes pour la Memory Bank:**
        *   (Conceptuel) L'agent observe (ou est informé par le développeur) des décisions techniques clés, des problèmes résolus, des snippets de code utiles. Ces informations seront incluses dans son résumé pour le Scribe.
*   **Memory Bank Interaction (via Scribe après résumé de l'agent):**
    *   Écriture: Le Scribe mettra à jour `memoryBank.tasks.{{activeTask.id}}` avec `developerNotes` (nouvelles notes, décisions), `affectedFiles`, `codeSnippets` (si pertinent). Si un commit est fait pendant la session, le `relatedCommit` sera mis à jour.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe` à la fin d'une session de travail ou sur demande explicite de "sauvegarde de contexte"):** Résumé NL détaillé comme décrit à la fin de la Phase 2, Actions & Tooling (`@developer-agent`).

---