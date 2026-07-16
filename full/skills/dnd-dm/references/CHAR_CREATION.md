# Character Creation and Advancement

Use `dnd_ability_roll` for visible rolls. Apply a confirmed standard array,
point-buy, or rolled assignment with `character_ability_apply`; never manually
write derived ability values.

For a new PC, call `character_build` with complete validated `sheet v2` and
`notes v2`. It creates the public template and independent campaign instance in one
transaction. Use `character_create` for a direct NPC/monster instance, and
`character_instantiate` to copy an existing library template.

When preparing an imported module, first verify whether the supplied artifact
actually contains pregenerated PCs. If it does not, do not attribute invented
characters to the module: create player-confirmed or explicitly labeled regression
PCs from the active content catalog, and retain that provenance in `notes`. For a
named module NPC or monster, the module supplies identity, role, disposition, and
scene-specific possessions while an inspected rule source supplies the mechanical
statblock. Record both sources. Fixed treasure may be placed on the card during
lobby setup; dice-denominated treasure stays unresolved until the real roll occurs.

Before creating any actor, read `character-schema-v2.md`. All PCs, NPCs, and
monsters require complete structured cards; NPCs and monsters require
`notes.profile.summary`. Do not persist an unconfirmed draft. After every creation
or advancement, call `character_get` and use its `derived` values for proficiency,
saves, skills, AC, HP, speed, spell DC, preparation, and encumbrance.

Recording a class, subclass, species, or subspecies name is not sufficient.
Before the first `play` phase and after every level-up, query `content_catalog_list`
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

Advancement changes the live campaign instance. Update only the affected validated
sheet fields, record a level-up `event_add`, and call `snapshot_create` after the
player confirms the new state.
