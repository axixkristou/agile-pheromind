# Workflow: Validation de l'Interface Utilisateur (UI Test Validation) (12_UI_Test_Validation.md)

**Objectif:** Assister le testeur dans la validation manuelle ou semi-automatisée de l'interface utilisateur (UI) d'une fonctionnalité ou User Story (US). L'agent utilise le Browser Tools MCP pour les interactions/captures, compare aux spécifications de design (avec une "chaîne de pensée" pour les écarts significatifs), et gère les erreurs ou ambiguïtés dans les specs.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@tester-ui-validator-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Utilisés:** Browser Tools MCP (Puppeteer ou Playwright), Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Testeur/PO demande validation UI pour US/Fonctionnalité + URL environnement de test (ex: `"AgilePheromind valide UI US Azure#12323 sur https://test.myapp.com"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Récupération Spécifications, ACs et Injection de Contexte.**
        *   UO délègue à `@devops-connector` pour ACs (via **ADO MCP**).
        *   UO **injecte contexte** à `@tester-ui-validator-agent` : ACs, `design_conventions.md`, mockups (depuis `documentationRegistry`/`memoryBank`).
        *   UO évalue clarté des specs UI. Si ambigu (ex: design contradictoire avec ACs), UO initie clarification via `@clarification-agent` avec PO/Designer.
        *   **onError:** Si ADO MCP ou accès aux specs échoue, notifier, arrêter.
    *   **Phase 2: Définition du Scénario de Test UI Détaillé.**
        *   UO délègue à `@tester-ui-validator-agent`.
    *   **Phase 3: Exécution Interactions et Captures via Browser Tools MCP.**
        *   UO délègue à `@tester-ui-validator-agent`.
        *   **onError (Browser Tools MCP):** Si MCP échoue (ex: élément non trouvé, navigation impossible), l'agent le note, tente de continuer si possible, et le signale dans le rapport.
    *   **Phase 4: Analyse des Résultats et Génération Rapport (avec "Chaîne de Pensée" pour Écarts).**
        *   UO délègue à `@tester-ui-validator-agent`. Pour les écarts majeurs, l'agent **documente sa "chaîne de pensée"** (pourquoi il considère que c'est un écart par rapport aux specs).
    *   **Phase 5: Enregistrement Rapport et Création Bugs (Optionnel).**
        *   Scribe enregistre. UO peut proposer création de bugs dans ADO via `@devops-connector`.

## Détails des Phases:

### Phase 1: Récupération Spécifications, ACs et Injection de Contexte
*   **Agent Responsable:** UO, `@devops-connector`, `@clarification-agent`.
*   **Inputs:** ID US/Fonctionnalité, URL env test.
*   **Actions (UO & `@devops-connector`):**
    1.  `@devops-connector` récupère ACs de l'US (si ID US) via **ADO MCP** `get_work_item_details`.
    2.  **onError (ADO MCP):** UO loggue via Scribe, notifie, arrête/continue avec avertissement.
    3.  Scribe met à jour `memoryBank.userStories.{{usId}}.acceptanceCriteria`.
    4.  UO injecte à `@tester-ui-validator-agent`:
        *   ACs récupérés.
        *   Chemin vers `design_conventions.md` (de `documentationRegistry`).
        *   Chemins vers mockups/prototypes pour l'US/feature (de `memoryBank.userStories.{{usId}}.designArtifactLinks` ou `documentationRegistry`).
        *   (Optionnel) Rapports de validation UI précédents pour des fonctionnalités similaires.
*   **Actions (UO - Évaluation Clarté):**
    1.  L'UO évalue si les ACs et les specs de design injectées sont cohérentes et suffisamment précises.
    2.  **Si ambiguïté majeure** (ex: AC décrit un comportement X, mockup montre Y):
        *   Mettre workflow en pause (`activeWorkflow.status: 'PendingClarification_UITestSpec'`).
        *   Déléguer à `@clarification-agent` avec le contexte et une question pour le PO/Designer (ex: "Pour US Azure#{{usId}}, l'AC X dit [comportement], mais le mockup Y montre [autre chose]. Quelle est l'attente correcte pour la validation UI ?").
        *   Attendre réponse via `01_AI-RUN/XX_Handle_Clarification_Response.md`.
