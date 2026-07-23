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
| Campaign | `campaign_create`, `campaign_query(list/get/party)`, `campaign_change`, `access_grant(campaign/actor)` |
| Rules | `rule_import(discover/stage/inspect/ingest/extract_candidates/review/compile/install/activate)`, `rule_document_page_render`, `import_query`, `rule_search`, `rule_expand`, `rule_seed_status`, `rule_seed_bundled`, `rule_pack_compile(draft/from_source)`, `rule_pack_query(list/inspect/test/content_catalog/sources)`, `rule_pack_change(install/remove)`, `campaign_rules(get_profile/set_profile/set_pack/remove_pack/explain/receipts)`, `character_content_apply` |
| Roll | `dnd_dice_roll`, `dnd_check`, `dnd_ability_roll`, `character_check` |
| Module artifact | `module_import(stage/inspect/validate/ingest/activate)`, `import_query` |
| Scene play | `module_query(list/index/scene/current/progress/assets/content/candidates/readiness)`, `module_page_render`, `module_content_review`, `module_search`, `module_expand`, `module_set_progress` |
| Chronology | `continuity_commit`, `campaign_event(add/list)`, `memory_change(add/upsert/revise/supersede)`, `memory_query(list/search)`, `actor_knowledge_change(add/revise)`, `actor_knowledge_query(list/search)`, `continuity_context` |
| Snapshot | `snapshot_create`, `snapshot_query(list/verify/lineage/recap)`, `snapshot_restore`, `branch_query(list/compare)`, `branch_change(create/checkout)` |
| Audit | `state_revision(history/undo/redo)` |

`module_search` only selects candidates. Call `module_expand` or
`module_query(view="scene")` before narrating a module fact. Always provide the active
`scope_id` to `module_query(view="current")` and `module_set_progress`.

## Structured content catalog

For a campaign locked to 2014, the installed `dnd5e.content.srd2014` catalog
provides source-linked class, subclass, species, background, feat, spell, and
item records. Use `rule_pack_query(view="content_catalog")` to discover only the core edition and
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

A discovered physical spellbook is a party or character inventory item, not a
free content grant. Store its resolved `spell_ids` and preserve catalog misses in
`unresolved_spell_names`; unresolved names remain non-executable. During `play`,
use `character_content_apply` with `method="spellbook_copy"`, exact
`source_owner`/`source_item_id`, and explicit exact coin payment. The transaction
includes the 2014 deciphering process, eligibility, payment, elapsed time,
matching actor/world duration expiry, and applicable rule modifiers such as
Evocation Savant. Ordinary `method="spellbook"` grants belong to lobby setup or
level advancement. A failure commits none of those state changes.

## Modules, space evidence, and temporary combat maps

Module re-imports are revisions: earlier sources are retained for snapshots and
scoped scene progress, while normal `module_query(view="index")` results show only the newest
active revision. Re-import orchestration must version parser behavior and restore
the entry tool phase if any staged refresh step fails; a rejected import must not
strand a live campaign in `lobby`. A D&D scene can contain conservative `spatial.locations`
evidence recovered from room headings and stated dimensions. Its optional
`spatial.connections` contains only edges supported by explicit route prose or
reviewed structured authoring, with confidence and source evidence; neither room
number order nor generic room references establish adjacency. Set
`current_location_key` with `module_set_progress` only when it names a location
in the current scene or exactly one spatial location elsewhere in the same
module. This lets an encounter scene reference a separately indexed room scene
without merging their narrative content; ambiguous or cross-module keys fail.
When the location lives in another scene, also persist its exact scene id as
`state.location_scene_id`. A later combat start resolves that recorded scene
first and fails if it is cross-module or does not contain the recorded key; it
must not silently select a different duplicate key elsewhere in the module.

For visual evidence, an owner/DM calls `module_query(view="assets")`, then
`module_page_render` for one page of the imported PDF. The tool returns an MCP
image and registers a content-addressed derived asset. After inspecting that
image, submit only observed edges through `module_set_progress.spatial_review`.
Core validates same-module unique endpoints, the PDF/rendered asset and page,
connection kind, reviewer, and active branch. Returned edges carry
`confidence="reviewed_image"` plus the asset checksum, page, reviewer, and branch
id. The review lives in scoped scene progress, so snapshots and branch checkout
restore it without mutating immutable imported metadata. A review-only write may
omit `status` and `progress`; existing values are preserved. See
`module-visual-atlas.md` for the full sequence and schema.

When a creature card exists only in the PDF image layer, render and inspect its
managed page, then call `module_content_review`. The MCP validates the normalized
2014 statblock before Core stores immutable module/scene/page/asset evidence.
Re-read it with `module_query(view="content")` and create campaign actors with
`character_create_from(mode="module_statblock")`. See
`module-image-content-review.md`; missing text extraction is not evidence that a
printed card is absent, and visual transcription is not permission to fill gaps.

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
`source.encounter_scene_id` retains the active narrative encounter. The current
progress scene remains the source of `current_location_key` and
`state.location_scene_id`, even when the DM supplies a different encounter
`scene_id`. If neither the spatial evidence nor the combat request provides
dimensions, the map falls back to a 12-by-12-cell canvas; clients must not
describe those fallback bounds as module-authored room dimensions.

