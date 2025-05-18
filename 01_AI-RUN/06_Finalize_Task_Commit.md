# Workflow: Finaliser une Tâche et Préparer le Commit (06_Finalize_Task_Commit.md)

**Objectif:** Guider le développeur à travers les étapes de finalisation d'une tâche technique. Cela inclut la vérification que tous les tests unitaires et d'intégration passent, la préparation d'un message de commit conforme aux normes "Conventional Commits", l'exécution du commit, et la mise à jour du statut de la tâche dans `.pheromone` et Azure DevOps.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@developer-agent`, `@commit-pr-formatter`, `@devops-connector`.

**MCPs Utilisés:** Git Tools MCP, Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Le développeur (Dev) signale la fin d'une tâche et l'US associée (ex: `"AgilePheromind: La tâche Azure#23223 est terminée. Prépare le commit pour l'US Azure#12323."`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Vérification Pré-Commit (Tests et Linters).**
        *   UO délègue à `@developer-agent` pour exécuter les vérifications locales.
    *   **Phase 2: Génération du Message de Commit.**
        *   UO délègue à `@commit-pr-formatter` pour proposer un message standardisé.
    *   **Phase 3: Validation Utilisateur et Exécution du Commit.**
        *   UO présente le message au développeur pour validation via `ask_followup_question`.
        *   Si validé, UO ordonne à `@developer-agent` (ou `@commit-pr-formatter`) d'utiliser **Git Tools MCP** pour commiter.
    *   **Phase 4: Mise à Jour des Statuts (Azure DevOps et `.pheromone`).**
        *   UO délègue à `@devops-connector` pour Azure DevOps.
        *   Scribe met à jour `.pheromone` basé sur les résumés.

## Détails des Phases:

### Phase 1: Vérification Pré-Commit (Tests et Linters)
*   **Agent Responsable:** `@developer-agent`
*   **Inputs:** Contexte de `activeTask` et `activeUserStory` depuis `.pheromone`. Accès au code source local.
*   **Actions & Tooling:**
    1.  **Exécuter les Linters:** Lancer les analyseurs statiques configurés pour le projet (.NET Roslyn Analyzers/StyleCop, Angular ESLint) sur les fichiers modifiés pour la tâche.
    2.  **Exécuter les Tests Unitaires et d'Intégration:**
        *   Identifier les tests pertinents pour `activeTask` et `activeUserStory` (peut consulter `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated`).
        *   Exécuter ces tests (ex: `dotnet test` pour .NET, `ng test --code-coverage` pour Angular).
    3.  **Rapporter les Résultats:**
        *   Si des erreurs de linting ou des échecs de tests sont détectés:
            *   Soumettre un résumé à l'UO/Scribe: "Vérification pré-commit pour Tâche Azure#{{activeTask.id}} ÉCHOUÉE. Erreurs de Linter: [Nombre/Détails]. Tests Échoués: [Nombre/Noms]. Veuillez corriger avant de commiter."
            *   Le workflow s'arrête ici pour cette action, en attente de correction par le développeur.
        *   Si tout est OK:
            *   Soumettre un résumé à l'UO/Scribe: "Vérification pré-commit pour Tâche Azure#{{activeTask.id}} RÉUSSIE. Linters OK. Tous les tests pertinents passent."
*   **Memory Bank Interaction (via Scribe):**
    *   En cas d'échec, une note peut être ajoutée à `memoryBank.tasks.{{activeTask.id}}.developerNotes`.
    *   En cas de succès, le Scribe peut enregistrer le log de test dans `documentationRegistry` si pertinent.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe` et UO):** Résumé NL indiquant le succès ou l'échec des vérifications pré-commit.

