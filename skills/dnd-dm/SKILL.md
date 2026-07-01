---
name: dnd-dm
description: "Run D&D 5e 2014 or 2024 sessions with rules-first adjudication and scene-bounded module retrieval."
---

# D&D Dungeon Master

## Runtime Context

Start with:

```powershell
sagasmith-dnd doctor --json
sagasmith-dnd campaign list --status active --json
```

Once selected, retain `campaign_id`. Read its rule profile:

```powershell
sagasmith-dnd campaign rules-get --campaign <id> --json
```

## Turn Loop

1. Read only the current scene or search the active module.
2. Ask for player intent when it is ambiguous.
3. Search rules before resolving a disputed or edition-sensitive mechanic.
4. Roll openly through the CLI.
5. Narrate the consequence without changing established module facts.
6. Persist important events, character state, scene progress, and memory.
7. Save at decision points, chapter transitions, and before dangerous restores.

## Rule Retrieval

```powershell
sagasmith-dnd rules search --campaign <id> --query "<question>" --limit 5 --json
sagasmith-dnd rules expand --chunk <chunk-id> --json
```

Use search snippets to select a result, then expand one result. Cite title, edition,
locale, and publication. The campaign profile is authoritative.

## Module Retrieval

```powershell
sagasmith-dnd module search --campaign <id> --query "<current situation>" --limit 5 --json
sagasmith-dnd module expand --chunk <chunk-id> --json
sagasmith-dnd module read-scene --campaign <id> --scene <scene-id> --json
```

Never reveal unseen rooms, future twists, hidden NPC motives, or appendix secrets.

## Dice

```powershell
sagasmith-dnd roll dice --expression "2d6+3" --json
sagasmith-dnd roll check --dc 15 --score 16 --level 5 --proficient --json
sagasmith-dnd roll attack --dc 17 --score 18 --level 5 --proficient --json
```

Use `--advantage` or `--disadvantage` only when the selected edition grants it.

## State Updates

```powershell
sagasmith-dnd event add --campaign <id> --type discovery --summary "<event>" --payload '<json>' --json
sagasmith-dnd module set-progress --campaign <id> --scene <scene-id> --progress 50 --state '<json>' --json
sagasmith-dnd memory add --campaign <id> --type fact --subject "<subject>" --content "<fact>" --json
sagasmith-dnd save create --campaign <id> --label "<decision point>" --json
```

## Portable Mode

If Runtime is unavailable, consult the selected directory under `srd/`. Default to
2024 unless the user selects 2014. The 2014 Chinese translation is a convenience
reference; verify important adjudications against 2014 English. Do not simulate
persistence.
