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

## Inventory

Use the item ledger for backpacks, treasure, currency, and containers. Do not
edit or rely on `sheet.inventory`; Actor-owned `game-item` and `game-activity`
documents are authoritative for combat abilities, equipment actions, spells, and
features.

```powershell
sagasmith-dnd item add --campaign <id> --name "Iron Key" --owner-type character --owner-id <character-id> --quantity 1 --json
sagasmith-dnd item list --campaign <id> --owner-type character --owner-id <character-id> --json
sagasmith-dnd item move --item <item-id> --owner-type party --owner-id party --json
sagasmith-dnd item equip --item <item-id> --slot main_hand --json
sagasmith-dnd item use --item <item-id> --quantity 1 --json
sagasmith-dnd item history --campaign <id> --item <item-id> --json
```

Create a snapshot after major treasure, shopping, or loadout changes.

## Combat

Use Foundry-style runtime commands for map, initiative, HP, conditions, action
economy, effects, periods, and turn flow. Do not use `combat act`.

```powershell
sagasmith-dnd scene create --campaign <id> --name "<battle map>" --width 1000 --height 800 --json
sagasmith-dnd token create --scene <scene-id> --name "<name>" --actor-type character --actor-id <character-id> --x 0 --y 0 --json
sagasmith-dnd token move --token <token-id> --x 30 --y 20 --json
sagasmith-dnd region create --scene <scene-id> --name "Web" --shape '{"type":"circle","x":10,"y":10,"radius":20}' --behavior difficult_terrain --json
sagasmith-dnd combat start --campaign <id> --scene <scene-id> --name "<encounter>" --json
sagasmith-dnd combat status --campaign <id> --json
sagasmith-dnd game-item create --campaign <id> --actor <actor-id> --name "Longsword" --type weapon --payload '{"equipped":true}' --json
sagasmith-dnd game-activity create --item <item-id> --name "Slash" --type attack --payload '{"activation":{"type":"action"},"system":{"attack_bonus":5,"damage":"1d8+3","damage_type":"slashing"}}' --json
sagasmith-dnd activity use --campaign <id> --actor <actor-id> --item <item-id> --activity <activity-id> --target-id <target-actor-id> --json
sagasmith-dnd condition add --campaign <id> --actor <actor-id> --condition prone --json
sagasmith-dnd effect add --campaign <id> --actor <actor-id> --name "Bless" --duration '{"period":"declared_minute","value":1}' --json
sagasmith-dnd time advance --campaign <id> --minutes 10 --reason "searching the room" --json
sagasmith-dnd rest short --campaign <id> --json
sagasmith-dnd combat end-turn --campaign <id> --actor <actor-id> --json
sagasmith-dnd combat end --campaign <id> --json
```

Before narrating any combat turn, read `combat status` and follow its `current`,
`legal_actions`, `legal_action_details`, `turn_budget`, and `effects`. Create a
snapshot before and after important encounters and map state changes. Wall-clock
time never advances duration; only runtime period commands do.

Use English runtime IDs in commands even during Chinese narration. Chinese labels
may follow fvtt-cn terminology, but command keys such as `main_action`,
`reaction`, `weapon`, `ActiveEffect`, and ruleset IDs remain stable English IDs.

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
