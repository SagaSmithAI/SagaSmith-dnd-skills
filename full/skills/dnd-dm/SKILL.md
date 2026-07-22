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
- module maps, diagrams, or missing topology:
  `../../references/module-visual-atlas.md`
- image-only module creature cards:
  `../../references/module-image-content-review.md`
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
5. Imported rulebook text is evidence, not executable mechanics. In `lobby`, use
   `rule_import` in order: `stage` -> `inspect` -> `ingest` ->
   `extract_candidates` -> `review` -> `compile` -> `install` -> `activate`, then
   search/expand the exact source. Compile a separate source-bound mechanic with
   `rule_pack_compile(action="from_source")` only after review, install it inactive,
   show the DM its report, and enable an exact version only with explicit DM approval. Never change the
   lock during combat or silently substitute a missing version.
   `campaign_rules(action="explain")` must also show the locked `dnd5e.core.2014` or
   `dnd5e.core.2024` provider; treat a missing or mismatched core fingerprint as
   a hard stop, not permission to bypass the existing engine boundaries.
   For an imported module PDF, also require every preview scene to carry a valid
   source page range. A parser profile/version change is a new normalized module
   revision even if the PDF checksum is unchanged; rerun the full staged import
   lifecycle and review the resulting index before play.
6. For character options, call `rule_pack_query(view="content_catalog")` and present only entries
   available to the campaign's locked Core edition and enabled branch packs.
   Apply only a returned id through `character_content_apply`; respect a
   `pending_ruling` response for unresolved prerequisites or effects. Supply
   the legal spell source class and grant method, the target base class for a
   multiclass subclass, every required background/species choice, and every
   class/subclass feature whose minimum level is met. A class or species name
   without its granted feature cards and traits is an incomplete actor, not a
   usable shortcut. Never patch the raw sheet to bypass selection validation.
7. Resolve openly with `dnd_dice_roll` or `dnd_check`.
8. Persist resolved scene continuity with one `continuity_commit`: one event,
   stable-key objective fact changes, exact per-actor knowledge changes, and an
   optional snapshot. Never infer who knows a fact from the fact itself.
9. Use an administrative `snapshot_create` only when no scene continuity unit is
   being written, such as immediately before a dangerous restore. Use
   `snapshot_query(view="verify" | "lineage")` before restore.

## MCP Tool Reference

| Workflow | MCP tools |
|---|---|
| Campaign | `campaign_create`, `campaign_query`, `campaign_change`, `access_grant` |
| Rules | `rule_import`, `import_query`, `rule_search`, `rule_expand`, `rule_pack_compile`, `rule_pack_query`, `rule_pack_change`, `campaign_rules`, `character_content_apply` |
| Module lifecycle | `module_import(stage/inspect/validate/ingest/activate)`, `import_query`, `module_query(list/index/assets/content/candidates/readiness)`, `module_page_render`, `module_content_review` |
| Scene play | `module_query(current/scene/progress)`, `module_search`, `module_expand`, `module_set_progress` including `spatial_review` |
| Rolls | `dnd_dice_roll`, `dnd_check`, `dnd_ability_roll`, `character_check` |
| World continuity | `continuity_commit`, `campaign_event`, `memory_change`, `memory_query` |
| Actor continuity | `actor_knowledge_change`, `actor_knowledge_query`, `continuity_context` |
| Saves and audit | `snapshot_create`, `snapshot_query`, `snapshot_restore`, `branch_query`, `branch_change`, `state_revision` |
| Combat | `combat_start`, `combat_join`, `combat_query`, `combat_preflight_attack`, `combat_resolve_attack`, `combat_movement`, `combat_common_action`, `combat_use_activity`, `combat_cast_spell`, `combat_ready`, `combat_reaction_attack`, `combat_end_turn`, `combat_check`, `combat_concentration_check`, `combat_hp_change`, `combat_map_patch`, `combat_end` |
| DM choices | `combat_choice(open/resolve/resolve_defense)` |

## Actor Cards and Party State

Every live PC, NPC, and monster is an authoritative v2 actor card. Use
`character_query(view="get")` after every write. Use granular facade tools instead of replacing a whole
sheet for a small change:

