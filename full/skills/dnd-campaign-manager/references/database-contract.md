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
- Bind combatants to scene tokens when a scene is available.
- Use `activity use` for action economy, resources, effects, and class features.
- Use `effect add/remove/list` for active effects.
- Use `rest short|long` and `time advance` for period recovery and durations.
- Never use `combat act`; it is intentionally outside the contract.
- Do not directly edit combat JSON, HP, resources, conditions, token position, or duration.

Snapshot/restore includes campaign state, characters, item instances, scene
progress, map scenes, scene tokens, scene regions, memories, and rule profile.
It does not copy static rules, module source text, or embeddings.

Durations advance only through declared runtime periods: combat turn/round, rest,
scene end, and `time advance`. Wall-clock time and LLM latency never count.
