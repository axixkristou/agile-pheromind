# Workflow: Mise en Place de l'Environnement de Projet .NET/Angular (13_Setup_Project_Environment.md)

**Objectif:** Initialiser un nouvel environnement de projet pour une application .NET (API Backend) et Angular (Frontend). Cela comprend la création de la structure de dossiers, la génération des fichiers de configuration de base via les CLIs .NET et Angular, l'initialisation d'un dépôt Git, la création de Dockerfiles et de stubs de pipeline Azure DevOps pour AKS, et la connexion au projet Azure DevOps spécifié. Une gestion robuste des erreurs pour les commandes CLI et les appels MCP est intégrée.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@project-setup-agent`, `@clarification-agent`.

**MCPs Utilisés:** Git Tools MCP, Azure DevOps MCP, `command_line_tool` (pour exécuter `dotnet new` et `ng new` si des MCPs dédiés ne sont pas disponibles/utilisés).

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Tech Lead/Architecte) demande la création d'un nouveau projet, en fournissant le nom de l'application, le nom de l'organisation Azure DevOps et le nom du projet Azure DevOps (ex: `"AgilePheromind initialise projet 'SuperApp' pour ADO org 'MaOrg', projet ADO 'SuperAppProject'"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Validation des Paramètres et Création de la Structure de Base.**
        *   UO délègue à `@project-setup-agent`.
        *   **onError:** Si les noms fournis sont invalides pour des chemins de dossier, UO demande clarification à l'utilisateur via `@clarification-agent`.
    *   **Phase 2: Initialisation du Backend .NET (avec gestion d'erreur CLI).**
        *   UO délègue à `@project-setup-agent`.
        *   **onError:** Si les commandes `dotnet new` échouent, l'agent loggue l'erreur, le signale à l'UO qui notifie l'utilisateur et peut arrêter ou proposer des solutions.
    *   **Phase 3: Initialisation du Frontend Angular (avec gestion d'erreur CLI).**
        *   UO délègue à `@project-setup-agent`.
        *   **onError:** Si `ng new` échoue, gestion similaire à la Phase 2.
    *   **Phase 4: Configuration Docker et Azure Pipelines (Stubs).**
        *   UO délègue à `@project-setup-agent`.
    *   **Phase 5: Initialisation du Dépôt Git et Connexion Azure DevOps.**
        *   UO délègue à `@project-setup-agent`.
        *   **onError:** Si Git MCP ou ADO MCP échouent, logguer, notifier l'utilisateur. Le workflow peut continuer avec un repo local non synchronisé si la connexion ADO échoue, avec un avertissement.
    *   **Phase 6: Mise à Jour de `.pheromone` et Rapport Final.**
        *   Scribe enregistre les informations.

## Détails des Phases:

### Phase 1: Validation des Paramètres et Création de la Structure de Base
*   **Agent Responsable:** `@project-setup-agent`, UO, `@clarification-agent`.
*   **Inputs:** Nom projet applicatif, nom org ADO, nom projet ADO (de l'UO).
*   **Actions (`@project-setup-agent`):**
    1.  Valider les noms pour les chemins de dossiers. S'ils contiennent des caractères invalides, le signaler à l'UO.
    2.  Si OK, créer répertoire racine, puis `backend/`, `frontend/`, `docs/`, `scripts/`, `.azuredevops/`.
*   **onError (Validation Noms par UO):**
    *   Si `@project-setup-agent` signale des noms invalides:
        *   UO met workflow en pause (`activeWorkflow.status: 'PendingClarification_ProjectNames'`).
        *   UO délègue à `@clarification-agent` pour demander à l'utilisateur de corriger les noms.
        *   Attendre réponse via `01_AI-RUN/XX_Handle_Clarification_Response.md`.
*   **Output (interne si succès):** Structure de dossiers de base créée.

### Phase 2: Initialisation du Backend .NET (avec gestion d'erreur CLI)
*   **Agent Responsable:** `@project-setup-agent`.
*   **Inputs:** Chemin `backend/`, NomProjetApplicatif.
*   **Actions (Utilisation de `command_line_tool` pour `dotnet`):**
    1.  `cd backend/`
    2.  `dotnet new sln -n {{NomProjetApplicatif}}Solution`
    3.  `dotnet new webapi -n {{NomProjetApplicatif}}.Api ...`
    4.  `dotnet new classlib ...` (pour Application, Domain, Infrastructure)
    5.  `dotnet sln add ...`
    6.  Ajouter dépendances de base (`dotnet add package ...`).
    7.  Créer/Modifier `appsettings.json`, `Program.cs`/`Startup.cs` stubs.
*   **onError (Commandes `dotnet`):**
    *   Si une commande `dotnet` échoue, `@project-setup-agent` capture la sortie d'erreur.
    *   Signale à l'UO: "Échec commande `dotnet {{commande}}` dans `backend/`. Erreur: [sortie d'erreur].".
    *   UO loggue via Scribe (`activeWorkflow.lastError`), notifie l'utilisateur. Le workflow peut s'arrêter ou l'UO peut demander à l'utilisateur s'il veut ignorer cette étape et continuer (avec un backend partiellement configuré).
*   **Output (interne si succès):** Projets .NET de base.

### Phase 3: Initialisation du Frontend Angular (avec gestion d'erreur CLI)
*   **Agent Responsable:** `@project-setup-agent`.
*   **Inputs:** Chemin `frontend/`, NomProjetApplicatif.
*   **Actions (Utilisation de `command_line_tool` pour `ng`):**
    1.  `cd frontend/`
    2.  `ng new {{NomProjetApplicatif}}App --routing --style=scss ...`
    3.  Configurer structure modules (`core`, `features`, `shared`), `proxy.conf.json`.
*   **onError (Commande `ng new`):**
    *   Si `ng new` échoue, `@project-setup-agent` capture la sortie d'erreur.
    *   Signale à l'UO: "Échec commande `ng new` dans `frontend/`. Erreur: [sortie d'erreur].".
    *   Gestion par l'UO similaire à la Phase 2.
*   **Output (interne si succès):** Projet Angular de base.

### Phase 4: Configuration Docker et Azure Pipelines (Stubs)
*   **Agent Responsable:** `@project-setup-agent`.
*   **Inputs:** Chemins vers projets API et App, `.azuredevops/`.
*   **Actions:**
    1.  Créer `Dockerfile` pour API .NET.
    2.  Créer `Dockerfile` pour App Angular.
    3.  Créer `docker-compose.yml` stub (optionnel).
    4.  Créer `.azuredevops/azure-pipelines.yml` stub (build .NET, build Angular, build Docker, push ACR, deploy AKS).
*   **onError (Création Fichiers):** Si erreur d'écriture de fichier, logguer et notifier. Le setup peut continuer mais ces fichiers seront manquants.
*   **Output (interne):** Dockerfiles et stub de pipeline.

### Phase 5: Initialisation du Dépôt Git et Connexion Azure DevOps
*   **Agent Responsable:** `@project-setup-agent`.
*   **Inputs:** Infos projet ADO (sera mis à jour dans `currentProject`).
*   **Actions (Git Tools MCP & Azure DevOps MCP):**
    1.  **Git Tools MCP:** `git_init`, `create_gitignore` (.NET & Node/Angular), `add_all_changes`, `commit_files {message: "feat: initial project structure for .NET API and Angular App"}`.
    2.  **Azure DevOps MCP:** `get_project_details {organizationName, projectName}`.
        *   **onError (ADO `get_project_details`):** Si échec, `@project-setup-agent` signale à l'UO: "Projet ADO '{{projectName}}' introuvable ou inaccessible: [Erreur MCP].". L'UO peut demander à l'utilisateur de vérifier/créer le projet dans ADO et fournir le nom correct, ou continuer sans configurer le remote Git.
    3.  Si ADO projet OK, obtenir URL repo et configurer remote: `Git Tools MCP` (`add_remote`, `push_commits`).
        *   **onError (Git Push/Remote):** Si échec, logguer, notifier le dev. Le repo local existe mais n'est pas sur ADO.
*   **Output (vers Scribe):** Résumé NL: "Init Git local OK. Commit initial. Connexion projet ADO '{{NomProjetAzure}}' [réussie/échouée (raison)]. Remote Git [configuré et pushé/non configuré]."

### Phase 6: Mise à Jour de `.pheromone` et Rapport Final
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** Résumés des phases, particulièrement de `@project-setup-agent`.
*   **Actions:**
    1.  Interpréter résumés via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `currentProject`: Infos ADO (ID, nom, org URL, repo URL si succès).
        *   `memoryBank.projectContext`: Stack, nom projet ADO, versions conventions "1.0_initial".
        *   `documentationRegistry`: Ajouter rapport de setup (`project_setup_details...md` créé par `@project-setup-agent` dans `02_AI-DOCS/Setup/`), Dockerfiles, `azure-pipelines.yml`.
        *   `systemHealth.mcpStatus.azureDevOpsMCP` (OK/Failed).
        *   `activeWorkflow.lastError` si des erreurs non bloquantes ont eu lieu.
*   **Output:** `.pheromone` à jour. UO informe l'utilisateur du statut final du setup.

---