---
name: dnd-dm
description: "Run D&D 5e 2014 or 2024 sessions through the SagaSmith D&D MCP server."
---

# D&D Dungeon Master

## Runtime

This full skill is MCP-first. Start with `storage_status`, then
`exposure_open` for the active campaign; search, inspect, and load only the
needed `lobby`, `play`, or `combat` capability group. Hosts that cannot refresh
native tool schemas must invoke a loaded domain tool through `exposure_call`.
All raw tool names below may be prefixed by the host, for example
`mcp_sagasmith_dnd_`.
For user rulebook import, also require the structured/source-bound flags from
`server_capabilities.rulebook_import` before exposing that workflow.
If the server is unavailable, stop using this skill and load `standalone/` rather
than switching to a local CLI.

Read `../../references/mcp-contract.md` before a mutation and
`references/DM_RULES.md` before a session. Load these only when needed:

- actor creation or advancement: `references/CHAR_CREATION.md`
- actor, items, wallet, spells, effects, or resources:
  `../../references/character-schema-v2.md`
- module preparation or scene transitions: `references/MODULE_INDEX.md` and
  `references/MODULE_ARC.md`
- user PDF/Markdown rulebook import: `../../references/rulebook-import.md`
- catalogued core or extension character options:
  `../../references/content-catalog.md`
- tactical positioning or reusable narration: `references/DM_MAP_SYS.md` and
  `references/DM_TEMPLATES.md`

## Turn Loop

Outside play, select `lobby` with `game_phase(action="set")` for module writing/import,
campaign setup, and character creation. Before the first in-character scene,
switch to `play`. `combat_start` enters `combat` automatically and `combat_end`
returns to `play`; never simulate a phase transition only in narration.
Before module import or character building, read the campaign rule profile and
explain output. The locked `dnd5e.core.2014` or `dnd5e.core.2024` provider must
match the adventure/table edition; do not let a default edition silently define
the campaign.

1. Resolve `scope_id` (`party`, `group:<id>`, or `player:<id>`), then call
   `module_query(view="current")`. Player scopes inherit party progress until they have their own.
2. Read that scene through `module_query(view="scene")`. Use `module_search` only to select a
   candidate, then call `module_expand` before relying on a chunk.
3. Ask for intent when it is ambiguous. Never reveal unseen rooms, future twists,
   hidden motives, or sibling-branch facts.
4. Use `rule_search` then `rule_expand` for disputed or edition-sensitive rules.
5. Imported rulebook text is evidence, not executable mechanics. In `lobby`,
   use `rule_document_stage` -> `rule_document_inspect` ->
   `rule_document_import`, then search/expand the exact source. Draft imported
   mechanics with `rule_pack_draft_from_source`, install it inactive, show the DM its report,
   and enable an exact version only with explicit DM approval. Never change the
   lock during combat or silently substitute a missing version.
   `campaign_rules_explain` must also show the locked `dnd5e.core.2014` or
   `dnd5e.core.2024` provider; treat a missing or mismatched core fingerprint as
   a hard stop, not permission to bypass the existing engine boundaries.
   For an imported module PDF, also require every preview scene to carry a valid
   source page range. A parser profile/version change is a new normalized module
   revision even if the PDF checksum is unchanged; rerun the full staged import
   lifecycle and review the resulting index before play.
6. For character options, call `content_catalog_list` and present only entries
   available to the campaign's locked Core edition and enabled branch packs.
   Apply only a returned id through `character_content_apply`; respect a
   `pending_ruling` response for unresolved prerequisites or effects. Supply
   the legal spell source class and grant method, the target base class for a
   multiclass subclass, every required background/species choice, and every
   class/subclass feature whose minimum level is met. A class or species name
   without its granted feature cards and traits is an incomplete actor, not a
   usable shortcut. Never patch the raw sheet to bypass selection validation.
7. Resolve openly with `dnd_dice_roll` or `dnd_check`.
8. Persist events, scene progress, actor/party state, and durable facts. Use
   `actor_knowledge_*` for what one PC/NPC believes, not `memory_*`.
9. Call `snapshot_create` at decision points, chapter transitions, and before a
   dangerous restore. Use `snapshot_verify` and `snapshot_lineage` before restore.

## MCP Tool Reference

