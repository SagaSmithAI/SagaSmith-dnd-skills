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
without double-counting numeric grants.

Before combat, audit at least: class and subclass features, species/subspecies
features, proficiencies and expertise, resources and recovery periods, equipped
weapons and ammunition, spellbook/known/prepared spells, AC, HP, speed, senses,
resistances, and every unresolved rule. Missing feature cards are setup defects,
not permissions for the DM agent to improvise abilities during combat.

For a 2014 class-prepared caster, eligible level 1+ class spells use
`grant.method: "class_prepared"`, identify the recorded class with
`grant.source_key`, and may keep `access.known: false`. Put the complete daily
choice in `spellcasting.preparation.selected_spell_ids`; the selected spells become
derived prepared spells. A Wizard selection must additionally be in the spellbook.
Cantrips are known but never consume a prepared-list selection.

Advancement changes the live campaign instance. Update only the affected validated
sheet fields, record a level-up `event_add`, and call `snapshot_create` after the
player confirms the new state.
