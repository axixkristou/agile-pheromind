# Workflow: Assistance au Product Owner - Analyse d'un Besoin Client (03_PO_Analyze_Need.md)

**Objectif:** Aider le Product Owner (PO) à analyser un besoin client exprimé en langage naturel. Le système doit décomposer le besoin de manière structurée (en utilisant le `Sequential Thinking MCP` et en enregistrant la "chaîne de pensée"), proposer des User Stories (US) potentielles avec des Critères d'Acceptation (ACs) initiaux, vérifier les US similaires existantes (via Azure DevOps MCP), gérer les ambiguïtés du besoin via `@clarification-agent`, et enregistrer l'analyse complète dans la Memory Bank.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@po-assistant`, `@devops-connector`, `@clarification-agent`.

**MCPs Utilisés:** Azure DevOps MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Le PO soumet une description du besoin client à AgilePheromind (ex: `"AgilePheromind analyse besoin : 'Nos utilisateurs se plaignent que le processus d'inscription est trop long...'"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Analyse Structurée du Besoin Client et Identification d'Ambiguïtés.**
        *   UO **injecte un contexte pertinent** (ex: personae existants, glossaire métier depuis `memoryBank`) à `@po-assistant`.
        *   `@po-assistant` utilise **Sequential Thinking MCP** pour la décomposition. Il doit **détailler sa "chaîne de pensée"**.
        *   Si des ambiguïtés majeures sont détectées dans le besoin client, `@po-assistant` les signale à l'UO. L'UO peut alors engager `@clarification-agent` pour demander des précisions au PO. Le workflow attend la réponse.
        *   **onError:** Si l'analyse initiale échoue ou reste trop vague, notifier le PO.
    *   **Phase 2: Génération de Propositions d'User Stories et Critères d'Acceptation.**
        *   UO délègue à `@po-assistant` (utilisant les informations clarifiées si Phase 1b a eu lieu).
    *   **Phase 3: Recherche d'User Stories Existantes dans le Backlog.**
        *   UO délègue à `@po-assistant`, qui interroge `@devops-connector`.
        *   **onError:** Si ADO MCP échoue, informer le PO que la vérification des doublons n'a pas pu être faite.
    *   **Phase 4: Compilation du Rapport d'Analyse et Interaction avec le PO.**
        *   `@po-assistant` génère un rapport (incluant la "chaîne de pensée" de l'analyse).
        *   Scribe enregistre le rapport et les US brouillons dans `.pheromone`.
        *   UO présente un résumé au PO et propose des actions de suivi.

## Détails des Phases:

### Phase 1: Analyse Structurée du Besoin Client et Identification d'Ambiguïtés
*   **Agent Responsable:** `@po-assistant` (avec support de l'UO pour clarification si besoin via `@clarification-agent`).
*   **Inputs (Injectés par l'UO):**
    *   Description du besoin client.
    *   Contexte de `memoryBank.projectContext` (ex: `targetAudienceDescription`, `businessGoals`).
    *   Liste des personas existants (`memoryBank.userPersonas`).
    *   Glossaire métier (`memoryBank.glossary`).
*   **Actions & Tooling (`@po-assistant`):**
    1.  Utiliser **Sequential Thinking MCP** pour l'analyse détaillée :
        *   `set_goal`: "Analyser le besoin client: '{{besoinClient}}' pour identifier problèmes, solutions et bénéfices."
        *   `add_step`: "Identifier les acteurs/personas concernés (en utilisant la liste de personas fournie comme référence)."
        *   `add_step`: "Extraire les problèmes spécifiques ('pain points')."
        *   `add_step`: "Identifier les solutions/désirs explicites ou implicites."
        *   `add_step`: "Déduire les bénéfices attendus."
        *   `add_step`: "Identifier les contraintes ou hypothèses."
        *   `add_step`: "**Évaluer la clarté du besoin.** Lister les points ambigus ou les informations manquantes qui empêchent une décomposition claire en US."
        *   `run_sequence`.
    2.  **Documenter la "Chaîne de Pensée":** Conserver le log détaillé de cette analyse séquentielle pour l'inclure dans le rapport final.
    3.  **Signaler Ambiguïtés à l'UO:** Si l'étape "Évaluer la clarté" identifie des ambiguïtés critiques:
        *   Formuler les points d'ambiguïté.
        *   Suggérer des questions spécifiques pour le PO.
*   **onError / Gestion de l'Ambiguïté (par l'UO):**
    1.  Si `@po-assistant` signale des ambiguïtés :
        *   UO met le workflow en pause (`activeWorkflow.status: 'PendingClarification_UserNeed'`).
        *   UO délègue à `@clarification-agent` avec le contexte du besoin et les questions suggérées par `@po-assistant`.
        *   La réponse sera traitée par `01_AI-RUN/XX_Handle_Clarification_Response.md`, qui mettra à jour la `memoryBank` et réactivera ce workflow.
    2.  Si le Sequential Thinking MCP échoue, Scribe loggue l'erreur, UO notifie le PO.
*   **Output (interne à `@po-assistant` après clarification si nécessaire):** Analyse structurée du besoin client (potentiellement enrichie par les réponses du PO).

### Phase 2: Génération de Propositions d'User Stories et Critères d'Acceptation
*   **Agent Responsable:** `@po-assistant`
*   **Inputs:** Analyse structurée du besoin (clarifiée si besoin en Phase 1).
*   **Actions & Tooling:**
    1.  Pour chaque ensemble {Problème -> Solution -> Bénéfice} :
        *   Formuler des US ("En tant que..., je veux..., afin de...").
        *   Rédiger des ACs initiaux (Gherkin ou listes).
    2.  **Documenter la "Chaîne de Pensée":** Expliquer brièvement dans le rapport final pourquoi chaque US a été formulée de cette manière par rapport à l'analyse du besoin.
*   **Memory Bank Interaction (via Scribe en Phase 4):**
    *   Les US brouillons et ACs seront stockés.
*   **Output (interne à `@po-assistant`):** Liste d'US candidates avec ACs.

### Phase 3: Recherche d'User Stories Existantes dans le Backlog
*   **Agent Responsable:** `@po-assistant` (coordonnant avec `@devops-connector`).
*   **Inputs:** US candidates (Phase 2). `.pheromone.currentProject.name`.
*   **Actions & Tooling:**
    1.  `@po-assistant` identifie mots-clés pour chaque US candidate.
    2.  `@po-assistant` demande à `@devops-connector`: "Recherche US existantes dans ADO projet `{{currentProject.name}}` pour mots-clés : [liste]."
    3.  `@devops-connector` utilise **Azure DevOps MCP** (`search_work_items`).
*   **onError Strategy (pour l'UO si `@devops-connector` signale échec MCP):**
    1.  Scribe loggue l'erreur.
    2.  UO informe `@po-assistant` que la recherche ADO a échoué. L'analyse continuera sans cette information, mais le rapport final le mentionnera.
*   **Output (`@devops-connector` vers `@po-assistant`):** Liste d'IDs/titres d'US ADO potentiellement similaires.

### Phase 4: Compilation du Rapport d'Analyse et Interaction avec le PO
*   **Agent Responsable:** `@po-assistant` (rapport), Scribe (enregistrement), UO (interaction PO).
*   **Inputs:** Résultats des phases précédentes.
*   **Actions & Tooling (`@po-assistant`):**
    1.  Compiler un rapport Markdown (`po_need_analysis_[timestamp].md`) dans `02_AI-DOCS/PO_Analyses/`. Il doit inclure:
        *   Besoin client initial.
        *   **Analyse structurée détaillée (avec la "chaîne de pensée" de la Phase 1).**
        *   US candidates avec ACs (et la "chaîne de pensée" pour leur formulation).
        *   Résultats de la recherche ADO (ou mention de l'échec si Phase 3 a échoué).
        *   Recommandations (créer, fusionner, priorités initiales).
*   **Output (`@po-assistant` vers Scribe):** Résumé NL: "Analyse besoin client '[résumé]' terminée. [N_us] US proposées. [N_exist] US existantes trouvées. Rapport (avec chaîne de pensée): `po_need_analysis_[timestamp].md`. Recommandations: [clés]."
*   **Actions & Tooling (Scribe):**
    1.  Enregistrer rapport dans `documentationRegistry`.
    2.  Stocker US candidates (avec ACs, liens vers US existantes, et un lien vers la section "chaîne de pensée" du rapport) dans `memoryBank.draftUserStories` ou `memoryBank.userStories.{{usId_draft}}.analysisSummaries[]` (si des ID brouillons sont générés).
*   **Actions & Tooling (UO):**
    1.  Utiliser `ask_followup_question` pour présenter résumé et options au PO : "Analyse du besoin terminée. [Résumé]. Rapport avec analyse détaillée disponible. Options: 1. Voir rapport. 2. Créer les nouvelles US suggérées dans ADO. 3. Discuter d'une US."
*   **Memory Bank Interaction (via Scribe):**
    *   Archivage du rapport, des US brouillons, et du lien vers la "chaîne de pensée".
*   **Outcome:** Le PO reçoit une analyse approfondie et traçable, des US exploitables, et des options claires, même si des clarifications ont été nécessaires.

---