# Workflow: Génération de Squelettes de Tests Unitaires (04_Generate_Unit_Tests.md)

**Objectif:** Assister le développeur en générant des squelettes de tests unitaires pour une méthode de classe C# (.NET) ou une méthode/fonction de service/composant TypeScript (Angular). L'agent se base sur le code source, les spécifications fonctionnelles (ACs de l'US/tâche), et les meilleures pratiques de test pour la stack technologique concernée.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@test-generator-agent`.

**MCPs Utilisés:** Git Tools MCP (pour lire le code source cible), Context7 MCP (pour la documentation des frameworks de test .NET/Angular et des librairies de mocking), Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Le développeur (Dev) spécifie la cible pour la génération de tests (ex: `"AgilePheromind génère tests unitaires pour la méthode calculatePrice dans OrderService"` ou `"AgilePheromind génère tests pour le composant ProductDetailComponent méthode loadProduct"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Analyse du Code Cible, des Spécifications et du Framework de Test.**
        *   UO délègue à `@test-generator-agent`. L'agent identifie le fichier, la méthode/fonction, ses dépendances, et consulte les ACs de l'US/tâche active.
    *   **Phase 2: Identification Méthodique des Cas de Test.**
        *   UO délègue à `@test-generator-agent`, qui utilise **Sequential Thinking MCP**.
    *   **Phase 3: Génération des Squelettes de Fichiers et Méthodes de Test.**
        *   UO délègue à `@test-generator-agent`.
    *   **Phase 4: Enregistrement et Rapport.**
        *   Scribe enregistre les informations dans `.pheromone`.

## Détails des Phases:

### Phase 1: Analyse du Code Cible, des Spécifications et du Framework de Test
*   **Agent Responsable:** `@test-generator-agent`
*   **Inputs:** Nom de la méthode/classe/composant/service et nom de la méthode/fonction spécifique à tester (fourni par l'UO). Accès au code source du projet via `memoryBank.currentProject.repositoryUrl` ou chemin local. `activeTask` et `activeUserStory` depuis `.pheromone` pour le contexte fonctionnel. `memoryBank.toolingConfigurations.testFrameworks` pour .NET et Angular.
*   **Actions & Tooling:**
    1.  **Localiser et Lire le Code Cible:**
        *   Identifier le chemin exact du fichier source (ex: `OrderService.cs`, `product-detail.component.ts`) basé sur le nom fourni et la structure du projet (inférée de `memoryBank.projectContext`).
        *   Utiliser **Git Tools MCP** (`get_file_contents {filePath}`) pour lire le contenu du fichier.
    2.  **Analyser la Signature et les Dépendances:**
        *   Parser la signature de la méthode/fonction cible (paramètres, types, type de retour).
        *   Identifier les dépendances injectées (services, repositories dans .NET ; services, pipes dans Angular) qui devront être mockées/stubées.
    3.  **Consulter les Spécifications Fonctionnelles:**
        *   Lire les descriptions et Critères d'Acceptation de `activeUserStory` et `activeTask` depuis `.pheromone` (ou `memoryBank`) pour comprendre le comportement attendu et les règles métier.
    4.  **Identifier le Framework de Test et les Librairies de Mocking:**
        *   Consulter `memoryBank.toolingConfigurations.testFrameworks` pour déterminer le framework de test utilisé (ex: MSTest, NUnit, xUnit pour .NET ; Jasmine, Jest pour Angular).
        *   Utiliser **Context7 MCP** (`get_library_docs`) pour obtenir la documentation :
            *   Du framework de test spécifique (ex: `get_library_docs {libraryName: "MSTest", topic: "TestMethod attribute"}`).
            *   Des librairies de mocking courantes pour la stack (ex: Moq, NSubstitute pour .NET ; `TestBed`, `jasmine.createSpyObj` pour Angular).
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.tasks.{{activeTask.id}}.description`, `memoryBank.userStories.{{activeUserStory.id}}.acceptanceCriteria`, `memoryBank.toolingConfigurations.testFrameworks`, `memoryBank.projectContext.codingConventionsLink`.
*   **Output (interne à `@test-generator-agent` pour la Phase 2):** Une compréhension détaillée de la cible à tester, de son contexte fonctionnel, et des outils de test à utiliser.

### Phase 2: Identification Méthodique des Cas de Test
*   **Agent Responsable:** `@test-generator-agent`
*   **Inputs:** L'analyse de la Phase 1.
*   **Actions & Tooling:**
    1.  Utiliser **Sequential Thinking MCP** pour une identification structurée des cas de test:
        *   `set_goal`: "Identifier tous les cas de test pertinents pour la méthode/fonction `{{targetName}}`."
        *   `add_step`: "Identifier les cas nominaux basés sur les ACs et le chemin principal de la logique." (Ex: entrée valide, résultat attendu).
        *   `add_step`: "Identifier les cas limites pour chaque paramètre d'entrée et condition logique." (Ex: valeurs nulles, vides, zéro, maximum, minimum, juste avant/après seuil).
        *   `add_step`: "Identifier les cas d'erreur et les exceptions attendues." (Ex: entrée invalide provoquant une `ArgumentNullException`, dépendance mockée lançant une exception).
        *   `add_step`: "Identifier les cas basés sur les interactions avec les dépendances mockées." (Ex: Que se passe-t-il si `dependency.Method()` retourne `null`, `true`, `false`, une liste vide, une erreur ?).
        *   `add_step`: "Pour les composants Angular, identifier les cas liés aux inputs (`@Input`), outputs (`@Output`), et états internes impactant l'affichage ou le comportement."
        *   `run_sequence`: Exécuter.
    2.  Pour chaque cas de test identifié, noter brièvement l'attendu (`expected outcome`).
*   **Memory Bank Interaction:**
    *   Lecture: Aucune interaction directe supplémentaire pour cette phase.
*   **Output (interne à `@test-generator-agent` pour la Phase 3):** Une liste exhaustive et structurée de scénarios de test à couvrir.

### Phase 3: Génération des Squelettes de Fichiers et Méthodes de Test
*   **Agent Responsable:** `@test-generator-agent`
*   **Inputs:** La liste des cas de test (Phase 2). La connaissance du framework de test et des librairies de mocking (Phase 1).
*   **Actions & Tooling:**
    1.  **Déterminer le Fichier de Test:**
        *   Basé sur les conventions du projet (ex: `OrderServiceTests.cs`, `product-detail.component.spec.ts`). Si le fichier existe, l'ouvrir pour y ajouter des tests ; sinon, le créer.
    2.  **Générer les Méthodes de Test:**
        *   Pour chaque cas de test identifié :
            *   Générer une méthode de test (.NET: `[TestMethod] public void MethodName_Scenario_ExpectedOutcome() {}`; Angular/Jasmine: `it('should ExpectedOutcome when Scenario', () => {});`).
            *   Inclure des commentaires `// Arrange`, `// Act`, `// Assert` (ou `// Given`, `// When`, `// Then`).
            *   **Arrange/Given:** Suggérer l'initialisation des objets, la création de données de test, et le setup des mocks pour les dépendances. Par exemple, pour .NET avec Moq: `var mockDependency = new Mock<IDependency>(); mockDependency.Setup(d => d.SomeMethod(It.IsAny<string>())).Returns(true);`. Pour Angular: `const mockService = jasmine.createSpyObj('MyService', ['getData']); mockService.getData.and.returnValue(of(mockData)); TestBed.configureTestingModule({ providers: [{ provide: MyService, useValue: mockService }] });`.
            *   **Act/When:** Indiquer l'appel à la méthode/fonction testée avec les données d'arrange.
            *   **Assert/Then:** Proposer des assertions pertinentes basées sur l'attendu du cas de test (ex: `Assert.AreEqual(expected, actual);`, `mockDependency.Verify(d => d.SomeMethod("specificValue"), Times.Once);` ou `expect(component.result).toEqual(expected);`, `expect(mockService.getData).toHaveBeenCalledWith(expectedParams);`).
    3.  S'assurer que la structure du fichier de test est correcte (classe de test pour .NET, `describe` block pour Angular/Jasmine).
    4.  Sauvegarder le fichier de test créé/modifié.
*   **Memory Bank Interaction (via Scribe après résumé):**
    *   Le Scribe enregistrera une référence au fichier de test généré/modifié et un résumé des cas couverts dans `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated` et dans `documentationRegistry`.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Squelettes de [Nombre] tests unitaires générés pour la méthode/fonction `{{targetName}}` dans le fichier `{{TestFilePath}}`. Les cas de test couvrent : [liste des types de cas, ex: nominaux, limites, erreurs, dépendances]. Le développeur doit maintenant compléter les sections Arrange/Act et finaliser les Assertions. Un rapport détaillé des cas de test a été généré : `unit_tests_scenarios_{{targetName}}_{{timestamp}}.md`." (Rapport enregistré dans `03_SPECS/Test_Scenarios/`).

### Phase 4: Enregistrement et Rapport
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@test-generator-agent`.
*   **Actions & Tooling:**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter le chemin vers `{{TestFilePath}}` et `unit_tests_scenarios_{{targetName}}_{{timestamp}}.md`.
        *   `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated`: Ajouter une entrée avec `{ path: "{{TestFilePath}}", scenarioReportPath: "unit_tests_scenarios_{{targetName}}_{{timestamp}}.md", description: "Squelettes de tests unitaires générés.", timestamp: "{{timestamp}}" }`.
*   **Memory Bank Interaction:**
    *   Écriture: Enregistrement des informations sur les tests générés.
*   **Output:** `.pheromone` mis à jour. L'UO est informé que la génération des squelettes de test est terminée.

---