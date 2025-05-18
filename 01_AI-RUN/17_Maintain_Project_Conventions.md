# Workflow: Maintenance des Conventions de Codage et de Design du Projet (17_Maintain_Project_Conventions.md)

**Objectif:** Assurer que les documents de conventions de codage (`coding_conventions.md`) et de design (`design_conventions.md`) du projet sont à jour, reflètent les meilleures pratiques actuelles, les décisions de l'équipe, et les standards de la stack technologique (.NET, Angular). Ce workflow peut être déclenché suite à des rétrospectives, des changements technologiques majeurs, ou sur demande du Tech Lead/Architecte.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@architecture-advisor-agent`, `@code-reviewer-assistant` (pour identifier les déviations récurrentes).

**MCPs Utilisés:** Context7 MCP (pour rechercher les dernières bonnes pratiques et conventions pour .NET/Angular), Git Tools MCP (pour commiter les changements aux fichiers de conventions).

## Pheromind Workflow Overview:

1.  **Initiation:**
    *   L'utilisateur (Tech Lead/Architecte) demande une revue et mise à jour des conventions (ex: `"AgilePheromind màj conventions de codage .NET"` ou `"AgilePheromind révise design_conventions.md pour intégrer les nouveaux tokens de couleur"`).
    *   Peut être déclenché par `@workflow-optimizer-agent` si des problèmes récurrents liés aux conventions sont détectés.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Collecte des Informations et Identification des Besoins de Mise à Jour.**
        *   UO délègue à `@architecture-advisor-agent`. L'agent lit les fichiers de conventions actuels, les suggestions de la `memoryBank` (ex: issues de rétrospectives, rapports de `@code-reviewer-assistant`), et recherche les bonnes pratiques à jour.
    *   **Phase 2: Proposition des Modifications aux Conventions.**
        *   UO délègue à `@architecture-advisor-agent`. L'agent rédige les propositions de changement.
    *   **Phase 3: Validation Humaine des Modifications (Cruciale).**
        *   UO présente les propositions au Tech Lead/Architecte via `ask_followup_question` pour validation.
    *   **Phase 4: Application des Modifications et Versionnement.**
        *   Si validé, UO délègue à `@architecture-advisor-agent` pour mettre à jour les fichiers .md.
        *   L'agent utilise **Git Tools MCP** pour commiter les changements.
    *   **Phase 5: Mise à Jour de `.pheromone` et Communication.**
        *   Scribe met à jour la version des conventions dans `memoryBank.projectContext` et `documentationRegistry`.

## Détails des Phases:

### Phase 1: Collecte des Informations et Identification des Besoins de Mise à Jour
*   **Agent Responsable:** `@architecture-advisor-agent`
*   **Inputs:** Type de convention à réviser (codage, design, ou les deux). Accès à `.pheromone`.
*   **Actions & Tooling:**
    1.  **Lire les Fichiers de Conventions Actuels:**
        *   Récupérer les chemins de `coding_conventions.md` et `design_conventions.md` depuis `.pheromone.documentationRegistry`.
        *   Utiliser **Git Tools MCP** (`get_file_contents`) pour lire leur contenu.
    2.  **Analyser la `MemoryBank` pour des Indicateurs:**
        *   Consulter `memoryBank.sprintRetrospectivesSummaries` pour des points liés aux conventions.
        *   Consulter `memoryBank.technicalDebtItems` ou les rapports de `@code-reviewer-assistant` (via `documentationRegistry`) pour des déviations fréquentes aux conventions actuelles ou des "code smells" qui pourraient être adressés par de nouvelles conventions.
        *   Consulter `memoryBank.architecturalDecisions` pour des décisions récentes impactant les conventions.
    3.  **Rechercher les Bonnes Pratiques Actuelles:**
        *   Utiliser **Context7 MCP** (`get_library_docs`) pour :
            *   Les guides de style officiels .NET (Microsoft) et C#.
            *   Les guides de style officiels Angular et TypeScript.
            *   Les meilleures pratiques pour Tailwind CSS, ou la librairie UI (ex: Angular Material) utilisée.
            *   Les tendances en design systems (pour `design_conventions.md`).
    4.  Identifier les sections des conventions actuelles qui sont obsolètes, manquantes, ou qui nécessitent des clarifications/améliorations.
*   **Memory Bank Interaction:**
    *   Lecture: `documentationRegistry`, `memoryBank.sprintRetrospectivesSummaries`, `memoryBank.technicalDebtItems`, `memoryBank.architecturalDecisions`.
*   **Output (interne à `@architecture-advisor-agent`):** Liste des points et sections des conventions nécessitant une mise à jour, avec justification et références aux bonnes pratiques.

