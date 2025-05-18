# Manuel du Débutant - AgilePheromind

**Version:** 1.0
**Date:** Mai 2025

## 1. Introduction à AgilePheromind

AgilePheromind est un système avancé d'intelligence artificielle conçu comme un essaim d'agents IA collaboratifs, basé sur le framework Pheromind. Ce système est spécialement conçu pour assister les équipes de développement logiciel Agile, en particulier celles utilisant des technologies comme .NET, Angular, Azure DevOps, Docker et Azure Kubernetes Service (AKS).

### 1.1 Qu'est-ce qu'AgilePheromind ?

AgilePheromind fonctionne comme un écosystème où des agents IA spécialisés interagissent indirectement pour accomplir des tâches complexes, apprendre du contexte du projet, et s'adapter aux besoins de l'équipe. Contrairement à un assistant IA monolithique, AgilePheromind divise les responsabilités entre différents agents, chacun expert dans son domaine.

### 1.2 Avantages clés

- **Assistance complète au cycle de vie Agile** : De l'analyse des besoins au déploiement
- **Intégration profonde avec les outils existants** : Azure DevOps, Git, MSSQL, etc.
- **Mémoire collective évolutive** : Apprentissage continu basé sur l'historique du projet
- **Spécialisation des agents** : Expertise ciblée pour chaque aspect du développement
- **Orchestration intelligente** : Coordination automatique des workflows complexes

## 2. Prérequis

### 2.1 Prérequis techniques

- **Roo Code** : AgilePheromind fonctionne exclusivement sur la plateforme Roo Code
- **Système d'exploitation** : Windows 10/11, macOS ou Linux (compatible avec Roo Code)
- **Navigateur web moderne** : Chrome, Edge, Firefox ou Safari récent
- **Connexion Internet stable** : Pour l'accès aux MCPs (Model Context Protocol servers)

### 2.2 Prérequis logiciels

- **Azure DevOps** : Compte et projet configuré
- **Git** : Installation locale et configuration de base
- **Stack de développement** :
  - .NET Core (version 6.0 ou supérieure)
  - Node.js (version 16.0 ou supérieure) pour Angular
  - SQL Server ou Azure SQL (pour les projets avec base de données)
  - Docker (pour les scénarios de conteneurisation)

### 2.3 Connaissances recommandées

- Principes de base des méthodologies Agiles (Scrum, Kanban)
- Familiarité avec Azure DevOps et Git
- Compréhension de base de l'architecture .NET et/ou Angular

## 3. Installation et Configuration

### 3.1 Installation de base

1. Assurez-vous d'avoir un compte Roo Code actif
2. Clonez le dépôt AgilePheromind :
   ```
   git clone https://github.com/votre-organisation/agile-pheromind.git
   cd agile-pheromind
   ```
3. Ouvrez le dossier dans Roo Code

### 3.2 Configuration initiale

1. **Configuration du projet Azure DevOps** :
   - Exécutez le script de bootstrap en activant le mode `🎩 @head-orchestrator` dans Roo Code
   - Entrez la commande : "Bootstrap AgilePheromind pour le projet [Nom du Projet Azure DevOps]"
   - Suivez les instructions pour fournir l'URL de l'organisation Azure DevOps et les identifiants

2. **Configuration des MCPs** :
   - Vérifiez que tous les MCPs nécessaires sont disponibles dans votre environnement Roo Code
   - Les MCPs requis sont : Azure DevOps, Git Tools, Context7, MSSQL, Sequential Thinking, Browser Tools, Fetch

3. **Initialisation du fichier .pheromone** :
   - Le script de bootstrap créera automatiquement le fichier `.pheromone` avec une structure de base
   - Vérifiez que le fichier a été correctement initialisé en examinant son contenu

## 4. Architecture du Système

### 4.1 Composants principaux

AgilePheromind est composé de plusieurs éléments clés :

1. **Fichiers de configuration** :
   - `.roomodes` : Définit les agents IA spécialisés
   - `.swarmConfig` : Contient la logique d'interprétation pour le Scribe
   - `.pheromone` : Stocke l'état du système et la mémoire collective