| Workflow | MCP tools |
|---|---|
| Campaign | `campaign_create`, `campaign_get`, `campaign_list` |
| Rules | `rule_document_stage`, `rule_document_inspect`, `rule_document_import`, `rule_ingest`, `rule_search`, `rule_expand`, `rule_pack_draft`, `rule_pack_draft_from_source`, `rule_pack_install`, `rule_pack_list`, `rule_pack_inspect`, `rule_pack_test`, `rule_pack_remove`, `campaign_rule_profile_get`, `campaign_rule_profile_set`, `campaign_rule_pack_set`, `campaign_rule_pack_remove`, `campaign_rules_explain`, `campaign_rule_receipts`, `content_catalog_list`, `character_content_apply`, `character_rule_artifact_add` |
| Module lifecycle | `module_import(stage/inspect/validate/ingest/activate)`, `import_query`, `module_query(list/index)` |
| Scene play | `module_query(current/scene/progress)`, `module_search`, `module_expand`, `module_set_progress` |
| Rolls | `dnd_dice_roll`, `dnd_check`, `dnd_ability_roll`, `character_check` |
| World continuity | `event_add`, `event_list`, `memory_add`, `memory_search` |
| Actor continuity | `actor_knowledge_add`, `actor_knowledge_revise`, `actor_knowledge_list`, `actor_knowledge_search`, `continuity_context` |
| Saves and audit | `snapshot_create`, `snapshot_list`, `snapshot_verify`, `snapshot_lineage`, `snapshot_restore`, `state_history`, `state_undo`, `state_redo`, `campaign_advance_effects` |
| Combat | `combat_start`, `combat_status`, `combat_available_actions`, `combat_preflight_attack`, `combat_resolve_attack`, `combat_move`, `combat_stand`, `combat_common_action`, `combat_use_activity`, `combat_cast_spell`, `combat_ready_spell`, `combat_readied_action_trigger`, `combat_readied_action_resolve`, `combat_readied_spell_trigger`, `combat_readied_spell_resolve`, `combat_reactions`, `combat_reaction_attack`, `combat_end_turn`, `combat_check`, `combat_concentration_check`, `combat_apply_damage`, `combat_heal`, `combat_map_patch`, `combat_end` |
| DM choices | `combat_choice_open`, `combat_choice_resolve` |

## Actor Cards and Party State

Every live PC, NPC, and monster is an authoritative v2 actor card. Use
`character_get` after every write. Use granular tools instead of replacing a whole
sheet for a small change:

```text
character_inventory_add | character_inventory_update | character_inventory_remove
character_inventory_transfer | character_inventory_equip | character_ammunition_consume
character_wallet_adjust | character_spell_prepare | character_spell_prepare_list | character_effect_add
character_effect_remove | character_resource_set | character_memory_add
character_memory_resolve | character_ability_apply | character_rest | character_cast_spell | character_use_activity
party_show | party_inventory_add | party_inventory_remove | party_inventory_transfer
party_wallet_adjust | party_wallet_transfer
```

Use `character_build` for a PC when a library template and its first campaign
instance must be created atomically. Always supply a stable idempotency key so a
transport retry replays that same pair. Use `character_library_list` and
`character_instantiate` for existing templates. `character_memory_*` stays a legacy
notes field; new subjective information belongs in the actor-knowledge ledger.

After item writes, treat `character_get(...).derived.inventory.weapon_attacks` and
`character_get(...).derived.inventory.encumbrance` as authoritative. Represent one
active concentration spell as one active effect with `concentration: true` and its
`source_spell_id`.

During lobby setup or level advancement, submit the complete level 1+ prepared list
through `character_spell_prepare_list`; use the singular prepare tool only for a
setup edit. During live play, a prepared-list change must be part of
`character_rest(..., rest_type="long_rest", prepared_spell_ids=[...])`. Do not
simulate a long rest by repeated toggles. The runtime enforces 2014/2024 class
timing and replacement count, class-level spell eligibility, Wizard spellbook
membership, always-prepared and cantrip exclusions, and multiclass
`grant.source_key` ownership. An unprepared level 1+ spell on a prepared caster's
card is not castable merely because its access record says it is known.

## Combat boundary