Call `module_import(action="inspect")` before validation; every later stage uses
the same D&D parser profile. Every write carries a stable stage-specific `idempotency_key`;
an exact retry returns the original result. A player may read `party` scope or
their own authorized `player:<actor_id>` scope only. Keeper scene reads expose
only the redaction marker, and player combat-map views omit blocked cells,
difficult terrain, world patches, checksums, and DM overrides.

## Actor cards and shared party state

| Intent | MCP tool |
|---|---|
| Create from direct/build/template/statblock input | `character_create_from(mode=...)` |
| Read campaign actors, reusable library, or classify a support document | `character_query(get/list/library/document)` |
| Replace a complete reviewed card | `character_sheet_replace` |
| Inventory | `inventory_change(add/update/remove/equip/consume_ammunition)`, `inventory_transfer` |
| Wallet, spell, effects, resources, advancement | `wallet_change(adjust/transfer)`, `character_spell_prepare(set/replace_all)`, `campaign_change(advancement_configure/experience_award/loot_acquire/consumable_use)`, `character_state_change(effect_add/effect_remove/resource_set/rest/level_advance/stable_recovery/stand)` |
| Ability scores | `dnd_ability_roll`, `character_ability_apply` |
| Actor-scoped knowledge | `actor_knowledge_change(add/revise)`, `actor_knowledge_query(list/search)` |
| Shared stash/wallet | `campaign_query(view="party")`, `inventory_change`, `inventory_transfer`, `wallet_change` |

The campaign instance is authoritative. After any actor or party mutation, read
`character_query(view="get")` or `campaign_query(view="party")` and use returned `derived` values. Do not use
`character_sheet_replace` for a one-field mutation.

Before passing an allowlisted PDF/text file to module import, use
`character_query(view="document")` when it may be a character sheet,
pregenerated-PC packet, or ability-score option document. The result stages and
normalizes the artifact, reports its checksum and `document_kind`, preserves
manual-input choices, and explicitly sets `module_import_allowed=false` for
character documents. Never force such a document through the module parser.

For a dead, missing, or departed PC, prefer an applicable unused module
pregenerated character and otherwise create one new legal character through the
same public Lobby tools. A replacement is a distinct actor and must have empty
ActorKnowledge before joining; never duplicate the predecessor's sheet or
knowledge. Preserve the predecessor and its independent ledger. A full-playthrough
driver may transition `play` to `lobby` for this build only through `game_phase`
and must restore the entry phase after either success or failure. Back in `play`,
register the replacement at the manifest's current source-cited Scene Atlas
location with one atomic continuity event. Give the replacement only the joining
fact it witnessed and explicit `told_by` handoff propositions, replace the
predecessor's active manifest slot, retain both actor ids and the handoff event in
replacement history, then create and verify a post-manifest checkpoint.

`inventory_transfer(mode="character_to_character")` mutates two private actor
documents and therefore requires the caller to control both actors (an owner/DM
satisfies both checks), plus current campaign/source/target revisions. A failed
authorization or stale revision moves nothing. Party transfers instead use
`party_to_character` or `character_to_party`; the facade maps those directions
to one atomic shared-stash/actor mutation.

Prepared-spell selection is edition- and class-aware. In `lobby`, use
`character_spell_prepare(mode="replace_all")` with the complete selected list and
`event: setup` or `event: level_up`; `mode="set"` is only a setup edit. In live
`play`, use `character_state_change(action="rest")` and supply the complete
`prepared_spell_ids` list so recovery and a legal long-rest replacement commit atomically.
In 2024, Cleric/Druid/Wizard may replace any number after a Long Rest,
Paladin/Ranger may replace one, and Bard/Sorcerer/Warlock replace one only when
gaining a class level. In 2014, Cleric/Druid/Paladin/Wizard may change their list
after a Long Rest, while Bard/Ranger/Sorcerer/Warlock use spells known. Always-
prepared spells and cantrips never occupy selections. Wizard selections must be
in the spellbook. Multiclass eligibility uses each spell's `grant.source_key`
and that class's own level. Campaign-bound characters inherit campaign edition.

`campaign_create` records `advancement_mode="milestone" | "xp"` in campaign
settings. Use `campaign_change(action="advancement_configure", payload={mode})`
only in `lobby`, outside active combat, with the current campaign revision. A
campaign missing this setting cannot award XP or advance until it is configured.

`campaign_change(action="experience_award")` is DM-authorized and valid only for
PCs in XP mode, outside active combat. Its payload contains nonempty `reason` and
`source_ref`, plus unique `awards` entries with `character_id`, positive `amount`,
and that PC's `expected_revision`; the call also requires the current campaign
revision and branch. All PC totals and a branch-local award record commit
atomically. It returns cumulative thresholds and `eligible`, but never changes a
level. Milestone mode rejects this action.

