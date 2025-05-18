# Workflow: Finaliser une Tâche et Préparer le Commit (06_Finalize_Task_Commit.md)

**Objectif:** Guider le développeur à travers les étapes de finalisation d'une tâche technique. Cela inclut la vérification rigoureuse que tous les tests (unitaires, intégration) passent et que les linters sont satisfaits. Ensuite, préparer un message de commit conforme "Conventional Commits", exécuter le commit, et mettre à jour les statuts dans `.pheromone` et Azure DevOps, avec une gestion des erreurs pour chaque étape.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@developer-agent`, `@commit-pr-formatter`, `@devops-connector`, `@clarification-agent`.

**MCPs Utilisés:** Git Tools MCP, Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Le développeur (Dev) signale la fin d'une tâche et l'US associée (ex: `"AgilePheromind: Tâche Azure#23223 est terminée. Prépare commit pour US Azure#12323."`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Vérification Pré-Commit Approfondie (Tests et Linters).**
        *   UO délègue à `@developer-agent`.
        *   **onError:** Si échecs, notifier le dev, stocker les détails de l'échec dans `memoryBank.tasks.{{activeTask.id}}.lastVerificationFailure`, arrêter le workflow pour cette action.
    *   **Phase 2: Génération du Message de Commit.**
        *   UO **injecte contexte** (titres US/tâche depuis `.pheromone`) à `@commit-pr-formatter`.
    *   **Phase 3: Validation Utilisateur et Exécution du Commit.**
        *   UO présente le message au dev via `ask_followup_question`.
        *   Si validé, UO ordonne à `@developer-agent` (ou `@commit-pr-formatter`) d'utiliser **Git Tools MCP**.
        *   **onError (Git Commit):** Si `commit_files` échoue (ex: hook pre-commit échoue, conflit de fusion non géré localement), logguer l'erreur, notifier le dev, suggérer des actions (ex: `git status`, résoudre conflits). Le workflow s'arrête jusqu'à résolution.
    *   **Phase 4: Mise à Jour des Statuts (Azure DevOps et `.pheromone`).**
        *   UO délègue à `@devops-connector` pour ADO.
        *   **onError (ADO Update):** Si la mise à jour ADO échoue, logguer l'erreur. `.pheromone` sera mis à jour, mais une note indiquera l'échec de synchro ADO. L'UO peut suggérer une nouvelle tentative ou une màj manuelle.
        *   Scribe met à jour `.pheromone`.

## Détails des Phases:

### Phase 1: Vérification Pré-Commit Approfondie (Tests et Linters)
*   **Agent Responsable:** `@developer-agent`
*   **Inputs:** Contexte de `activeTask`, `activeUserStory` depuis `.pheromone`. Accès code local.
*   **Actions & Tooling:**
    1.  Exécuter linters (.NET, Angular) via `memoryBank.toolingConfigurations.linters`.
    2.  Exécuter tests unitaires/intégration pertinents (identifiés via `memoryBank.tasks.{{activeTask.id}}.testCasesGenerated` et tests auto-découverts).
*   **onError Strategy (pour `@developer-agent` à remonter à UO):**
    1.  Si erreurs de linting ou échecs de tests :
        *   Compiler un rapport d'échec détaillé (quels linters/tests, messages d'erreur).
        *   Soumettre à l'UO/Scribe: "Vérification pré-commit Tâche Azure#{{activeTask.id}} ÉCHOUÉE. Rapport: `pre_commit_check_failure_{{activeTask.id}}_{{timestamp}}.md`. Corrigez avant commit." (Rapport dans `03_SPECS/Verification_Failures/`).
        *   Scribe enregistre le lien vers le rapport dans `memoryBank.tasks.{{activeTask.id}}.lastVerificationFailure`.
        *   UO arrête le workflow pour cette action.
*   **Output (vers UO/Scribe si succès):** "Vérification pré-commit Tâche Azure#{{activeTask.id}} RÉUSSIE. Linters OK. Tests OK. Log: `pre_commit_check_success_{{activeTask.id}}_{{timestamp}}.log`." (Log dans `03_SPECS/Verification_Logs/`).