```text
inventory_change | inventory_transfer | wallet_change
character_spell_prepare | character_state_change | character_action
character_metadata_update | character_content_apply | character_ability_apply
campaign_query(view="party")
```

Use `character_create_from(mode="build")` for a PC when a library template and its first campaign
instance must be created atomically. Always supply a stable idempotency key so a
transport retry replays that same pair. Use `character_query(view="library")` and
`character_create_from(mode="template")` for existing templates. New subjective
information belongs in the actor-knowledge ledger.

For ability generation, `manual` preserves explicitly entered scores without
claiming a dice audit, while `standard_array` and `point_buy` enforce their rule
budgets. Rolled generation is two-phase through `character_ability_apply`: omit
assignments to let the engine persist the six 4d6-drop-lowest results, then assign
those exact results under the returned character revision. Never send `rolls` or
reroll a pending set. See `references/CHAR_CREATION.md` for the complete contract.
When presenting character creation choices, keep all four paths visible; never
remove `manual` merely because a rules-validated alternative is available.

For a module NPC or monster, use the exact imported standard statblock or an
immutable reviewed image card. If the module explicitly changes only HP, AC,
creature type, languages, or an action, pass a source-cited `variant` to the
statblock creation call. Never rebuild or raw-patch the full sheet for a small
module instance change. A type replacement such as beast to undead must use
`creature_type` and cite the exact managed module chunk or review that says so.

After item writes, treat `character_query(view="get").derived.inventory.weapon_attacks` and
`character_query(view="get").derived.inventory.encumbrance` as authoritative. Represent one
active concentration spell as one active effect with `concentration: true` and its
`source_spell_id`.

When a module yields a found spellbook, add one `kind="spellbook"` inventory item
for each physically distinct book. Preserve its edition, exact source scene/key,
copyability, owner mark, resolved catalog `spell_ids`, and
`unresolved_spell_names`. A name absent from the active content catalog stays
unresolved and non-executable; never drop it, substitute a similar spell, or
fabricate an artifact id. Record discovery through `continuity_commit`, with the
objective item fact and separate ActorKnowledge entries only for actual witnesses.

Discovery does not add spells to a Wizard's personal spellbook. During `play`,
copy exactly one returned spell id with `character_content_apply` using
`method="spellbook_copy"`, the source owner/item id, payment owner, and an exact
coin payment. The runtime validates Wizard/class-level eligibility, performs the
2014 decipher-and-copy process, advances time and all timed actor/world effects,
and applies source-bound discounts such as Evocation Savant. It does not invent
currency exchange or change. A missing source, unresolved spell name,
insufficient exact payment, unavailable Core lock, or failed validation must
leave character, wallet, clock, inventory, and effects unchanged.

During lobby setup or level advancement, submit the complete level 1+ prepared list
through `character_spell_prepare(mode="replace_all")`; use `mode="set"` only for a
setup edit. During live play, a prepared-list change must be part of
`character_state_change(action="rest", rest_type="long_rest", prepared_spell_ids=[...])`. Do not
simulate a long rest by repeated toggles. The runtime enforces 2014/2024 class
timing and replacement count, class-level spell eligibility, Wizard spellbook
membership, always-prepared and cantrip exclusions, and multiclass
`grant.source_key` ownership. An unprepared level 1+ spell on a prepared caster's
card is not castable merely because its access record says it is known.

A rest benefit and elapsed narrative time are separate writes. First establish
that the uninterrupted rest actually occurred and advance the branch-local clock
by its source-required duration; only then call
`character_state_change(action="rest")` for each eligible actor. For a short
rest, submit `hit_dice_spends=[{"key": <recorded hit-die key>, "count": <positive
integer>}]`. Never submit a die result: the engine validates the available pool,
rolls every spent die, adds the card's Constitution modifier, and returns
`hit_dice_rolls` for audit. Long rests reject `hit_dice_spends`; short rests
reject long-rest hit-die recovery allocations and `food_and_drink`. A creature
at 0 HP or Dead receives no rest benefit.

