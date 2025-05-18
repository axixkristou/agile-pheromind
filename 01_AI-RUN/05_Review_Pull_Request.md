# Workflow: Assistance à la Revue de Pull Request (05_Review_Pull_Request.md)

**Objectif:** Aider le Tech Lead à réviser une Pull Request (PR) Azure DevOps. Le système récupère les détails de la PR, analyse les modifications de code (qualité, sécurité, conventions), génère un rapport de revue détaillé (avec "chaîne de pensée" pour les points soulevés), le stocke dans un répertoire spécifique à la branche, et gère les erreurs de récupération ou d'analyse.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@code-reviewer-assistant`, `@security-analyst-agent`, `@devops-connector`, `@clarification-agent`.

**MCPs Utilisés:** Azure DevOps MCP, Git Tools MCP, Context7 MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Le Tech Lead fournit l'ID de la PR Azure DevOps (ex: `"AgilePheromind analyse PR Azure#456"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Récupération des Informations de la PR et Création/Validation du Répertoire de Revue.**
        *   UO délègue à `@devops-connector` pour les détails de la PR (incluant `sourceBranchName`).
        *   UO (ou `@code-reviewer-assistant`) s'assure de l'existence du répertoire `04_PR_REVIEWS/[nom_branche_source_PR_sanitized]/`.
        *   **onError:** Si ADO MCP échoue pour `get_pull_request_details`, logguer, notifier, arrêter.
    *   **Phase 2: Récupération et Préparation des Modifications de Code.**
        *   UO délègue à `@code-reviewer-assistant` pour obtenir les diffs (via ADO ou Git MCP).
        *   **onError:** Si récupération des diffs échoue, logguer, notifier, arrêter.
    *   **Phase 3: Analyse Statique, Heuristique et de Sécurité du Code avec "Chaîne de Pensée".**
        *   UO **injecte un contexte ciblé** (conventions de codage, historique de dette technique pertinent depuis `memoryBank`) à `@code-reviewer-assistant`.
        *   `@code-reviewer-assistant` collabore avec `@security-analyst-agent`. Utilise **Context7 MCP**. Doit **documenter la "chaîne de pensée"** pour les problèmes majeurs identifiés.
        *   Si l'analyse est bloquée par du code très ambigu, `@code-reviewer-assistant` le signale à l'UO, qui peut initier une clarification via `@clarification-agent` demandant au développeur de la PR d'expliquer une section de code.
    *   **Phase 4: Génération et Stockage du Rapport de Revue Détaillé.**
        *   UO délègue à `@code-reviewer-assistant` pour compiler et sauvegarder le rapport (incluant la "chaîne de pensée") dans le répertoire dédié.
    *   **Phase 5: Notification et Mise à Jour de `.pheromone`.**
        *   Scribe enregistre les informations et le lien vers le rapport.

## Détails des Phases:

### Phase 1: Récupération des Informations de la PR et Création/Validation du Répertoire de Revue
*   **Agent Responsable:** `@devops-connector`, puis `🧐 @uber-orchestrator` (ou `@code-reviewer-assistant`).
*   **Inputs:** ID de la PR.
*   **Actions & Tooling (`@devops-connector`):**
    1.  Utiliser **Azure DevOps MCP** `get_pull_request_details {id: ID_PR}` (sourceBranchName, targetBranchName, etc.).
