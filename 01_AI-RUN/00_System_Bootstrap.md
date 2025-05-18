# Workflow: Amorçage et Configuration du Système AgilePheromind (00_System_Bootstrap.md)

**Objectif:** Initialiser ou vérifier la configuration fondamentale du système AgilePheromind. Cela inclut la création/validation du fichier `.pheromone`, la configuration du contexte de base du projet dans la Memory Bank, l'initialisation des documents de conventions, la vérification de la connexion à Azure DevOps, et la confirmation de la présence des agents système essentiels comme `@clarification-agent`. Ce workflow est crucial pour assurer la robustesse et l'intelligence contextuelle du système dès le départ.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@project-setup-agent`, `@devops-connector`, `@architecture-advisor-agent`.

**MCPs Utilisés:** Azure DevOps MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** L'utilisateur lance ce workflow (ex: `"AgilePheromind bootstrap système pour projet 'SuperApp' dans ADO org 'MaSuperOrg'"`). L'UO peut demander des précisions si les informations sont incomplètes.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Initialisation/Vérification de `.pheromone` et des Agents Système.**
        *   UO s'assure que le Scribe est activé (qui bootstrap `.pheromone` si besoin).
        *   UO vérifie la présence de `@clarification-agent` dans `.roomodes` (conceptuel, via lecture de la config ou une capacité de l'UO).
    *   **Phase 2: Configuration du Contexte du Projet dans la Memory Bank.**
        *   UO délègue à `@project-setup-agent` pour collecter/confirmer les informations du projet.
        *   Scribe enregistre ces informations dans `.pheromone.currentProject` et `memoryBank.projectContext`.
    *   **Phase 3: Initialisation des Conventions de Codage et de Design.**
        *   UO délègue à `@architecture-advisor-agent` pour créer/vérifier les fichiers de conventions à partir de modèles.
    *   **Phase 4: Vérification de la Connexion Azure DevOps.**
        *   UO délègue à `@devops-connector`.
    *   **Phase 5: Rapport de Bootstrap et État de Préparation.**
        *   L'UO compile un résumé des actions. Scribe enregistre le rapport et l'état du système.

## Détails des Phases:

### Phase 1: Initialisation/Vérification de `.pheromone` et des Agents Système
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe` (pour `.pheromone`), `🧐 @uber-orchestrator` (pour vérification agents).
*   **Inputs:** Aucun input spécifique pour le Scribe. L'UO a accès à la configuration `.roomodes` (conceptuellement).
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  L'UO active le Scribe.
    2.  Le Scribe charge `.pheromone`. S'il est manquant/invalide, il l'initialise avec la structure de base (incluant `systemVersion`, `clarificationContext` vide, `memoryBank` avec ses sections, `documentationRegistry`, `currentUser` à null, `systemHealth` avec `lastPheromoneWriteSuccess: true`).
    3.  Si `.pheromone` existe, le Scribe valide sa structure de base.
*   **Actions & Tooling (`🧐 @uber-orchestrator`):**
    1.  (Conceptuel) Vérifier que l'agent `@clarification-agent` est défini dans `.roomodes`. Si manquant, émettre un avertissement critique dans le rapport final car il est essentiel pour la gestion des ambiguïtés.
*   **Memory Bank Interaction:**
    *   Écriture (Scribe): Initialisation de la structure de la Memory Bank si `.pheromone` est nouveau.
