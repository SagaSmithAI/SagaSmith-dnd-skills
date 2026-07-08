---
name: dnd-dm
description: "Run D&D 5e 2014 or 2024 sessions with rules-first adjudication and scene-bounded module retrieval."
---

# D&D Dungeon Master

## Mode Detection

First, detect which mode is available:

```powershell
sagasmith-dnd doctor --json 2>nul && set SAGASMITH_MODE=runtime || set SAGASMITH_MODE=portable
if "%SAGASMITH_MODE%"=="portable" python tools/portable.py doctor
```

If `sagasmith-dnd` is found → **Runtime mode** (full persistence, FTS5 search, vector store).
If not → **Portable mode** (file-based, zero pip deps, Python stdlib only).

In Portable mode, locate `tools/portable.py` (in the skill repo root) and use it in place of `sagasmith-dnd`.

## Turn Loop

1. Resolve the acting scope (`party`, `group:<id>`, or `player:<id>`). Run `module current`
   to get that scope's current scene. A player scope inherits the party scene until it
   records its own.
2. Read that scene. Search only when it lacks the needed fact; expand before using.
3. Ask for player intent when it is ambiguous.
4. Search rules before resolving a disputed or edition-sensitive mechanic.
5. Roll openly through the CLI.
6. Narrate the consequence without changing established module facts.
7. Merge discoveries and room state into that scope's existing progress.
8. Persist important events, character state, scene progress, and memory.
9. Save at decision points, chapter transitions, and before dangerous restores.

Before running a session, read `references/DM_RULES.md`. Load the other references
only when their workflow is active:

- character creation or advancement: `references/CHAR_CREATION.md`
- module preparation and scene transitions: `references/MODULE_INDEX.md` and
  `references/MODULE_ARC.md`
- tactical positioning: `references/DM_MAP_SYS.md`
- reusable narration and state shapes: `references/DM_TEMPLATES.md`

## Command Reference

Use the left column in Runtime mode, the right column in Portable mode.

### Service / Health

| Runtime | Portable |
|---------|----------|
| `sagasmith-dnd doctor --json` | `python tools/portable.py doctor` |

### Rules Retrieval

| Runtime | Portable |
|---------|----------|
| `sagasmith-dnd rules search --campaign <id> --query "<q>" --limit 5 --json` | `python tools/portable.py rules search --campaign <id> --query "<q>" --limit 5` |
| `sagasmith-dnd rules expand --chunk <id> --json` | (read the SRD file directly) |

Portable mode searches the bundled SRD `.md` files in `skills/dnd-dm/srd/` using
lexical scoring with Chinese ↔ English query expansion.

### Module Retrieval

| Runtime | Portable |
|---------|----------|
| `sagasmith-dnd module current --campaign <id> --scope <scope> --json` | `python tools/portable.py module current --campaign <id> --scope <scope>` |
| `sagasmith-dnd module search --campaign <id> --query "<q>" --limit 5 --json` | `python tools/portable.py module search --campaign <id> --query "<q>" --limit 5` |
| `sagasmith-dnd module expand --chunk <id> --json` | (read the scene from its `.md` file) |
| `sagasmith-dnd module read-scene --campaign <id> --scene <id> --json` | `python tools/portable.py module read-scene --campaign <id> --scene <id>` |
| `sagasmith-dnd module inspect --path <path> --json` | `python tools/portable.py module inspect --path <path>` |
| `sagasmith-dnd module ingest --campaign <id> --path <path> --json` | `python tools/portable.py module ingest --campaign <id> --path <path>` |
| `sagasmith-dnd module index --campaign <id> --json` | `python tools/portable.py module index --campaign <id>` |

Never reveal unseen rooms, future twists, hidden NPC motives, or appendix secrets.

### Campaign

| Runtime | Portable |
|---------|----------|
| `sagasmith-dnd campaign start --name <name> --edition 2024 --json` | `python tools/portable.py campaign start --name <name> --edition 2024` |
| `sagasmith-dnd campaign list --json` | `python tools/portable.py campaign list` |
| `sagasmith-dnd campaign rules-get --campaign <id> --json` | (not needed in portable — SRD is bundled) |

### Dice

| Runtime | Portable |
|---------|----------|
| `sagasmith-dnd roll dice --expression "2d6+3" --json` | `python tools/portable.py roll dice --expression "2d6+3"` |
| `sagasmith-dnd roll check --dc 15 --score 16 --proficient --level 5 --json` | `python tools/portable.py roll check --dc 15 --score 16 --proficient --level 5` |
| `sagasmith-dnd roll attack --dc 17 --score 18 --proficient --level 5 --json` | `python tools/portable.py roll attack --dc 17 --score 18 --proficient --level 5` |

Use `--advantage` or `--disadvantage` only when the selected edition grants it.

### 2014 Checks

For 2014 character adjudication, prefer character-based checks over hand-built
`roll check` math. The agent decides whether a check is needed, the DC, and any
advantage/disadvantage after reading rules or module text; the CLI computes the
character's ability modifier, proficiency, expertise, and total.