2. **Scripts de workflow** :
   - Situés dans le dossier `01_AI-RUN/*.md`
   - Définissent les séquences d'actions pour différentes tâches

3. **Agents IA spécialisés** :
   - Définis dans `.roomodes` comme des modes personnalisés de Roo Code
   - Chaque agent a un rôle et des compétences spécifiques

4. **Model Context Protocol (MCP) Servers** :
   - Permettent l'interaction avec des outils et services externes
   - Incluent Azure DevOps MCP, Git Tools MCP, Context7 MCP, etc.

### 4.2 Flux d'information

1. L'utilisateur interagit avec le `🎩 @head-orchestrator`
2. Le Head Orchestrator délègue au `🧐 @uber-orchestrator`
3. L'Uber Orchestrator exécute un script de workflow
4. Les agents spécialisés sont activés selon les besoins
5. Le `✍️ @orchestrator-pheromone-scribe` met à jour le fichier `.pheromone`

## 5. Premiers Pas avec AgilePheromind

### 5.1 Activation du système

1. Ouvrez votre projet dans Roo Code
2. Activez le mode `🎩 @head-orchestrator` dans la barre latérale de Roo Code
3. Entrez une commande initiale, par exemple : "Bootstrap AgilePheromind"

### 5.2 Commandes de base

- **"Analyse le besoin [description]"** : Déclenche l'analyse d'un nouveau besoin client
- **"Démarre l'US Azure#[ID]"** : Initialise le travail sur une User Story existante
- **"Continue la tâche Azure#[ID]"** : Reprend le travail sur une tâche en cours
- **"Revue la PR Azure#[ID]"** : Lance la revue d'une Pull Request

### 5.3 Navigation dans l'interface

- Utilisez la barre latérale de Roo Code pour basculer entre les différents modes (agents)
- Les résultats des agents sont généralement stockés dans les dossiers :
  - `02_AI-DOCS/` : Documentation et rapports
  - `03_SPECS/` : Spécifications techniques
  - `04_PR_REVIEWS/` : Rapports de revue de Pull Requests

## 6. Scénarios d'Utilisation Détaillés

### 6.1 Gestion des User Stories

#### 6.1.1 Analyse d'un nouveau besoin

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Analyse le besoin [description détaillée du besoin]"
3. Le système activera le workflow `03_PO_Analyze_Need.md`
4. L'agent `🧑‍💼 @po-assistant` analysera le besoin et proposera des User Stories
5. Consultez le rapport d'analyse dans `02_AI-DOCS/PO_Analyses/`

#### 6.1.2 Démarrage d'une User Story

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Démarre l'US Azure#[ID]"
3. Le système récupérera les détails de l'US depuis Azure DevOps
4. L'agent `📊 @task-breakdown-estimator` décomposera l'US en tâches techniques
5. Les tâches seront créées dans Azure DevOps et liées à l'US

### 6.2 Développement de Code

#### 6.2.1 Travail sur une tâche

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Continue la tâche Azure#[ID]"
3. Le système récupérera les détails de la tâche et le contexte associé
4. L'agent `💻 @developer-agent` sera activé pour implémenter le code
5. Le code sera écrit dans les fichiers appropriés du projet

#### 6.2.2 Génération de tests unitaires

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Génère des tests pour [composant/classe]"
3. Le système activera le workflow `04_Generate_Unit_Tests.md`
4. L'agent `🧪 @test-generator-agent` analysera le code et générera des tests
5. Les tests seront écrits dans les fichiers de test appropriés

### 6.3 Revue de Code

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Revue la PR Azure#[ID]"
3. Le système récupérera les détails de la PR et les fichiers modifiés
4. L'agent `🧐 @code-reviewer-assistant` analysera le code pour :
   - Respect des conventions de codage
   - Bugs potentiels
   - Problèmes de sécurité (via `🛡️ @security-analyst-agent`)
   - Code smells
5. Un rapport de revue sera généré dans `04_PR_REVIEWS/[branch]/`

### 6.4 Déploiement

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Déploie [AppName] v[Version] sur AKS [Environnement]"
3. Le système activera le workflow `16_Manage_Docker_AKS_Deployment.md`
4. L'agent `🚀 @deployment-agent-aks` gérera le déploiement via Azure Pipelines
5. Un rapport de déploiement sera généré dans `03_SPECS/Deployments/`