`campaign_change(action="loot_acquire")` is the play-phase transaction for one
source-defined treasure parcel. Supply a stable branch-local `acquisition_id`,
positive denomination amounts in `coins`, normalized shared-stash `items` with
stable ids, a nonempty reason, and the exact JSON `source_ref` from the selected
module chunk. The Runtime verifies that the chunk belongs to the campaign,
module, and scene and that `content_sha256` matches its expanded text. Currency,
items, and the branch-local acquisition record commit atomically; an exact
idempotent retry returns the original parcel, while a second key cannot reuse the
same acquisition id. Use separate `wallet_change` or `inventory_change` calls
only for genuinely separate in-world transactions, not to decompose one chest.
Regression orchestration may distinguish the occurrence scene/location from the
source scene for a delayed promised reward. The transaction's exact `source_ref`
still identifies the original promise chunk, while continuity records the later
scene and a location that exists in that scene's atlas.

Outside combat, use `campaign_change(action="consumable_use")` to drink one
standard identified `Potion of Healing` from the shared stash. Supply a stable
branch-local `use_id`, the stack `item_id`, target PC id and current character
revision, reason, current campaign revision, and idempotency key. The 2014/2024
Core boundary supplies `2d4+2`; the service rolls it from the campaign random
stream and atomically removes one potion, heals with maximum-HP clamping, stores
the use record, advances the random position, and emits a rule receipt. During
combat, use the combat action path instead; never bypass action economy with this
play-phase transaction. Do not emulate potion use with separate inventory removal,
dice, and healing calls.

`character_state_change(action="level_advance")` is DM-authorized and valid only
in `lobby`, outside active combat. It requires the current actor revision, a fresh
idempotency key, the exact existing `class_name`, `hp_method` (`fixed` or
`rolled`; never provide `hp_roll`), and nonempty `reason` and `source_ref`. In XP
mode it requires the actor's current cumulative XP to meet the next-level
threshold; milestone mode relies on the cited trigger. It currently
advances a 2014 single-class actor exactly one level. The atomic mutation raises
maximum HP without healing current damage, adds the new Hit Die, adds only newly
gained spell-slot capacity to available slots, recalculates preparation maximum,
and applies source-bound per-level HP modifiers from installed content to both
maximum HP and the matching HP-growth ledger entry. Its
`advancement.follow_up` lists eligible feature artifacts, subclass options, and
spell-choice counts. Those are mandatory subsequent catalog operations; after a
subclass choice, query again for its features. Finish with a complete
`character_spell_prepare(mode="replace_all", event="level_up")`, actor re-read,
and snapshot before returning to `play`.

## Branch-aware continuity

World facts, chronology, and subjective actor knowledge are different ledgers.

| Ledger | MCP tool |
|---|---|
| Branches | `branch_query(list/compare)`, `branch_change(create/checkout)` |
| World facts | `memory_change`, `memory_query(list/search)` |
| Events | `campaign_event(add/list)` |
| One actor's belief/knowledge | `actor_knowledge_change(add/revise)`, `actor_knowledge_query(list/search)` |
| Safe retrieved context | `continuity_context` with one shared `budget_chars` limit |
| Atomic post-scene write | `continuity_commit` |
| Continuity health | `continuity_diagnostics` (owner/DM only) |

Pass `branch_id` for an explicit historical branch. For player-safe narration,
use `continuity_context` with the acting `actor_id`, `scope_id`, and audience;
never substitute a broad `memory_query(view="search")` result for that context.

Use the three ledgers deliberately:

- `campaign_event` records what happened. For a witnessed subset, set
  `audience_scope="actor"`, list `known_by_actor_ids`, and set
  `knowledge_disclosure_scope="owner"`; use `party` only when every party member
  may know it.
- `memory_change` records objective world facts worth retrieving later. Prefer
  `action="upsert"` with a deterministic `fact_key`; revising an existing key
  requires its current `expected_revision_id`. Use `supersede` or a revised status
  instead of deleting history. Omitted revision fields remain unchanged.
- `actor_knowledge_change` records one PC or NPC's belief, inference, secret, or
  misinformation. Revising one actor must never revise another actor's ledger.

After a meaningful scene or combat outcome, call one `continuity_commit` with the
event, accepted fact changes, per-actor knowledge changes, and an optional
snapshot. The transaction rolls back as a unit if any actor, revision token, or
fact is invalid. Require a fresh `idempotency_key`; existing fact or knowledge
revisions require their current `expected_revision_id`. The MCP adds the installed
D&D and module-generation Skill SHA-256 manifest to the event payload, and the
snapshot captures that provenance.

Use the individual event, memory, knowledge, and snapshot tools only for isolated
administrative work or when the server does not advertise
`atomic_continuity_commit`; never present a partially completed fallback as a
successful scene save. A snapshot contains a full restorable payload; its recap is
the branch delta. Before a restore call `snapshot_query(view="verify")`; after
restore verify the new head and refresh campaign, characters, module progress,
events, and continuity context.

