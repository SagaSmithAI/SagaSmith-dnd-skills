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
| Wallet, spell, effects, resources | `character_wallet_adjust`, `character_spell_prepare`, `character_spell_prepare_list`, `character_effect_add`, `character_effect_remove`, `character_resource_set` |
| Ability scores | `dnd_ability_roll`, `character_ability_apply` |
| Legacy notes memory | `character_memory_add`, `character_memory_resolve` |
| Shared stash/wallet | `party_show`, `party_inventory_add`, `party_inventory_remove`, `party_inventory_transfer`, `party_wallet_adjust`, `party_wallet_transfer` |

The campaign instance is authoritative. After any actor or party mutation, read
`character_get` or `party_show` and use returned `derived` values. Do not use
`character_sheet_replace` for a one-field mutation.

Prepared-spell selection is edition- and class-aware. In `authoring`, use
`character_spell_prepare_list` with the complete selected list and `event: setup`
or `event: level_up`; the singular `character_spell_prepare` is only a setup
edit. In live `play`, supply the complete `prepared_spell_ids` list to
`character_rest` so recovery and a legal long-rest replacement commit atomically.
In 2024, Cleric/Druid/Wizard may replace any number after a Long Rest,
Paladin/Ranger may replace one, and Bard/Sorcerer/Warlock replace one only when
gaining a class level. In 2014, Cleric/Druid/Paladin/Wizard may change their list
after a Long Rest, while Bard/Ranger/Sorcerer/Warlock use spells known. Always-
prepared spells and cantrips never occupy selections. Wizard selections must be
in the spellbook. Multiclass eligibility uses each spell's `grant.source_key`
and that class's own level. Campaign-bound characters inherit campaign edition.

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

## Tool profiles and game phase

The MCP publishes three exact, server-owned tool profiles. `server_tool_profiles`
returns their current membership; clients must consume the tool metadata instead
of maintaining a second hard-coded list.

| Profile | Intended state | Distinct writes |
|---|---|---|
| `authoring` | Outside play: module writing/import/indexing, campaign setup, character creation/building, rule ingestion | `module_write`, `module_import`, `character_create`, `character_build`, `rule_ingest` |
| `play` | Live non-combat exploration and downtime | inventory/wallet transfers, rests, non-combat spells/activities, scene progress, memory and actor knowledge |
| `combat` | Active structured encounter | `combat_*` settlement plus narrow read/effect/resource tools |

Use `game_phase_set` only to enter `authoring` or `play`. `combat_start`
automatically returns `tool_profile=combat`; `combat_end` returns
`tool_profile=play`. On a new or resumed agent session, call `game_phase_get`
before acting so the client can restore the persisted campaign phase. The
server's authorization, revision, and idempotency checks still apply even if a
client fails to hide an out-of-profile tool.

## Integrity and identity contract

Campaign creation accepts `principal_id`; the server creates an owner membership
for that principal. Platform users must be resolved to stable principal IDs before
calling sensitive tools. Roles and actor grants are checked by MCP, not supplied by
the model as trusted claims.

`campaign_create` and `character_create` require a fresh `idempotency_key`; their
entity rows, initial branch/owner membership where applicable, and replay receipt
commit atomically.

Use `campaign_member_grant` for campaign roles and `actor_grant` for explicit PC/NPC
control or private-sheet visibility. A player's `player_name` field is descriptive
only and is never an authorization mechanism.

All campaign-bound granular character writes require the character's current
`expected_revision` and a fresh `idempotency_key`. Inventory transfer additionally
requires `expected_campaign_revision`, `expected_source_revision`, and
`expected_target_revision`.

Append-only `event_add`, `memory_add`, and `actor_knowledge_add` require a fresh
`idempotency_key`. `actor_knowledge_revise` also requires the current
`expected_revision_id`; this is the knowledge ledger's revision token, not the
character-card revision.

Branch creation/checkout and snapshot restore require the current campaign
revision, active `expected_branch_id`, and a fresh key. Snapshot creation requires
the current campaign revision, current `expected_head_snapshot_id` (use `""` when
the branch has no head), and a fresh key. `state_undo` / `state_redo` require the
current `expected_history_sequence` from `state_history` plus a fresh key.

`branch_create(checkout=true)` returns both the new branch and the materialized
snapshot; pointer changes and state restoration are one transaction.
The transfer is one mutation group, so one `state_undo` restores every affected
wallet, item stack, and character revision together. Retrying the same key replays
the original result; reusing it with different arguments is an error.

Use `branch_compare` before discussing alternate timelines. There is no implicit
branch merge: world facts and each actor's subjective knowledge require explicit
conflict decisions.

