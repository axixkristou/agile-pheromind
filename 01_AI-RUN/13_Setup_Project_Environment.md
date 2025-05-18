# Workflow: Mise en Place de l'Environnement de Projet .NET/Angular (13_Setup_Project_Environment.md)

**Objectif:** Initialiser un nouvel environnement de projet pour une application .NET (API Backend) et Angular (Frontend). Cela comprend la création de la structure de dossiers standard, la génération des fichiers de configuration de base, l'initialisation d'un dépôt Git, la création de Dockerfiles et d'un stub de pipeline Azure DevOps pour le déploiement sur AKS, et la connexion au projet Azure DevOps spécifié.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@project-setup-agent`.

**MCPs Utilisés:** Git Tools MCP, Azure DevOps MCP, (potentiellement `command_line_tool` pour les CLIs .NET et Angular si des MCPs plus spécifiques ne sont pas disponibles ou si l'agent est configuré pour les utiliser).

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur (Tech Lead/Architecte) demande la création d'un nouveau projet (ex: `"AgilePheromind initialise nouveau projet 'NomProjetSuperApp' pour Azure DevOps org 'MaSuperOrganisation', projet Azure DevOps 'SuperAppProject'."`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Validation des Paramètres et Création de la Structure de Base.**
        *   UO délègue à `@project-setup-agent`.
    *   **Phase 2: Initialisation du Backend .NET.**
        *   UO délègue à `@project-setup-agent`.
    *   **Phase 3: Initialisation du Frontend Angular.**
        *   UO délègue à `@project-setup-agent`.
    *   **Phase 4: Configuration Docker et Azure Pipelines (Stubs).**
        *   UO délègue à `@project-setup-agent`.
    *   **Phase 5: Initialisation du Dépôt Git et Connexion Azure DevOps.**
        *   UO délègue à `@project-setup-agent`.
    *   **Phase 6: Mise à Jour de `.pheromone` et Rapport.**
        *   Scribe enregistre les informations du nouveau projet.

## Détails des Phases:

### Phase 1: Validation des Paramètres et Création de la Structure de Base
*   **Agent Responsable:** `@project-setup-agent`
*   **Inputs:** Nom du projet applicatif (ex: `NomProjetSuperApp`), nom de l'organisation Azure DevOps, nom du projet Azure DevOps (fournis par l'UO).
*   **Actions & Tooling:**
    1.  Valider les noms pour éviter les caractères invalides pour les chemins de dossiers ou les noms de projets.
    2.  Créer le répertoire racine du projet (ex: `NomProjetSuperApp/`).
    3.  À l'intérieur, créer une structure de dossiers standard :
        *   `backend/` (pour le code .NET)
        *   `frontend/` (pour le code Angular)
        *   `docs/` (pour la documentation générale du projet)
        *   `scripts/` (pour les scripts d'aide divers)
        *   `.azuredevops/` (pour les stubs de pipelines)
        *   `.github/` (si GitHub est utilisé pour le repo, sinon ignorer ou adapter pour Azure Repos)
*   **Memory Bank Interaction:**
    *   Aucune pour cette phase, prépare le terrain.
*   **Output (interne à `@project-setup-agent`):** Structure de dossiers de base créée.

### Phase 2: Initialisation du Backend .NET
*   **Agent Responsable:** `@project-setup-agent`
*   **Inputs:** Chemin vers le répertoire `backend/`.
*   **Actions & Tooling:**
    1.  **Création de la Solution et des Projets .NET:**
        *   (Via `command_line_tool` ou un MCP .NET si disponible)
        *   `cd backend/`
        *   `dotnet new sln -n {{NomProjetApplicatif}}Solution`
        *   `dotnet new webapi -n {{NomProjetApplicatif}}.Api -o {{NomProjetApplicatif}}.Api --no-openapi --no-https` (ou avec OpenAPI si préféré)
        *   `dotnet new classlib -n {{NomProjetApplicatif}}.Application -o {{NomProjetApplicatif}}.Application`
        *   `dotnet new classlib -n {{NomProjetApplicatif}}.Domain -o {{NomProjetApplicatif}}.Domain`
        *   `dotnet new classlib -n {{NomProjetApplicatif}}.Infrastructure -o {{NomProjetApplicatif}}.Infrastructure`
        *   `dotnet sln {{NomProjetApplicatif}}Solution.sln add ./**/*.csproj`
        *   Ajouter des dépendances de base (ex: EF Core pour `.Infrastructure`, MediatR pour `.Application`).
    2.  **Configuration de Base:**
        *   Créer/Modifier `appsettings.json` et `appsettings.Development.json` dans `.Api` avec des stubs pour la connexion DB.
        *   Configurer `Program.cs` (ou `Startup.cs`) pour les services de base (Contrôleurs, EF Core DbContext stub, CORS simple).
*   **Memory Bank Interaction:**
    *   Les détails de la structure du projet .NET seront notés pour le rapport final.
*   **Output (interne à `@project-setup-agent`):** Projets .NET de base créés et configurés.

### Phase 3: Initialisation du Frontend Angular
*   **Agent Responsable:** `@project-setup-agent`
*   **Inputs:** Chemin vers le répertoire `frontend/`.
*   **Actions & Tooling:**
    1.  **Création du Projet Angular:**
        *   (Via `command_line_tool` Angular CLI)
        *   `cd frontend/`
        *   `ng new {{NomProjetApplicatif}}App --routing --style=scss --strict --standalone=false --ssr=false` (options à ajuster selon préférences)
    2.  **Configuration de Base:**
        *   Mettre en place des dossiers pour `core`, `features`, `shared` modules/components.
        *   Configurer `proxy.conf.json` pour le proxy vers l'API backend .NET pendant le développement.
        *   Installer des librairies Angular de base (ex: Angular Material ou autre UI lib si spécifié, ngx-translate pour i18n stub).
*   **Memory Bank Interaction:**
    *   Les détails de la structure du projet Angular seront notés.
*   **Output (interne à `@project-setup-agent`):** Projet Angular de base créé.

### Phase 4: Configuration Docker et Azure Pipelines (Stubs)
*   **Agent Responsable:** `@project-setup-agent`
*   **Inputs:** Chemins vers `backend/{{NomProjetApplicatif}}.Api/` et `frontend/{{NomProjetApplicatif}}App/`. Chemin vers `.azuredevops/`.
*   **Actions & Tooling:**
    1.  **Dockerfiles:**
        *   Créer `backend/{{NomProjetApplicatif}}.Api/Dockerfile` pour l'API .NET (multi-stage build, publication ASP.NET Core).
        *   Créer `frontend/{{NomProjetApplicatif}}App/Dockerfile` pour l'application Angular (multi-stage build, serveur Nginx pour servir les statiques).
    2.  **Docker Compose (Optionnel, pour dev local):**
        *   Créer `docker-compose.yml` à la racine du projet pour lancer l'API, l'App Angular, et une instance MSSQL de développement.
    3.  **Azure Pipelines Stub:**
        *   Créer `.azuredevops/azure-pipelines.yml` avec des étapes de base (stubs) pour :
            *   Build de l'API .NET (restore, build, test, publish).
            *   Build de l'App Angular (npm install, build).
            *   Build des images Docker pour API et App, push vers Azure Container Registry (ACR).
            *   Déploiement sur Azure Kubernetes Service (AKS) (utilisation de manifests Kubernetes stubs).
*   **Memory Bank Interaction:**
    *   Les chemins vers ces fichiers de configuration seront notés.
*   **Output (interne à `@project-setup-agent`):** Dockerfiles et stub de pipeline Azure DevOps créés.

### Phase 5: Initialisation du Dépôt Git et Connexion Azure DevOps
*   **Agent Responsable:** `@project-setup-agent`
*   **Inputs:** Informations du projet Azure DevOps (`currentProject` sera mis à jour).
*   **Actions & Tooling:**
    1.  **Initialisation Git:**
        *   Utiliser **Git Tools MCP**:
            *   `git_init` (à la racine du projet).
            *   `create_gitignore` avec des templates combinés pour .NET (VisualStudio) et Node/Angular.
            *   `add_all_changes`.
            *   `commit_files {message: "Initial project structure for .NET API and Angular App"}`.
    2.  **Connexion Azure DevOps:**
        *   Utiliser **Azure DevOps MCP**:
            *   `get_project_details {organizationName, projectName}` pour confirmer l'existence du projet Azure DevOps. Si le projet n'existe pas, l'agent doit le signaler et demander sa création ou un nom de projet valide.
            *   (Conceptuel) Obtenir l'URL du dépôt Git Azure Repos pour le projet.
    3.  **Configurer Remote Git (si l'URL du repo est obtenue):**
        *   Utiliser **Git Tools MCP**:
            *   `add_remote {remoteName: "origin", remoteUrl: "URL_DU_REPO_AZURE"}`.
            *   `push_commits {remoteName: "origin", branchName: "main", setUpstream: true}` (ou `develop` si c'est la branche par défaut).
*   **Memory Bank Interaction (via Scribe):**
    *   Les informations `currentProject.id`, `currentProject.name`, `currentProject.azureDevOpsOrgUrl`, `currentProject.repositoryUrl` seront mises à jour/complétées.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Initialisation du dépôt Git local terminée pour projet '[NomProjetApplicatif]'. Premier commit effectué. Connexion au projet Azure DevOps '[NomProjetAzure]' dans l'organisation '[NomOrgAzure]' [réussie/échouée - si échec, raison]. URL du dépôt distant configurée: '[URLRepoAzure]' (si réussi)."

### Phase 6: Mise à Jour de `.pheromone` et Rapport
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumés des phases précédentes, particulièrement de `@project-setup-agent`.
*   **Actions & Tooling:**
    1.  Interpréter les résumés via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `currentProject`: Remplir avec `id` (si retourné par Azure DevOps MCP ou généré), `name`, `azureDevOpsOrgUrl`, `repositoryUrl`, `defaultBranch`.
        *   `memoryBank.projectContext`: Confirmer/Mettre à jour `techStack`, `azureDevOpsProject: currentProject.name`. Initialiser `codingConventionsVersion` et `designConventionsVersion` à "1.0_initial".
        *   `documentationRegistry`: Ajouter les chemins vers les Dockerfiles, `azure-pipelines.yml`, et un rapport de setup (`project_setup_details_[NomProjetApplicatif]_[timestamp].md` que `@project-setup-agent` aura généré et sauvegardé dans `02_AI-DOCS/Setup/`).
    3.  Mettre à jour `systemHealth.mcpStatus.azureDevOpsMCP` en fonction du résultat de la connexion.
*   **Memory Bank Interaction:**
    *   Écriture: Initialisation et remplissage de `currentProject` et des sections pertinentes de `memoryBank.projectContext`.
*   **Output:** `.pheromone` mis à jour. L'UO est informé que la configuration du projet est terminée. Un message de confirmation est envoyé à l'utilisateur.

---