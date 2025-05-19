# Workflow: Handle User Response to Clarification Request (XX_Handle_Clarification_Response.md)

**Objective:** Process a user's response (provided in `currentUser.lastInteractionLanguage`) to a clarification question previously posed by `@clarification-agent`. This workflow updates the `memoryBank` (English) with the clarification details, clears the pending clarification context, and then attempts to resume the original workflow that was paused, providing it with the new (conceptually translated to English if necessary) information.

**Key AI Agents:** `🧐 @uber-orchestrator` (UO), `✍️ @orchestrator-pheromone-scribe` (Scribe).

**MCPs Used:** None directly in this specific workflow, but it enables other workflows that use MCPs to resume.

## Pheromind Workflow Overview:

1.  **Initiation:** This workflow is triggered internally by the Pheromind system when a user provides a response to a question identified by `clarificationContext.pendingClarificationId`. The UO receives the `clarificationId` and the `userResponseInOriginalLang`. `currentUser.lastInteractionLanguage` is assumed to be the language of the response.
2.  **`🧐 @uber-orchestrator`** (UO) takes control.
    *   **Phase 1: Retrieve Clarification Context.**
        *   UO reads `.pheromone.clarificationContext` and `memoryBank.clarificationHistory` to validate `clarificationId` and retrieve original context (original agent, English prompt context, English question).
        *   **onError:** If ID invalid or context missing, log critical English error, notify admin, stop.
    *   **Phase 2: Record User Response in MemoryBank (English Context).**
        *   UO tasks Scribe to update `memoryBank.clarificationHistory` with `userResponseInOriginalLang` and other details. Scribe clears `.pheromone.clarificationContext.pendingClarificationId`.
        *   The UO (or Scribe, conceptually) ensures an English version or summary of the user's response (`userResponse_en`) is available for internal use by other agents if the original response was not in English.
    *   **Phase 3: Resume Original Paused Workflow.**
        *   UO identifies the original workflow and phase that was paused (using info from `clarificationContext` or `activeWorkflow.pausedContext`).
        *   UO re-activates the original workflow/agent, injecting the now-clarified information (the English version/summary of the user's response, `userResponse_en`).

## Phase Details:

### Phase 1: Retrieve Clarification Context
*   **Responsible Agent:** `🧐 @uber-orchestrator`.
*   **Inputs:** `clarificationId` and `userResponseInOriginalLang` (from user interaction mechanism).
*   **Actions & Tooling (UO):**
    1.  Read `.pheromone.clarificationContext`.
    2.  Validate `clarificationContext.pendingClarificationId == clarificationId`.
    3.  **onError:** If ID mismatch or `pendingClarificationId` is null: Scribe logs critical English error ("Clarification response received for unknown/non-pending ID: {{clarificationId}}"). UO stops this workflow. This indicates a system logic issue.
    4.  Retrieve `originalAgent`, `originalPromptContext_en`, `clarificationQuestion_en`, `timestampRequested` from the (now cleared) `clarificationContext` or from the corresponding entry in `memoryBank.clarificationHistory`.
*   **Output (internal to UO, English context):** Original clarification request details and the user's response (in original language).

### Phase 2: Record User Response in MemoryBank (English Context)
*   **Responsible Agent:** `✍️ @orchestrator-pheromone-scribe` (instructed by UO).
*   **Inputs:** `clarificationId`, `userResponseInOriginalLang`, and original context (Phase 1).
*   **Actions & Tooling (UO prepares summary for Scribe):**
    1.  UO formulates English NL summary for Scribe. This summary must include the `userResponseInOriginalLang` and also an English translation/summary of it (`userResponse_en`) if the original response was not English (UO's LLM capability handles this conceptual translation).
*   **Output (UO to Scribe, English NL Summary):** "User response for clarification ID '{{clarificationId}}' received. Original English question: '{{originalQuestion_en}}'. User response (original lang): '{{userResponseInOriginalLang}}'. English interpretation/summary of response: '{{userResponse_en}}'. Update history and clear pending context."
*   **Actions & Tooling (Scribe, guided by `.swarmConfig` rule `process_clarification_response_from_user_v2_en`):**
    1.  Interpret English summary.
    2.  Update `.pheromone`:
        *   Find entry in `memoryBank.clarificationHistory` with matching `clarificationId`.
        *   Update it with `userResponse_originalLang`, `userResponse_en`, and `timestampResponded`. Update `impactOnWorkflow_en`.
        *   Reset `.pheromone.clarificationContext` fields to `null`.
*   **Output:** `.pheromone` updated. `userResponse_en` is now available for the original workflow.

### Phase 3: Resume Original Paused Workflow
*   **Responsible Agent:** `🧐 @uber-orchestrator`.
*   **Inputs:** Original clarification context (especially `originalAgent` and `originalPromptContext_en`). `userResponse_en` (from Scribe-updated `.pheromone` or UO's internal state). Current `.pheromone` state.
*   **Actions & Tooling (UO):**
    1.  **Identify Workflow and Phase to Resume:**
        *   Use information previously stored when the clarification was initiated (e.g., from `activeWorkflow.pausedContext = { scriptPath, phaseName, originalAgent, originalTaskParams }` if UO saved it, or from `clarificationContext.originalAgent` and `clarificationContext.originalPromptContext_en` to deduce where to re-enter).
    2.  **Prepare New Context for Resumed Workflow:** The key new information is `userResponse_en`.
    3.  **Re-activate Original Workflow/Agent:**
        *   The UO will typically re-delegate the task to the `clarificationContext.originalAgent` for the phase that was paused.
        *   The UO **injects the `userResponse_en`** along with the original task parameters and any other relevant context.
        *   The `customInstructions` for the `originalAgent` should be designed to handle this newly provided clarification.
        *   Example: If `@po-assistant` was paused during `01_AI-RUN/03_PO_Analyze_Need.md` Phase 1 due to an ambiguous need, the UO now re-delegates Phase 1 (or a sub-part) to `@po-assistant` with the original need AND `userResponse_en`.
    4.  Update `.pheromone.activeWorkflow.status` from "PendingClarification_..." to "InProgress" (or appropriate).
*   **Output:** The original workflow is resumed with the clarified English information. AgilePheromind continues its operation.

---
**Important Considerations for Resuming Workflows:**

*   The UO needs a robust mechanism to "remember" which workflow and specific phase was paused awaiting clarification. Storing this in `activeWorkflow.pausedContext` (including the original script path and parameters) is a good approach.
*   Specialized agents need to be designed in their `.roomodes` `customInstructions` to accept and utilize clarified information when a task is re-delegated to them after a clarification cycle.
*   If the user's response to a clarification is "I don't know" or doesn't resolve the ambiguity, the UO needs a strategy (defined in the original `01_AI-RUN/*.md` script's `onError` section or by escalating to a human Tech Lead).

---