Fresh MCP storage automatically seeds the compact bundled SRD corpus. Use
`rule_seed_status` and `rule_seed_bundled` to inspect or re-run the idempotent seed.
Structured combat is an auditable preflight/commit surface. Initiative, turn
budgets, movement spend, canonical derived weapon attacks, typed damage,
resistances, vulnerabilities, immunities, temporary HP, concentration windows,
healing, and death saves are automatic. Surprise is a supplied scene fact and
is interpreted by the selected 2014/2024 ruleset. The DM may provide a
`participant_config` at `combat_start` with token position, disposition, reach,
`hidden`, explicit `visible_to_actor_ids`, surprise, and initiative. Omit
`visible_to_actor_ids` for an ordinarily visible creature; provide it when known
special senses let only named participants see a hidden or Invisible creature.
With valid grid positions, `combat_move`
verifies the declared five-foot grid distance and creates an owned
`opportunity_attack` reaction window only when a mover leaves an eligible
hostile's reach; `combat_reaction_attack` settles that window and its attack in
one mutation. Collision, terrain, forced movement, line of sight, unrecorded
triggers, and narrative consequences remain explicit DM choices through
`combat_choice_open` / `combat_choice_resolve`.
`combat_common_action` settles the action payment for Dash, Disengage, Dodge,
Help, Hide, Search, and non-spell Ready without inventing their narrative result;
`combat_reactions` exposes an eligible actor's pending reaction windows.
The generic Ready action rejects spell payloads. Use the dedicated spell-ready
transaction instead:

1. `combat_ready_spell` accepts only a spell with an Action casting time. It pays
   the action and the spell slot or other casting resource immediately, replaces
   any prior concentration, and records an explicit perceivable trigger.
2. `combat_readied_spell_trigger` is a DM/owner confirmation that the trigger has
   occurred and opens an owned reaction window. It does not infer trigger truth
   from prose.
3. `combat_readied_spell_resolve` either releases the spell and consumes the
   caster's reaction, or declines that occurrence without spending the reaction.
   Declining rearms the same held spell for a later occurrence before expiry.

The held spell always requires concentration, including a spell that normally
does not. Concentration loss, the start of the caster's next turn, or combat end
dissipates it without effect. When a normally-concentration spell is released,
its original concentration duration continues; otherwise the holding effect
ends. Release returns `pending_ruling`, because targeting, spell attacks, saves,
damage, areas, and narrative consequences still require the relevant settlement
tools and DM decisions. Reaction spells and activities otherwise require an
owned pending reaction window; they cannot be invoked merely because it is not
the actor's turn.
Numeric attack modifiers and damage formulas supplied by a client are ignored;
they must come from `derived.inventory.weapon_attacks` or an explicit DM ruling.

Every combat write should provide `expected_revision` and `idempotency_key`.
`combat_preflight_attack` never mutates; `combat_resolve_attack`,
`combat_move`, `combat_end_turn`, `combat_check`, `combat_use_activity`,
`combat_ready_spell`, `combat_readied_spell_trigger`, `combat_readied_spell_resolve`,
`combat_concentration_check`, `combat_apply_damage`, and `combat_heal` commit one
atomic mutation group. Sensitive combat writes require both
`expected_revision` and `idempotency_key`. Player views are filtered by campaign
membership; keeper logs, target mechanics, and rulings are not exposed to
players.

Treat an `idempotency_key` as unique for the whole campaign, not merely one
tool name. A successful state mutation is recorded in the same transaction as
its revision group. If a process stops before its rich response receipt is
stored, retry returns the safe opaque replay `{status: committed,
response_recovery: read_current_state}` rather than applying the mutation
again; then read the relevant campaign, character, or combat state.

`character_use_activity` and `combat_use_activity` work with the normalized
`content.activities`, `content.features`, and `content.feats` cards. They spend
one `resource_key` resource when present, otherwise one limited card use, and
pay the card's action/bonus-action/reaction timing in combat. Card prose,
choices, targeting, and any non-deterministic result are returned for an
explicit DM ruling rather than automatically materialized.

`character_rest` applies v2-card short/long-rest recovery with a character
revision and idempotency key. Timed card effects advance at the ending actor's
turn; any non-deterministic healing or narrative rest consequence remains a DM
ruling.

`module_set_progress` requires the current `expected_state_version` for that
scene/scope row (`0` for its first write) and a fresh idempotency key.

`campaign_advance_effects` is DM-only and advances matching `minute`, `hour`,
`day`, `round`, or `encounter` effect durations across campaign actors as one
atomic group. It must be called with an explicit established period; conversation
time is never treated as campaign time automatically.

## Player-safe module reads

`module_read_scene`, `module_index`, and `module_search` accept `principal_id`.
DM/owner reads may include keeper content; player reads are filtered to `public` or
`party` visibility and keeper content is replaced with a redaction marker.