Use `combat_preflight_attack` before every attack commit. The engine automatically
settles initiative, turn resources, canonical weapon attack data, attack nat-1/
nat-20, damage dice and typed trait ordering, temporary HP, concentration save
windows, healing, movement budget, and death saves. Surprise is an explicit scene
fact and follows the selected 2014/2024 ruleset. It does not infer missing map
geometry, line of sight, cover, hidden targets, or story consequences. It may
create an opportunity-attack window only from recorded positions, reach,
hostility, visibility, and movement mode.
Open a choice window for those decisions and resolve it explicitly; never encode
an unverified ruling as a character-card fact.

For a structured monster Multiattack, pass `multiattack_option_id` on its first
`combat_preflight_attack` and `combat_resolve_attack`, then make only the exact
weapon/mode/count sequence still recorded by that option. Do not declare a raw
`attacks_per_action` override. A melee weapon with the Thrown property remains a
melee attack by default; pass `attack_mode: "ranged"` when it is actually thrown.
This distinction controls reach, range, disadvantage, and melee-only modifiers.

When an attack returns `status: pending_reaction`, no damage has been rolled or
applied. The target actor reads its owned window with
`combat_query(view="reactions")`, chooses a listed defense or `decline`, and calls
`combat_choice(action="resolve_defense")` with the returned choice id and current
campaign revision. Only that resolution spends the Reaction when used, updates AC
for the stored attack roll, and then resolves damage if the attack still hits.
Never roll damage early, manually patch HP, or use generic choice resolution for
this window. The same sequence applies when an opportunity attack opens a
post-hit defense. A source-bound `Shield` spell appears as a spell candidate only
when it is prepared/available, has a legal casting resource, and is legal under
the current turn's 2014/2024 spell limit. Select one of its returned
`cast_levels` by sending both its spell id and `cast_level`; the same transaction
spends the Reaction and slot, applies +5 AC to the triggering roll, and records
the +5 AC effect until the start of that caster's next turn. Do not model Shield
as an activity or add its AC manually.

A source-bound Core `Magic Missile` is the exception to generic spell
`pending_ruling`. Call `combat_cast_spell` with `target_allocations`, where every
entry supplies `target_id` and a positive `darts` count; the total must be three
at level 1 plus one per higher slot. The server validates current map distance and
recorded visibility. If a target can cast Shield, the cast returns
`pending_reaction` before any dart is rolled or damage applied. That target reads
its owned reaction and uses `combat_choice(action="resolve_defense")` with Shield
and one returned `cast_level`, or `decline`. The server settles all target choices,
then rolls and applies every unblocked dart as a separate force-damage instance;
each dart can cause its own concentration save or 0-HP failure. An already-active
Shield blocks that target's darts without spending another Reaction. Never roll
darts externally, combine them into `combat_hp_change`, or forge an attack-hit
window for this targeting trigger.

Declare 2014 Sneak Attack with `use_sneak_attack: true`; the engine checks the
recorded Rogue feature, finesse/ranged weapon, advantage or adjacent active enemy,
disadvantage, once-per-turn token, and critical dice. For Second Wind, first use
`combat_use_activity` to pay the bonus action and feature use, then roll `1d10 +
fighter level` and pass that exact total to `combat_hp_change(action=heal)`.
For levelled spell healing, also pass `source_actor_id`, `spell_id`, and the
actual `spell_level`; this lets the engine validate the actor card and settle
source-linked features such as 2014 Life Domain's Disciple of Life. Never fold
that feature bonus into the base amount yourself. Halfling Lucky is
resolved automatically for attacks, checks, saves, death saves, and initiative;
retain its `rerolls` audit instead of rolling a second untracked check.

To stabilize a dying creature with Medicine, call
`combat_check(kind="stabilize", ability="wisdom", target_id=...)`. The MCP
requires the current actor's turn, both creatures in the encounter, recorded map
positions within 5 feet, and a living target at exactly 0 HP. It derives the
fixed DC 10 and the actor card's complete Medicine modifier; do not pass a client
`bonus`, `proficient`, or alternate DC. The same atomic transaction spends the
main action. On success it resets the target's death-save tally and records
Stable while preserving 0 HP, Unconscious, and existing conditions such as
Prone; on failure it spends the action without changing the target. Do not patch
conditions or death saves by hand. Use Spare the Dying only when that exact
source spell is present and castable on the actor card; never grant it merely
because stabilization is needed.

