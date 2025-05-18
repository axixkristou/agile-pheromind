# Workflow: Traitement de la Réponse à une Demande de Clarification (XX_Handle_Clarification_Response.md)

**Objectif:** Traiter la réponse fournie par un utilisateur suite à une question posée par `@clarification-agent`. Ce workflow met à jour la `memoryBank` avec la réponse, nettoie le contexte de clarification en attente, et réactive le workflow original qui était en pause.

**Agents IA Clés:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe).

**MCPs Utilisés:** Aucun MCP direct dans ce workflow spécifique, mais il interagit avec l'état défini par d'autres workflows utilisant des MCPs.

## Pheromind Workflow Overview:

1.  **Initiation:** Ce workflow est déclenché par le système Pheromind (probablement par l'interface utilisateur ou un gestionnaire d'événements) lorsque l'utilisateur fournit une réponse à une question en attente (identifiée par `clarificationContext.pendingClarificationId`). L'UO reçoit l'ID de clarification et la réponse de l'utilisateur.
2.  **`🧐 @uber-orchestrator`** prend le contrôle.
    *   **Phase 1: Récupération du Contexte de Clarification.**
        *   UO lit `.pheromone.clarificationContext` et `memoryBank.clarificationHistory` (pour s'assurer que l'ID est valide et récupérer le contexte original).
    *   **Phase 2: Enregistrement de la Réponse Utilisateur.**
        *   UO délègue au Scribe pour mettre à jour `memoryBank.clarificationHistory` avec la réponse et nettoyer `clarificationContext.pendingClarificationId`.
    *   **Phase 3: Reprise du Workflow Original.**
        *   UO identifie le workflow original qui était en pause (information potentiellement stockée avec la demande de clarification, ou l'UO doit avoir un mécanisme pour savoir quel workflow reprendre avec cette nouvelle information).
        *   UO réactive le workflow original, en lui fournissant la réponse de clarification comme nouveau contexte.

## Détails des Phases:

### Phase 1: Récupération du Contexte de Clarification
*   **Agent Responsable:** `🧐 @uber-orchestrator`.
*   **Inputs:** `clarificationId` et `userResponse` (fournis par le mécanisme d'interface utilisateur qui a collecté la réponse).
*   **Actions & Tooling (UO):**
    1.  Lire `.pheromone.clarificationContext`.
    2.  Valider que `clarificationContext.pendingClarificationId` correspond au `clarificationId` reçu.
        *   **onError:** Si les IDs ne correspondent pas ou si `pendingClarificationId` est nul, c'est une situation anormale. L'UO loggue une erreur via le Scribe ("Réponse de clarification reçue pour un ID inconnu ou non en attente: {{clarificationId}}") et arrête ce workflow.
    3.  Récupérer `clarificationContext.originalAgent`, `clarificationContext.originalPromptContext`, `clarificationContext.clarificationQuestion`, `clarificationContext.timestampRequested`.
*   **Memory Bank Interaction:**
    *   Lecture: `.pheromone.clarificationContext`.
*   **Output (interne à l'UO):** Contexte de la demande de clarification initiale et la réponse de l'utilisateur.

### Phase 2: Enregistrement de la Réponse Utilisateur
*   **Agent Responsable:** `✍️ @orchestrator-pheromone-scribe` (via une instruction de l'UO).
*   **Inputs:** `clarificationId`, `userResponse`, et le contexte récupéré en Phase 1.
*   **Actions & Tooling (UO prépare le résumé pour le Scribe):**
    1.  L'UO formule un résumé pour le Scribe.
*   **Output (UO vers Scribe):** Résumé NL: "Réponse utilisateur reçue pour clarification ID '{{clarificationId}}'. Question: '{{clarificationContext.clarificationQuestion}}'. Réponse: '{{userResponse}}'. Contexte original: Agent '{{clarificationContext.originalAgent}}', Prompt: '{{clarificationContext.originalPromptContext}}'. Mettre à jour l'historique et nettoyer le contexte en attente."
*   **Actions & Tooling (Scribe):**
    1.  Interpréter le résumé via `.swarmConfig`.
    2.  Mettre à jour `.pheromone`:
        *   Rechercher dans `memoryBank.clarificationHistory` l'entrée avec le `clarificationId` correspondant (ou créer une nouvelle entrée si la demande initiale n'a pas été loggée de manière complète, bien qu'elle devrait l'être).
        *   Ajouter/Mettre à jour `userResponse` et `timestampResponded` pour cette entrée.
        *   Mettre à jour `impactOnWorkflow` (ex: "Réponse fournie, en attente de reprise du workflow original.").
        *   Réinitialiser `.pheromone.clarificationContext`:
            *   `pendingClarificationId: null`
            *   `originalAgent: null`
            *   `originalPromptContext: null`
            *   `clarificationQuestion: null`
            *   `timestampRequested: null`
*   **Memory Bank Interaction:**
    *   Écriture: Mise à jour de `memoryBank.clarificationHistory`. Réinitialisation de `clarificationContext`.
*   **Output:** `.pheromone` mis à jour.

### Phase 3: Reprise du Workflow Original
*   **Agent Responsable:** `🧐 @uber-orchestrator`.
*   **Inputs:** Contexte de la clarification initiale (surtout `originalAgent` et `originalPromptContext`). `userResponse`. État actuel de `.pheromone`.
*   **Actions & Tooling (UO):**
    1.  **Identifier le Workflow à Reprendre:**
        *   Le plus simple est si le script `01_AI-RUN/*.md` original (celui qui a été mis en pause) est conçu pour vérifier `clarificationContext` au début de ses phases critiques.
        *   Alternativement, l'UO pourrait avoir stocké une référence au script et à la phase en pause (ex: dans `activeWorkflow.pausedContext = { scriptPath, phase, originalParams }`).
    2.  **Préparer le Nouveau Contexte pour le Workflow Original:**
        *   Le `userResponse` est la nouvelle information clé.
    3.  **Réactiver le Workflow Original:**
        *   Si le workflow original est toujours "actif" mais en attente (ex: l'UO attendait une boucle), l'UO peut maintenant continuer son script `01_AI-RUN/*.md`, en passant la `userResponse` comme contexte supplémentaire à l'agent qui était bloqué (`clarificationContext.originalAgent`).
        *   Si le workflow original s'était terminé en attendant, l'UO pourrait avoir besoin de relancer une version modifiée de ce workflow ou une partie spécifique, en commençant par la phase qui nécessitait la clarification.
        *   **Exemple de reprise:** Si `@po-assistant` attendait une clarification pour `01_AI-RUN/03_PO_Analyze_Need.md` Phase 2, l'UO pourrait maintenant redéléguer la Phase 2 à `@po-assistant` en lui fournissant le besoin client original ET la `userResponse` à la question d'ambiguïté.
    4.  Mettre à jour `.pheromone.activeWorkflow.status` de "PendingClarification_..." à "InProgress" (ou le statut approprié).
*   **Memory Bank Interaction:**
    *   Lecture: Pour récupérer le contexte du workflow en pause si nécessaire.
*   **Output:** Le workflow original est relancé avec les informations clarifiées. AgilePheromind continue son opération.

---
**Note Importante sur la Reprise du Workflow Original (Phase 3):**

L'implémentation exacte de la reprise dépendra de la manière dont l'UO gère les états de "pause" et de "reprise". Une approche robuste pourrait être :

*   Lorsqu'une clarification est demandée, l'UO enregistre dans `.pheromone.activeWorkflow` des informations sur le script et la phase en cours, ainsi que les paramètres initiaux.
*   Après que `XX_Handle_Clarification_Response.md` a mis à jour la `memoryBank` avec la réponse, le `🎩 @head-orchestrator` réactive l'UO.
*   L'UO, en voyant qu'il n'y a plus de `pendingClarificationId` mais qu'il y a un `activeWorkflow.pausedContext` et une réponse fraîche dans `memoryBank.clarificationHistory`, sait qu'il doit reprendre ce workflow. Il relance alors le script `pausedContext.scriptPath` à la phase `pausedContext.phase` avec les `pausedContext.originalParams` PLUS la `userResponse` comme nouveau contexte.

Cela nécessite que les scripts `01_AI-RUN/*.md` soient conçus pour pouvoir potentiellement démarrer ou reprendre à des phases spécifiques avec un contexte enrichi.

---