---
name: dnd-campaign-manager
description: "Create and maintain D&D campaigns through the SagaSmith D&D MCP server."
---

# D&D Campaign Manager

`full/` is MCP-first. Use the raw MCP names below (a client may prefix them), not
shell `sagasmith-dnd` commands. Read `../../references/mcp-contract.md` and
`../dnd-dm/references/DM_RULES.md` before mutating a campaign.

Call `game_phase_get` when resuming a campaign. Use the `authoring` profile for
the setup, module, import, indexing, and character-building workflows below;
call `game_phase_set(tool_profile="play")` only when live in-character play begins.

## Start and Modules

1. Call `campaign_create` with name, edition, locale, and optional description.
2. Persist the returned `campaign_id`.
3. Resolve the caller's stable `principal_id`; use `campaign_member_grant` and
   `actor_grant` for access instead of treating `player_name` as authorization.
4. Generate Markdown through `module_write`, inspect it with `module_inspect`, then
   call `module_import` with a stable campaign-wide `idempotency_key`. This
   preserves an editable MCP-managed artifact before ingestion and makes an
   exact retry return the original import result.
5. Use `module_index` to choose a scene; use `module_set_progress` with an explicit
   `scope_id` to enter it. Do not narrate from a `module_search` snippet until
   `module_expand` or `module_read_scene` has been called.

## Characters

| Need | MCP tool |
|---|---|
| Create a public template or direct instance | `character_create` |
| List templates | `character_library_list` |
| Instantiate a template | `character_instantiate` |
| Atomically create PC template + instance | `character_build` |
| List or read live actors | `character_list`, `character_get` |
| Full reviewed replacement | `character_sheet_replace` |
| Ability generation | `dnd_ability_roll`, `character_ability_apply` |

All live actors use `sheet v2` and `notes v2`. Read
`../../references/character-schema-v2.md` before creation or mutation. Do not
persist an unconfirmed draft.

For normal play, mutate only the affected structure:

```text
character_inventory_add | character_inventory_update | character_inventory_remove
character_inventory_transfer | character_inventory_equip | character_ammunition_consume
character_wallet_adjust | character_rest | character_effect_add
character_effect_remove | character_resource_set | character_memory_add
character_memory_resolve
party_show | party_inventory_add | party_inventory_remove | party_inventory_transfer
party_wallet_adjust | party_wallet_transfer
```

Use `character_spell_prepare` or `character_spell_prepare_list` only in
`authoring` for setup and level advancement. During live play, pass the complete
new list as `prepared_spell_ids` on the actor's `character_rest` long-rest
transaction; do not toggle preparations one by one.

After each actor or party mutation call `character_get` or `party_show`. Use their
derived values rather than recalculating weapon attacks, AC, or encumbrance in
prose.

Before every write, read the matching optimistic token and send a fresh
`idempotency_key`. Character tools use the actor revision; scene progress uses
`state_version`; knowledge revisions use `revision_id`; branch/snapshot tools use
the campaign revision plus the current branch or snapshot-head ID. The exact
fields are listed in `../../references/mcp-contract.md`.

## Saves, Branches, and Audit

| Need | MCP tool |
|---|---|
| Create/list a save | `snapshot_create`, `snapshot_list` |
| Validate / inspect lineage | `snapshot_verify`, `snapshot_lineage` |
| Restore without deleting future history | `snapshot_restore` |
| Regenerate recap | `snapshot_regenerate_recap` |
| List/create/switch timeline | `branch_list`, `branch_create`, `branch_checkout` |
| Audit / undo / redo | `state_history`, `state_undo`, `state_redo` |

Restore is a branch fork, never destructive overwrite. Verify the target first,
then refresh campaign actors, party state, scene progress, events, and continuity
context after restoring.

## Continuity

Use `memory_*` for branch-scoped durable world facts and `event_*` for immutable
chronology. Use `actor_knowledge_*` only for one PC/NPC/monster's subjective
knowledge. The separate `character_memory_*` tools keep legacy notes memories and
are not imported into the actor-knowledge ledger.

For player-safe retrieval, call `continuity_context` with `actor_id`, `scope_id`,
audience, and optionally `branch_id`. Do not substitute broad `memory_search` for
that context; it can expose DM facts or sibling-branch history.
