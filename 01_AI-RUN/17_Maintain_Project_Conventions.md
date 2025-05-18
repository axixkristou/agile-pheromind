# Workflow: Maintenance des Conventions de Codage et de Design du Projet (17_Maintain_Project_Conventions.md)

**Objectif:** Assurer que les documents de conventions de codage (`coding_conventions.md`) et de design (`design_conventions.md`) du projet sont à jour. L'agent analyse les conventions actuelles, les feedbacks de l'équipe (via `memoryBank`), les bonnes pratiques (via `Context7 MCP`), propose des modifications justifiées (avec "chaîne de pensée"), et après validation humaine, applique et commite ces changements.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe), `@architecture-advisor-agent`, `@code-reviewer-assistant` (en tant que source de feedback sur les déviations), `@clarification-agent`.

**MCPs Utilisés:** Context7 MCP, Git Tools MCP, Sequential Thinking MCP.

## Pheromind Workflow Overview:

1.  **Initiation:** Par utilisateur (Tech Lead/Architecte: `"AgilePheromind màj conventions de codage .NET"`) ou déclenché par `@workflow-optimizer-agent`.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Collecte d'Informations et Identification des Besoins de MàJ (avec Injection de Contexte).**
        *   UO **injecte contexte** à `@architecture-advisor-agent`: conventions actuelles (via `documentationRegistry` & **Git Tools MCP**), `memoryBank` (rétrospectives, dette technique liée aux conventions, décisions architecturales).
        *   `@architecture-advisor-agent` recherche bonnes pratiques à jour (via **Context7 MCP**).
        *   **onError:** Si les fichiers de conventions sont introuvables ou Context7 MCP échoue, le signaler.
    *   **Phase 2: Proposition Structurée des Modifications aux Conventions (avec "Chaîne de Pensée").**
        *   UO délègue à `@architecture-advisor-agent`. L'agent utilise **Sequential Thinking MCP** pour structurer ses propositions. **Doit documenter la "chaîne de pensée"** justifiant chaque changement majeur.
        *   Si des informations manquent pour justifier un changement (ex: impact d'une nouvelle règle pas clair), l'agent le signale à l'UO pour clarification potentielle avec l'équipe via `@clarification-agent`.
    *   **Phase 3: Validation Humaine des Modifications Proposées.**
        *   UO présente les propositions (avec justifications) au Tech Lead/Architecte via `ask_followup_question`.
    *   **Phase 4: Application des Modifications et Versionnement.**
        *   Si validé, UO délègue à `@architecture-advisor-agent` pour màj des fichiers .md et commit via **Git Tools MCP**.
        *   **onError (Git Commit):** Si échec, logguer, notifier.
    *   **Phase 5: Mise à Jour de `.pheromone` et Communication.**
        *   Scribe met à jour versions des conventions dans `memoryBank.projectContext` et `documentationRegistry`.

## Détails des Phases:

### Phase 1: Collecte d'Informations et Identification des Besoins de MàJ (avec Injection de Contexte)
*   **Agent Responsable:** `@architecture-advisor-agent`.
*   **Inputs (Injectés par l'UO):**
    *   Type de convention à réviser.
    *   Contenu des fichiers `coding_conventions.md` / `design_conventions.md` actuels (lus via **Git Tools MCP** `get_file_contents` à partir des chemins dans `documentationRegistry`).
    *   Extraits pertinents de `memoryBank`: `sprintRetrospectivesSummaries` (points sur conventions), `technicalDebtItems` (déviations aux conventions), `architecturalDecisions` (impactant conventions), `commonIssuesAndSolutions` (si liés à des manques dans les conventions).
*   **Actions & Tooling (`@architecture-advisor-agent`):**
    1.  Analyser les conventions actuelles et les inputs de la `memoryBank`.
    2.  Rechercher via **Context7 MCP** (`get_library_docs`): guides de style officiels .NET/C#, Angular/TypeScript, bonnes pratiques Tailwind/librairie UI, tendances design systems.
    3.  Identifier sections obsolètes, manquantes, ou nécessitant clarification/amélioration.
*   **onError (Git Tools / Context7 MCP):**
    *   Si fichiers de conventions non trouvés ou MCP Context7 inaccessible/ne retourne rien d'utile :
        *   `@architecture-advisor-agent` signale à l'UO: "Impossible de [lire conventions actuelles / récupérer bonnes pratiques à jour pour X]. Erreur: [Détail]. Analyse limitée."
        *   L'UO peut décider de continuer avec une analyse limitée ou d'arrêter et demander une intervention.
*   **Output (interne):** Liste des points de convention à mettre à jour, avec justification et références.

### Phase 2: Proposition Structurée des Modifications aux Conventions (avec "Chaîne de Pensée")
*   **Agent Responsable:** `@architecture-advisor-agent`, UO, `@clarification-agent`.
*   **Inputs:** Analyse Phase 1. Fichiers de conventions actuels.
*   **Actions (`@architecture-advisor-agent`):**
    1.  Utiliser **Sequential Thinking MCP** pour chaque section majeure à modifier:
        *   `set_goal`: "Proposer une mise à jour pour la section '[NomSection]' des conventions [Codage/Design]."
        *   `add_step`: "État actuel de la convention: [Extrait convention actuelle]."
        *   `add_step`: "Problème identifié / Opportunité d'amélioration: [Basé sur Phase 1]."
        *   `add_step`: "Bonne pratique/Référence (Context7/Autre): [Référence]."
        *   `add_step`: "Proposition de nouvelle convention / modification: [Nouveau texte]."
        *   `add_step`: "Justification / Chaîne de Pensée: [Expliquer pourquoi ce changement est bénéfique, comment il résout le problème, son alignement avec les bonnes pratiques]."
        *   `run_sequence`.
    2.  Compiler toutes les propositions (avec leur "chaîne de pensée") dans un document de "changements proposés" (diff ou version annotée).
    3.  **Gestion d'Ambiguïté/Information Manquante:** Si pour une proposition, l'impact sur un aspect du projet n'est pas clair ou si un consensus d'équipe serait préférable (ex: choix entre deux styles de nommage valides):
        *   L'agent formule le point et la question. Ex: "Pour la convention de nommage des services Angular, deux options sont viables: `XService` ou `XDataService`. Quel est le consensus de l'équipe ou la préférence du Tech Lead ?".
        *   Signaler à l'UO pour potentielle clarification via `@clarification-agent` avant de finaliser cette proposition spécifique.
*   **Output (vers UO):** Document des modifications proposées (avec justifications/"chaînes de pensée").

### Phase 3: Validation Humaine des Modifications Proposées
*   **Agent Responsable:** `🧐 @uber-orchestrator`.
*   **Inputs:** Propositions de `@architecture-advisor-agent`.
*   **Actions (UO):**
    1.  `ask_followup_question` au Tech Lead/Architecte: "Mises à jour proposées pour conventions [Codage/Design] (avec justifications et chaîne de pensée pour chaque point majeur). Rapport des changements: `[lien_vers_doc_changements_proposés]`. Approuver ? (approuver / rejeter / demander modifs)".
    2.  Gérer la réponse (itération avec `@architecture-advisor-agent` si "demander modifs", arrêt si "rejeter").
*   **Output (vers `@architecture-advisor-agent` si approuvé):** Confirmation.

### Phase 4: Application des Modifications et Versionnement
*   **Agent Responsable:** `@architecture-advisor-agent`.
*   **Inputs:** Confirmation d'approbation. Propositions validées.
*   **Actions:**
    1.  Appliquer modifs aux fichiers `.md` dans `02_AI-DOCS/Conventions/`.
    2.  Mettre à jour version et date dans les documents.
    3.  **Git Tools MCP**: `add_files`, `commit_files {message: "docs(conventions): update [coding/design] conventions v[NewVersion]\n\n[Résumé des changements clés]"}`, (optionnel) `push_commits`.
*   **onError (Git Commit):** Si échec, `@architecture-advisor-agent` signale à l'UO. UO loggue via Scribe, notifie utilisateur.
*   **Output (vers Scribe):** Résumé NL: "Conventions [Codage/Design] màj (v[NewVersion]). Commit: `{{commitHash}}`. Changements: [Liste]."

### Phase 5: Mise à Jour de `.pheromone` et Communication
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe`.
*   **Inputs:** Résumé NL de `@architecture-advisor-agent`.
*   **Actions:**
    1.  Mettre à jour `.pheromone`:
        *   `memoryBank.projectContext.codingConventionsVersion` / `designConventionsVersion`.
        *   `documentationRegistry`: Vérifier `lastModified` pour les fichiers de conventions.
        *   `memoryBank.architecturalDecisions` ou `conventionUpdateHistory`: Enregistrer la mise à jour (commit, résumé, lien vers diff si possible, et lien vers la "chaîne de pensée" si le rapport des propositions est stocké).
    2.  (Optionnel) UO notifie l'équipe.
*   **Output:** `.pheromone` à jour. Conventions du projet maintenues.

---