If a Wizard chooses Arcane Recovery at the end of that short rest, include
`arcane_recovery={"<slot level>": <count>}` in the same rest call. The engine
requires the recorded feature, limits the combined recovered slot levels to half
the Wizard level rounded up, forbids level 6+ slots, records the use, and restores
only actually missing slots. For 2014, this is once per
campaign day, not once per long rest: the MCP requires the branch-local clock,
records the last-used day on the feature, and a long rest does not reset it. Do
not apply the rest first and patch spell slots afterward.

Level advancement is a `lobby` transaction, not a sheet replacement. Preserve
the exact award evidence and call
`character_state_change(action="level_advance")` with the existing class, fixed
or rolled HP method, `reason`, and `source_ref`. Never provide `hp_roll`: for the
rolled method the engine rolls the class Hit Die after idempotency, revision,
content, and rules checks and returns the roll in `advancement.hit_points.roll`.
Current HP is not healed.
Then exhaust `advancement.follow_up`: apply eligible class features, resolve any
subclass and spell choices from the active content catalog, apply newly eligible
subclass features, replace the complete prepared list with `event="level_up"`,
re-read derived state, and snapshot before returning to `play`. Follow the full
ordering and stop conditions in `references/CHAR_CREATION.md`.

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

For a weapon with a recorded `ammunition_item_id`, attack preflight also requires
at least one unit in that exact linked ammunition stack. The commit consumes one
only after the declaration is otherwise valid; the final shot leaves the
auditable stack at quantity 0 so the weapon link remains valid. Do not substitute
another stack, delete the empty linked item, or roll before successful preflight.
An actor can always explicitly select `weapon_id: "unarmed-strike"`, including
while holding a weapon whose ammunition is exhausted. The Core attack is
proficient, uses Strength, has 5-foot reach, and deals `1 + Strength modifier`
bludgeoning damage. Do not require the actor to delete or unequip a weapon first.

A source-bound weapon can carry multiple simultaneous typed damage parts. Let the
engine roll every recorded part and apply resistance, immunity, and vulnerability
per type as one hit; never collapse them into one type or manually add the second
part. If the same hit also returns `on_hit_ruling.required`, the damage is already
committed but the quoted secondary effect is still a DM boundary. Resolve that
effect explicitly and do not silently omit it or apply the damage again.

Multiattack is a distinct action choice. For a structured monster Multiattack,
pass `multiattack_option_id` on its first `combat_preflight_attack` and
`combat_resolve_attack`, then make only the exact weapon/mode/count sequence still
recorded by that option. Omit the id to choose one ordinary Attack. A descriptive
Multiattack without options remains a DM boundary only when selected and does not
block an ordinary weapon attack. Do not declare a raw `attacks_per_action`
override. A melee weapon with the Thrown property remains a
melee attack by default; pass `attack_mode: "ranged"` when it is actually thrown.
This distinction controls reach, range, disadvantage, and melee-only modifiers.
On a positioned combat map, a ranged attack without a recorded normal range is a
DM-ruling boundary, never permission to skip distance enforcement. Repair the
source-grounded card in lobby when the source states the range; otherwise choose
a legal recorded mode such as melee or preserve the ruling explicitly.

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

For a source-bound structured spell attack such as `Scorching Ray`, call
`combat_cast_spell` exactly once. That transaction pays the casting action and
spell resource once and returns `status="pending_resolution"`, a
`resolution_id`, and the authoritative attack count. Select each target only when
calling `combat_resolve_attack`; pass
`action.spell_resolution_id=<resolution_id>` and a freshly read campaign revision
once for every remaining attack. Never call `combat_cast_spell` per ray, substitute
a weapon id, roll damage externally, or patch HP. A hit can still open an owned
Shield reaction; resolve it normally before attempting the next attack. The
pending spell resolution blocks the caster's turn end and encounter end until its
remaining count reaches zero.