*   **Output:** Contexte riche et clarifié (si besoin) pour `@tester-ui-validator-agent`.

### Phase 2: Définition du Scénario de Test UI Détaillé
*   **Agent Responsable:** `@tester-ui-validator-agent`.
*   **Inputs:** Contexte injecté (Phase 1).
*   **Actions:**
    1.  Définir étapes de validation (actions, attendus visuels/fonctionnels) basé sur ACs et specs design.
    2.  Documenter scénario (pour rapport final).
*   **Output (interne):** Plan de test UI.

### Phase 3: Exécution Interactions et Captures via Browser Tools MCP
*   **Agent Responsable:** `@tester-ui-validator-agent`.
*   **Inputs:** Scénario de test (Phase 2), URL env test.
*   **Actions (Browser Tools MCP - Puppeteer/Playwright):**
    1.  Pour chaque étape: `navigate`, `click`, `fill_form_field`, `take_screenshot` (différents viewports, dans `04_PR_REVIEWS/[branche]/screenshots/` ou `03_SPECS/UI_Validation_Screenshots/[feature]/`), `execute_script` (CSS, contenu), `get_console_logs`.
*   **onError (Browser Tools MCP):**
    *   Si une action MCP échoue (ex: `click {selector}` car élément non trouvé):
        *   L'agent loggue l'erreur précise du MCP.
        *   Tente de continuer le scénario si l'échec n'est pas bloquant pour les étapes suivantes.
        *   Tous les échecs MCP seront explicitement listés dans le rapport de validation.
*   **Output (interne):** Captures d'écran, données navigateur, logs d'erreurs MCP.

### Phase 4: Analyse des Résultats et Génération Rapport (avec "Chaîne de Pensée" pour Écarts)
*   **Agent Responsable:** `@tester-ui-validator-agent`.
*   **Inputs:** Données de la Phase 3, specs de la Phase 1.
*   **Actions:**
    1.  Comparer captures/données avec mockups/conventions/ACs. Vérifier logs console.
    2.  Rédiger rapport MD (`ui_validation_report_[US_ID_ou_Feature]_[timestamp].md`) dans `03_SPECS/UI_Validation_Reports/` ou dir PR.
        *   Structure: Périmètre, Résumé, Points de validation (Attendu, Constaté (avec screenshot/data), Statut, Commentaire).
        *   **"Chaîne de Pensée" pour Écarts Significatifs:** Pour chaque "Échec" majeur, expliquer *pourquoi* c'est un écart par rapport aux specs (ex: "Le bouton X utilise la couleur `colors.red['300']` alors que `design_conventions.md` Section 4.1 spécifie `colors.red['500']` pour les actions destructives. Cela manque de contraste et ne respecte pas la sémantique définie.").
        *   Lister échecs MCP de la Phase 3.
        *   Lister bugs/problèmes avec sévérité.
*   **Output (vers Scribe):** Résumé NL: "Validation UI '[US_ID/Feature]' terminée. Statut: [Global]. [N_bugs] bugs. Rapport (avec chaîne de pensée pour écarts): `ui_validation_report...md`."

### Phase 5: Enregistrement Rapport et Création Bugs (Optionnel)
*   **Agent Responsable:** Scribe, UO, `@devops-connector`.
*   **Inputs:** Résumé NL de `@tester-ui-validator-agent`.
*   **Actions (Scribe):**
    1.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter chemin rapport.
        *   `memoryBank.userStories.{{usId}}.uiValidationHistory[]`: Ajouter `{reportPath, status, timestamp}`.
        *   `memoryBank.userStories.{{usId}}.reasoningChainLinks.uiValidation`: Lier au rapport (contenant la chaîne de pensée pour les écarts).
        *   (Optionnel) `memoryBank.identifiedBugs`: Ajouter les bugs listés.
*   **Actions (UO):**
    1.  Si bugs majeurs: `ask_followup_question` à l'utilisateur "Bugs UI trouvés. Voulez-vous créer des items Bug dans ADO pour : [liste bugs majeurs] ?".
    2.  Si oui, UO délègue à `@devops-connector` (**ADO MCP** `create_work_item {type: "Bug", ...}`).
*   **Output:** `.pheromone` à jour. Bugs potentiellement créés dans ADO.

---