*   **onError Strategy (pour l'UO si `@devops-connector` signale échec):**
    1.  Scribe loggue dans `activeWorkflow.lastError`.
    2.  UO notifie l'utilisateur: "Impossible de récupérer détails de PR Azure#{{prId}}. Erreur MCP: [Message]. Vérifiez ID ou connexion ADO."
    3.  Arrêter workflow.
*   **Output (`@devops-connector` vers Scribe/UO):** Résumé NL: "Détails PR Azure#{{prId}} récupérés. Branche source: '{{sourceBranchName}}'. Log: `azure_pr_{{prId}}_details_{{timestamp}}.json`."
*   **Actions & Tooling (UO ou `@code-reviewer-assistant` après màj `.pheromone` par Scribe):**
    1.  Lire `activePullRequest.sourceBranchName`. Créer `reviewDirPath = "04_PR_REVIEWS/{{activePullRequest.sourceBranchName_sanitized}}/"`. Créer le répertoire si absent.
    2.  Stocker `reviewDirPath` dans `.pheromone.activePullRequest.reviewDirectoryPath` (via Scribe).

### Phase 2: Récupération et Préparation des Modifications de Code
*   **Agent Responsable:** `@code-reviewer-assistant`
*   **Inputs:** `activePullRequest` depuis `.pheromone`.
*   **Actions & Tooling:**
    1.  Utiliser **Azure DevOps MCP** (`get_pull_request_changed_files`, `get_pull_request_commits`/`get_diff`).
    2.  Ou, si besoin, **Git Tools MCP** (`git_fetch_origin`, `git_diff`).
*   **onError Strategy (pour l'UO si `@code-reviewer-assistant` signale échec):**
    1.  Scribe loggue.
    2.  UO notifie: "Impossible de récupérer les diffs pour PR Azure#{{prId}}. Erreur MCP/Git: [Message]."
    3.  Arrêter workflow.
*   **Output (interne à `@code-reviewer-assistant`):** Diffs de code prêts.

### Phase 3: Analyse Statique, Heuristique et de Sécurité du Code avec "Chaîne de Pensée"
*   **Agent Responsable:** `@code-reviewer-assistant` (collabore avec `@security-analyst-agent`).
*   **Inputs (Injectés par l'UO):**
    *   Modifications de code (Phase 2).
    *   Contexte de `memoryBank`: `projectContext.codingConventionsLink`, `projectContext.designConventionsLink`, `technicalDebtItems` (pour voir si la PR adresse/introduit de la dette), `architecturalDecisions` pertinentes.
*   **Actions & Tooling (`@code-reviewer-assistant`):**
    1.  **Analyse Conventions & Qualité:** Vérifier vs conventions. Identifier code smells, bugs potentiels. Évaluer lisibilité, maintenabilité, tests. Utiliser **Context7 MCP** pour usage librairies.
    2.  **Coordination Sécurité:** Déléguer analyse de sécurité des diffs à `@security-analyst-agent`.
    3.  **"Chaîne de Pensée":** Pour chaque problème significatif (Majeur/Critique) identifié, documenter brièvement le raisonnement: "Problème X détecté car [condition Y observée] viole [convention Z] ou introduit [risque W]. La bonne pratique (Context7/convention) est [pratique]."
    4.  **Gestion d'Ambiguïté:** Si une section de code est trop complexe ou son intention est totalement obscure, rendant l'analyse impossible:
        *   Signaler à l'UO: "Analyse bloquée sur fichier [X] ligne [Y] pour PR Azure#{{prId}}. Code ambigu. Suggestion de question pour dev: 'Pouvez-vous expliquer la logique/l'intention de cette section ?'".
        *   L'UO peut alors mettre le workflow en pause et initier une clarification via `@clarification-agent`.
*   **Actions & Tooling (`@security-analyst-agent`):**
    *   Analyse sécurité ciblée (OWASP, etc.). Rapport concis à `@code-reviewer-assistant`.
*   **Output (compilé par `@code-reviewer-assistant`):** Liste consolidée de problèmes, avec sévérité et "chaîne de pensée" pour les points majeurs.

### Phase 4: Génération et Stockage du Rapport de Revue Détaillé
*   **Agent Responsable:** `@code-reviewer-assistant`
*   **Inputs:** Problèmes consolidés (Phase 3). `activePullRequest.reviewDirectoryPath`.
*   **Actions & Tooling:**
    1.  Rédiger rapport Markdown (`pr_{{activePullRequest.id}}_review_report_{{timestamp}}.md`). Inclure: Infos PR, Résumé, Détail des Problèmes (avec sévérité, localisation, **raisonnement/chaîne de pensée pour les points clés**, suggestion).
    2.  Sauvegarder dans `{{activePullRequest.reviewDirectoryPath}}`.
*   **Output (vers Scribe):** Résumé NL: "Revue PR Azure#{{activePullRequest.id}} terminée. [Stats problèmes]. Rapport (avec chaîne de pensée): `{{fullReportPath}}`. Recommandation: [Action]."

### Phase 5: Notification et Mise à Jour de `.pheromone`
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@code-reviewer-assistant`.
*   **Actions & Tooling:**
    1.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter chemin du rapport.
        *   `memoryBank.pullRequests.{{activePullRequest.id}}`: Mettre à jour `status: "ReviewedByAI"`, `reviewReportPath`, `lastReviewTimestamp`, `issuesFoundCounts`, `summary`, et `reasoningChainLinks.reviewReport` (qui pointe vers le rapport contenant les chaînes de pensée).
    2.  (Optionnel) UO instruit `@devops-connector` de commenter la PR sur ADO.
*   **Output:** `.pheromone` mis à jour. UO informe Tech Lead.

---