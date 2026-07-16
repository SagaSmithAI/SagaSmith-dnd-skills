# D&D Content Catalog Contract

The Content Catalog is the common character-option surface for bundled Core
rules and source-bound extension packs. It separates three facts that must not
be conflated:

1. **Catalogued**: a source-linked record is searchable and has a stable id.
2. **Available**: its core edition is locked by the campaign, or its extension
   pack is enabled on the current branch.
3. **Executable**: a reviewed rule mechanic covers the requested outcome.

For the bundled 2014 SRD, `dnd5e.content.srd2014@1.3.0` is installed during MCP
startup when the full D&D skill repository is configured. Its records retain a
`bundled:srd2014/...` reference to the original Markdown file. Optional books
must use `rule_pack_draft_from_source`; every artifact supplies imported
`source_chunk_ids`, which the MCP resolves to the exact document chunk/page
citations before the pack can be installed.

## Agent procedure

1. Read `campaign_rule_profile_get` and the current branch state.
2. Call `content_catalog_list(campaign_id, kind, query)`.
3. Present only returned options and their source references to the player. Read
   `selection_requirements` for spell eligibility, subclass class/level,
   species grants, class/subclass feature level, background choices, and feat
   prerequisites; do not infer these from names.
4. For a supported card target, call `character_content_apply` with the
   character's latest revision, an idempotency key, and every required
   `selection` value. A spell selection identifies its `source_class` and grant
   `method`; a subclass identifies its target class on multiclass cards; a
   background supplies its required language choices. A species supplies every
   listed language, skill, tool, ability, and cantrip choice. If an imported
   finished sheet already includes all numeric species bonuses, set
   `values_include_species_grants: true` explicitly so provenance and traits are
   linked without adding the bonuses twice. If only part is already included,
   use `ability_scores_include_species_grants` and
   `hit_points_include_species_grants` separately; for example, a printed Hill
   Dwarf card may already include ability increases but still be missing the
   per-level HP grant.
5. If the response is `pending_ruling`, obtain the required choices or resolve
   the effect as a DM decision. Do not bypass the result by editing raw sheets.

An imported extension is not automatically enabled, and installation is not a
mechanics claim. The DM selects its exact pack version per branch. Snapshots
then retain that version/checksum lock for replay and audit.

The bundled catalog is built from leaf records, not index pages: individual
spell files, twelve class files, twelve subclass sections, thirteen base or
subspecies cards, source-linked class/subclass feature sections, the SRD
Acolyte background, the Grappler feat, and structured equipment rows. A base
class card is `catalog_only`: its name is not proof that level features have
been applied. Complex species such as Dragonborn also remain `catalog_only`
until every ancestry-dependent grant is structured. Spell catalog cards retain
class eligibility, but a character card records only the class actually chosen
in `grant.source_key`.

## Character completeness gate

Before play or combat, compare every recorded class level and subclass against
the catalog and apply every feature available at that level. Then compare the
selected species/subspecies against its structured grants. A card is incomplete
if it records only `progression.classes`, `progression.species`, or a subclass
name while the corresponding `content.features`, proficiencies, traits,
resources, spell grants, or required choices are absent.

The engine recognizes source-linked feature cards, not prose guesses. For 2014
Rogue Sneak Attack, pass `use_sneak_attack: true` to the attack declaration;
the engine validates finesse/ranged weapon eligibility, advantage or an active
enemy adjacent to the target, disadvantage, once-per-turn use, and critical
dice. For Fighter Second Wind, call `combat_use_activity` first to pay the bonus
action and use, roll `1d10 + fighter level`, then apply that exact amount through
`combat_hp_change(action=heal)`. For spell healing, supply the source actor,
recorded spell id, and actual slot level separately; the engine adds a recorded
Disciple of Life modifier and preserves it in the healing receipt. Halfling Lucky
rerolls are automatic and appear in the roll's `rerolls` audit field.
