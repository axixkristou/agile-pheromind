# Workflow: Assistance à la Revue de Pull Request (05_Review_Pull_Request.md)

**Objectif:** Aider le Tech Lead à réviser une Pull Request (PR) Azure DevOps. Le système récupère les détails de la PR, analyse les modifications de code pour des problèmes de qualité, de sécurité (en collaboration avec `@security-analyst-agent`), et de conformité aux conventions. Un rapport de revue est généré et stocké dans un répertoire spécifique à la branche de la PR.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@code-reviewer-assistant`, `@security-analyst-agent`, `@devops-connector`.

**MCPs Utilisés:** Azure DevOps MCP, Git Tools MCP (pour une analyse de diff plus fine si nécessaire), Context7 MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Le Tech Lead fournit l'ID de la PR Azure DevOps (ex: `"AgilePheromind analyse PR Azure#456"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Récupération des Informations de la PR et Création du Répertoire de Revue.**
        *   UO délègue à `@devops-connector` pour les détails de la PR (y compris le nom de la branche source).
        *   UO (ou `@code-reviewer-assistant`) crée un répertoire `04_PR_REVIEWS/[nom_branche_source_PR]/` si non existant.
    *   **Phase 2: Récupération et Préparation des Modifications de Code.**
        *   UO délègue à `@code-reviewer-assistant` pour obtenir les diffs via **Azure DevOps MCP** ou **Git Tools MCP**.
    *   **Phase 3: Analyse Statique, Heuristique et de Sécurité du Code.**
        *   UO délègue à `@code-reviewer-assistant`, qui collabore avec `@security-analyst-agent`. Utilisation de **Context7 MCP** pour la documentation des librairies.
    *   **Phase 4: Génération et Stockage du Rapport de Revue.**
        *   UO délègue à `@code-reviewer-assistant` pour compiler et sauvegarder le rapport dans le répertoire dédié.
    *   **Phase 5: Notification et Mise à Jour de `.pheromone`.**
        *   Scribe enregistre les informations.

## Détails des Phases:

### Phase 1: Récupération des Informations de la PR et Création du Répertoire de Revue
*   **Agent Responsable:** `@devops-connector`, puis `🧐 @uber-orchestrator` (ou `@code-reviewer-assistant` pour la création du dir).
*   **Inputs:** ID de la PR fourni par l'UO.
*   **Actions & Tooling (`@devops-connector`):**
    1.  Utiliser **Azure DevOps MCP**:
        *   `get_pull_request_details {id: ID_PR}`: Récupérer les informations de la PR, y compris `sourceBranchName`, `targetBranchName`, auteur, description, US/tâches liées.
