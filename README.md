# AgilePheromind: Système d'Assistance IA Agile Propulsé par Pheromind

AgilePheromind est un système avancé d'intelligence artificielle en essaim, conçu pour augmenter les capacités des équipes de développement logiciel Agile. Basé sur le framework Pheromind, il orchestre une collection d'agents IA spécialisés pour automatiser, assister et optimiser diverses tâches tout au long du cycle de vie du développement, de la compréhension des besoins à la livraison et à la maintenance.

## Vision

L'objectif d'AgilePheromind est de devenir un partenaire IA indispensable pour les équipes utilisant des méthodologies Agiles et des technologies telles que .NET, Angular, Azure DevOps, Docker et AKS. Il vise à :

*   **Augmenter la productivité:** En automatisant les tâches répétitives et en fournissant une assistance contextuelle rapide.
*   **Améliorer la qualité:** En intégrant des analyses de code, des générations de tests et des vérifications de conformité.
*   **Faciliter la collaboration:** En centralisant l'information et en fournissant des outils de support pour les rituels Agiles.
*   **Promouvoir l'apprentissage continu:** En capitalisant sur l'expérience du projet via une "Memory Bank" évolutive.
*   **S'intégrer de manière transparente:** En utilisant des Model Context Protocol (MCP) servers pour interagir avec les outils existants de l'équipe (Azure DevOps, Git, etc.).

## Architecture du Système

AgilePheromind s'appuie sur les composants clés du framework Pheromind :

1.  **Scripts de Workflow (`01_AI-RUN/*.md`):** Ces fichiers Markdown définissent la logique de haut niveau et la séquence des phases pour des opérations spécifiques (ex: démarrer une User Story, analyser un besoin PO, réviser une PR).
2.  **Agents IA Spécialisés (`.roomodes`):** Un ensemble d'agents, chacun avec un rôle et des instructions spécifiques (ex: `@po-assistant`, `@developer-agent`, `@code-reviewer-assistant`, `@migration-analyst-agent`). Leurs capacités sont définies dans le fichier `.roomodes`.
3.  **État Partagé et Mémoire Collective (`.pheromone`):** Un fichier JSON central qui sert de "cerveau" au système. Il contient :
    *   L'état actuel du projet (utilisateur actif, US/tâche en cours).
    *   Une `documentationRegistry` pour suivre tous les artefacts générés.
    *   Une `memoryBank` détaillée pour stocker l'historique des décisions, analyses, estimations, contextes utilisateurs, conventions du projet, et autres informations persistantes cruciales pour l'apprentissage et la cohérence du système.
4.  **Orchestrateurs Pheromind:**
    *   `🎩 @head-orchestrator`: Le point d'entrée qui initie les workflows `01_AI-RUN/` en réponse aux commandes de l'utilisateur.
    *   `🧐 @uber-orchestrator`: L'agent principal qui lit les scripts `01_AI-RUN/`, consulte `.pheromone` pour le contexte, et délègue les phases aux agents spécialisés.
    *   `✍️ @orchestrator-pheromone-scribe`: L'agent vital qui interprète les résumés en langage naturel des autres agents et met à jour `.pheromone` (y compris la `memoryBank`) de manière précise et structurée, en utilisant la logique définie dans `.swarmConfig`.
5.  **Configuration du Swarm (`.swarmConfig`):** Ce fichier JSON contient la `interpretationLogic` que le `✍️ @orchestrator-pheromone-scribe` utilise pour traduire les rapports des agents en mises à jour structurées de `.pheromone`.
6.  **Model Context Protocol (MCP) Servers:** Des serveurs spécialisés qui permettent aux agents IA d'interagir avec des outils externes tels que Azure DevOps, Git, des bases de données MSSQL, des outils d'analyse de code, des services de documentation (Context7), etc.

## Fonctionnalités Clés (Exemples)

*   Assistance à la gestion des User Stories et des tâches avec Azure DevOps.
*   Aide à la décomposition et à l'estimation des tâches.
*   Génération de code (snippets, tests unitaires) pour .NET et Angular.
*   Analyse de code pour la qualité, la sécurité et la conformité aux conventions.
*   Assistance à la revue de Pull Requests.
*   Génération de messages de commit standardisés.
*   Analyse de code legacy pour les projets de migration.
*   Génération et maintenance de la documentation technique.
*   Support pour les rituels Agiles (Sprint Planning, Daily Stand-up).
*   Gestion proactive des risques et de la dette technique.
*   Automatisation des déploiements Docker sur AKS via Azure Pipelines.

## Stack Technologique Cible de l'Équipe

Le système est optimisé pour les équipes travaillant avec :

*   **Backend:** .NET (C#)
*   **Frontend:** Angular (TypeScript)
*   **Gestion de Projet Agile:** Azure DevOps
*   **Base de Données:** MSSQL (Azure SQL)
*   **CI/CD:** Azure Pipelines
*   **Conteneurisation & Orchestration:** Docker, Azure Kubernetes Service (AKS)

## Démarrage

1.  Configurer les fichiers `.roomodes` et `.swarmConfig` pour définir les agents et la logique du Scribe.
2.  S'assurer que les MCPs requis sont accessibles et configurés.
3.  Interagir avec le `🎩 @head-orchestrator` via l'interface Roo Code en utilisant les commandes définies pour lancer les workflows `01_AI-RUN/`.

AgilePheromind est conçu pour évoluer avec les besoins de l'équipe, en intégrant continuellement de nouvelles capacités et en affinant son intelligence collective.