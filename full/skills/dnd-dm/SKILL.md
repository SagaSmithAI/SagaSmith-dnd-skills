---
name: dnd-dm
description: "Run D&D 5e 2014 or 2024 sessions through the SagaSmith D&D MCP server."
---

# D&D Dungeon Master

## Runtime

This full skill is MCP-first. Start by calling `storage_status`; all raw tool
names below may be prefixed by the host, for example `mcp_sagasmith_dnd_`.
If the server is unavailable, stop using this skill and load `standalone/` rather
than switching to a local CLI.

Read `../../references/mcp-contract.md` before a mutation and
`references/DM_RULES.md` before a session. Load these only when needed:

- actor creation or advancement: `references/CHAR_CREATION.md`
- actor, items, wallet, spells, effects, or resources:
  `../../references/character-schema-v2.md`
- module preparation or scene transitions: `references/MODULE_INDEX.md` and
  `references/MODULE_ARC.md`
- tactical positioning or reusable narration: `references/DM_MAP_SYS.md` and
  `references/DM_TEMPLATES.md`

## Turn Loop

1. Resolve `scope_id` (`party`, `group:<id>`, or `player:<id>`), then call
   `module_current`. Player scopes inherit party progress until they have their own.
2. Read that scene through `module_read_scene`. Use `module_search` only to select a
   candidate, then call `module_expand` before relying on a chunk.
3. Ask for intent when it is ambiguous. Never reveal unseen rooms, future twists,
   hidden motives, or sibling-branch facts.
4. Use `rule_search` then `rule_expand` for disputed or edition-sensitive rules.
5. Resolve openly with `dnd_dice_roll` or `dnd_check`.
6. Persist events, scene progress, actor/party state, and durable facts. Use
   `actor_knowledge_*` for what one PC/NPC believes, not `memory_*`.
7. Call `snapshot_create` at decision points, chapter transitions, and before a
   dangerous restore. Use `snapshot_verify` and `snapshot_lineage` before restore.

## MCP Tool Reference

| Workflow | MCP tools |
|---|---|
| Campaign | `campaign_create`, `campaign_get`, `campaign_list` |
| Rules | `rule_ingest`, `rule_search`, `rule_expand` |
| Module lifecycle | `module_write`, `module_inspect`, `module_import`, `module_list`, `module_index` |
| Scene play | `module_current`, `module_search`, `module_expand`, `module_read_scene`, `module_set_progress` |
| Rolls | `dnd_dice_roll`, `dnd_check`, `dnd_ability_roll` |
| World continuity | `event_add`, `event_list`, `memory_add`, `memory_search` |
| Actor continuity | `actor_knowledge_add`, `actor_knowledge_revise`, `actor_knowledge_list`, `actor_knowledge_search`, `continuity_context` |
| Saves and audit | `snapshot_create`, `snapshot_list`, `snapshot_verify`, `snapshot_lineage`, `snapshot_restore`, `state_history`, `state_undo`, `state_redo` |
| Combat state | `combat_start`, `combat_status`, `combat_act`, `combat_end` |

## Actor Cards and Party State

Every live PC, NPC, and monster is an authoritative v2 actor card. Use
`character_get` after every write. Use granular tools instead of replacing a whole
sheet for a small change:

```text
character_inventory_add | character_inventory_update | character_inventory_remove
character_inventory_transfer | character_inventory_equip | character_ammunition_consume
character_wallet_adjust | character_spell_prepare | character_effect_add
character_effect_remove | character_resource_set | character_memory_add
character_memory_resolve | character_ability_apply
party_show | party_inventory_add | party_inventory_remove | party_inventory_transfer
party_wallet_adjust | party_wallet_transfer
```

Use `character_build` for a PC when a library template and its first campaign
instance must be created atomically. Use `character_library_list` and
`character_instantiate` for existing templates. `character_memory_*` stays a legacy
notes field; new subjective information belongs in the actor-knowledge ledger.

After item writes, treat `character_get(...).derived.inventory.weapon_attacks` and
`character_get(...).derived.inventory.encumbrance` as authoritative. Represent one
active concentration spell as one active effect with `concentration: true` and its
`source_spell_id`.