*   **Output (`@devops-connector` vers Scribe et UO):** Résumé NL: "Détails de PR Azure#{{prId}} récupérés. Branche source: '{{sourceBranchName}}', Branche cible: '{{targetBranchName}}'. Liée à US/Tâches: [IDs]. Log: `azure_pr_{{prId}}_details_{{timestamp}}.json`." (Log dans `03_SPECS/AzureDevOps_Logs/`).
*   **Actions & Tooling (UO ou `@code-reviewer-assistant` après que le Scribe ait mis à jour `.pheromone` avec les détails de la PR):**
    1.  Lire `activePullRequest.sourceBranchName` depuis `.pheromone`.
    2.  Construire le chemin du répertoire de revue: `reviewDirPath = "04_PR_REVIEWS/{{activePullRequest.sourceBranchName_sanitized}}/"`. (Sanitiser le nom de la branche pour qu'il soit valide comme nom de répertoire).
    3.  Vérifier si `reviewDirPath` existe. Si non, le créer.
    4.  Stocker `reviewDirPath` dans `.pheromone.activePullRequest.reviewDirectoryPath`.
*   **Memory Bank Interaction (via Scribe):**
    *   Le Scribe met à jour `.pheromone.activePullRequest` avec les détails de la PR, y compris `sourceBranchName` et `reviewDirectoryPath`.
    *   `documentationRegistry` est mis à jour avec `azure_pr_{{prId}}_details_{{timestamp}}.json`.

### Phase 2: Récupération et Préparation des Modifications de Code
*   **Agent Responsable:** `@code-reviewer-assistant`
*   **Inputs:** `activePullRequest.id` et `activePullRequest.targetBranchName` depuis `.pheromone`.
*   **Actions & Tooling:**
    1.  Utiliser **Azure DevOps MCP**:
        *   `get_pull_request_changed_files {id: activePullRequest.id}`: Obtenir la liste des fichiers modifiés.
        *   `get_pull_request_commits {id: activePullRequest.id}` (et/ou `get_diff` entre `sourceBranchName` et `targetBranchName` si disponible) pour obtenir les modifications de code détaillées.
    2.  Alternativement, si une analyse plus fine est nécessaire et que le repo local est synchronisé, utiliser **Git Tools MCP**:
        *   `git_fetch_origin`.
        *   `git_diff {base_commit: origin/activePullRequest.targetBranchName, head_commit: origin/activePullRequest.sourceBranchName}`.
    3.  Stocker temporairement les diffs ou les références aux fichiers modifiés pour la phase d'analyse.
*   **Memory Bank Interaction:**
    *   Lecture: `.pheromone.activePullRequest` pour les IDs et noms de branches. Lire `memoryBank.projectContext.codingConventionsLink`.
*   **Output (interne à `@code-reviewer-assistant`):** Diffs de code et liste des fichiers modifiés prêts pour analyse.

### Phase 3: Analyse Statique, Heuristique et de Sécurité du Code
*   **Agent Responsable:** `@code-reviewer-assistant`, en collaboration avec `@security-analyst-agent`.
*   **Inputs:** Modifications de code (Phase 2). Conventions de codage (`memoryBank.projectContext.codingConventionsLink`).
*   **Actions & Tooling (`@code-reviewer-assistant`):**
    1.  **Analyse de Conventions et Qualité:**
        *   Pour chaque fichier modifié, vérifier l'adhérence aux conventions de codage .NET/Angular.
        *   Identifier les "code smells" (duplication, complexité cyclomatique, méthodes/classes longues, etc.).
        *   Rechercher les anti-patterns .NET/Angular connus.
        *   Évaluer la lisibilité, la maintenabilité et la clarté du code.
        *   Vérifier si des tests unitaires pertinents ont été ajoutés ou mis à jour pour couvrir les modifications.
    2.  **Analyse de Dépendances et Librairies:**
        *   Si de nouvelles librairies sont introduites ou des fonctionnalités complexes de librairies existantes sont utilisées, consulter **Context7 MCP** (`get_library_docs`) pour vérifier l'utilisation correcte des APIs et les meilleures pratiques.
    3.  **Coordination avec `@security-analyst-agent`:**
        *   Transmettre les diffs de code ou les fichiers modifiés à `@security-analyst-agent` avec la tâche "Analyser ce code pour les vulnérabilités de sécurité".
        *   Attendre le rapport de `@security-analyst-agent`.
*   **Actions & Tooling (`@security-analyst-agent`):**
    *   Effectuer une analyse de sécurité ciblée sur les modifications : recherche de vulnérabilités OWASP Top 10, injections, XSS, gestion incorrecte des secrets, problèmes d'autorisation/authentification spécifiques à .NET/Angular.
    *   Produire un rapport de sécurité concis pour `@code-reviewer-assistant`.
*   **Memory Bank Interaction:**
    *   Lecture: `memoryBank.projectContext.codingConventionsLink`, `memoryBank.technicalDebtItems` (pour voir si la PR adresse ou introduit de la dette).
*   **Output (compilé par `@code-reviewer-assistant`):** Une liste consolidée de tous les problèmes identifiés (qualité, conventions, sécurité), classés par sévérité.

### Phase 4: Génération et Stockage du Rapport de Revue
*   **Agent Responsable:** `@code-reviewer-assistant`
*   **Inputs:** Liste consolidée des problèmes (Phase 3). `activePullRequest.reviewDirectoryPath` depuis `.pheromone`.
*   **Actions & Tooling:**
    1.  Rédiger un rapport de revue Markdown structuré (ex: `pr_{{activePullRequest.id}}_review_report_{{timestamp}}.md`).
    2.  Le rapport doit inclure:
        *   **Informations sur la PR:** ID, Titre, Auteur, Branches, Liens vers US/Tâches.
        *   **Résumé de la Revue:** Appréciation globale, nombre de problèmes par sévérité.
        *   **Détail des Problèmes:** Pour chaque problème :
            *   Sévérité (Critique, Majeur, Mineur, Informatif).
            *   Description claire du problème.
            *   Fichier(s) et ligne(s) concerné(s).
            *   Suggestion de correction ou question au développeur.
        *   **Points Positifs (Optionnel):** Bonnes pratiques observées.
    3.  Sauvegarder le rapport dans le répertoire `{{activePullRequest.reviewDirectoryPath}}`.
*   **Memory Bank Interaction (via Scribe):**
    *   Le chemin du rapport sera ajouté à `documentationRegistry` et lié à l'entrée de la PR dans `memoryBank.pullRequests`.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Revue de PR Azure#{{activePullRequest.id}} terminée. [N_crit] critiques, [N_maj] majeurs, [N_min] mineurs trouvés. Rapport sauvegardé dans `{{activePullRequest.reviewDirectoryPath}}pr_{{activePullRequest.id}}_review_report_{{timestamp}}.md`. Recommandation: [Action principale suggérée, ex: Adresser les points critiques avant merge]."

### Phase 5: Notification et Mise à Jour de `.pheromone`
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@code-reviewer-assistant`.
*   **Actions & Tooling:**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `documentationRegistry`: Ajouter le chemin complet du rapport de revue.
        *   `memoryBank.pullRequests.{{activePullRequest.id}}`:
            *   `status`: "ReviewedByAI".
            *   `reviewReportPath`: Chemin du rapport.
            *   `lastReviewTimestamp`: `{{timestamp}}`.
            *   `issuesFoundCounts`: { `critical`: N_crit, `major`: N_maj, `minor`: N_min }.
            *   `summary`: Extrait du résumé de `@code-reviewer-assistant`.
    3.  (Optionnel, si configuré et MCP disponible) `@devops-connector` pourrait être invoqué par l'UO pour poster un commentaire sur la PR dans Azure DevOps avec un lien vers le rapport.
*   **Memory Bank Interaction:**
    *   Écriture: Mise à jour détaillée de l'entrée de la PR dans `memoryBank.pullRequests`.
*   **Output:** `.pheromone` mis à jour. L'UO est informé que la revue est terminée, et peut notifier le Tech Lead.

---