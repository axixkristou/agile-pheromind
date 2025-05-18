# AgilePheromind: Système d'Assistance IA Agile Propulsé par Pheromind

**Version:** 1.1 (intégrant améliorations de contexte et de résilience)

AgilePheromind est un système avancé d'intelligence artificielle en essaim, conçu pour augmenter les capacités des équipes de développement logiciel Agile. Basé sur le framework Pheromind, il orchestre une collection d'agents IA spécialisés pour automatiser, assister et optimiser diverses tâches tout au long du cycle de vie du développement, de la compréhension des besoins à la livraison et à la maintenance. Ce système met un accent particulier sur la **compréhension contextuelle** grâce à une `memoryBank` riche et des mécanismes d'**injection de contexte ciblée**, ainsi que sur la **résilience** grâce à des stratégies de gestion d'erreur et de clarification.

## Vision

L'objectif d'AgilePheromind est de devenir un partenaire IA indispensable pour les équipes utilisant des méthodologies Agiles et des technologies telles que .NET, Angular, Azure DevOps, Docker et AKS. Il vise à :

*   **Augmenter la productivité:** En automatisant les tâches répétitives et en fournissant une assistance contextuelle rapide et pertinente.
*   **Améliorer la qualité:** En intégrant des analyses de code, des générations de tests et des vérifications de conformité, basées sur une compréhension approfondie du projet.
*   **Faciliter la collaboration:** En centralisant l'information, en assurant la clarté des communications (y compris avec les agents IA via `@clarification-agent`), et en fournissant des outils de support pour les rituels Agiles.
*   **Promouvoir l'apprentissage continu:** En capitalisant sur l'expérience du projet via une `memoryBank` évolutive, enrichie par les "chaînes de pensée" des agents.
*   **S'intégrer de manière transparente et robuste:** En utilisant des Model Context Protocol (MCP) servers pour interagir avec les outils existants de l'équipe, avec des mécanismes de gestion d'erreur.

## Architecture du Système

AgilePheromind s'appuie sur les composants clés du framework Pheromind :

1.  **Scripts de Workflow (`01_AI-RUN/*.md`):** Ces fichiers Markdown définissent la logique de haut niveau, la séquence des phases, et incluent des **stratégies de gestion d'erreur** pour des opérations spécifiques. L'`🧐 @uber-orchestrator` est responsable de l'**injection de contexte ciblé** depuis la `memoryBank` vers les agents spécialisés, comme spécifié dans ces scripts.
2.  **Agents IA Spécialisés (`.roomodes`):** Un ensemble d'agents, chacun avec un rôle et des instructions spécifiques. Les instructions incluent désormais la nécessité de **détailler leur "chaîne de pensée"** pour les analyses complexes et la capacité à interagir avec `@clarification-agent` en cas d'ambiguïté. Un nouvel agent, `@clarification-agent`, est introduit pour gérer les interactions avec l'utilisateur lorsque le système a besoin de lever une ambiguïté.
3.  **État Partagé & Mémoire Collective (`.pheromone`):** Un fichier JSON central qui sert de "cerveau" au système. (Structure inchangée, mais son contenu est enrichi par des informations plus contextuelles et des historiques de clarification).
4.  **Orchestrateurs Pheromind:** (Rôles principaux inchangés, mais l'UO a une responsabilité accrue dans la gestion du contexte et des erreurs).
5.  **Configuration du Swarm (`.swarmConfig`):** Ce fichier JSON contient la `interpretationLogic`. Des règles peuvent être ajoutées pour traiter les réponses de `@clarification-agent` ou pour stocker les "chaînes de pensée" des agents.
6.  **Model Context Protocol (MCP) Servers:** (Liste inchangée, mais leur utilisation par les agents est maintenant encadrée par des stratégies de gestion d'erreur plus explicites).

## Fonctionnalités Clés (Exemples, avec améliorations)

*   Assistance à la gestion des User Stories et des tâches avec Azure DevOps, avec une meilleure gestion des ambiguïtés grâce à `@clarification-agent`.
*   Aide à la décomposition et à l'estimation des tâches, avec des agents détaillant leur raisonnement (Chaîne de Pensée).
*   Génération de code (.NET, Angular) et de tests unitaires, informée par un contexte projet plus riche.
*   Analyse de code pour la qualité, la sécurité et la conformité aux conventions, avec des stratégies de reprise si les outils MCP échouent.
*   Assistance à la revue de Pull Requests, avec possibilité de demander des clarifications sur des points spécifiques du code.
*   Génération de messages de commit standardisés.
*   Analyse de code legacy pour les projets de migration, avec une documentation détaillée du processus d'analyse.
*   Génération et maintenance de la documentation technique, en s'assurant que le contexte est bien compris.
*   Support pour les rituels Agiles (Sprint Planning, Daily Stand-up).
*   Gestion proactive des risques et de la dette technique.
*   Automatisation des déploiements Docker sur AKS via Azure Pipelines, avec une meilleure gestion des erreurs de déploiement.

## Stack Technologique Cible de l'Équipe

(Inchangée)

*   **Backend:** .NET (C#)
*   **Frontend:** Angular (TypeScript)
*   **Gestion de Projet Agile:** Azure DevOps
*   **Base de Données:** MSSQL (Azure SQL)
*   **CI/CD:** Azure Pipelines
*   **Conteneurisation & Orchestration:** Docker, Azure Kubernetes Service (AKS)

## Démarrage

(Similaire, mais le bootstrap inclura la vérification du `@clarification-agent`)

AgilePheromind, avec ces améliorations, vise une interaction plus intelligente, plus robuste et plus transparente, minimisant les erreurs dues aux malentendus et augmentant la fiabilité globale du système.