Use `continuity_diagnostics` to inspect active/inactive ledger counts, orphan
source-event references, unsnapshotted events, latest checkpoint size, recap
provenance, and Skill-manifest drift. It returns health metadata rather than
narrative content. A non-null drift or growing unsnapshotted count is an
operational signal, not permission to rewrite history automatically.

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
agent must open and load its own exposure. A session/principal has exactly one
active exposure. Calling `exposure_open` again replaces the previous exposure;
load multiple compatible same-phase groups into the current exposure and discard
older exposure ids.

| Phase | Intended state | Example groups |
|---|---|---|
| `lobby` | Game-outside setup, imports, campaign administration, character building | player-safe `lobby.characters`, `lobby.memory`; owner/DM `lobby.campaign`, `lobby.rules`, `lobby.modules`, `lobby.memory_control` |
| `play` | Live non-combat exploration and downtime | player-safe `play.scene`, `play.characters`, `play.resolution`; owner/DM `play.scene_control`, `play.combat_control` |
| `combat` | Active structured encounter | player-safe `combat.observe`, `combat.turn`, `combat.actions`; owner/DM `combat.control`, `combat.save`, `combat.map` |

For native clients supporting MCP `tools/list_changed`, loading or unloading a
group changes the actual tool list. If a host cannot refresh its native schemas,
it keeps the core and calls the same loaded domain tool through
`exposure_call(exposure_id, tool_id, arguments)`. It returns the same structured
facade result as a native domain-tool call, not an internal content/result tuple.
This is a transport fallback, not a permission bypass: it performs the identical
session and phase check. For image-producing tools, the fallback keeps the JSON
envelope in the first text/structured-content block and forwards each MCP image
block separately. Read the envelope for provenance and inspect the image block;
never expect base64 image data inside `result` or discard non-text content.

An exposure without `campaign_id` may load only `lobby.bootstrap` (system list
and campaign creation), plus the `system:local`-only storage administration
group. After campaign creation or selection, call `exposure_open` again with the
campaign id. Campaign administration, rules and module-import groups additionally
require owner/DM membership. A campaign-bound exposure rejects arguments that
target a different campaign, including character ids resolved to that campaign.
Objective memory, actor-knowledge writes, snapshot history, state history, rule
receipts, scene progression, combat start/end/join, and map patches are likewise
kept in owner/DM control groups. Do not load a control group merely to make its
tools visible to a player Agent; use the player-safe read group and let actor
authorization govern that Agent's own card and subjective knowledge.

The runtime enforces the same boundary even when a caller bypasses progressive
discovery and invokes a facade directly. In particular, players cannot inspect
snapshot labels or lineage, reversible state history, or settlement receipts.
Non-local reusable-character library reads retain the reusable sheet but omit
private template notes.

Use `game_phase(action="set", tool_profile="lobby" | "play")` only for the
non-combat transition. `combat_start` moves the campaign to `combat` and
`combat_end` returns it to `play`; the server invalidates incompatible loaded
groups. Authorization, revision, idempotency, and engine checks apply even if a
client presents a stale schema.
Direct character-card mutations (sheet replacement, inventory, wallet, effects,
resources, rests, non-combat casts, and activities) are rejected while an
encounter is active. Do not use a profile mismatch to bypass combat action
economy.

## Source-bound actors and scene readiness

Create likely combatants and reinforcements in `lobby`, before combat. For a
creature present in an imported rule source, use
`character_create_from(mode="statblock")` with the imported `source_id` and, when
needed, reviewed `chunk_ids`. The parser preserves source provenance, exact AC,
HP, abilities, defenses, senses, weapon attacks, and structured Multiattack.
For an image-only module card, use the reviewed visual workflow and
`mode="module_statblock"` instead. Current automatic import supports reviewed
English 2014 SRD-style numeric weapon and spell attacks. A spell-only card without
numeric attack facts, 2024, ambiguous, or otherwise unsupported block must
remain unresolved; do not replace it with a similar SRD creature or invent a card.
When a complete statblock action repeats a known spell, the action is authoritative
for that creature. Hydration preserves the Core card's components and provenance
but overlays the displayed effect/range and structured attack resolution together.
Clients must reject a newly prepared actor if those views disagree; do not show a
base-spell range while the engine enforces the statblock override.

Before `combat_start`, call `module_query(view="readiness")` with a manifest:

```json
{
  "schema_version": 1,
  "groups": [
    {
      "key": "dead_three",
      "label": "Dead Three attackers",
      "role": "combatant",
      "required_count": 8,
      "actor_ids": ["canonical campaign actor ids"],
      "source_scene_id": "same-module scene id",
      "source_excerpt": "exact normalized module excerpt, 8 to 500 characters"
    },
    {
      "key": "tavern_reserves",
      "label": "Bribable tavern patrons",
      "role": "reinforcement",
      "required_count": 5,
      "actor_ids": ["canonical campaign actor ids"],
      "source_scene_id": "same-module scene id",
      "source_excerpt": "exact normalized module excerpt, 8 to 500 characters"
    }
  ]
}
```