Declare 2014 Sneak Attack with `use_sneak_attack: true`; the engine checks the
recorded Rogue feature, finesse/ranged weapon, advantage or adjacent active enemy,
disadvantage, once-per-turn token, and critical dice. The canonical 2014 Fighter
Second Wind card is engine-owned: call `combat_use_activity` with its exact
feature id. One atomic transaction consumes the card use and bonus action, rolls
`1d10 + fighter level`, applies the clamped healing, and returns the roll,
before/after HP, applied amount, and Core receipt. Never roll it externally or
follow it with `combat_hp_change`; that would double-settle the feature. For
levelled spell healing, pass `source_actor_id`, `spell_id`, and the
actual `spell_level`; this lets the engine validate the actor card and settle
source-linked features such as 2014 Life Domain's Disciple of Life. Never fold
that feature bonus into the base amount yourself. Halfling Lucky is
resolved automatically for attacks, checks, saves, death saves, and initiative;
retain its `rerolls` audit instead of rolling a second untracked check.
The canonical 2014 Fighter Action Surge card is engine-owned: call
`combat_use_activity` with its exact feature id on the Fighter's turn. The same
transaction consumes one use and grants one `extra_action`; it returns
`committed`, not `pending_ruling`. Use the returned action normally. An unused
extra action expires at the next turn and Action Surge cannot be activated twice
on the same turn. Do not patch `turn_budget` or invent another Attack.

The canonical 2014 Rogue Cunning Action card is also engine-owned. Call
`combat_use_activity` with its exact feature id and a declaration whose `action`
is `dash`, `disengage`, or `hide`. Dash spends the bonus action and adds the
actor's recorded Speed to remaining movement; Disengage spends it and records
the no-opportunity-attack turn flag. Hide spends the bonus action and records a
source-linked Hide declaration, but remains `pending_ruling`: the DM must still
decide whether the circumstances permit hiding and resolve the Stealth/observer
boundary. Never spend a second main action for the same declaration, and never
mark the actor Hidden merely because the bonus action was paid.

The canonical 2014 Cleric Channel Divinity card's `Turn Undead` option is
engine-owned. Call `combat_use_activity` with activity id
`dnd5e.content.srd2014.feature.cleric-channel-divinity` and declaration
`{option: "turn_undead", perception: [...]}`. The DM must include exactly one
perception entry for every living Undead whose recorded battle-map position is
within 30 feet: `{target_id, can_see_or_hear, reason?}`. Use `reason` whenever an
Undead is excluded because it can neither see nor hear the cleric. Do not omit a
hidden, blinded, deafened, silenced, or obscured target instead of making that
explicit sensory ruling. The server derives the cleric spell save DC, rolls each
included target's Wisdom save, spends the main action and Channel Divinity, and
atomically updates every failed target. A turned creature cannot react, must try
to move away, and may use its action only to Dash, escape an effect preventing
movement, or Dodge when the DM confirms it has nowhere to move. Damage ends the
effect immediately; otherwise the combat duration clock ends it after one minute
(ten rounds). Use the returned target saves, effects, combat state, and Core
receipt. Never roll the saves separately, patch `turned`, edit target reactions,
or spend Channel Divinity before this call.

At the start of a current combatant's turn, treat `HP == 0`, that combatant's
`death_saves: true`, and the absence of Dead/Stable as the complete death-save
gate. Do not wait for or create a synthetic `Dying` condition. Confirm
`combat_query(view="available_actions", actor_id=...)` returns `death_save`, then
call `combat_check(kind="death_save")` with no `ability`, bonus, proficiency, DC,
or target. Resolve it before any other action or `combat_end_turn`. Refresh the
actor card and combat state immediately; a natural 20 can restore 1 HP and leave
the action available, three successes add Stable, and three failures add Dead.

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
If reaching the target requires movement, commit `combat_movement` first and
inspect all returned reaction windows. Resolve or explicitly decline every owned
opportunity attack before calling stabilization; a blocking reaction window must
never be skipped. Re-read both actors afterward because the rescuer can be
damaged or incapacitated before administering aid.

When an imported scene allows an action-bound social or investigative check,
pass the skill name as `ability` and the action payment in the same
`combat_check` transaction. For example, the Elfsong Tavern bribe is
`combat_check(kind="check", ability="persuasion", action="improvise", dc=15)`
(or `ability="deception"` when the offered reward is insufficient). The engine
derives the complete skill modifier from the actor card and spends the main
action whether the check succeeds or fails; do not pass `proficient` or `bonus`
for a named skill. Advantage from an offer of at least 10 gp is a verified scene
fact and may be supplied only when the actual offer meets that threshold.