*   **Output (interne à l'UO):** `.pheromone` est prêt. Statut de la présence de `@clarification-agent`.

### Phase 2: Configuration du Contexte du Projet dans la Memory Bank
*   **Agent Responsable:** `@project-setup-agent`
*   **Inputs:** Nom du projet Azure DevOps et URL de l'organisation (fournis par l'utilisateur via l'UO). L'UO injecte le `techStack` par défaut depuis `memoryBank.projectContext` (si déjà partiellement rempli) ou les valeurs par défaut connues.
*   **Actions & Tooling:**
    1.  Collecter/Confirmer les informations projet : Nom projet ADO, URL org ADO, URL repo Git principal, stack technique (par défaut .NET/Angular/AzureSQL/Docker/AKS, mais peut être affiné ici si l'utilisateur spécifie des versions ou des composants particuliers).
    2.  Préparer un objet JSON pour `.pheromone.currentProject` et pour `.pheromone.memoryBank.projectContext`.
*   **Memory Bank Interaction:**
    *   Préparation des données.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Contexte du projet '[NomProjet]' collecté. Stack: [.NET, Angular, Azure SQL, Docker, AKS]. Azure DevOps Org: '[URLOrg]', Projet: '[NomProjet]', Repo: '[URLRepo]'. Prêt à mettre à jour `.pheromone`."

### Phase 3: Initialisation des Conventions de Codage et de Design
*   **Agent Responsable:** `@architecture-advisor-agent`
*   **Inputs:** Chemins vers les modèles (`02_AI-DOCS/Conventions/coding_conventions_template.md`, `02_AI-DOCS/Conventions/design_conventions_template.md`). L'UO injecte le contexte de la stack confirmée en Phase 2.
*   **Actions & Tooling:**
    1.  Vérifier si `coding_conventions.md` et `design_conventions.md` existent dans `02_AI-DOCS/Conventions/`.
    2.  Si non, copier les modèles vers ces emplacements.
    3.  Effectuer des ajustements initiaux minimes dans les copies pour refléter la stack (.NET/Angular), et ajouter des placeholders pour les décisions de design spécifiques au projet.
*   **Memory Bank Interaction:**
    *   Les chemins seront ajoutés à `documentationRegistry` par le Scribe.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Fichiers de conventions `coding_conventions.md` et `design_conventions.md` initialisés dans `02_AI-DOCS/Conventions/`. Prêts pour personnalisation et versionnement (v1.0_initial)."

### Phase 4: Vérification de la Connexion Azure DevOps
*   **Agent Responsable:** `@devops-connector`
*   **Inputs:** Informations du projet depuis `.pheromone.currentProject` (après mise à jour par le Scribe suite à Phase 2).
*   **Actions & Tooling:**
    1.  Utiliser **Azure DevOps MCP**:
        *   `get_project_details {projectName: .pheromone.currentProject.name}`.
        *   `get_user_identity`.
    2.  Si l'une des commandes échoue, cela indique un problème de configuration MCP ou d'accès au projet ADO.
*   **Memory Bank Interaction:**
    *   Le Scribe mettra à jour `.pheromone.systemHealth.mcpStatus.azureDevOpsMCP`.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Connexion à Azure DevOps pour projet '[NomProjet]' [réussie/échouée]. Identité MCP: '[IdentitéMCP]'." Si échec, inclure le message d'erreur de l'ADO MCP.

### Phase 5: Rapport de Bootstrap et État de Préparation
*   **Agent Responsable:** `🧐 @uber-orchestrator` (pour la compilation), `✍️ @orchestrator-pheromone-scribe` (pour l'enregistrement final).
*   **Inputs:** Résumés des phases précédentes.
*   **Actions & Tooling (`🧐 @uber-orchestrator`):**
    1.  Compiler un résumé global. Noter explicitement le statut de la connexion ADO et la présence de `@clarification-agent`.
*   **Output (`🧐 @uber-orchestrator` vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Bootstrap AgilePheromind terminé pour projet '[NomProjet]'. `.pheromone` [initialisé/vérifié]. Contexte projet [OK]. Conventions [initialisées/existantes]. Connexion Azure DevOps [OK/Échouée]. Agent `@clarification-agent` [Présent/Manquant - ALERTE SI MANQUANT]. Système [Prêt pour opérations / Nécessite attention sur points X, Y]. Rapport: `system_bootstrap_report_[timestamp].md`."
*   **Actions & Tooling (`✍️ @orchestrator-pheromone-scribe`):**
    1.  Enregistrer le rapport (`system_bootstrap_report_[timestamp].md`) dans `documentationRegistry` (ex: `02_AI-DOCS/System_Admin/`).
    2.  Persister toutes les mises à jour de `.pheromone` des phases précédentes (contexte projet, versions conventions, statut MCP).
    3.  Mettre à jour `.pheromone.systemVersion` si nécessaire.
*   **Memory Bank Interaction:**
    *   Écriture (Scribe): Finalisation du contexte projet, enregistrement du rapport.
*   **Outcome:** AgilePheromind est configuré, son état initial est enregistré, et les points d'attention pour une opérabilité complète sont signalés.

---