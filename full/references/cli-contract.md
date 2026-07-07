# SagaSmith D&D CLI Contract

- Executable: `sagasmith-dnd`
- Agent calls always include `--json`.
- stdout contains exactly one JSON document.
- stderr may contain logs or model download progress.

Success:

```json
{"ok":true,"data":{},"error":null,"meta":{"command":"group.action","version":"0.2.0"}}
```

Failure:

```json
{"ok":false,"data":null,"error":{"code":"not_found","message":"..."},"meta":{}}
```

Exit codes: 0 success, 2 invalid input, 3 not found, 4 conflict, 5 configuration,
6 optional dependency, 10 internal error.

Keep responses bounded with `--limit`. Search first and expand one selected chunk.

Compatibility workflows include:

- `module index` / `module export-scenes`
- `save regenerate-recap`
- `memory scope` / `memory status`
- `state history` / `state undo` / `state redo`

Foundry-style runtime workflows:

- Rulesets: `ruleset list/show/validate`
- Pack import: `pack import --campaign <id> --path <foundry-pack-or-file>`
- Actor documents: `actor create/list/show`
- Map documents: `scene create/list/show`, `token create/list/show/move`, `region create/list`
- Combat: `combat start/status/attack/damage/heal/condition/death-save/end-turn/end`
- Activities: `activity use --campaign <id> --actor <actor-or-combatant-id> --item <item-id> --activity <activity-id>`
- Reactions: `reaction list/resolve/decline`
- Effects, conditions, damage, and rolls: `effect recalculate`, `condition add/remove`, `damage apply`, `roll ability/skill/save/initiative`
- Effects and periods: `effect add/remove/list`, `time status/advance`, `rest short/long`

Runtime authority rules:

- Treat `scene -> token -> combatant -> actor` as the map/combat chain.
- Treat `ruleset.activityActivationTypes`, `ruleset.activityTypes`, `ruleset.limitedUsePeriods`,
  `ruleset.conditionTypes`, and `ruleset.conditionEffects` as the structured rule contract.
- Do not use `combat act`; it is intentionally disabled.
- Do not directly edit combat JSON, HP, conditions, action economy, resources, token position, or duration.
- If `activity use` returns `pending` reaction windows, resolve or decline them before narrating final resolution.
- Use `effect recalculate` after adding/removing ActiveEffects that change actor math.
- Use `condition add/remove` for Actor document conditions and then `effect recalculate`.
- Use `damage apply` for Actor document damage so resistance, vulnerability, and immunity are applied.
- Use `roll ability/skill/save/initiative --campaign <id> --actor <actor-id>` when an Actor document exists.
- Use `time advance --period <period>` for narrative or combat period durations.
- Use `time advance --minutes <n>` for declared in-world elapsed time. Wall-clock time and model latency never advance durations.
