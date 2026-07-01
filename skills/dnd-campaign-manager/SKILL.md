---
name: dnd-campaign-manager
description: "Create and maintain D&D campaigns, characters, modules, saves, and long-term memory through sagasmith-dnd."
---

# D&D Campaign Manager

All commands end in `--json`.

## Start

```powershell
sagasmith-dnd campaign start --name "<name>" --edition 2014 --locale zh --json
sagasmith-dnd campaign start --name "<name>" --edition 2024 --locale en --json
```

Store the returned campaign ID. Import a module only after the campaign exists:

```powershell
sagasmith-dnd module inspect --path "<module.pdf>" --json
sagasmith-dnd module ingest --campaign <id> --path "<module.pdf>" --json
```

Existing output from `SagaSmith-module-gen-skills` can be passed directly to
`module ingest`.

## Characters

```powershell
sagasmith-dnd character create --campaign <id> --name "<name>" --type pc --player "<player>" --sheet '<json>' --json
sagasmith-dnd character list --campaign <id> --json
sagasmith-dnd character show --id <character-id> --json
sagasmith-dnd character update --id <character-id> --sheet '<json>' --json
```

Do not persist a draft until the user confirms it.

## Saves

```powershell
sagasmith-dnd save create --campaign <id> --label "<label>" --json
sagasmith-dnd save list --campaign <id> --json
sagasmith-dnd save verify --campaign <id> --slot <n> --json
sagasmith-dnd save lineage --campaign <id> --json
sagasmith-dnd save restore --campaign <id> --slot <n> --json
```

Restore automatically preserves the current state and creates a new branch. Never
describe restore as overwriting history.

## Continuity

```powershell
sagasmith-dnd event list --campaign <id> --limit 30 --json
sagasmith-dnd memory search --campaign <id> --query "<question>" --limit 8 --json
sagasmith-dnd state history --campaign <id> --limit 30 --json
```

Use memory for durable facts and events for chronology.
