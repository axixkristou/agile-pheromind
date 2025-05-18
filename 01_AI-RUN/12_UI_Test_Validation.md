# Workflow: Validation de l'Interface Utilisateur (UI Test Validation) (12_UI_Test_Validation.md)

**Objectif:** Assister le testeur dans la validation manuelle ou semi-automatisée de l'interface utilisateur (UI) d'une fonctionnalité ou d'une User Story (US) spécifique. L'agent utilise le Browser Tools MCP pour interagir avec l'application et prendre des captures d'écran, puis compare les résultats avec les spécifications de design et les critères d'acceptation.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@tester-ui-validator-agent` (nouveau rôle), `@devops-connector`.

**MCPs Utilisés:** Browser Tools MCP (Puppeteer ou Playwright), Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Testeur/PO) demande la validation UI pour une US ou une fonctionnalité spécifique (ex: `"AgilePheromind valide UI pour US Azure#12323"` ou `"AgilePheromind teste UI page Inscription"`). L'URL de l'environnement de test doit être fournie ou configurable.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Récupération des Spécifications et Critères d'Acceptation.**
        *   UO délègue à `@devops-connector` pour les ACs de l'US.
        *   UO s'assure que `@tester-ui-validator-agent` a accès aux conventions de design et mockups (via `.pheromone.documentationRegistry` et `memoryBank`).
    *   **Phase 2: Définition du Scénario de Test UI.**
        *   UO délègue à `@tester-ui-validator-agent` pour créer un plan de test UI basé sur les ACs et les spécifications de design.
    *   **Phase 3: Exécution des Interactions et Captures d'Écran via Browser Tools MCP.**
        *   UO délègue à `@tester-ui-validator-agent`.
    *   **Phase 4: Analyse des Résultats et Génération du Rapport de Validation.**
        *   UO délègue à `@tester-ui-validator-agent`.
    *   **Phase 5: Enregistrement du Rapport et des Problèmes.**
        *   Scribe enregistre le rapport. Si des bugs sont trouvés, l'UO peut proposer de les créer dans Azure DevOps via `@devops-connector`.

## Détails des Phases:

### Phase 1: Récupération des Spécifications et Critères d'Acceptation
*   **Agent Responsable:** `@devops-connector` (pour les ACs), `@tester-ui-validator-agent` (pour les specs de design).
*   **Inputs:** ID de l'US ou nom de la fonctionnalité/page à tester. URL de l'environnement de test.
*   **Actions & Tooling (`@devops-connector`):**
    1.  Si un ID d'US est fourni, utiliser **Azure DevOps MCP** (`get_work_item_details {id: ID_US}`) pour récupérer les Critères d'Acceptation et la description.
*   **Output (`@devops-connector` vers Scribe):** Résumé NL: "Critères d'Acceptation pour US Azure#{{usId}} récupérés. Log: `azure_wi_{{usId}}_ACs_{{timestamp}}.json`."
*   **Actions & Tooling (`@tester-ui-validator-agent`):**
    1.  Lire `.pheromone.documentationRegistry` et `.pheromone.memoryBank` pour:
        *   Le chemin vers `design_conventions.md` (et `coding_conventions.md` pour les aspects liés à la structure HTML/CSS attendue).
        *   Les chemins vers d'éventuels mockups, wireframes, ou prototypes de design stockés pour l'US/fonctionnalité.
*   **Memory Bank Interaction (via Scribe):**
    *   Scribe met à jour `memoryBank.userStories.{{usId}}.acceptanceCriteria` avec les ACs récupérés.
    *   Enregistre le log d'Azure DevOps dans `documentationRegistry`.
*   **Output (interne à `@tester-ui-validator-agent`):** Ensemble des spécifications (ACs, design) pour la validation.

### Phase 2: Définition du Scénario de Test UI
*   **Agent Responsable:** `@tester-ui-validator-agent`
*   **Inputs:** Spécifications de la Phase 1.
*   **Actions & Tooling:**
    1.  Basé sur les ACs et les spécifications de design, définir une série d'étapes de validation (scénario de test).
    2.  Pour chaque étape, identifier :
        *   L'action à effectuer (ex: naviguer vers URL, cliquer sur bouton X, remplir champ Y).
        *   L'attendu visuel (ex: le header doit être bleu, le texte doit être de taille 16px, le modal Z doit apparaître).
        *   L'attendu fonctionnel (ex: après clic, redirection vers page B ; après soumission, message de succès).
    3.  Documenter ce scénario dans un fichier temporaire ou une section du rapport final.
*   **Memory Bank Interaction:**
    *   Le scénario sera inclus dans le rapport final.
*   **Output (interne à `@tester-ui-validator-agent`):** Plan de test UI détaillé.

