# Character Creation and Advancement

Use `dnd_ability_roll` for visible rolls. Apply a confirmed standard array,
point-buy, or rolled assignment with `character_ability_apply`; never manually
write derived ability values.

For a new PC, call `character_create_from(mode="build")` with complete validated `sheet v2` and
`notes v2`. It creates the public template and independent campaign instance in one
transaction. Use `character_create_from(mode="direct")` for a direct NPC/monster
instance, `mode="template"` to copy an existing library template, and
`mode="statblock"` to create an NPC/monster from an exact imported rule source.

When preparing an imported module, first verify whether the supplied artifact
actually contains pregenerated PCs. If it does not, do not attribute invented
characters to the module: create player-confirmed or explicitly labeled regression
PCs from the active content catalog, and retain that provenance in `notes`. For a
named module NPC or monster, the module supplies identity, role, disposition, and
scene-specific possessions while an inspected rule source supplies the mechanical
statblock. Record both sources. Fixed treasure may be placed on the card during
lobby setup; dice-denominated treasure stays unresolved until the real roll occurs.
Statblock import currently accepts reviewed English 2014 SRD-style weapon
statblocks. If the exact creature is absent, spell-only, 2024, ambiguous, or
unsupported, keep it unresolved instead of substituting a similar creature.

Before creating any actor, read `character-schema-v2.md`. All PCs, NPCs, and
monsters require complete structured cards; NPCs and monsters require
`notes.profile.summary`. Do not persist an unconfirmed draft. After every creation
or advancement, call `character_query(view="get")` and use its `derived` values for proficiency,
saves, skills, AC, HP, speed, spell DC, preparation, and encumbrance.

Recording a class, subclass, species, or subspecies name is not sufficient.
Before the first `play` phase and after every level-up, query
`rule_pack_query(view="content_catalog")`
and reconcile every class/subclass feature whose `minimum_level` is met, plus every
species grant and required choice, through `character_content_apply`. Treat
`catalog_only` as a stop condition that needs reviewer/DM completion, never as an
applicable card. When importing a finished character whose printed scores and HP
already include species bonuses, pass `values_include_species_grants: true` while
applying the species so the catalog provenance and nonnumeric traits are retained
without double-counting numeric grants. When only the printed ability scores or
only HP already includes the grant, use the narrower
`ability_scores_include_species_grants` or
`hit_points_include_species_grants` flag and let the other value settle normally.

Before combat, audit at least: class and subclass features, species/subspecies
features, proficiencies and expertise, resources and recovery periods, equipped
weapons and ammunition, spellbook/known/prepared spells, AC, HP, speed, senses,
resistances, and every unresolved rule. Missing feature cards are setup defects,
not permissions for the DM agent to improvise abilities during combat.

For an NPC or monster statblock with Multiattack, record its exact legal attack
compositions in one Action activity. Do not flatten it to a generic
`attacks_per_action` value: that would allow illegal substitutions. Each option
uses stable inventory weapon ids, an explicit `attack_mode`, and a count:

```json
{
  "id": "bandit-captain-multiattack",
  "name": "Multiattack",
  "source_key": "Bandit Captain",
  "activation": {"type": "action"},
  "choices": {
    "multiattack_options": [
      {
        "id": "melee",
        "attacks": [
          {"weapon_id": "scimitar", "attack_mode": "melee", "count": 2},
          {"weapon_id": "dagger", "attack_mode": "melee", "count": 1}
        ]
      },
      {
        "id": "ranged",
        "attacks": [
          {"weapon_id": "dagger", "attack_mode": "ranged", "count": 2}
        ]
      }
    ]
  }
}
```

After creation, verify `derived.multiattack_options` against the source and make
sure every referenced weapon id exists. Preserve the source document and rule
reference on the activity card just like any other imported mechanic.

An AC-changing defensive reaction must also be structured instead of inferred
from its description. For example, the 2014 Bandit Captain's Parry activity keeps
its Reaction activation and records:

```json
{
  "reaction_defense": {
    "kind": "armor_class_bonus",
    "bonus": 2,
    "attack_modes": ["melee"],
    "requires_visible_attacker": true,
    "requires_wielded_melee_weapon": true
  }
}
```

Put this object under `choices`, and retain the source citation on the activity.
Do not turn reaction text into an always-on AC bonus. If any source prerequisite
cannot be represented as a verified field, leave the mechanic unresolved for the
DM instead of silently weakening it.

For a 2014 class-prepared caster, eligible level 1+ class spells use
`grant.method: "class_prepared"`, identify the recorded class with
`grant.source_key`, and may keep `access.known: false`. Put the complete daily
choice in `spellcasting.preparation.selected_spell_ids`; the selected spells become
derived prepared spells. A Wizard selection must additionally be in the spellbook.
Cantrips are known but never consume a prepared-list selection.

Advancement changes the live campaign instance and must use the audited lobby
workflow; never patch `progression.level`, HP, Hit Dice, slots, or preparation
limits by hand:

1. Inspect the module or campaign award and retain its exact `source_ref`. Record
   a level-up `campaign_event(action="add")` that says why the level was earned.
2. End active combat, switch the campaign to `lobby`, read the actor's latest
   revision, and call
   `character_state_change(action="level_advance", payload={class_name,
   hp_method, hp_roll?, reason, source_ref})`. The current implementation advances
   an existing 2014 single class by exactly one level; multiclass and 2024
   advancement are stop conditions, not permissions to replace the sheet.
3. Use `hp_method="fixed"` for the class fixed value, or use `"rolled"` only with
   the actual Hit Die result. The transaction updates maximum HP, the Hit Die
   pool, spell-slot capacity, and preparation maximum. It does not heal existing
   damage. Newly gained slot capacity becomes available, but spent old slots do
   not refresh. Source-bound per-level modifiers such as Dwarven Toughness are
   resolved from already applied catalog provenance.
4. Read `advancement.follow_up`. Apply every eligible base-class and already
   selected-subclass feature in the listed order through
   `character_content_apply`. If `subclass_options` is nonempty, obtain the
   player's choice, apply that subclass, query the content catalog again, and
   apply its newly eligible feature cards. A feature that names a shared resource
   must be applied after the feature that grants that resource.
5. Resolve each reported cantrip, known-spell, or spellbook choice from
   `rule_pack_query(view="content_catalog")`; apply only eligible artifact ids.
   A Wizard adds the reported spells with `method="spellbook"`. Then submit the
   complete legal prepared list with
   `character_spell_prepare(mode="replace_all", event="level_up")`.
6. Re-read the actor and audit level, HP maximum/current HP, Hit Dice, spell slots,
   preparation, feature resources, subclass, spells, and `derived`. Do not return
   to `play` while any required catalog item or player choice is missing.
7. After confirmation, create a post-level `snapshot_create`, then return to
   `play` and reopen the phase-appropriate exposure.