If that check succeeds and the chosen NPC already has a canonical campaign actor
card, call `combat_join` with its explicit position, disposition, initiative (or
let the engine roll), and a `tie_breaker` whenever its initiative ties another
participant. The actor is stored under `reinforcements`, cannot act, be targeted,
or trigger reactions during the current round, and is inserted into initiative
only when the next round starts. Do not patch the combatant list, create a
mid-combat placeholder card, or queue the NPC after a failed check. Establish
potential participant cards during lobby/module preparation.

A Stable creature at 0 HP cannot take a short or long rest. If the established
scene permits waiting, call `character_state_change(action="stable_recovery")`.
The engine rolls the source-required `1d4` hours, restores exactly 1 HP, clears
Stable and Unconscious, and keeps unrelated conditions such as Prone. Never patch
HP or choose the duration yourself. Once that actor is conscious and above 0 HP,
use the restricted `character_state_change(action="stand")` to clear Prone; do
not replace the whole sheet or expose arbitrary condition deletion.

Before combat, call `module_query(view="readiness")` with source-grounded groups
for required combatants, reinforcements, and optional actors. Each group includes
canonical campaign actor ids, a same-module `source_scene_id`, and an exact
normalized `source_excerpt`. Required combatants must be in the initial participant
list; reinforcements must not be. Treat missing cards, 0 HP/Dead actors, and
unresolved executable rules as blockers. Surface manual rulings without silently
marking them resolved. Read the returned `settlement`, `manual_rulings`,
`automatic_spell_ids`, `ruling_spell_ids`, and `unavailable_attack_ids`; `ready`
means the actor may enter the encounter, not that every action on the card is
automatically executable. `unarmed_attack_id` remains available even when every
recorded weapon is unavailable. When an exact imported rule source contains the creature,
create it in lobby with `character_create_from(mode="statblock")`; never substitute
a similar creature when the named statblock is unavailable or unsupported. When
the exact card is visible only on a module PDF page, follow
`../../references/module-image-content-review.md` and use
`character_create_from(mode="module_statblock")` only after the reviewed record
validates. Never create or repair a required actor after combat begins.
When the statblock prints a complete numeric action for a known spell, its
creature-specific range, damage, and effect override the base spell for that actor.
After creation, verify that the spell card's displayed definition and structured
resolution agree; a mismatch is a lobby blocker because Agent narration and engine
settlement would otherwise contradict each other.
The excerpt is evidence, not a search hint: copy an exact normalized substring
from the expanded same-module scene or a verified `module_search` hit. Never
paraphrase, translate, or copy text from a different occurrence of the room key.
For a source-bound statblock spell marked with components not repeated in its
reviewed card, obtain an explicit source or DM confirmation and pass
`component_ruling.source_components_confirmed=true` before casting. The engine
checks this before paying the action, slot, or concentration. Never spend first
and ask for the component ruling afterward.
If a hidden caster uses a spell with verbal, somatic, or source-unknown
components, include `component_ruling.casting_perception` before casting. It must
contain exactly one `{observer_id, perceived, reason?}` entry for every living
combatant that does not already know the caster's position; a negative ruling
requires `reason`. Only the DM owns this observer matrix. The MCP rejects an
incomplete matrix before spending the action or spell resource and then updates
per-observer visibility atomically with the cast. Do not leave `hidden=true`
unchanged after audible or visible casting. If an already-recorded legacy or
interrupted workflow needs correction, use `combat_map_patch` with a
`combatant_visibility` patch containing `actor_id`, `hidden` and/or
`visible_to_actor_ids`, and an explicit DM `reason`; never edit encounter state
outside MCP.

For a room such as `D13` nested inside a larger indexed location scene, call
`module_search("D13")` and verify that the first hit's last `heading_path` entry
is the exact room heading before using its content. Preserve that hit's chunk id,
scene id, and page range. Do not select another occurrence where `D13` merely
means a DC value or appears in unrelated prose.