Required combatants must be present in the initial `participant_ids`.
Reinforcements must not be initial participants. Readiness is false when a
required actor is missing, at 0 HP/Dead, or has unresolved executable card rules;
manual rulings are surfaced but do not falsely disappear. The excerpt must be an
exact normalized substring of that same module scene; it is not a fuzzy query and
a paraphrase or another occurrence of the room label is rejected. `ready=true`
means that the actor may enter combat, not that every card entry is automatic.
`required_count` is the complete branch-local group count established by the
source, a recorded source-table roll, or an explicit DM composition fact. It is
not derived from the current length of `actor_ids`; missing cards must keep the
group unready. If an excerpt names a larger count or additional hostile groups,
manifest the complete composition or record the source-supported branch change
that removed them. Never select a shorter substring merely to evade that count.
`automatic_spell_ids` describes structured effect settlement, while component,
targeting, passive, and on-hit uncertainty can still appear in `manual_rulings` or
`ruling_spell_ids`. A scene offer such as
“pay at least 10 gp, then DC 15 Charisma (Persuasion)” is resolved with an
action-bound `combat_check(action="improvise", ability="persuasion", dc=15)` and
advantage only when the stated offer condition is satisfied. On success call
`combat_join`; the canonical actor enters at the next round boundary with a full
turn. On failure no actor joins.

Rule text retrieval and executable rules are separate. For user documents, use
`rule_import(action="stage")`; it accepts only configured import roots, and every
later action reads only the checksum-addressed MCP-managed artifact. Core performs
the shared PDF/Markdown normalization and records the original checksum, parser
warnings, and per-chunk page ranges. Direct ingestion helpers are internal and are
not part of the public contract. Only the safe declarative IR accepted by
`rule_pack_compile(action="draft" | "from_source")` can settle mechanics;
arbitrary Python, expression evaluation,
network access, and database paths are forbidden. Installation does not enable
a pack. A DM explicitly pins a validated version per branch, and snapshots keep
the exact version/checksum lock. Missing locked versions never fall back to a
newer version. Use `campaign_rules(action="explain")` to audit applied mechanic
ids, citations, and the deterministic fingerprint. Use
`campaign_rules(action="receipts")` for the fingerprint and citations stored
atomically with historical settlements.
For a user-imported executable rule, use
`rule_pack_compile(action="from_source")`: citations
must be imported chunk ids and are resolved server-side to the exact source id,
document checksum, heading path, and page range. Use `character_check` outside
combat and `combat_check` during combat when an enabled `check.before` rule needs
DM-established `rule_facts`.

Treat rule-profile and branch rule-pack changes as campaign writes. First read
`campaign_rules(action="get_profile")`, then pass its latest `campaign_revision`
as `expected_revision` together with a stable `idempotency_key` to
`campaign_rules(action="set_profile" | "set_pack" | "remove_pack")`. Reuse the
same key only for an exact retry; a stale revision requires a fresh read and review.

The base engine is not an implicit fallback. Every new campaign locks either
`dnd5e.core.2014` or `dnd5e.core.2024`, including a fingerprinted coverage list
for the existing combat, movement, reaction, damage, rest, spell, character,
and MCP mutation boundaries. Optional packs layer on top of that core provider.
If a runtime upgrade changes the locked core fingerprint, settlement fails until
the DM explicitly reviews and relocks the campaign profile. During active combat,
first finish the current atomic write, call `snapshot_create`, and require
`snapshot_query(view="verify")` to report that checkpoint valid and current. Then
call `campaign_core_relock` with the exact old Core fingerprint from
`campaign_rules(action="get_profile")`, current branch id, campaign revision,
checkpoint head id, a bounded reason, and a fresh idempotency key. Re-read the
effective profile, add a DM-visible maintenance event, and create/verify a second
snapshot before resuming settlement. This tool changes only the built-in Core
lock; edition, locale, publications, user options, and optional pack activations
remain unchanged. Never use `campaign_rules(action="set_profile")` to bypass this
checkpointed combat path.
Snapshot restore and branch checkout check the saved Core lock before changing
live state. A legacy save without that lock, or a save requiring an unavailable
Core fingerprint, needs an explicit conversion path and is never silently
upgraded.

## Integrity and identity contract

Campaign creation accepts `principal_id`; the server creates an owner membership
for that principal. Platform users must be resolved to stable principal IDs before
calling sensitive tools. Roles and actor grants are checked by MCP, not supplied by
the model as trusted claims.

`campaign_create` and every `character_create_from` mode require a fresh
`idempotency_key`; their entity rows, initial branch/owner membership where
applicable, and replay receipt commit atomically. A replay of build mode
returns its original library template and campaign instance as one pair.

Use `access_grant(scope="campaign")` for campaign roles and
`access_grant(scope="actor")` for explicit PC/NPC control or private-sheet
visibility. A player's `player_name` field is descriptive
only and is never an authorization mechanism.