### Phase 2: Proposition des Modifications aux Conventions
*   **Agent Responsable:** `@architecture-advisor-agent`
*   **Inputs:** Analyse de la Phase 1. Fichiers de conventions actuels.
*   **Actions & Tooling:**
    1.  Pour chaque point identifié, rédiger une proposition de modification claire et concise.
    2.  **Pour `coding_conventions.md`:**
        *   Suggérer de nouvelles règles de nommage, de formatage, d'utilisation des fonctionnalités de langage (.NET Linq, C# async/await, TypeScript types/interfaces, décorateurs Angular).
        *   Proposer des patrons de conception recommandés ou à éviter.
        *   Clarifier les règles pour la gestion des erreurs, le logging, les commentaires.
        *   Mettre à jour les configurations recommandées pour les linters (ESLint, StyleCop).
    3.  **Pour `design_conventions.md`:**
        *   Suggérer des mises à jour pour la palette de couleurs, la typographie, les grilles de mise en page, l'espacement.
        *   Proposer de nouveaux composants UI standards ou des variations de composants existants.
        *   Affiner les principes d'interaction design, d'accessibilité (A11Y).
        *   Mettre à jour les directives pour l'utilisation de Tailwind CSS ou de la librairie UI.
    4.  Préparer un document de "changements proposés" (diff ou version annotée des fichiers .md) pour faciliter la revue humaine.
*   **Memory Bank Interaction:**
    *   Aucune écriture directe.
*   **Output (vers `🧐 @uber-orchestrator`):** Un document ou un texte clair présentant les modifications proposées pour `coding_conventions.md` et/ou `design_conventions.md`, avec justifications.

### Phase 3: Validation Humaine des Modifications (Cruciale)
*   **Agent Responsable:** `🧐 @uber-orchestrator`
*   **Inputs:** Propositions de modifications de `@architecture-advisor-agent`.
*   **Actions & Tooling:**
    1.  Utiliser `ask_followup_question` pour présenter les changements proposés au Tech Lead/Architecte (ou à l'équipe désignée) :
        *   "L'agent `@architecture-advisor-agent` a analysé les conventions du projet et propose les mises à jour suivantes pour [coding_conventions.md / design_conventions.md]:\n\n[Résumé des propositions majeures OU lien vers le document des changements proposés]\n\nVoulez-vous approuver ces changements ? (approuver / rejeter / demander modifications)"
    2.  Si "demander modifications", transmettre le feedback à `@architecture-advisor-agent` pour une nouvelle itération (retour à Phase 2).
    3.  Si "rejeter", le workflow s'arrête pour ces propositions. Le Scribe peut enregistrer la décision.
    4.  Si "approuver", passer à la Phase 4.
*   **Memory Bank Interaction (via Scribe si rejet ou modifications demandées):**
    *   Enregistrer la décision dans `memoryBank.architecturalDecisions` ou une section `conventionUpdateHistory`.
*   **Output (vers `@architecture-advisor-agent` si approuvé):** Confirmation d'approbation.

### Phase 4: Application des Modifications et Versionnement
*   **Agent Responsable:** `@architecture-advisor-agent`
*   **Inputs:** Confirmation d'approbation de l'UO. Propositions de modifications validées.
*   **Actions & Tooling:**
    1.  Appliquer les modifications approuvées aux fichiers `coding_conventions.md` et/LOU `design_conventions.md` dans le répertoire `02_AI-DOCS/Conventions/`.
    2.  Mettre à jour le numéro de version dans les documents (ex: `Version: 1.1`, `Date de mise à jour: {{timestamp}}`).
    3.  Utiliser **Git Tools MCP**:
        *   `add_files {filePaths: ["02_AI-DOCS/Conventions/coding_conventions.md", "02_AI-DOCS/Conventions/design_conventions.md"]}` (selon les fichiers modifiés).
        *   `commit_files {message: "docs(conventions): update coding and design conventions v1.1\n\n[Résumé des changements clés approuvés]"}`.
        *   (Optionnel, selon workflow Git) `push_commits`.
*   **Memory Bank Interaction (via Scribe):**
    *   Le Scribe enregistrera le nouveau hash de commit et mettra à jour les versions des conventions.
*   **Output (vers `✍️ @orchestrator-pheromone-scribe`):** Résumé NL: "Conventions de projet mises à jour. `coding_conventions.md` (vX.Y) et/ou `design_conventions.md` (vZ.A) modifiés et commité (Commit: `{{commitHash}}`). Principaux changements: [Liste]."

### Phase 5: Mise à Jour de `.pheromone` et Communication
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`
*   **Inputs:** Résumé NL de `@architecture-advisor-agent`.
*   **Actions & Tooling:**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   `memoryBank.projectContext.codingConventionsVersion`: Mettre à jour avec la nouvelle version.
        *   `memoryBank.projectContext.designConventionsVersion`: Mettre à jour avec la nouvelle version.
        *   `documentationRegistry`: S'assurer que les entrées pour `coding_conventions.md` et `design_conventions.md` sont correctes et que leur `lastModified` timestamp est à jour.
        *   `memoryBank.architecturalDecisions` ou `conventionUpdateHistory`: Enregistrer un item pour cette mise à jour, avec lien vers le commit et résumé des changements.
    3.  (Optionnel) L'UO peut être instruit de notifier l'équipe des mises à jour des conventions via un canal approprié.
*   **Memory Bank Interaction:**
    *   Écriture: Mise à jour des versions des conventions et de l'historique des décisions.
*   **Output:** `.pheromone` mis à jour. L'équipe est informée (ou peut consulter) des nouvelles conventions.

---