### Phase 2: Génération du Message de Commit
*   **Agent Responsable:** `@commit-pr-formatter`
*   **Inputs:** `activeTask`, `activeUserStory` depuis `.pheromone`. Information que les vérifications pré-commit sont OK (de l'UO).
*   **Actions & Tooling:**
    1.  Utiliser **Git Tools MCP** (`get_changed_files_staged` ou `git status --porcelain` via `command_line_tool` si MCP plus direct non dispo) pour identifier les fichiers modifiés et prêts à être commité (staged).
    2.  Consulter `memoryBank.tasks.{{activeTask.id}}.title` et `memoryBank.userStories.{{activeUserStory.id}}.title`.
    3.  Proposer un message de commit structuré selon "Conventional Commits":
        *   **Type:** `feat` (nouvelle fonctionnalité pour l'utilisateur), `fix` (correction de bug), `build` (affecte le système de build ou dépendances externes), `chore` (pas de changement de code de prod), `ci` (changements CI/CD), `docs` (documentation uniquement), `perf` (amélioration de performance), `refactor` (ni fix ni feat), `style` (formatage), `test` (ajout/correction de tests).
        *   **Scope (Optionnel):** Le module ou la partie de l'application impactée (ex: `auth`, `order-processing`, `product-details-ui`). Peut être dérivé du nom de la tâche/US.
        *   **Description:** Sujet concis en impératif présent.
        *   **Body (Optionnel):** Motivation du changement et contraste avec le comportement précédent.
        *   **Footer (Optionnel):** `BREAKING CHANGE: description`, ou `Resolves: Azure#{{activeUserStory.id}}`, `Closes: Azure#{{activeTask.id}}`.
        *   *Exemple:* `feat(auth): implement Google sign-in for user registration\n\nAllows new users to register quickly using their existing Google account, reducing friction in the onboarding process.\n\nResolves: Azure#12323\nCloses: Azure#23223`
*   **Memory Bank Interaction:**
    *   Lecture: Titres/descriptions de l'US et de la tâche depuis la `memoryBank`.
*   **Output (vers `🧐 @uber-orchestrator`):** Le message de commit formaté.

### Phase 3: Validation Utilisateur et Exécution du Commit
*   **Agent Responsable:** `🧐 @uber-orchestrator` (pour l'interaction), puis `@developer-agent` (ou `@commit-pr-formatter` pour l'exécution du commit).
*   **Inputs:** Message de commit proposé par `@commit-pr-formatter`.
*   **Actions & Tooling (`🧐 @uber-orchestrator`):**
    1.  Utiliser `ask_followup_question` pour présenter le message au développeur: "Les vérifications pré-commit sont OK. Voici le message de commit proposé pour la tâche Azure#{{activeTask.id}}:\n```\n{{proposedCommitMessage}}\n```\nConfirmez-vous pour commiter ces changements ? (oui/non/modifier)"
*   **Si l'utilisateur confirme "oui" (`@developer-agent` ou `@commit-pr-formatter`):**
    1.  Utiliser **Git Tools MCP**:
        *   `add_all_changes` (ou `add_specific_files` si une liste est maintenue et plus précise).
        *   `commit_files {message: validatedCommitMessage}`.
        *   (Optionnel, peut être une action séparée) `push_commits {remoteName: 'origin', branchName: currentBranch}`.
*   **Si l'utilisateur demande à "modifier":**
    1.  UO demande au développeur de fournir le message modifié.
    2.  UO redéclenche l'action de commit avec le message fourni par l'utilisateur.
*   **Output (`@developer-agent` ou `@commit-pr-formatter` vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Commit effectué avec succès pour Tâche Azure#{{activeTask.id}}. Hash du commit: `{{commitHash}}`. [Optionnel: Push vers 'origin/{{currentBranch}}' effectué.]"

### Phase 4: Mise à Jour des Statuts (Azure DevOps et `.pheromone`)
*   **Agent Responsable:** `@devops-connector` (pour Azure DevOps), `✍️ @orchestrator-pheromone-scribe` (pour `.pheromone`).
*   **Inputs:** Confirmation que le commit a été effectué (de la Phase 3). `activeTask.id` et `activeUserStory.id`.
*   **Actions & Tooling (`@devops-connector`):**
    1.  Utiliser **Azure DevOps MCP**:
        *   `update_work_item_status {id: activeTask.id, status: "Done" (ou "Resolved", ou "Closed" - selon le workflow Azure DevOps du projet), comment: "Implémentation terminée et commité. Commit: {{commitHash}}"}`.
*   **Output (`@devops-connector` vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Statut de la tâche Azure#{{activeTask.id}} mis à jour à 'Done' dans Azure DevOps avec référence au commit {{commitHash}}."
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  Interpréter les résumés de la phase de commit et de la mise à jour Azure DevOps.
    2.  Mettre à jour `.pheromone`:
        *   `activeUserStory.tasks` (pour la tâche concernée): `status: "DoneByPheromind"`.
        *   `activeTask`: Mettre à `null` ou indiquer la complétion.
        *   `memoryBank.tasks.{{activeTask.id}}`:
            *   `status`: "Done".
            *   `statusHistory`: Ajouter { `status`: "Done", `timestamp`: "{{timestamp}}", `actor`: "AgilePheromindSystem", `details`: "Commit {{commitHash}}" }.
            *   `relatedCommit`: "{{commitHash}}".
        *   `memoryBank.commits.{{commitHash}}`: { `message`: "{{validatedCommitMessage}}", `author`: "{{currentUser.azureDevOpsUsername}}", `timestamp`: "{{timestamp}}", `relatedTask`: "Azure#{{activeTask.id}}", `relatedUS`: "Azure#{{activeUserStory.id}}" }.
    3.  **Vérification de complétion de l'US:**
        *   Vérifier si toutes les tâches dans `memoryBank.userStories.{{activeUserStory.id}}.tasks` sont maintenant "Done".
        *   Si oui, mettre à jour `memoryBank.userStories.{{activeUserStory.id}}.status` à "ReadyForReview" (ou "ReadyForTesting").
        *   Notifier l'UO ou le PO/Tech Lead de la complétion de l'US et de la disponibilité pour la revue/PR.
*   **Memory Bank Interaction (via Scribe):**
    *   Mise à jour détaillée du statut de la tâche, enregistrement du commit, et potentiellement mise à jour du statut de l'US.
*   **Outcome:** Le code est commité avec un message standardisé. La tâche est marquée comme terminée dans `.pheromone` et synchronisée avec Azure DevOps. Si l'US est complétée, le système le signale.

---