All campaign-bound granular character writes require the character's current
`expected_revision` and a fresh `idempotency_key`. Inventory transfer additionally
requires `expected_campaign_revision`, `expected_source_revision`, and
`expected_target_revision`.

`continuity_commit`, append-only `campaign_event(action="add")`, every
`memory_change`, and `actor_knowledge_change(action="add")` require a fresh
`idempotency_key`. Existing `memory_change(action="upsert" | "revise" |
"supersede")` targets require the current fact `expected_revision_id`.
`actor_knowledge_change(action="revise")` also requires the current knowledge
`expected_revision_id`; neither token is the character-card revision.

Branch creation/checkout and snapshot restore require the current campaign
revision, active `expected_branch_id`, and a fresh key. Snapshot creation requires
the current campaign revision, current `expected_head_snapshot_id` (use `""` when
the branch has no head), and a fresh key. `state_revision(action="undo" | "redo")`
requires the current `expected_history_sequence` from
`state_revision(action="history")` plus a fresh key.

`branch_change(action="create", payload.checkout=true)` returns both the new branch and the materialized
snapshot; pointer changes and state restoration are one transaction.
The transfer is one mutation group, so one `state_revision(action="undo")` restores every affected
wallet, item stack, and character revision together. Retrying the same key replays
the original result; reusing it with different arguments is an error.

Use `branch_query(view="compare")` before discussing alternate timelines. There is no implicit
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
When a source-bound weapon records additional typed damage, one successful hit
rolls all parts and applies per-type defenses as one simultaneous damage instance.
The result's `damage.roll_parts` preserves every roll. A nonempty
`on_hit_ruling` means damage is committed while the quoted secondary condition or
choice still requires explicit DM settlement; it is not permission to repeat the hit.
For 2014 surprise, first satisfy any imported scene prerequisites that merely
avoid automatic detection, then resolve each hiding actor's canonical Stealth
check and compare the individual results with each opposing creature's passive
Perception. An opponent that notices any threat is not surprised. Surprise is
therefore a per-participant scene fact, not the result of applying the ordinary
half-success group-check rule to the party. `hidden` and `surprised` are distinct
facts. Store the source prerequisite and comparison matrix in an auditable
campaign event before supplying `participant_config.surprised`.
Multiattack is an explicit action choice. `derived.attacks_per_action` represents
the actor's ordinary Attack action (including a real Extra Attack feature); it is
not inflated from a monster's Multiattack card. Pass a canonical
`multiattack_option_id` only when selecting that structured Multiattack and omit
it for one ordinary weapon attack. A descriptive Multiattack without executable
options requires a DM ruling only if selected and must not disable ordinary attacks.
With valid grid positions, `combat_movement(action="move")`
verifies the declared five-foot grid distance and creates an owned
`opportunity_attack` reaction window only when a mover leaves an eligible
hostile's reach; `combat_reaction_attack` settles that window and its attack in
one mutation. Collision, terrain, forced movement, line of sight, unrecorded
triggers, and narrative consequences remain explicit DM choices through
`combat_choice(action="open" | "resolve" | "resolve_defense")`.
`combat_common_action` settles the action payment for Dash, Disengage, Dodge,
Help, Hide, Search, Influence, Study, Utilize/Use an Object, and non-spell Ready
without inventing their narrative result;
`combat_query(view="reactions")` exposes an eligible actor's pending reaction windows.
For a scene-defined skill use that consumes an action, `combat_check` accepts the
skill name as `ability` and one matching `action` payment. A 2014 improvised
action uses `action="improvise"`. The server derives the named skill from the
actor card and rejects caller-supplied proficiency or bonus values. The check
and action payment commit together even when the check fails.

`combat_join` queues an existing canonical campaign actor as a reinforcement for
the next round. The queued actor remains outside `combatants` until the round
boundary, is omitted from player combat views, and cannot act, be targeted, or
participate in reaction geometry early. At the boundary it is inserted by
initiative without changing the actor whose turn was already in progress.
Joining initiative ties require an explicit `tie_breaker`. Create likely scene
participants and their source-bound cards during lobby import, not during an
active encounter.
`combat_end` accepts an optional structured outcome with a bounded public
`summary` and a status of victory, defeat, withdrawal, truce, or interrupted.
It persists that outcome on the final encounter audit. It still refuses to end
while a death-save participant remains dying rather than Dead or Stable.
Medicine stabilization is not a generic narrative check. Call
`combat_check(kind="stabilize", ability="wisdom", target_id=...)`; the server
requires the acting turn, recorded adjacency within 5 feet, and a living target
at 0 HP, then derives DC 10 and the actor card's Medicine modifier. It consumes
the main action whether the check succeeds or fails. Success atomically clears
death-save successes/failures and records Stable without healing; failure leaves
the target unchanged. A client must not supply a replacement DC, proficiency,
bonus, or manual condition patch.
Death saves are discovered separately from ordinary actions. At the start of the
current combatant's turn, `combat_query(view="available_actions", actor_id=...)`
returns only `death_save` when the canonical card is at 0 HP, the encounter grants
death saves, and the actor is neither Dead nor Stable. Call
`combat_check(kind="death_save")`; `ability`, target, client bonus, proficiency,
DC, and `rule_facts` are absent. A successful write marks the turn's save used and
returns the natural roll, tally, and outcome. Do not infer eligibility from a
nonexistent Dying condition. If a rescuer must move into range, resolve every
opportunity-reaction window from that movement before attempting stabilization.
The generic Ready action rejects spell payloads. Use the dedicated spell-ready
transaction instead:

