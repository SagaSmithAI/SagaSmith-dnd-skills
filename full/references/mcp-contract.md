# Full Runtime MCP Contract

`full/` is MCP-first. Connect the `sagasmith_dnd` server and call the raw tool
names below; a client may expose them with a prefix such as
`mcp_sagasmith_dnd_`. Do not require a local `sagasmith-dnd` executable.

## Core turn loop

| Intent | MCP tool |
|---|---|
| Health and owned storage | `storage_status`, `storage_migrate`, `server_capabilities` |
| Campaign | `campaign_create`, `campaign_get`, `campaign_list`, `campaign_update`, `campaign_member_grant`, `actor_grant` |
| Rules | `rule_ingest`, `rule_search`, `rule_expand`, `rule_seed_status`, `rule_seed_bundled` |
| Roll | `dnd_dice_roll`, `dnd_check`, `dnd_ability_roll` |
| Module artifact | `module_write`, `module_inspect`, `module_import` |
| Scene play | `module_current`, `module_search`, `module_expand`, `module_read_scene`, `module_index`, `module_set_progress` |
| Chronology | `event_add`, `event_list`, `memory_add`, `memory_list`, `memory_search` |
| Snapshot | `snapshot_create`, `snapshot_list`, `snapshot_verify`, `snapshot_lineage`, `snapshot_restore`, `snapshot_regenerate_recap`, `branch_compare` |
| Audit | `state_history`, `state_undo`, `state_redo` |

`module_search` only selects candidates. Call `module_expand` or
`module_read_scene` before narrating a module fact. Always provide the active
`scope_id` to `module_current` and `module_set_progress`.

## Actor cards and shared party state

| Intent | MCP tool |
|---|---|
| Template/library | `character_create` without `campaign_id`, `character_library_list`, `character_instantiate` |
| Atomic PC build | `character_build` |
| Read/replace card | `character_get`, `character_list`, `character_sheet_replace` |
| Inventory | `character_inventory_add`, `character_inventory_update`, `character_inventory_remove`, `character_inventory_transfer`, `character_inventory_equip`, `character_ammunition_consume` |
| Wallet, spell, effects, resources | `character_wallet_adjust`, `character_spell_prepare`, `character_effect_add`, `character_effect_remove`, `character_resource_set` |
| Ability scores | `dnd_ability_roll`, `character_ability_apply` |
| Legacy notes memory | `character_memory_add`, `character_memory_resolve` |
| Shared stash/wallet | `party_show`, `party_inventory_add`, `party_inventory_remove`, `party_inventory_transfer`, `party_wallet_adjust`, `party_wallet_transfer` |

The campaign instance is authoritative. After any actor or party mutation, read
`character_get` or `party_show` and use returned `derived` values. Do not use
`character_sheet_replace` for a one-field mutation.

## Branch-aware continuity

World facts, chronology, and subjective actor knowledge are different ledgers.

| Ledger | MCP tool |
|---|---|
| Branches | `branch_list`, `branch_create`, `branch_checkout` |
| World facts | `memory_add`, `memory_list`, `memory_search` |
| Events | `event_add`, `event_list` |
| One actor's belief/knowledge | `actor_knowledge_add`, `actor_knowledge_revise`, `actor_knowledge_list`, `actor_knowledge_search` |
| Safe retrieved context | `continuity_context` |

Pass `branch_id` for an explicit historical branch. For player-safe narration,
use `continuity_context` with the acting `actor_id`, `scope_id`, and audience;
never substitute a broad `memory_search` result for that context.

## Deliberate boundaries

- `full` uses MCP-owned SQLite, optional ChromaDB, and managed module artifacts.
- `standalone/` remains a separate portable file workflow; it does not call MCP.
- `references/cli-contract.md` documents the legacy CLI compatibility path only.

## Integrity and identity contract

Campaign creation accepts `principal_id`; the server creates an owner membership
for that principal. Platform users must be resolved to stable principal IDs before
calling sensitive tools. Roles and actor grants are checked by MCP, not supplied by
the model as trusted claims.

Use `campaign_member_grant` for campaign roles and `actor_grant` for explicit PC/NPC
control or private-sheet visibility. A player's `player_name` field is descriptive
only and is never an authorization mechanism.

Cross-entity transfers should include an `idempotency_key` and expected revisions.
The transfer is one mutation group, so one `state_undo` restores every affected
wallet, item stack, and character revision together. Retrying the same key replays
the original result; reusing it with different arguments is an error.

Use `branch_compare` before discussing alternate timelines. There is no implicit
branch merge: world facts and each actor's subjective knowledge require explicit
conflict decisions.

Fresh MCP storage automatically seeds the compact bundled SRD corpus. Use
`rule_seed_status` and `rule_seed_bundled` to inspect or re-run the idempotent seed.
The structured combat engine is intentionally not part of this contract yet;
existing combat tools remain a generic auditable state surface.

## Player-safe module reads

`module_read_scene`, `module_index`, and `module_search` accept `principal_id`.
DM/owner reads may include keeper content; player reads are filtered to `public` or
`party` visibility and keeper content is replaced with a redaction marker.