```powershell
sagasmith-dnd check ability --character <character-id> --ability strength --dc 15 --json
sagasmith-dnd check skill --character <character-id> --skill perception --dc 15 --json
sagasmith-dnd check save --character <character-id> --ability dexterity --dc 13 --json
sagasmith-dnd check tool --character <character-id> --ability dexterity --tool thieves_tools --dc 15 --json
sagasmith-dnd check initiative --character <character-id> --json
```

### Combat

Runtime combat is structured state. Do not hand-edit campaign combat JSON during
normal play.

```powershell
sagasmith-dnd scene create --campaign <id> --name "<battle map>" --width 1000 --height 800 --json
sagasmith-dnd scene activate --campaign <id> --scene <scene-id> --json
sagasmith-dnd scene show --scene <scene-id> --json
sagasmith-dnd token create --scene <scene-id> --name "<name>" --actor-id <actor-id> --actor-type character --x 0 --y 0 --json
sagasmith-dnd combat start --campaign <id> --scene <scene-id> --name "<encounter>" --json
sagasmith-dnd combat status --campaign <id> --json
sagasmith-dnd game-item create --campaign <id> --actor <actor-id> --name "Longsword" --type weapon --payload '{"equipped":true}' --json
sagasmith-dnd game-activity create --item <item-id> --name "Slash" --type attack --payload '{"activation":{"type":"action"},"system":{"attack_bonus":5,"damage":"1d8+3","damage_type":"slashing"}}' --json
sagasmith-dnd activity use --campaign <id> --actor <actor-id> --item <item-id> --activity <activity-id> --target-id <target-actor-id> --json
sagasmith-dnd combat death-save --campaign <id> --target-id <actor-id> --json
sagasmith-dnd condition add --campaign <id> --actor <actor-id> --condition prone --json
sagasmith-dnd condition remove --campaign <id> --actor <actor-id> --condition prone --json
sagasmith-dnd rest short --campaign <id> --actor <actor-id> --payload '{"hit_dice":1}' --json
sagasmith-dnd rest long --campaign <id> --json
sagasmith-dnd combat end-turn --campaign <id> --actor <actor-id> --json
sagasmith-dnd combat end --campaign <id> --json
```

Before each combat narration, run `combat status` and use its `current` and
`legal_actions` fields. Combat starts from visible Scene Tokens linked to Actor
documents; do not pass free-form participants.

Use `scene show` and `token show` for map narration; prepared token runtime
contains Actor summary, HP bar, targetability, size, position, and vision derived
from Actor senses. Use `scene activate` when changing tactical scenes so
`scene_end` durations advance.

Use `activity use` as the normal combat action entry point. Cast activities handle
spell slots, cantrip scaling, upcasting, ritual payloads, spell attack/DC defaults,
concentration, saves, damage, healing, effects, and pending reactions. Low-level
`combat attack/damage/heal/condition` commands are debugging fallbacks, not the
DM narration path.

### State Updates

| Runtime | Portable |
|---------|----------|
| `sagasmith-dnd event add --campaign <id> --type discovery --summary "<s>" --payload '<json>' --json` | `python tools/portable.py event add --campaign <id> --type discovery --summary "<s>" --payload '<json>'` |
| `sagasmith-dnd module set-progress --campaign <id> --scope <scope> --scene <id> --progress 50 --state '<json>' --json` | `python tools/portable.py module set-progress --campaign <id> --scope <scope> --scene <id> --progress 50 --state '<json>'` |
| `sagasmith-dnd memory add --campaign <id> --type fact --subject "<s>" --content "<f>" --json` | `python tools/portable.py memory add --campaign <id> --type fact --subject "<s>" --content "<f>"` |
| `sagasmith-dnd save create --campaign <id> --label "<label>" --json` | `python tools/portable.py save create --campaign <id> --label "<label>"` |
| `sagasmith-dnd save list --campaign <id> --json` | `python tools/portable.py save list --campaign <id>` |

### Characters

| Runtime | Portable |
|---------|----------|
| `sagasmith-dnd character create --campaign <id> --name <name> --sheet '<json>' --json` | `python tools/portable.py character create --campaign <id> --name <name> --sheet '<json>'` |
| `sagasmith-dnd character list --campaign <id> --json` | `python tools/portable.py character list --campaign <id>` |
| `sagasmith-dnd character get --campaign <id> --name <name> --json` | `python tools/portable.py character get --campaign <id> --name <name>` |

## Portable Mode Data

Data lives in `~/.sagasmith/`:

```
~/.sagasmith/
  campaigns.json                  # campaign index
  <campaign_id>/
    campaign.json                 # metadata
    characters/<name>.json        # character sheets
    modules/<source_key>.md       # imported modules (Markdown)
    progress.json                 # scoped scene progress
    events.jsonl                  # event log
    memories.jsonl                # campaign memory
    saves/<slot>/                 # snapshot copies
```

Module files are plain Markdown with `##` headings as scenes. Progress merges
state per-scope (`party`, `group:<id>`, `player:<id>`). Save snapshots copy the
campaign directory with `shutil.copytree`.

## Portable Mode Limitations

- No SQL FTS5 — search uses Python lexical scoring with Chinese ↔ English expansion
- No ChromaDB vector search
- No PDF import — convert PDF to Markdown first, then `module ingest`
- No transaction atomicity — file writes use atomic rename (write tmp → replace)
  for individual file safety, but multi-file operations (like save) are best-effort