### Phase 3: Exécution des Interactions et Captures d'Écran via Browser Tools MCP
*   **Agent Responsable:** `@tester-ui-validator-agent`
*   **Inputs:** Scénario de test UI (Phase 2). URL de l'environnement de test.
*   **Actions & Tooling:**
    1.  Utiliser **Browser Tools MCP** (Puppeteer ou Playwright - l'agent doit être configuré pour l'un ou l'autre, ou le MCP doit être abstrait):
        *   Pour chaque étape du scénario:
            *   `navigate {url}`
            *   `click {selector}`
            *   `fill_form_field {selector, value}`
            *   `select_dropdown_option {selector, optionValue}`
            *   `hover_element {selector}`
            *   `take_screenshot {filePath, fullPage: true/false, clip: {x,y,width,height} (optionnel)}` -> sauvegarder les captures dans le répertoire de revue de la PR (ex: `04_PR_REVIEWS/[nom_branche_PR_actuelle_ou_feature_testee]/screenshots/`) ou un répertoire de test UI dédié.
            *   `execute_script {scriptContent}` pour récupérer des propriétés CSS, des contenus textuels, des états d'éléments.
            *   `get_console_logs` pour vérifier les erreurs JS.
    2.  Stocker les résultats des `execute_script` et les chemins des captures d'écran.
*   **Memory Bank Interaction:**
    *   Les chemins des captures d'écran seront dans le rapport.
*   **Output (interne à `@tester-ui-validator-agent`):** Ensemble de captures d'écran et de données extraites du navigateur.

### Phase 4: Analyse des Résultats et Génération du Rapport de Validation
*   **Agent Responsable:** `@tester-ui-validator-agent`
*   **Inputs:** Captures d'écran et données du navigateur (Phase 3). Spécifications (Phase 1).
*   **Actions & Tooling:**
    1.  Comparer les captures d'écran avec les mockups/conventions de design.
    2.  Comparer les données extraites (styles CSS, textes) avec les attendus.
    3.  Vérifier si les interactions ont produit les résultats fonctionnels attendus (basé sur ACs).
    4.  Vérifier les logs de la console pour des erreurs JavaScript.
    5.  Rédiger un rapport de validation UI Markdown (`ui_validation_report_[US_ID_ou_Feature]_[timestamp].md`) dans un répertoire dédié (ex: `03_SPECS/UI_Validation_Reports/` ou le répertoire de revue de PR si pertinent `04_PR_REVIEWS/[nom_branche]/`).
    6.  Le rapport doit inclure:
        *   Périmètre de la validation (US/Fonctionnalité, URL testée).
        *   Résumé: Globalement Conforme / Conforme avec problèmes mineurs / Non Conforme.
        *   Liste des points de validation (du scénario de test). Pour chaque point:
            *   Attendu.
            *   Constaté (avec lien/embed de capture d'écran si pertinent).
            *   Statut (OK, Échec, Attention).
            *   Commentaire (si Échec/Attention, description du problème).
        *   Liste des bugs/problèmes identifiés avec leur sévérité.
*   **Memory Bank Interaction (via Scribe):**
    *   Le chemin du rapport sera stocké. Les bugs majeurs peuvent être signalés pour création dans `memoryBank.technicalDebtItems` ou `memoryBank.tasks` (type bug).
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Validation UI pour '[US_ID/Feature]' terminée. Statut global: [Conforme/Avec Problèmes/Non Conforme]. [N_bugs] bugs identifiés. Rapport détaillé: `ui_validation_report_[US_ID_ou_Feature]_[timestamp].md`."

### Phase 5: Enregistrement du Rapport et des Problèmes
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`, puis `🧐 @uber-orchestrator` pour actions de suivi.
*   **Inputs:** Résumé NL de `@tester-ui-validator-agent`.
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  Interpréter le résumé.
    2.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter le chemin vers le rapport de validation UI.
        *   `memoryBank.userStories.{{usId_if_contextual}}.uiValidationHistory`: Ajouter une entrée `{ reportPath: "[CheminRapport]", status: "[StatutGlobal]", timestamp: "{{timestamp}}" }`.
        *   Si des bugs sont listés dans le résumé ou le rapport, le Scribe pourrait les enregistrer dans une section `memoryBank.identifiedBugs` avec des détails pour un traitement ultérieur.
*   **Actions & Tooling (`🧐 @uber-orchestrator`):**
    1.  Lire le statut global de la validation depuis `.pheromone`.
    2.  Si des bugs majeurs sont identifiés:
        *   Utiliser `ask_followup_question` pour demander à l'utilisateur (Testeur/PO): "Des problèmes UI ont été trouvés pour [US_ID/Feature]. Souhaitez-vous que je crée des tâches de bug dans Azure DevOps pour les problèmes suivants: [Liste des problèmes majeurs] ?"
        *   Si oui, UO délègue à `@devops-connector` pour utiliser `create_work_item {type: "Bug", ...}`.
*   **Memory Bank Interaction:**
    *   Écriture: Enregistrement du rapport et potentiellement des bugs.
*   **Output:** `.pheromone` mis à jour. L'équipe est informée des résultats de la validation UI et des actions de suivi peuvent être entreprises pour les bugs.

---