1. `combat_ready(action="ready_spell")` accepts only a spell with an Action casting time. It pays
   the action and the spell slot or other casting resource immediately, replaces
   any prior concentration, and records an explicit perceivable trigger.
2. `combat_ready(action="trigger_spell")` is a DM/owner confirmation that the trigger has
   occurred and opens an owned reaction window. It does not infer trigger truth
   from prose.
3. `combat_ready(action="resolve_spell")` either releases the spell and consumes the
   caster's reaction, or declines that occurrence without spending the reaction.
   Declining rearms the same held spell for a later occurrence before expiry.

For a generic non-spell Ready action, use `combat_common_action(action="ready")`,
then let the DM confirm the trigger with `combat_ready(action="trigger_action")`
and let the actor release or decline with `combat_ready(action="resolve_action")`. Releasing pays
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
`combat_movement`, `combat_end_turn`, `combat_check`, `combat_use_activity`,
`combat_ready`,
`combat_concentration_check`, and `combat_hp_change` commit one
atomic mutation group. Sensitive combat writes require both
`expected_revision` and `idempotency_key`. Player views are filtered by campaign
membership; keeper logs, target mechanics, and rulings are not exposed to
players.
`campaign_query(view="get" | "list")` is also audience-filtered: a non-DM sees
only the whitelisted party/game-phase/clock state, audience-visible world
effects, and the already-redacted combat projection. It cannot be used as a raw
state back door around domain-specific visibility checks.

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

`character_action(action="use_activity")` and `combat_use_activity` work with the normalized
`content.activities`, `content.features`, and `content.feats` cards. They spend
one `resource_key` resource when present, otherwise one limited card use, and
pay the card's action/bonus-action/reaction timing in combat. Card prose,
choices, targeting, and any non-deterministic result are returned for an
explicit DM ruling rather than automatically materialized.
The canonical 2014 Fighter Action Surge id is a narrow Core exception:
`combat_use_activity` consumes its use and atomically grants one current-turn
`extra_action`. It rejects off-turn or twice-on-one-turn activation, and any
unused extra action is cleared when the actor's next turn begins. Its Core receipt
is `dnd5e.core.activity.action_surge`; clients must not edit the turn budget.

`combat_preflight_attack` and `combat_resolve_attack` accept
`multiattack_option_id` for the first attack of a structured Multiattack. The
engine pays the Action once, stores the remaining source-defined weapon/mode
entries in turn state, and rejects substitutions or excess attacks. For a melee
weapon with the Thrown property, `attack_mode` defaults to `melee`; send
`attack_mode: "ranged"` to use its thrown range. The selected mode also determines
whether melee-only modifiers apply.

A successful attack may return `status: pending_reaction` with no damage. The
engine has committed its attack roll, Action/attack payment, ammunition use, Help
consumption, and hidden-attacker reveal, while blocking further actions. The
target reads its owned candidate list through `combat_query(view="reactions")`
and calls `combat_choice(action="resolve_defense")` with that choice id and either
a listed reaction activity id or `decline`. The resolver spends the Reaction only
when used, re-evaluates the stored roll against the structured AC bonus, then
rolls/applies damage at most once. Generic `combat_choice(action="resolve")`
rejects this window. Non-DM reaction reads omit the stored attack plan and raw
mechanical internals. A source-bound Core `Shield` spell candidate additionally
returns `cast_levels`; select its spell id with an explicit `cast_level`. That one
mutation pays the Reaction and canonical casting resource, persists the +5 AC
effect with `turn_start: 1`, re-evaluates the stored attack, and never rolls
damage twice. An unavailable/unprepared spell, exhausted slot, incapacitated
caster, spent Reaction, or edition spell-per-turn conflict removes the candidate.
The attack-hit window does not represent Shield's separate `Magic Missile`
targeting trigger; clients must not synthesize one from prose. A source-bound Core
Magic Missile instead uses `combat_cast_spell(..., target_allocations=[...])`.
Allocations contain `target_id` and `darts`; their total is three at level 1 plus
one for each higher slot. Targets must be current combatants visible to the caster
and within 120 feet on the recorded grid. The cast pays the caster's action and
resource once, then opens an owned `magic_missile_targeted` reaction window for
each target with a legal Shield cast. Resolve each through
`combat_choice(action="resolve_defense")`; no dart is rolled until all such windows
are settled. Active or newly cast Shield negates every dart allocated to that
target. Remaining darts are rolled and applied as separate force-damage instances,
so concentration and 0-HP consequences are per dart. Never merge them into one
damage packet or manually patch HP.

