# Full Runtime MCP Contract

`full/` is MCP-first. Connect the `sagasmith_dnd` server and call the raw tool
names below; a client may expose them with a prefix such as
`mcp_sagasmith_dnd_`. Do not require a local `sagasmith-dnd` executable.
`server_capabilities.rulebook_import` is the machine-readable contract for the
ordered import stages, canonical citation fields, and play/combat settlement tools.

## Core turn loop

| Intent | MCP tool |
|---|---|
| Health and owned storage | `storage_status`, `storage_migrate`, `server_capabilities` |
| Campaign | `campaign_create`, `campaign_get`, `campaign_list`, `campaign_update`, `campaign_member_grant`, `actor_grant` |
| Rules | `rule_document_stage`, `rule_document_inspect`, `rule_document_import`, `rule_ingest`, `rule_search`, `rule_expand`, `rule_seed_status`, `rule_seed_bundled`, `rule_pack_draft`, `rule_pack_draft_from_source`, `rule_pack_install`, `rule_pack_list`, `rule_pack_inspect`, `rule_pack_test`, `rule_pack_remove`, `campaign_rule_profile_get`, `campaign_rule_profile_set`, `campaign_rule_pack_set`, `campaign_rule_pack_remove`, `campaign_rules_explain`, `campaign_rule_receipts`, `content_catalog_list`, `character_content_apply`, `character_rule_artifact_add` |
| Roll | `dnd_dice_roll`, `dnd_check`, `dnd_ability_roll`, `character_check` |
| Module artifact | `module_import(stage/inspect/validate/ingest/activate)`, `import_query` |
| Scene play | `module_query(list/index/scene/current/progress)`, `module_search`, `module_expand`, `module_set_progress` |
| Chronology | `event_add`, `event_list`, `memory_add`, `memory_list`, `memory_search` |
| Snapshot | `snapshot_create`, `snapshot_list`, `snapshot_verify`, `snapshot_lineage`, `snapshot_restore`, `snapshot_regenerate_recap`, `branch_compare` |
| Audit | `state_history`, `state_undo`, `state_redo` |

`module_search` only selects candidates. Call `module_expand` or
`module_query(view="scene")` before narrating a module fact. Always provide the active
`scope_id` to `module_current` and `module_set_progress`.

## Structured content catalog

For a campaign locked to 2014, the installed `dnd5e.content.srd2014` catalog
provides source-linked class, subclass, species, background, feat, spell, and
item records. Use `content_catalog_list` to discover only the core edition and
extension packs enabled on the current branch. Do not search an inactive pack
and then apply its option by id. The list response includes compact
`selection_requirements` and source citations without copying an entire rule
entry into the catalog listing.

`character_content_apply` safely records catalog spells, feats, backgrounds,
and a selected subclass on a character card, preserving its pack version and
source references. Catalog presence is not a claim that every narrative effect
is executable: an item, species, class, or an option with unresolved choices
returns `pending_ruling` rather than inventing a settlement. Use a source-bound
rule pack mechanic only when the rule has been reviewed and validated.

Supply `selection.source_class` and an appropriate spell grant `method` when a
spell has class eligibility, `selection.target_class_name` for a multiclass
subclass choice, and all required background choices. The runtime rejects a
spell outside the selected class list or class-level limit and never silently
assigns a subclass to the first class on a multiclass card.

## Modules, space evidence, and temporary combat maps

Module re-imports are revisions: earlier sources are retained for snapshots and
scoped scene progress, while normal `module_index` results show only the newest
active revision. A D&D scene can contain conservative `spatial.locations`
evidence recovered from room headings and stated dimensions. Set
`current_location_key` with `module_set_progress` only when it names a location
in the current scene or exactly one spatial location elsewhere in the same
module. This lets an encounter scene reference a separately indexed room scene
without merging their narrative content; ambiguous or cross-module keys fail.

In `lobby`, run `module_import` in strict order: `stage` -> `inspect` ->
`validate` -> `ingest` -> `activate`. `stage` accepts either generated
`payload.name` + `payload.content`, or a user document in `payload.source_path`.
The latter is limited to `SAGASMITH_DND_MCP_MODULE_IMPORT_ROOTS`; the server
copies PDF/Markdown/text into checksum-addressed MCP storage before Core reads it.
Inspect exposes the Core PDF-to-Markdown page/bookmark statistics and complete
scene/spatial preview. Ingest remains inactive, and only activation changes the
campaign's active revision under optimistic concurrency.

