# AgilePheromind: AI-Powered Agile Development Assistant

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/badge/build-alpha-orange.svg)]()
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)]()

**Transform your Agile development lifecycle with AgilePheromind, an intelligent AI swarm designed to assist .NET/Angular teams using Azure DevOps.**

AgilePheromind is an advanced AI-powered multi-agent system built to enhance and streamline Agile software development processes. It leverages a sophisticated architecture of specialized AI agents that collaborate to automate, assist, and optimize tasks from requirements analysis through to deployment and maintenance.

**Inspired by the innovative [Pheromind Framework by Chris Royse](https://github.com/ChrisRoyse/Pheromind)**, AgilePheromind adapts its principles of swarm intelligence, decentralized state management, and workflow orchestration to specifically address the needs of development teams working within the .NET, Angular, and Azure DevOps ecosystems.

## 🌟 Overview: What is AgilePheromind?

AgilePheromind acts as an intelligent co-pilot for your Agile team. It's not just a collection of scripts, but a dynamic system where AI agents, each with a specific role (like PO Assistant, Developer Agent, Code Reviewer, Migration Analyst), work together. They interact by reading and modifying a shared environment and memory (`.pheromone` file), enabling complex, context-aware assistance.

**Key Goals:**
*   **Boost Productivity:** Automate repetitive tasks, accelerate analysis, and provide rapid contextual support.
*   **Enhance Quality:** Integrate automated code analysis, test generation, and adherence to project conventions.
*   **Improve Collaboration:** Centralize project knowledge and support Agile ceremonies.
*   **Facilitate Continuous Learning:** Build a persistent `memoryBank` of project decisions, solutions, and learnings.
*   **Seamless Tool Integration:** Natively interact with Azure DevOps, Git, MSSQL, and other essential development tools via Model Context Protocol (MCP) servers.

## 🤔 Why Use AgilePheromind?

This framework is built to solve key challenges in modern Agile development:

*   **Contextual AI Assistance:** Agents understand your project's specifics (tech stack, conventions, history) stored in the `memoryBank`.
*   **Full Lifecycle Support:** From analyzing Product Owner needs to assisting with PR reviews and legacy code migration.
*   **User-Centric Interaction:** Communicate with AgilePheromind in your preferred language; it handles internal processing and data storage consistently in English.
*   **Robust Onboarding:** A guided setup process ensures AgilePheromind is correctly configured for your existing Azure DevOps project and local Git repository.
*   **Resilient Workflows:** Built-in error handling and clarification loops make the system more robust.
*   **Adaptable & Evolutive:** Designed to be customized and to learn from project interactions.

## ✨ Core Features

*   🧠 **Pheromone-Based Swarm Intelligence:** Decentralized agent coordination via a shared state file (`.pheromone`) managed by the `✍️ @orchestrator-pheromone-scribe`.
*   🎯 **Workflow-Driven Execution:** Project operations are defined by clear, modifiable scripts in the `01_AI-RUN/` directory.
*   🛠️ **Specialized AI Agents (`.roomodes`):** A team of virtual experts (PO Assistant, .NET/Angular Developer, Test Generator, Code Reviewer, Migration Analyst, Risk Manager, etc.).
*   🔗 **Deep MCP Integration:** Native interaction with Azure DevOps (Work Items, PRs, Pipelines), Git, Context7 (documentation), MSSQL, Browser Tools, and more.
*   🌍 **Multilingual User Interface:** Interact in your language; the system manages internal data in English.
*   📚 **Rich `memoryBank`:** Stores project history, decisions, conventions, analyses, risks, and learnings.
*   ⚙️ **Automated Onboarding:** Configures itself for your existing Azure DevOps project and local Git setup.
*   💡 **"Chain of Thought" Logging:** Analytical agents document their reasoning for transparency and learning.
*   🛡️ **Integrated Error Handling & Clarification Loops:** For more robust and user-friendly operation.

## ⚙️ How It Works: The AgilePheromind Flow

1.  **User Onboarding (`01_AI-RUN/00_System_Bootstrap.md`):**
    *   On first interaction (or if critical info is missing), AgilePheromind guides you to provide your Azure DevOps identity, preferred language, and details about your existing ADO project and local Git repository. This information is stored locally in `.agilepherominduserinfo` (gitignored) and mirrored in `.pheromone`.
2.  **User Command:** You issue a command in your language to the `🎩 @head-orchestrator` (e.g., "AgilePheromind start US Azure#12323").
3.  **Workflow Initiation:** The `🎩 @head-orchestrator` detects your language, selects the appropriate `01_AI-RUN/*.md` script, and tasks the `🧐 @uber-orchestrator` (UO).
4.  **Orchestration & Context:** The UO reads the script and `.pheromone` (for English project state and `memoryBank`). It injects targeted English context into specialized agents.
5.  **Agent Execution:** Specialized agents (e.g., `@po-assistant`, `@developer-agent`) perform their tasks (analysis, code generation, MCP calls), operating with English inputs and producing English internal results/summaries.
6.  **User Interaction & Document Localization:**
    *   If an agent needs clarification, the UO tasks `@clarification-agent` to ask you a question (translated to your language).
    *   Agents producing final documents (reports, specs) will draft them in English and then translate them into your language before saving.
7.  **State Update:** Agents send English Natural Language (NL) summaries to the `✍️ @orchestrator-pheromone-scribe`. The Scribe uses `.swarmConfig` to interpret these and update `.pheromone` (English data for `memoryBank`, paths to localized documents in `documentationRegistry`).
8.  **Cycle Continuation:** The UO proceeds to the next phase or concludes the workflow.

For a conceptual overview with diagrams, see [`ARCHITECTURE.md`](./ARCHITECTURE.md) (assuming you'll create this based on my previous output).

## 🚀 Getting Started

1.  **Clone this Repository:** This provides the core AgilePheromind framework files.
2.  **Prerequisites:**
    *   Ensure you have a compatible AI agent environment (e.g., Roo Code, or another LLM interface capable of managing modes/agents and running scripts).
    *   Access to an LLM (e.g., GPT-4, Claude 3+).
    *   Azure DevOps project and a local Git clone of its repository.
    *   Necessary MCP servers installed and running (Azure DevOps MCP, Git Tools MCP, Context7 MCP, MSSQL MCP, etc.). Refer to their respective documentation for setup.
3.  **Initial Configuration:**
    *   Review and potentially customize `.roomodes` (agent definitions) and `.swarmConfig` (Scribe logic) for your specific needs, although the provided defaults are a good starting point.
    *   **There is no manual creation of `.agilepherominduserinfo` or `.pheromone` needed.**
4.  **First Interaction (Onboarding):**
    *   Issue your first command to AgilePheromind via your LLM interface, targeting the `🎩 @head-orchestrator`. For example:
        `@head-orchestrator Help me get started with AgilePheromind.`
    *   The `🧐 @uber-orchestrator` will detect that onboarding is needed and automatically trigger the `01_AI-RUN/00_System_Bootstrap.md` workflow.
    *   Follow the prompts to provide your Azure DevOps user details, preferred language, ADO project information, and local Git repository path.
5.  **Start Using Workflows:** Once onboarding is complete, you can use commands like:
    *   `@head-orchestrator Start US Azure#<ID>`
    *   `@head-orchestrator Analyze PR Azure#<ID>`
    *   `@head-orchestrator Analyze need: "..."`
    *   Refer to the specific `01_AI-RUN/*.md` files for the exact functionality and expected parameters of each workflow.

## 📚 Key System Files

*   **`.agilepherominduserinfo` (Local, Gitignored):** Stores your specific user identity, project connection details (local Git path!), and language preference. Created during onboarding.
*   **`.pheromone` (Local, typically Gitignored):** The dynamic state file. Contains `currentUser`, `currentProject`, `activeWorkflow`, `memoryBank`, `documentationRegistry`, etc. All core data is in English.
*   **`.roomodes`:** JSON definitions for all AgilePheromind AI agents and their English `customInstructions`.
*   **`.swarmConfig`:** JSON defining the `interpretationLogic` for the `✍️ @orchestrator-pheromone-scribe` to process English NL summaries.
*   **`01_AI-RUN/` (Directory):** Markdown scripts defining the **project workflow phases and logic** that AgilePheromind executes. These are the "programs" for the UO.
*   **`02_AI-DOCS/` (Directory):** Contains project-specific English convention templates, and will store generated English documents (like PO analyses, migration reports, system optimization reports). Localized user-facing reports will also be linked from here or `03_SPECS/`.
*   **`03_SPECS/` (Directory):** Will store more granular English specifications, logs, and specific reports. Localized user-facing reports might also be stored in relevant subdirectories here.
*   **`04_PR_REVIEWS/` (Directory, Gitignored or cleaned periodically):** Stores artifacts from PR reviews.

## 🛠️ Technology Stack Focus

AgilePheromind is primarily designed to assist teams working with:
*   **Backend:** .NET (C#)
*   **Frontend:** Angular (TypeScript)
*   **DevOps & ALM:** Azure DevOps (Boards, Repos, Pipelines)
*   **Database:** MSSQL (Azure SQL)
*   **Containerization:** Docker
*   **Orchestration:** Azure Kubernetes Service (AKS) (Primarily for Docker image management in current scope)

## 🤝 Contributing / Future Vision

AgilePheromind is an ambitious project with vast potential. We envision it evolving into an even more deeply integrated and proactive AI partner for Agile teams. Contributions, ideas, and feedback are highly welcome!

**Future Directions:**
*   More sophisticated learning capabilities for agents based on `memoryBank` analysis.
*   Richer MCP integrations (e.g., SonarQube, advanced testing tools).
*   Visual dashboard for `.pheromone` state and `memoryBank` insights.
*   Enhanced natural language understanding for user commands.
*   More proactive suggestions and risk alerts.

---