End an encounter with a structured `combat_end.outcome`: `status` is one of
`victory`, `defeat`, `withdrawal`, `truce`, or `interrupted`, and `summary`
states the scene-supported reason and immediate public result. Do not close a
fight merely because the regression has enough samples. The engine rejects an
end while any death-save actor is still at 0 HP without Dead or Stable; settle
those actors first. Record longer consequences as post-combat events and memory,
not by hiding them inside the outcome summary.
After `combat_end`, `combat_query(view="status")` is a historical final encounter
record. Require `snapshot_role: "historical_final_encounter"` and
`combatant_state_is_current: false`, and read current HP, conditions, resources,
and recovery from `character_query`; never overwrite a recovered actor from the
historical combatant projection.

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
2014 surprise decision must compare every hiding creature's canonical Stealth
result against each opponent's passive Perception. Do not substitute the general
half-success group-check rule, and do not treat satisfying an adventure's
"careful/no light" prerequisite as guaranteed surprise unless the source says so.
An opponent that notices any threat is not surprised; record the comparison and
set `surprised` per participant. Hidden and surprised are separate facts. A
contextual observer feature such as Keen Smell changes passive Perception by
`+5` only when the DM confirms that the particular attempt can be perceived by
that sense; preserve that sensory ruling per observer instead of raising every
observer's passive score globally. For every d20 result, retain `roll_mode`,
`advantage_applied`, and `disadvantage_applied` with the raw `rolls` and rule
receipts. The effective roll-mode fields are authoritative: two raw d20 values
alone do not distinguish an applied mode from rerolls or other audited effects.
A current module scene produces a frozen temporary battle map. An encounter scene
may use `current_location_key` to reference exactly one spatial location in
another scene of the same module; persist that source scene as
`state.location_scene_id`, and preserve the current progress, encounter, and
spatial source ids as separate evidence. If the spatial evidence states no room
dimensions and the DM supplies no bounds, the temporary map uses a conservative
12-by-12-cell canvas; this is workspace, not inferred room geometry. The map
may render imported `spatial.connections` only when each edge is backed by
`confidence="explicit_text"` or `confidence="reviewed_image"` evidence. For a
PDF map, follow `../../references/module-visual-atlas.md`: render the managed
page, inspect the image, then persist the branch/snapshot-managed review through
`module_set_progress(spatial_review=...)`. Never
connect rooms by heading order, room number, or a generic cross-reference such
as an encounter's reinforcement note. An empty connection list is an explicit
unknown-topology state. The temporary map enforces only its explicit bounds and
blocked cells; it never turns
room prose into inferred walls, cover, line of sight, or terrain. Record a real door, hazard, or similar
post-combat world change through `combat_map_patch`, not by rewriting the module.
Player map views intentionally omit blocked cells, difficult terrain, DM
overrides, checksums, and world patches; do not disclose those fields from a DM
read or an earlier tool result.
A voluntary grid move cannot end in another living combatant's recorded space.
Set `participant_config.can_share_space=true` only when a source-bound trait
(for example, a swarm trait) or an explicit DM ruling permits it; preserve that
decision when the creature joins later. Passing through occupied spaces,
different creature sizes, and effect-specific forced/teleport overlap are not
inferred from endpoints. A forced or teleport destination that is already
occupied requires an explicit effect-specific ruling unless one participant has
the recorded sharing exception. A grid move that leaves an eligible
hostile's recorded reach opens an owned opportunity window; read it through
`combat_query(view="reactions")`, decline it with
`combat_choice(action="resolve")`, or settle it
atomically with `combat_reaction_attack`. Do not claim other map collision, terrain,
forced movement, line of sight, or a trigger not represented in encounter state.
Use `combat_movement(action="move")` with `payload.path` for bent grid routes. Set `movement_mode` to `forced` or
`teleport` when the scene establishes that the move does not provoke a normal
opportunity attack; do not encode terrain cost or collision unless it is part of
the supplied scene facts.
When the temporary battle map records `difficult_cells`, provide a cell-by-cell
`payload.path` for any move longer than one square. The engine charges one extra
foot per foot spent entering those reviewed cells and returns the reduced movement
budget with a Core receipt. Do not add that surcharge to `distance` yourself:
`distance` remains the geometric route length. Unmapped terrain still requires a
DM ruling rather than an invented cell cost.
If Prone, either use `combat_movement(action="stand")` (half speed, no action) or
use `combat_movement(action="move", payload.crawl=true)`; crawling costs double movement.