Preserve the source spell card's canonical casting time during import. Standard
cards commonly use `1 action`, `1 bonus action`, or `1 reaction, ...`; do not
strip the leading count or replace these with a free-form timing label. In
combat, `combat_cast_spell` maps that card timing to the actor's matching action
budget. A bonus-action spell spends only the bonus action and leaves the main
action available, subject to the edition's same-turn spell restrictions; a
reaction spell still requires its owned pending reaction window.

Use `combat_common_action` for the core non-attack actions. It records their
action payment and tactical state; it deliberately does not fabricate the
outcome of a Hide, Search, or Help declaration. At encounter start, provide
DM-authored `participant_config` positions, disposition, reach, initiative, and
visibility (`hidden` and `visible_to_actor_ids`) when those facts are known. A
current module scene produces a frozen temporary battle map. An encounter scene
may use `current_location_key` to reference exactly one spatial location in
another scene of the same module; persist that source scene as
`state.location_scene_id`, and preserve the current progress, encounter, and
spatial source ids as separate evidence. If the spatial evidence states no room
dimensions and the DM supplies no bounds, the temporary map uses a conservative
12-by-12-cell canvas; this is workspace, not inferred room geometry. The map
enforces only its explicit bounds and blocked cells; it never turns
room prose into inferred walls, cover, line of sight, or terrain. Record a real door, hazard, or similar
post-combat world change through `combat_map_patch`, not by rewriting the module.
Player map views intentionally omit blocked cells, difficult terrain, DM
overrides, checksums, and world patches; do not disclose those fields from a DM
read or an earlier tool result.
A grid move that leaves an eligible
hostile's recorded reach opens an owned opportunity window; read it through
`combat_reactions`, decline it with `combat_choice_resolve`, or settle it
atomically with `combat_reaction_attack`. Do not claim map collision, terrain,
forced movement, line of sight, or a trigger not represented in encounter state.
Use `combat_move.path` for bent grid routes. Set `movement_mode` to `forced` or
`teleport` when the scene establishes that the move does not provoke a normal
opportunity attack; do not encode terrain cost or collision unless it is part of
the supplied scene facts.
If Prone, either use `combat_stand` (half speed, no action) or use
`combat_move(..., crawl=true)`; crawling costs double movement.

Every campaign, character, party, combat, rest, continuity, branch, snapshot,
scene-progress, and actor-knowledge mutation must carry the current optimistic
token exposed by that tool and a fresh `idempotency_key`. The token may be a
campaign/character revision, actor-knowledge revision ID, branch/head ID, history
sequence, or scene `state_version`; read the MCP contract for the exact field.
Re-read the relevant state after a conflict; never retry a changed payload under
the same key. Shared wallet and
inventory adjustments are campaign writes and follow the same contract.

Use `combat_use_activity` or `character_use_activity` for cards in
`content.activities`, `features`, or `feats`. These tools pay a recorded use or
resource and the activation timing, then return `pending_ruling` when the card
has choices; they never infer a result from prose.

Reaction spells and activities require an owned pending reaction window. Do not
call them solely because it is another actor's turn. Do not hide a spell inside
the generic Ready payload. For a spell with an Action casting time, call
`combat_ready_spell`: it pays the action and spell slot or other casting resource
immediately, replaces any existing concentration, and arms one perceivable
trigger until the start of the caster's next turn. The DM confirms that the
trigger occurred with `combat_readied_spell_trigger`. The caster then uses
`combat_readied_spell_resolve` to release the spell with its reaction or decline
that occurrence without spending the reaction; declining leaves the spell armed.
Losing concentration, reaching the caster's next turn, or ending combat makes the
held spell dissipate without effect. A released spell returns `pending_ruling`:
resolve its targets, attack, save, damage, area, and narrative consequences with
the appropriate combat tools and DM ruling rather than treating release as the
spell's complete effect.

Use `campaign_advance_effects` only after the DM establishes an elapsed
minute/hour/day (or explicit round/encounter boundary). It advances matching
canonical effect durations across all campaign actors atomically; it does not
invent passage of time from chat pacing.
