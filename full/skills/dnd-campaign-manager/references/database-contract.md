# Runtime Data Contract

Skills never access the database directly. Every mutation goes through
`sagasmith-dnd ... --json`.

- SQL is the source of truth for mutable campaign state.
- Rules, modules, and embeddings are static/reference layers.
- Campaigns, characters, item instances, scene progress, map scenes, scene tokens,
  scene regions, snapshots, events, and memories are mutable runtime state.
- Each campaign's edition, locale, publications, and options are decided by its
  rule profile.
- Dense retrieval is optional; exact/lexical retrieval remains available.

## Foundry-Style Runtime

SagaSmith runtime follows this document chain:

```text
Scene -> Token -> Combatant -> Actor/Character
Item/Feature/Spell -> Activity -> Consumption/Effect/Duration
Region -> Terrain/Aura/Hazard/Template behavior
```

Runtime rules:

- Use `scene`, `token`, and `region` commands for map state.
- Use `scene activate` when the tactical scene changes, and read `scene show`
  for prepared Token runtime data.
- Bind combatants to visible scene tokens; free-form combat participants are not
  part of the runtime contract.
- Use `activity use` for action economy, resources, effects, and class features.
- Use `effect add/remove/list` for active effects.
- Use `rest short|long`, `combat death-save`, and `time advance` for recovery,
  death-save, and duration state.
- Never use `combat act`; it is intentionally outside the contract.
- Do not directly edit combat JSON, HP, resources, conditions, token position, or duration.
- Do not use `sheet.inventory`, `inventory_managed`, or legacy sheet fields as
  runtime authority. Actor Items/Activities own abilities and equipment actions;
  the item ledger owns treasure, backpack, currency, containers, and mundane
  ownership.

Snapshot/restore includes campaign state, characters, item instances, scene
progress, map scenes, scene tokens, scene regions, memories, and rule profile.
It does not copy static rules, module source text, or embeddings.

Durations advance only through declared runtime periods: combat turn/round, rest,
scene end, and `time advance`. Wall-clock time and LLM latency never count.

Rest and death-save state lives on Actor documents. Short rests can spend hit
dice through `rest short --payload '{"hit_dice":n}'`; long rests restore HP,
spell slots, rest-recovered resources, death saves, and part of spent hit dice.
