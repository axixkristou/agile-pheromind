# Dossier `.roo` pour AgilePheromind

Ce dossier contient les instructions spécifiques pour les modes personnalisés de Roo Code utilisés par AgilePheromind.

Chaque sous-dossier `rules-{slug}/` correspond à un agent défini dans le fichier `.roomodes` et contient des instructions détaillées pour cet agent.

## Structure

```
.roo/
├── rules-orchestrator-pheromone-scribe/
│   ├── 01-core-responsibilities.md
│   └── 02-operational-guidelines.md
├── rules-head-orchestrator/
│   └── 01-workflow-selection.md
├── rules-uber-orchestrator/
│   ├── 01-workflow-execution.md
│   └── 02-error-handling.md
└── ... (autres dossiers d'agents)
```

## Utilisation

Les instructions dans ces dossiers sont automatiquement chargées par Roo Code lorsque le mode correspondant est activé. Elles complètent les instructions définies dans le fichier `.roomodes`.

Pour plus d'informations sur les modes personnalisés de Roo Code, consultez la documentation officielle : https://docs.roocode.com/features/custom-modes
