---
name: dnd-campaign-manager
description: "Create and maintain D&D campaigns, characters, modules, saves, and long-term memory through sagasmith-dnd."
---

# D&D Campaign Manager

All commands end in `--json`.

Read `../dnd-dm/references/DM_RULES.md` before mutating a campaign. For character
creation, module lifecycle, save/restore, undo, recap, memory, player assignments,
or advancement, load the matching section of that reference and the linked
specialized reference. The CLI is authoritative; never emulate a successful write.

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

For the complete v2 contract, read `references/database-contract.md` and
`../../references/character-schema-v2.md`. During play, mutate only the affected
state through granular commands:

```powershell
sagasmith-dnd character inventory add --id <id> --payload '<item-json>' --json
sagasmith-dnd character inventory transfer --id <source> --target <target> --item <item-id> --json
sagasmith-dnd character wallet credit --id <id> --denomination gp --amount 10 --json
sagasmith-dnd character equipment equip --id <id> --item <item-id> --slot-name main_hand --json
sagasmith-dnd character spell prepare --id <id> --spell <spell-id> --json
sagasmith-dnd character effect add --id <id> --payload '<effect-json>' --json
sagasmith-dnd character memory add --id <npc-id> --payload '<memory-json>' --json
sagasmith-dnd party inventory deposit --campaign <id> --id <character-id> --item <item-id> --json
```

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

Use `state undo` and `state redo` for audited mutations; these do not delete
snapshots. Keep player-to-character assignments in campaign state so they remain
portable across agent platforms.
