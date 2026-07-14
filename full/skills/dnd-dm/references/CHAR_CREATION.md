# Character Creation and Advancement

Use `dnd_ability_roll` for visible rolls. Apply a confirmed standard array,
point-buy, or rolled assignment with `character_ability_apply`; never manually
write derived ability values.

For a new PC, call `character_build` with complete validated `sheet v2` and
`notes v2`. It creates the public template and independent campaign instance in one
transaction. Use `character_create` for a direct NPC/monster instance, and
`character_instantiate` to copy an existing library template.

Before creating any actor, read `character-schema-v2.md`. All PCs, NPCs, and
monsters require complete structured cards; NPCs and monsters require
`notes.profile.summary`. Do not persist an unconfirmed draft. After every creation
or advancement, call `character_get` and use its `derived` values for proficiency,
saves, skills, AC, HP, speed, spell DC, preparation, and encumbrance.

Advancement changes the live campaign instance. Update only the affected validated
sheet fields, record a level-up `event_add`, and call `snapshot_create` after the
player confirms the new state.