## 7. Personnalisation et Extension

### 7.1 Ajout d'un nouvel agent

1. Modifiez le fichier `.roomodes` pour ajouter une nouvelle entrée dans `customModes`
2. Créez un dossier `.roo/rules-{slug}/` avec les instructions spécifiques
3. Mettez à jour les scripts de workflow pertinents pour utiliser le nouvel agent

### 7.2 Modification des workflows

1. Modifiez ou créez des fichiers dans le dossier `01_AI-RUN/`
2. Suivez la structure existante pour définir les phases et les agents responsables
3. Testez le nouveau workflow en l'invoquant via le `🎩 @head-orchestrator`

## 8. Dépannage

### 8.1 Problèmes courants et solutions

- **Erreur de connexion MCP** : Vérifiez que tous les MCPs sont correctement configurés
- **Fichier .pheromone corrompu** : Utilisez la sauvegarde `.pheromone.bak`
- **Agent bloqué** : Activez le mode `🩺 @swarm-monitor-agent` pour diagnostiquer

### 8.2 Logs et diagnostics

- Consultez le dossier `memlog/` pour les journaux d'activité
- Utilisez l'agent `🩺 @swarm-monitor-agent` pour générer des rapports de santé
- Vérifiez la section `systemHealth` dans le fichier `.pheromone`

## 9. Glossaire

- **Agent** : Un LLM configuré avec un rôle et des instructions spécifiques
- **MCP (Model Context Protocol)** : Interface permettant aux agents d'interagir avec des outils externes
- **Pheromone** : Fichier d'état central stockant le contexte et la mémoire du système
- **Scribe** : Agent responsable de la mise à jour du fichier `.pheromone`
- **Workflow** : Script définissant une séquence d'actions pour accomplir une tâche
- **Memory Bank** : Section du fichier `.pheromone` stockant la connaissance collective du système

## 10. Scénarios d'Utilisation Avancés

### 10.1 Support aux Rituels Agiles

#### 10.1.1 Assistance à la planification de sprint

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Assiste à la planification du sprint [ID Sprint]"
3. Le système activera le workflow `07_Sprint_Planning_Assistant.md`
4. L'agent `🧑‍🏫 @scrum-facilitator-agent` analysera :
   - Les User Stories candidates pour le sprint
   - Les estimations et la capacité de l'équipe
   - Les dépendances entre les tâches
5. Un plan de sprint sera généré dans `02_AI-DOCS/Sprint_Plans/`

#### 10.1.2 Support au Daily Stand-up

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Prépare le daily stand-up"
3. Le système activera le workflow `08_Daily_Standup_Support.md`
4. L'agent `🧑‍🏫 @scrum-facilitator-agent` préparera un résumé :
   - Tâches terminées la veille
   - Tâches en cours
   - Blocages identifiés
5. Un résumé sera généré dans `03_SPECS/Daily_Summaries/`

### 10.2 Analyse et Optimisation

#### 10.2.1 Analyse de la dette technique

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Analyse la dette technique dans [composant/module]"
3. Le système activera le workflow `11_Analyze_Tech_Debt.md`
4. Les agents appropriés analyseront le code pour identifier :
   - Code obsolète ou redondant
   - Violations des principes SOLID
   - Dépendances problématiques
   - Opportunités de refactoring
5. Un rapport sera généré dans `03_SPECS/Tech_Debt/`

#### 10.2.2 Optimisation des performances

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Optimise les performances de [composant]"
3. L'agent `⚡ @performance-optimization-agent` analysera le code pour :
   - Goulots d'étranglement potentiels
   - Requêtes SQL inefficaces
   - Problèmes de gestion de mémoire
   - Optimisations spécifiques à .NET ou Angular
4. Un rapport d'optimisation sera généré avec des recommandations concrètes

### 10.3 Migration et Modernisation

#### 10.3.1 Analyse de code legacy

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Analyse le code legacy [nom du projet]"
3. Le système activera le workflow `09_Legacy_Migration_Analysis.md`
4. L'agent `🔍 @migration-analyst-agent` analysera :
   - La structure du code legacy
   - Les dépendances et technologies utilisées
   - La logique métier à préserver
   - Les défis potentiels de migration