A source-bound structured spell attack uses a two-stage contract. Call
`combat_cast_spell` once without a target declaration. A successful cast pays its
action and casting resource once and returns `status="pending_resolution"`, an
opaque `resolution_id`, `attack_count`, and `remaining_attacks`. For each attack,
call `combat_resolve_attack` with the chosen `target_id` and
`action={"spell_resolution_id": resolution_id}` using the latest campaign
revision. The engine derives the spell attack bonus, range, damage, and critical
dice from the source-bound card; the per-attack calls do not pay another action or
slot. A pending Shield defense is resolved through its actor-owned reaction window
before the next attack. Pending spell attacks block `combat_end_turn` and
`combat_end`; both become legal only after `remaining_attacks` reaches zero. Never
model the attacks as weapon actions, repeat the cast, combine damage packets, or
patch HP.

`character_state_change(action="rest")` applies v2-card short/long-rest recovery with a character
revision and idempotency key. For a Short Rest, provide each spent hit die and
its rolled result through `hit_dice_spends`; the runtime applies Constitution
and the edition's minimum. A 2014 Long Rest may require an explicit
`hit_dice_recovery` allocation across multiclass pools. A 2024 Long Rest restores
all expended Hit Dice; exhaustion falls by one. In 2014 exhaustion recovery needs
the DM-confirmed `food_and_drink=true` condition. Timed card effects advance at
the ending actor's turn; any narrative rest consequence remains a DM ruling. A
resource marked `recovers_on: short_rest` also recovers on a Long Rest; the marker
means the earliest rest that restores it, not that a longer rest fails to do so.

A Stable creature at 0 HP cannot benefit from a rest. When the party can safely
wait for the automatic recovery, call
`character_state_change(action="stable_recovery")`. The engine rolls `1d4` hours,
restores exactly 1 HP, clears Stable and Unconscious, preserves unrelated
conditions such as Prone, and stores the Core rule receipt atomically. Do not
manually set HP or choose the recovery duration. A recovered, conscious actor
above 0 HP may then use `character_state_change(action="stand")`; this narrowly
clears Prone under the Core movement boundary and does not permit arbitrary
condition edits.

Except for source-bound spell workflows such as Core Magic Missile,
`character_action(action="cast_spell")` and `combat_cast_spell` settle only timing, casting
resources, concentration, and recorded components. Generic spells return
`pending_ruling` for targets and effects. Cantrips and rituals cannot be upcast; a ritual cannot
complete in active combat. Costly or consumed material components require
`component_ruling.material_confirmed=true` before resources are spent. Pact Magic
uses the recorded `pact_magic.slot_level` and is counted as a slot expenditure.
A custom source-bound statblock spell whose component details were not present in
the reviewed card requires `component_ruling.source_components_confirmed=true`
before it pays an action, slot, or concentration. Confirm this only from an
explicit DM ruling or an active exact spell rule; the later `pending_ruling`
still covers targets and effects.

`module_set_progress` requires the current `expected_state_version` for that
scene/scope row (`0` for its first write) and a fresh idempotency key.

`campaign_change(action="clock_set")` establishes the branch-local campaign day,
hour, and minute. `campaign_change(action="clock_advance")` advances an explicit
`minute`, `hour`, `day`, `round`, or `encounter` count. Narrative-time advances
update the snapshotted `state.world_time` and settle matching effect durations
across all campaign actors and `state.world_effects` as one atomic group;
round/encounter advances do not
move the world clock. The clock must be set first, cannot change during active
combat, and conversation time is never treated as elapsed campaign time. Once
set, a different `clock_set` value is rejected; use `clock_advance`.

`campaign_change(action="party_rest")` is the only long-rest write. Its
`members` array contains `character_id`, that actor's `expected_revision`, and
optional `prepared_spell_ids`, `hit_dice_recovery`, and `food_and_drink` choices.
`duration_minutes` defaults to 480 and cannot be lower. The MCP advances time and
all timed actor/world effects once, then settles the named actors and records
their completion minute in the same mutation. It rejects 0-HP/dead starters and
a second benefit less than 1,440 minutes after the previous one. Individual
`character_rest` remains the short-rest surface and rejects `long_rest`.

Use `campaign_change(action="effect_add" | "effect_remove")` for a structured
effect on a campaign, scene, location, or object. Each effect has a stable id,
source, target, active flag, duration period/remaining count, and visibility
`public|party|dm`. Timed effects require an established campaign clock. Do not
store a timed Light, hazard, ward, weather effect, or similar object state only
inside arbitrary scene-progress JSON, because that bypasses the duration engine.

## Player-safe module reads

`module_query(view="scene" | "index")` and `module_search` accept `principal_id`.
DM/owner reads may include keeper content; player reads are filtered to `public` or
`party` visibility and keeper content is replaced with a redaction marker. A
player cannot select another player or group scope merely by knowing its ID.