### Phase 2: Génération du Message de Commit
*   **Agent Responsable:** `@commit-pr-formatter`
*   **Inputs (Injectés par l'UO):**
    *   Confirmation que Phase 1 OK.
    *   `activeTask.title` et `activeUserStory.title` (depuis `.pheromone`).
    *   (Optionnel) Résumé des changements clés si `@developer-agent` l'a fourni.
*   **Actions & Tooling:**
    1.  **Git Tools MCP** (`get_changed_files_staged`) pour la liste des fichiers.
    2.  Proposer message "Conventional Commit" (type, scope, description, body, footer avec `Resolves Azure#...`, `Closes Azure#...`).
*   **Output (vers UO):** Message de commit formaté.

### Phase 3: Validation Utilisateur et Exécution du Commit
*   **Agent Responsable:** `🧐 @uber-orchestrator` (interaction), `@developer-agent` (exécution).
*   **Inputs:** Message de commit proposé.
*   **Actions & Tooling (UO):**
    1.  `ask_followup_question` pour validation du message par le dev.
*   **Si utilisateur confirme "oui" (`@developer-agent`):**
    1.  Utiliser **Git Tools MCP**: `add_all_changes`, `commit_files {message: validatedCommitMessage}`.
    2.  **onError (Git Commit):** Si `commit_files` échoue:
        *   `@developer-agent` remonte l'erreur Git exacte à l'UO.
        *   UO notifie le dev: "Échec du commit pour Tâche Azure#{{activeTask.id}}. Erreur Git: [Message Erreur]. Veuillez vérifier `git status` et résoudre les problèmes (conflits, hooks ?)."
        *   Scribe loggue l'échec dans `memoryBank.tasks.{{activeTask.id}}.commitHistory`.
        *   Workflow arrêté jusqu'à résolution manuelle ou nouvelle tentative.
    3.  Si commit OK, (Optionnel) `push_commits`.
*   **Output (`@developer-agent` vers Scribe si succès):** Résumé NL: "Commit Tâche Azure#{{activeTask.id}} OK. Hash: `{{commitHash}}`. [Push OK/Non effectué]."

### Phase 4: Mise à Jour des Statuts (Azure DevOps et `.pheromone`)
*   **Agent Responsable:** `@devops-connector`, Scribe.
*   **Inputs:** Confirmation commit OK (Phase 3). `activeTask.id`, `activeUserStory.id`, `commitHash`.
*   **Actions & Tooling (`@devops-connector`):**
    1.  **Azure DevOps MCP**: `update_work_item_status {id: activeTask.id, status: "Done", comment: "Implémentation commité. Commit: {{commitHash}}"}`.
*   **onError Strategy (pour l'UO si `@devops-connector` signale échec ADO Update):**
    1.  Scribe loggue l'erreur dans `activeWorkflow.lastError` et note l'échec de synchro ADO dans `memoryBank.tasks.{{activeTask.id}}.syncIssues`.
    2.  UO notifie le dev: "Commit {{commitHash}} réussi, mais la mise à jour du statut de la Tâche Azure#{{activeTask.id}} dans Azure DevOps a échoué: [Erreur MCP]. Le statut local est 'Done'. Veuillez vérifier ADO ou relancer la synchro."
    3.  Le workflow continue pour la mise à jour `.pheromone`.
*   **Output (`@devops-connector` vers Scribe):** Résumé NL: "Statut Tâche Azure#{{activeTask.id}} màj à 'Done' dans ADO (Commit: {{commitHash}})." ou "Échec màj statut Tâche Azure#{{activeTask.id}} dans ADO."
*   **Actions & Tooling (Scribe):**
    1.  Mettre à jour `.pheromone`:
        *   `activeUserStory.tasks` (pour la tâche): `status: "DoneByPheromind"`. `activeTask` -> `null`.
        *   `memoryBank.tasks.{{activeTask.id}}`: `status: "Done"`, `statusHistory` màj, `relatedCommit: "{{commitHash}}"`. Si échec synchro ADO, ajouter à `syncIssues`.
        *   `memoryBank.commits.{{commitHash}}`: { `message`, `author`, `timestamp`, `relatedTask`, `relatedUS` }.
    2.  **Vérifier complétion US:** Si toutes tâches `Done`, màj `memoryBank.userStories.{{activeUserStory.id}}.status` -> "ReadyForReview". Notifier UO.
*   **Output:** `.pheromone` mis à jour. UO informé pour notifier dev/prochaines étapes.

---