5. Un rapport d'analyse sera généré dans `02_AI-DOCS/Migration_Analyses/`

#### 10.3.2 Refactoring automatisé

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Refactore [composant/classe] pour [objectif]"
3. Le système activera le workflow `19_Automated_Code_Refactoring.md`
4. Les agents appropriés analyseront et refactoreront le code
5. Un rapport de refactoring sera généré avec les changements effectués

### 10.4 Accessibilité et Conformité

1. Activez le mode `🎩 @head-orchestrator`
2. Entrez : "Vérifie l'accessibilité de [composant UI]"
3. Le système activera le workflow `21_Accessibility_Compliance_Check.md`
4. L'agent `♿ @accessibility-compliance-agent` analysera :
   - La conformité WCAG
   - Les problèmes d'accessibilité
   - Les améliorations recommandées
5. Un rapport d'accessibilité sera généré dans `03_SPECS/Accessibility_Reports/`

## 11. Bonnes Pratiques

### 11.1 Organisation du travail avec AgilePheromind

- **Commencez par l'analyse des besoins** : Utilisez `🧑‍💼 @po-assistant` pour clarifier les besoins avant de commencer le développement
- **Décomposez les User Stories** : Utilisez `📊 @task-breakdown-estimator` pour créer des tâches de taille gérable
- **Revue de code systématique** : Utilisez `🧐 @code-reviewer-assistant` pour chaque Pull Request
- **Documentation continue** : Utilisez `📚 @documentation-writer-agent` pour maintenir la documentation à jour

### 11.2 Optimisation de l'utilisation

- **Fournissez un contexte clair** : Plus vos instructions sont précises, meilleurs seront les résultats
- **Utilisez les clarifications** : Si un agent a besoin de plus d'informations, répondez clairement aux questions de `❓ @clarification-agent`
- **Enrichissez la Memory Bank** : Plus le système accumule d'informations sur votre projet, plus il devient efficace
- **Surveillez la santé du système** : Utilisez régulièrement `🩺 @swarm-monitor-agent` pour identifier les problèmes potentiels

### 11.3 Sécurité et gouvernance

- **Revue humaine obligatoire** : Vérifiez toujours le code et les documents générés avant de les valider
- **Contrôle des accès** : Limitez l'accès aux modes sensibles selon les rôles dans l'équipe
- **Sauvegarde régulière** : Exportez périodiquement le fichier `.pheromone` pour sauvegarder la mémoire du système
- **Validation des décisions critiques** : Utilisez AgilePheromind comme conseiller, mais gardez la décision finale sur les aspects critiques

## 12. Ressources Supplémentaires

### 12.1 Documentation officielle

- Documentation complète d'AgilePheromind : `02_AI-DOCS/`
- Documentation de Roo Code : [https://docs.roocode.com/](https://docs.roocode.com/)
- Documentation des MCPs : Disponible via les liens spécifiques à chaque MCP

### 12.2 Formation et support

- Tutoriels vidéo : Disponibles sur la chaîne YouTube officielle
- Forum de support : Communauté active pour l'entraide
- Sessions de formation : Webinaires mensuels sur les fonctionnalités avancées

### 12.3 Mises à jour et évolution

- Consultez régulièrement le dépôt GitHub pour les mises à jour
- Utilisez `⚙️ @workflow-optimizer-agent` pour obtenir des suggestions d'amélioration
- Participez à la communauté pour partager vos retours et idées d'amélioration

## Conclusion

AgilePheromind représente une nouvelle approche de l'assistance IA pour les équipes Agile, combinant la puissance des grands modèles de langage avec une architecture en essaim spécialisée. En suivant ce manuel et en explorant progressivement les capacités du système, vous découvrirez un assistant capable de transformer votre façon de développer des logiciels, d'améliorer la qualité de vos livrables et d'accélérer votre cycle de développement.

N'hésitez pas à expérimenter avec différents agents et workflows pour découvrir tout le potentiel d'AgilePheromind dans votre contexte spécifique. Comme le système apprend et s'améliore avec chaque interaction, son utilité ne fera que croître au fil du temps.