For PDFs, `preview.valid` also requires every scene to have an in-bounds
`page_start`/`page_end`; the preview exposes those fields together with source
line ranges. The normalized revision identity is the source checksum plus
`parser_profile` plus `parser_version`. Re-run inspect -> validate -> ingest ->
activate after a parser upgrade even when the document checksum and semantic
scene diff are unchanged, then verify the inactive and active indexes before
starting play.

When `combat_start` has a current scene (or receives `scene_id`), the server
creates and freezes an encounter-local `battle_map`. It may enforce supplied
grid bounds and DM-confirmed blocked cells, but never invents walls, line of
sight, doors, terrain cost, or a global tactical map. Use `combat_map_patch`
only for DM-confirmed world changes; it stores the patch in the encounter audit
and the scene runtime state. End combat before treating that temporary map as a
different scene or module revision. When progress references a same-module
spatial scene, map `source.scene_id` identifies that spatial evidence and
`source.encounter_scene_id` retains the active narrative encounter.

Call `module_import(action="inspect")` before validation; every later stage uses
the same D&D parser profile. Every write carries a stable stage-specific `idempotency_key`;
an exact retry returns the original result. A player may read `party` scope or
their own authorized `player:<actor_id>` scope only. Keeper scene reads expose
only the redaction marker, and player combat-map views omit blocked cells,
difficult terrain, world patches, checksums, and DM overrides.

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

Prepared-spell selection is edition- and class-aware. In `lobby`, use
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

## Session exposure and game phase

The MCP, not an agent configuration, owns tool exposure. Every connection starts
with a compact core: `exposure_open`, `exposure_status`, `exposure_search`,
`exposure_inspect`, `exposure_load`, `exposure_unload`, `exposure_call`,
`campaign_query`, `game_phase`, and server diagnostics. Start or resume with
`exposure_open(campaign_id, principal_id)`, search/inspect a group, then load it.
Loaded groups are scoped to that MCP session and principal; another connected
agent must open and load its own exposure.

| Phase | Intended state | Example groups |
|---|---|---|
| `lobby` | Game-outside setup, imports, campaign administration, character building | `lobby.bootstrap`, `lobby.campaign`, `lobby.characters`, `lobby.rules`, `lobby.modules` |
| `play` | Live non-combat exploration and downtime | `play.scene`, `play.characters`, `play.resolution` |
| `combat` | Active structured encounter | `combat.observe`, `combat.turn`, `combat.actions`, `combat.save`, `combat.map` |

For native clients supporting MCP `tools/list_changed`, loading or unloading a
group changes the actual tool list. If a host cannot refresh its native schemas,
it keeps the core and calls the same loaded domain tool through
`exposure_call(exposure_id, tool_id, arguments)`. This is a transport fallback,
not a permission bypass: it performs the identical session and phase check.

An exposure without `campaign_id` may load only `lobby.bootstrap` (system list
and campaign creation), plus the `system:local`-only storage administration
group. After campaign creation or selection, call `exposure_open` again with the
campaign id. Campaign administration, rules and module-import groups additionally
require owner/DM membership. A campaign-bound exposure rejects arguments that
target a different campaign, including character ids resolved to that campaign.

Use `game_phase(action="set", tool_profile="lobby" | "play")` only for the
non-combat transition. `combat_start` moves the campaign to `combat` and
`combat_end` returns it to `play`; the server invalidates incompatible loaded
groups. Authorization, revision, idempotency, and engine checks apply even if a
client presents a stale schema.
Direct character-card mutations (sheet replacement, inventory, wallet, effects,
resources, rests, non-combat casts, and activities) are rejected while an
encounter is active. Do not use a profile mismatch to bypass combat action
economy.

Rule text retrieval and executable rules are separate. For user documents,
`rule_document_stage` accepts only configured import roots, and inspect/import
read only the MCP-managed artifact. Core performs the shared PDF/Markdown
normalization and records the original checksum, parser warnings, and per-chunk
page ranges. `rule_ingest` remains the direct Markdown compatibility path.
Only the safe declarative IR accepted by
`rule_pack_draft` can settle mechanics; arbitrary Python, expression evaluation,
network access, and database paths are forbidden. Installation does not enable
a pack. A DM explicitly pins a validated version per branch, and snapshots keep
the exact version/checksum lock. Missing locked versions never fall back to a
newer version. Use `campaign_rules_explain` to audit applied mechanic ids,
citations, and the deterministic fingerprint.
Use `campaign_rule_receipts` for the fingerprint and citations stored atomically
with historical settlements.
For a user-imported executable rule, use `rule_pack_draft_from_source`: citations
must be imported chunk ids and are resolved server-side to the exact source id,
document checksum, heading path, and page range. Use `character_check` outside
combat and `combat_check` during combat when an enabled `check.before` rule needs
DM-established `rule_facts`.

