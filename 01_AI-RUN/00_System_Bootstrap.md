# Workflow: Amorçage et Configuration du Système AgilePheromind (00_System_Bootstrap.md)

**Objectif:** Initialiser ou vérifier la configuration fondamentale du système AgilePheromind, y compris le fichier `.pheromone`, les conventions de base, et la connexion aux services essentiels comme Azure DevOps. Ce workflow est typiquement exécuté une fois au début d'un projet ou après des changements de configuration majeurs.

**Agents IA Clés:** `🧐 @uber-orchestrator`, `✍️ @orchestrator-pheromone-scribe`, `@project-setup-agent`, `@devops-connector`, `@architecture-advisor-agent`.

**MCPs Utilisés:** Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur lance ce workflow (ex: `"Bootstrap AgilePheromind pour le projet [NomProjetAzureDevOps] dans l'organisation [NomOrganisationAzureDevOps]"`).
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Initialisation/Vérification de `.pheromone`.**
        *   Délègue au `✍️ @orchestrator-pheromone-scribe` (qui a une logique de bootstrap interne).
    *   **Phase 2: Configuration du Contexte du Projet dans la Memory Bank.**
        *   Délègue au `@project-setup-agent` pour collecter les informations et au Scribe pour enregistrer.
    *   **Phase 3: Initialisation des Conventions de Codage et de Design.**
        *   Délègue à `@architecture-advisor-agent` pour créer les fichiers de conventions initiaux à partir de modèles.
    *   **Phase 4: Vérification de la Connexion Azure DevOps.**
        *   Délègue au `@devops-connector`.
    *   **Phase 5: Rapport de Bootstrap.**
        *   L'UO compile un résumé des actions et le Scribe enregistre.

## Détails des Phases:

### Phase 1: Initialisation/Vérification de `.pheromone`
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe` (via l'UO)
*   **Inputs:** Aucun input spécifique requis pour cette phase, le Scribe a une logique de bootstrap.
*   **Actions & Tooling:**
    1.  L'UO s'assure que le Scribe est activé.
    2.  Le Scribe, lors de son premier chargement (ou si `.pheromone` est manquant/invalide), initialise le fichier `.pheromone` avec la structure de base définie (incluant `systemVersion`, `memoryBank` vide avec ses sections, `documentationRegistry` vide, `currentUser` à null, etc.).
    3.  Si `.pheromone` existe, le Scribe le charge et valide sa structure de base.
*   **Memory Bank Interaction:**
    *   Écriture: Initialisation de la structure de la Memory Bank si le fichier est nouveau.
*   **Output (implicite, état du Scribe):** Fichier `.pheromone` initialisé et prêt.

### Phase 2: Configuration du Contexte du Projet dans la Memory Bank
*   **Agent Responsable:** `@project-setup-agent`
*   **Inputs:** Nom du projet Azure DevOps et URL de l'organisation (fournis par l'utilisateur ou l'UO).
*   **Actions & Tooling:**
    1.  Collecter/Confirmer les informations de base du projet:
        *   Nom du projet Azure DevOps.
        *   URL de l'organisation Azure DevOps.
        *   URL du dépôt Git principal (déduit ou demandé).
        *   Stack technologique par défaut (déjà dans `.roomodes` de `@project-setup-agent` mais confirmé ici).
    2.  Préparer un objet JSON pour `memoryBank.projectContext` et `currentProject`.
*   **Memory Bank Interaction:**
    *   Préparation des données pour la Memory Bank.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Contexte du projet '[NomProjet]' collecté. Stack: [.NET, Angular, Azure SQL, Docker, AKS]. Azure DevOps Org: '[URLOrg]', Projet: '[NomProjet]', Repo: '[URLRepo]'. Prêt à mettre à jour `.pheromone`."

### Phase 3: Initialisation des Conventions de Codage et de Design
*   **Agent Responsable:** `@architecture-advisor-agent`
*   **Inputs:** Chemins vers les modèles de conventions (ex: `02_AI-DOCS/Conventions/coding_conventions_template.md`, `02_AI-DOCS/Conventions/design_conventions_template.md`).
*   **Actions & Tooling:**
    1.  Vérifier si les fichiers de conventions finaux (ex: `02_AI-DOCS/Conventions/coding_conventions.md`) existent.
    2.  Si non existants, copier les fichiers modèles vers les emplacements finaux.
    3.  Peut faire des ajustements initiaux minimes basés sur la stack technologique confirmée (ex: sections spécifiques .NET/Angular).
*   **Memory Bank Interaction:**
    *   Préparation des entrées pour `documentationRegistry`.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Fichiers de conventions initialisés: `coding_conventions.md` et `design_conventions.md` créés à partir des modèles dans `02_AI-DOCS/Conventions/`. Ces documents sont prêts pour une personnalisation plus poussée."

### Phase 4: Vérification de la Connexion Azure DevOps
*   **Agent Responsable:** `@devops-connector`
*   **Inputs:** Informations du projet depuis `.pheromone.currentProject` (mis à jour par le Scribe après Phase 2).
*   **Actions & Tooling:**
    1.  Utiliser **Azure DevOps MCP**:
        *   `get_project_details {projectName: .pheromone.currentProject.name}`: Pour vérifier que le projet est accessible.
        *   `get_user_identity`: Pour s'assurer que l'authentification MCP fonctionne.
*   **Memory Bank Interaction:**
    *   Peut mettre à jour `systemHealth.mcpStatus.azureDevOpsMCP` dans `.pheromone` via le Scribe.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Connexion à Azure DevOps pour le projet '[NomProjet]' vérifiée avec succès. Identité utilisateur MCP: '[IdentitéMCP]'." ou "Échec de la connexion à Azure DevOps: [Message d'erreur]."

### Phase 5: Rapport de Bootstrap
*   **Agent Responsable:** `🧐 @uber-orchestrator` (pour la compilation), `✍️ @orchestrator-pheromone-scribe` (pour l'enregistrement final)
*   **Inputs:** Résumés des phases précédentes.
*   **Actions & Tooling (`🧐 @uber-orchestrator`):**
    1.  Compiler un résumé global du processus de bootstrap.
*   **Output (`🧐 @uber-orchestrator` vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Bootstrap du système AgilePheromind terminé. `.pheromone` initialisé. Contexte du projet configuré pour '[NomProjet]'. Conventions de base établies. Connexion Azure DevOps [OK/Échouée]. Système prêt pour opérations. Rapport détaillé: `system_bootstrap_report_[timestamp].md`."
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  Enregistrer le rapport de bootstrap dans `documentationRegistry` (ex: `02_AI-DOCS/System_Admin/system_bootstrap_report_[timestamp].md`).
    2.  S'assurer que toutes les mises à jour des phases précédentes sont bien persistées dans `.pheromone`.
*   **Memory Bank Interaction:**
    *   Écriture: Archivage du rapport de bootstrap.
*   **Outcome:** Le système AgilePheromind est configuré avec un `.pheromone` initialisé, le contexte du projet de base dans la Memory Bank, des fichiers de conventions prêts, et une connectivité vérifiée avec Azure DevOps.

---