Every campaign, character, party, combat, rest, continuity, branch, snapshot,
scene-progress, and actor-knowledge mutation must carry the current optimistic
token exposed by that tool and a fresh `idempotency_key`. The token may be a
campaign/character revision, actor-knowledge revision ID, branch/head ID, history
sequence, or scene `state_version`; read the MCP contract for the exact field.
Re-read the relevant state after a conflict; never retry a changed payload under
the same key. Shared wallet and
inventory adjustments are campaign writes and follow the same contract.

Use `combat_use_activity` or `character_action(action="use_activity")` for cards in
`content.activities`, `features`, or `feats`. These tools pay a recorded use or
resource and the activation timing. They settle explicitly supported Core
mechanics such as Action Surge, Second Wind, Cunning Action, and Turn Undead;
other choices or prose outcomes return `pending_ruling`. Never treat a generic
`committed` payment as proof that unstructured prose was resolved.

Core Preserve Life is an exception with a complete deterministic contract. In
noncombat play, its `declaration.allocations` must contain every target's id,
current character revision, positive healing amount, and DM-confirmed
`within_30_ft: true`. Submit the whole allocation once. The MCP enforces the
five-times-Cleric-level pool, the half-maximum-HP ceiling, and the Undead/Construct
exclusion, then atomically spends Channel Divinity and updates all target cards.
Never pay the activity first and heal targets through separate calls.

Reaction spells and activities require an owned pending reaction window. Do not
call them solely because it is another actor's turn. Do not hide a spell inside
the generic Ready payload. For a spell with an Action casting time, call
`combat_ready(action="ready_spell")`: it pays the action and spell slot or other casting resource
immediately, replaces any existing concentration, and arms one perceivable
trigger until the start of the caster's next turn. The DM confirms that the
trigger occurred with `combat_ready(action="trigger_spell")`. The caster then uses
`combat_ready(action="resolve_spell")` to release the spell with its reaction or decline
that occurrence without spending the reaction; declining leaves the spell armed.
Losing concentration, reaching the caster's next turn, or ending combat makes the
held spell dissipate without effect. A released spell returns `pending_ruling`:
resolve its targets, attack, save, damage, area, and narrative consequences with
the appropriate combat tools and DM ruling rather than treating release as the
spell's complete effect.

Before a module can branch on opening hours, daylight, watches, or travel time,
establish its branch-local clock with
`campaign_change(action="clock_set", payload={day, hour, minute, label})` and cite
the narrative/source assumption. After the DM establishes elapsed time, use
`campaign_change(action="clock_advance", payload={period, count})`. Minute, hour,
and day advances update the snapshotted campaign clock and settle all matching
canonical actor and campaign-space effect durations in the same mutation; round and encounter durations
advance without changing narrative time. Never infer elapsed time from chat
pacing, and never set or advance the narrative clock during active combat. Once
set, do not jump the clock with another `clock_set`; use `clock_advance` so no
duration is skipped.

Resolve every completed long rest through
`campaign_change(action="party_rest", payload={members, duration_minutes})`, even
for a one-character party. Each member supplies its current character revision
and any prepared-spell or 2014 hit-die recovery choices. This one write advances
the campaign clock once, advances timed effects for every campaign actor and
world object, applies benefits to only the named members, records completion on
each card, and enforces the one-benefit-per-24-hours rule. A creature must have at
least 1 HP at the start. Do not call individual `character_rest` for a long rest
or advance eight hours separately before/after the party rest.

An effect on a room, object, scene, or the campaign belongs in structured
`campaign_change(action="effect_add")`, not an arbitrary `module_set_progress.state`
blob. Give it a stable id, target, duration, source, and `visibility` of
`public`, `party`, or `dm`; dismiss it with `effect_remove`. Player campaign
reads are audience projections, but never reuse a DM `campaign_query` result in
player narration or assume an exposure layer repairs already-read private data.