Treat rule-profile and branch rule-pack changes as campaign writes. First read
`campaign_rule_profile_get`, then pass its latest `campaign_revision` as
`expected_revision` together with a stable `idempotency_key` to
`campaign_rule_profile_set`, `campaign_rule_pack_set`, or
`campaign_rule_pack_remove`. Reuse the same key only for an exact retry; a stale
revision requires a fresh read and review.

The base engine is not an implicit fallback. Every new campaign locks either
`dnd5e.core.2014` or `dnd5e.core.2024`, including a fingerprinted coverage list
for the existing combat, movement, reaction, damage, rest, spell, character,
and MCP mutation boundaries. Optional packs layer on top of that core provider.
If a runtime upgrade changes the locked core fingerprint, settlement fails until
the DM explicitly reviews and relocks the campaign profile.
Snapshot restore and branch checkout check the saved Core lock before changing
live state. A legacy save without that lock, or a save requiring an unavailable
Core fingerprint, needs an explicit conversion path and is never silently
upgraded.

## Integrity and identity contract

Campaign creation accepts `principal_id`; the server creates an owner membership
for that principal. Platform users must be resolved to stable principal IDs before
calling sensitive tools. Roles and actor grants are checked by MCP, not supplied by
the model as trusted claims.

`campaign_create`, `character_create`, and `character_build` require a fresh
`idempotency_key`; their entity rows, initial branch/owner membership where
applicable, and replay receipt commit atomically. A replay of `character_build`
returns its original library template and campaign instance as one pair.

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
Help, Hide, Search, Influence, Study, Utilize/Use an Object, and non-spell Ready
without inventing their narrative result;
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

For a generic non-spell Ready action, use `combat_common_action(action="ready")`,
then let the DM confirm the trigger with `combat_readied_action_trigger` and let
the actor release or decline with `combat_readied_action_resolve`. Releasing pays
the reaction and returns `pending_ruling`; it never fabricates the declared
effect.

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
`combat_concentration_check`, and `combat_hp_change` commit one
atomic mutation group. Sensitive combat writes require both
`expected_revision` and `idempotency_key`. Player views are filtered by campaign
membership; keeper logs, target mechanics, and rulings are not exposed to
players.

For `combat_hp_change(action=heal)`, `payload.amount` is the rolled or otherwise
resolved base healing. Spell healing additionally carries `source_actor_id`,
`spell_id`, and `spell_level`. The server verifies the spell on that source card,
rejects illegal cast levels, applies source-linked modifiers such as 2014
Disciple of Life, clamps once to maximum HP, and returns the base, bonus, effective,
and actually restored amounts separately.

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
revision and idempotency key. For a Short Rest, provide each spent hit die and
its rolled result through `hit_dice_spends`; the runtime applies Constitution
and the edition's minimum. A 2014 Long Rest may require an explicit
`hit_dice_recovery` allocation across multiclass pools. A 2024 Long Rest restores
all expended Hit Dice; exhaustion falls by one. In 2014 exhaustion recovery needs
the DM-confirmed `food_and_drink=true` condition. Timed card effects advance at
the ending actor's turn; any narrative rest consequence remains a DM ruling.

`character_cast_spell` and `combat_cast_spell` settle only timing, casting
resources, concentration, and recorded components. They return `pending_ruling`
for targets and effects. Cantrips and rituals cannot be upcast; a ritual cannot
complete in active combat. Costly or consumed material components require
`component_ruling.material_confirmed=true` before resources are spent. Pact Magic
uses the recorded `pact_magic.slot_level` and is counted as a slot expenditure.

`module_set_progress` requires the current `expected_state_version` for that
scene/scope row (`0` for its first write) and a fresh idempotency key.

`campaign_advance_effects` is DM-only and advances matching `minute`, `hour`,
`day`, `round`, or `encounter` effect durations across campaign actors as one
atomic group. It must be called with an explicit established period; conversation
time is never treated as campaign time automatically.

## Player-safe module reads

`module_read_scene`, `module_index`, and `module_search` accept `principal_id`.
DM/owner reads may include keeper content; player reads are filtered to `public` or
`party` visibility and keeper content is replaced with a redaction marker. A
player cannot select another player or group scope merely by knowing its ID.
