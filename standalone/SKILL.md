---
name: dnd-dm-lite
description: "D&D 5e 2014/2024 standalone — no pip install needed. File-based persistence, zero dependencies."
---

# D&D Dungeon Master (Standalone)

Standalone mode uses **Python stdlib only** (`portable.py`) + plain Markdown files.
No `sagasmith-dnd` runtime required.

## Quick Start

```powershell
python portable.py doctor
python portable.py campaign start --name "Campaign" --edition 2024
python portable.py character create --campaign <id> --name "Aragorn" --sheet '{"hp":10}'
python portable.py module ingest --campaign <id> --path "module.md" --title "Keep"
python portable.py module current --campaign <id> --scope party
```

Data lives in `~/.sagasmith/`. Create a campaign first.

## Turn Loop

1. Resolve scope (`party`/`group:<id>`/`player:<id>`). Run `module current`.
2. Read that scope's scene. Search if the scene lacks the needed fact.
3. Ask player intent.
4. Search rules via `rules search` before resolving disputed mechanics.
5. Roll openly.
6. Narrate without changing established facts.
7. Merge new state into that scope's progress.
8. Persist events, state, memory.
9. Save at decision points and chapter transitions.

## Commands

### Campaign

```powershell
python portable.py campaign start --name "Campaign" --edition 2024
python portable.py campaign list
python portable.py campaign get --campaign <id>
```

### Characters

```powershell
python portable.py character create --campaign <id> --name "Aragorn" --sheet '{"str":15,"dex":14,"con":13,"int":10,"wis":12,"cha":8}'
python portable.py character list --campaign <id>
python portable.py character get --campaign <id> --name "Aragorn"
```

### Modules

```powershell
python portable.py module ingest --campaign <id> --path "module.md" --title "Module"
python portable.py module index --campaign <id>
python portable.py module current --campaign <id> --scope party
python portable.py module read-scene --campaign <id> --scene <id>
python portable.py module search --campaign <id> --query "<situation>" --limit 5
python portable.py module set-progress --campaign <id> --scope party --scene <id> --progress 50 --room "A1" --state '{"discovered":["key"]}'
```

Import Markdown only. Convert PDF first.

### Rules

```powershell
python portable.py rules search --campaign <id> --query "<question>" --limit 5
```

Searches the bundled SRD in `skills/dnd-dm/srd/`. Chinese queries auto-expand
with English equivalents.

### Dice

```powershell
python portable.py roll dice --expression "2d6+3"
python portable.py roll check --dc 15 --score 16 --proficient --level 5
python portable.py roll check --dc 12 --score 14 --advantage
python portable.py roll attack --ac 17 --score 18 --proficient --level 5
```

When Runtime mode is available for a 2014 campaign, prefer character-based checks
over hand-built `roll check` math:

```powershell
sagasmith-dnd check ability --character <character-id> --ability strength --dc 15 --json
sagasmith-dnd check skill --character <character-id> --skill perception --dc 15 --json
sagasmith-dnd check save --character <character-id> --ability dexterity --dc 13 --json
sagasmith-dnd check tool --character <character-id> --ability dexterity --tool thieves_tools --dc 15 --json
sagasmith-dnd check initiative --character <character-id> --json
```

When Runtime mode is available, use structured Foundry-style combat commands
instead of hand-editing campaign combat JSON:

```powershell
sagasmith-dnd scene create --campaign <id> --name "<battle map>" --width 1000 --height 800 --json
sagasmith-dnd scene activate --campaign <id> --scene <scene-id> --json
sagasmith-dnd actor create-monster --campaign <id> --monster goblin --json
sagasmith-dnd token create --scene <scene-id> --name "<name>" --actor-id <actor-id> --actor-type npc --x 30 --y 20 --json
sagasmith-dnd combat start --campaign <id> --scene <scene-id> --name "<encounter>" --json
sagasmith-dnd combat status --campaign <id> --json
sagasmith-dnd advancement grant-class --campaign <id> --actor <actor-id> --class-id fighter --level 2 --json
sagasmith-dnd advancement grant-subclass --campaign <id> --actor <actor-id> --subclass champion --level 3 --json
sagasmith-dnd advancement grant-feature --campaign <id> --actor <actor-id> --feature action-surge --json
sagasmith-dnd advancement grant-spell --campaign <id> --actor <actor-id> --spell fire-bolt --json
sagasmith-dnd activity use --campaign <id> --actor <actor-id> --item <item-id> --activity <activity-id> --target-id <target-actor-id> --json
sagasmith-dnd resolution list --campaign <id> --actor <actor-id> --json
sagasmith-dnd resolution resolve --campaign <id> --id <reaction-window-id> --payload '{"item_id":"<shield-item>","activity_id":"<shield-activity>"}' --json
sagasmith-dnd combat end-turn --campaign <id> --actor <actor-id> --json
```

Do not pass free-form combat participants. Create Actors, place Actor-linked
Tokens in a Scene, start combat from the Scene, and resolve actions through
`activity use`. Low-level `combat attack/damage/heal/condition` commands are
debugging fallbacks when no Activity exists.

### Events & Memory

```powershell
python portable.py event add --campaign <id> --type discovery --summary "Event" --payload '{}'
python portable.py memory add --campaign <id> --type fact --subject "Key" --content "Value"
python portable.py memory search --campaign <id> --query "key"
```

### Save / Restore

```powershell
python portable.py save create --campaign <id> --label "Before boss"
python portable.py save list --campaign <id>
python portable.py save restore --campaign <id> --slot 3
```

## Reference

- `skills/dnd-dm/references/DM_RULES.md` — adjudication guide
- `skills/dnd-dm/references/DM_TEMPLATES.md` — output templates
- `skills/dnd-dm/references/CHAR_CREATION.md` — character creation
- `skills/dnd-dm/references/MODULE_INDEX.md` — scene navigation
- `skills/dnd-dm/references/MODULE_ARC.md` — narrative structure

## Limitations

- No PDF import — convert to Markdown first
- No SQL FTS5 — lexical scoring with CJK query expansion
- No ChromaDB vector search
