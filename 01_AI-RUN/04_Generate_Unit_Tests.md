# Workflow: Génération de Squelettes de Tests Unitaires (04_Generate_Unit_Tests.md)

**Objectif:** Assister le développeur en générant des squelettes de tests unitaires pour une méthode de classe C# (.NET) ou une méthode/fonction de service/composant TypeScript (Angular). L'agent se base sur le code source, les spécifications fonctionnelles (ACs de l'US/tâche), les conventions de test du projet, et documente sa "chaîne de pensée" pour la sélection des cas de test.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@test-generator-agent`, `@clarification-agent`.

**MCPs Utilisés:** Git Tools MCP (pour lire le code source cible), Context7 MCP (pour la documentation des frameworks de test .NET/Angular et des librairies de mocking), Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Le développeur (Dev) spécifie la cible pour la génération de tests (ex: `"AgilePheromind génère tests unitaires pour la méthode calculatePrice dans OrderService"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Injection de Contexte et Analyse Préliminaire.**
        *   UO récupère le code source cible (via **Git Tools MCP**) et le contexte pertinent de `.pheromone` (`activeTask`, `activeUserStory` ACs, `memoryBank.toolingConfigurations.testFrameworks`, `coding_conventions.md`).
        *   UO évalue si le code source ou les specs sont suffisamment clairs pour `@test-generator-agent`. Si ambigu, UO peut initier une clarification via `@clarification-agent`.
        *   UO délègue à `@test-generator-agent` avec le code et le contexte injecté.
        *   **onError:** Si Git Tools MCP échoue ou si le code est introuvable, notifier l'utilisateur et arrêter.
    *   **Phase 2: Analyse du Code Cible, Spécifications, et Framework de Test par l'Agent.**
        *   `@test-generator-agent` analyse la cible, ses dépendances, les ACs. Utilise **Context7 MCP** pour les docs des frameworks de test/mocking.
    *   **Phase 3: Identification Méthodique des Cas de Test avec "Chaîne de Pensée".**
        *   `@test-generator-agent` utilise **Sequential Thinking MCP** et doit **documenter sa "chaîne de pensée"** pour la sélection des cas.
        *   **onError:** Si l'agent ne peut pas déduire les cas de test à cause d'ambiguïtés persistantes, il le signale à l'UO. L'UO peut relancer une clarification ou notifier le développeur.
    *   **Phase 4: Génération des Squelettes de Fichiers et Méthodes de Test.**
        *   `@test-generator-agent` génère les fichiers.
    *   **Phase 5: Enregistrement et Rapport.**
        *   Scribe enregistre les informations (y compris le lien vers le rapport des scénarios/chaîne de pensée) dans `.pheromone`.

## Détails des Phases:

### Phase 1: Injection de Contexte et Analyse Préliminaire
*   **Agent Responsable:** `🧐 @uber-orchestrator` (pour la collecte et l'évaluation), `@clarification-agent` (si besoin).
*   **Inputs:** Nom de la méthode/classe/composant/service et nom de la méthode/fonction spécifique à tester (de l'utilisateur).
*   **Actions & Tooling (UO):**
    1.  **Localiser et Lire le Code Cible:**
        *   Identifier le chemin du fichier source.
        *   Utiliser **Git Tools MCP** (`get_file_contents {filePath}`) pour lire le code.
        *   **onError (Git Tools MCP):** Si échec, logguer via Scribe, notifier l'utilisateur "Impossible de lire le fichier source [filePath]. Vérifiez le chemin et les droits.", arrêter workflow.
    2.  **Collecter le Contexte de `.pheromone`:**
        *   `activeTask.description` et `activeUserStory.acceptanceCriteria`.
        *   `memoryBank.toolingConfigurations.testFrameworks` (.NET et Angular).
        *   Lien vers `coding_conventions.md` depuis `memoryBank.projectContext`.
        *   (Optionnel) Tests existants pour des méthodes/composants similaires dans `memoryBank.tasks.{{any_task}}.testCasesGenerated`.
    3.  **Évaluation de Clarté:**
        *   L'UO effectue une première évaluation : le code source est-il présent et lisible ? Les ACs sont-ils suffisamment précis pour guider la génération de tests ?
        *   **Si ambiguïté détectée** (ex: code source commenté massivement, ACs trop vagues pour déduire des comportements testables):
            *   Mettre workflow en pause (`activeWorkflow.status: 'PendingClarification_TestGen'`).
            *   Déléguer à `@clarification-agent` avec le contexte et une question pour le développeur (ex: "Le code de la méthode `calculatePrice` semble incomplet. Pouvez-vous fournir une version plus finalisée ou clarifier son comportement attendu pour [cas X] ?").
            *   Attendre réponse via `01_AI-RUN/XX_Handle_Clarification_Response.md`.
    4.  Si clair, déléguer à `@test-generator-agent` en injectant le code source et tout le contexte collecté.
*   **Output:** Si clair, tâche déléguée à `@test-generator-agent` avec contexte riche. Si ambigu, workflow en pause.

### Phase 2: Analyse du Code Cible, Spécifications, et Framework de Test par l'Agent
*   **Agent Responsable:** `@test-generator-agent`
*   **Inputs (Injectés par l'UO):** Code source cible, ACs de l'US/tâche, framework de test à utiliser, conventions de codage.
*   **Actions & Tooling:**
    1.  Analyser la signature de la méthode/fonction, les dépendances injectées, la logique interne.
    2.  Utiliser **Context7 MCP** (`get_library_docs`) pour la documentation du framework de test spécifique (ex: MSTest, Jasmine) et des librairies de mocking (ex: Moq, `jasmine.createSpyObj`).
*   **Output (interne à `@test-generator-agent`):** Compréhension approfondie de la cible et des outils de test.

### Phase 3: Identification Méthodique des Cas de Test avec "Chaîne de Pensée"
*   **Agent Responsable:** `@test-generator-agent`
*   **Inputs:** Analyse de la Phase 2.
*   **Actions & Tooling:**
    1.  Utiliser **Sequential Thinking MCP** pour identifier les cas (nominaux, limites, erreurs, dépendances, états UI pour Angular).
    2.  **Documenter la "Chaîne de Pensée":** Pour chaque cas de test majeur identifié, expliquer brièvement *pourquoi* il a été sélectionné (ex: "Ce cas teste la condition limite X identifiée dans l'AC Y", "Ce cas vérifie le comportement si la dépendance Z retourne une erreur, ce qui est un scénario d'échec attendu"). Cette "chaîne de pensée" sera incluse dans le rapport de scénarios.
*   **onError Strategy (pour l'agent, à signaler à l'UO):**
    1.  Si, malgré le contexte fourni, il est impossible de déduire des cas de test pertinents (ex: logique trop complexe et non commentée, ACs toujours vagues):
        *   L'agent formule le point de blocage précis.
        *   L'UO peut alors relancer une clarification via `@clarification-agent` ou notifier le développeur qu'une intervention manuelle est nécessaire pour définir les cas de test.
*   **Output (interne à `@test-generator-agent`):** Liste structurée de cas de test et leur justification ("chaîne de pensée").

### Phase 4: Génération des Squelettes de Fichiers et Méthodes de Test
*   **Agent Responsable:** `@test-generator-agent`
*   **Inputs:** Liste des cas de test et leur justification (Phase 3). Connaissance du framework de test.
*   **Actions & Tooling:**
    1.  Déterminer nom et emplacement du fichier de test. Le créer ou l'ouvrir.
    2.  Pour chaque cas de test, générer la méthode de test (avec nom descriptif, commentaires Arrange/Act/Assert), suggérer initialisations, mocks, et assertions pertinentes.
    3.  Sauvegarder le fichier de test.
    4.  Compiler le rapport des scénarios/chaîne de pensée (`unit_tests_scenarios_{{targetName}}_{{timestamp}}.md`) dans `03_SPECS/Test_Scenarios/`.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Squelettes de [Nombre] tests unitaires générés pour `{{targetName}}` dans `{{TestFilePath}}`. Rapport des scénarios (avec chaîne de pensée) disponible à `unit_tests_scenarios_{{targetName}}_{{timestamp}}.md`. Le développeur doit compléter les sections et finaliser les assertions."

### Phase 5: Enregistrement et Rapport
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@test-generator-agent`.
*   **Actions & Tooling:**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter `{{TestFilePath}}` et `unit_tests_scenarios_{{targetName}}_{{timestamp}}.md`.
        *   `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated`: Ajouter `{ path: "{{TestFilePath}}", scenarioReportPath: "unit_tests_scenarios_{{targetName}}_{{timestamp}}.md", description: "Squelettes de tests unitaires avec chaîne de pensée pour les scénarios.", timestamp: "{{timestamp}}" }`.
        *   `memoryBank.tasks.{{activeTask.id}}.reasoningChainLinks.unitTestGeneration`: Lier au rapport des scénarios.
*   **Output:** `.pheromone` mis à jour. UO informé.

---