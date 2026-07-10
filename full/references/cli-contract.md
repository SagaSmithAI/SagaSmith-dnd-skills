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

- Rulesets: `ruleset list/show/validate/coverage`; build a canonical pack with
  `ruleset compile --path <foundry-source> --output <pack.json>`
- Pack import: `pack import --campaign <id> --path <foundry-pack-or-file>`
- Actor documents: `actor create/create-monster/list/show/update/prepare`
- Actor Item documents: `game-item create/list/show/update`
- Activity documents: `game-activity create/list/show/update`, then `activity use`
- Advancement: `advancement apply`, `advancement grant-class`,
  `advancement grant-subclass`, `advancement grant-feature`, and
  `advancement grant-spell`
- Map documents: `scene create/list/show/activate`, `token create/list/show/update/move`, `region create/list`
- Measured templates: `template place --scene <id> --item <id> --activity <id> --x <n> --y <n>`
- Cover: `cover check --scene <id> --token <attacker-token-id> --target-id <target-token-id>`
- Combat: `combat start/status/death-save/end-turn/end`; low-level
  `combat attack/damage/heal/condition` commands are debug fallbacks, not the
  normal AI DM action path
- Activities: `activity use --campaign <id> --actor <actor-or-combatant-id> --item <item-id> --activity <activity-id>`
- Resolution windows: `resolution list/resolve/decline` (`reaction` remains an
  alias for existing callers)
- Ready actions: `ready set/trigger/clear`
- Effects, conditions, damage, concentration, and rolls: `effect recalculate`, `condition add/remove`, `damage apply`, `concentration pass/fail`, `roll ability/skill/save/initiative`
- Effects and periods: `effect add/remove/list`, `time status/preview/declare`,
  and `rest short/long`

Runtime authority rules:

- Treat `scene -> token -> combatant -> actor` as the map/combat chain.
- When a Scene has Actor-linked Tokens, `combat start --scene <scene-id>` derives
  combatants from visible tokens and prepared Actor data. Do not pass free-form
  participants; create or update Actor and Token documents first.
- Treat `ruleset.activityActivationTypes`, `ruleset.activityTypes`, `ruleset.limitedUsePeriods`,
  `ruleset.conditionTypes`, `ruleset.conditionEffects`, `ruleset.spellcasting`,
  `ruleset.durationPeriods`, `ruleset.mapRuntime`, `ruleset.restRecovery`, and
  `ruleset.deathSaves` as the structured rule contract.
- Do not use `combat act`; it is intentionally disabled.
- Do not directly edit combat JSON, HP, conditions, action economy, resources, token position, or duration.
- Use `game-item` and `game-activity` for Foundry-style Actor Items and executable
  activities. Use the campaign item ledger only for inventory ownership, treasure,
  currency, containers, and mundane item accounting.
- If `activity use` returns `pending` reaction windows, resolve or decline them before narrating final resolution.
- If it returns `deferred: true`, its action cost is already spent but its
  continuation has not rolled or dealt damage. Resolve or decline the eligible
  response with `resolution resolve|decline`; only its `continuation_result`
  is the final action result.
- If `activity use` returns `execution`, treat that attack/damage/heal/save result as authoritative and do not recalculate it in prose.
- Activity execution resolves common Foundry roll data formulas such as `@prof`,
  `@mod`, `@abilities.dex.mod`, `@classes.<class>.levels`, and
  `@item.uses.spent`. Do not pre-expand those formulas in prose.
- Activity execution reads Foundry-style structured `system.damage.parts`,
  `system.damage.onSave`, `system.healing`, and `system.save.dc.formula`.
  Prefer these imported/structured fields over custom flat damage strings.
- Cast activities consume spell slots, support cantrip/slot scaling, use Actor
  spellcasting attack/DC defaults, create concentration effects, and honor ritual
  casts only when the activity/item data and payload explicitly mark ritual casting.
- Actor preparation applies Foundry numeric ActiveEffect change modes
  `1=multiply`, `2=add`, `3=downgrade`, `4=upgrade`, `5=override`, with
  priority ordering. Keep imported numeric modes intact.
- If `token move` returns movement `pending` reaction windows, resolve opportunity attacks before final movement narration.
- Use `ready set` for the Ready action and `ready trigger` when the stated trigger occurs.
- Use `effect recalculate` after adding/removing ActiveEffects that change actor math.
- Use `condition add/remove` for Actor document conditions and then `effect recalculate`.
- Use `actor prepare` after equipment, advancement, item, condition, or ActiveEffect
  changes that can affect derived actor math. Rolls, attacks, saves, damage, and
  narration should read the prepared Actor state instead of hand-calculating AC,
  proficiency, resistances, immunities, vulnerabilities, or condition statuses.
- Prepared Actor data includes basic ability modifiers, save totals, skill
  modifiers, and passive skill values under `derived.effective_system` and
  `derived.rolls`.
- Use `damage apply` for Actor document damage so resistance, vulnerability, and immunity are applied.
- If `damage apply` returns `concentration_save_required`, roll the CON save and call `concentration pass` or `concentration fail`.
- Use `roll ability/skill/save/initiative --campaign <id> --actor <actor-id>` when an Actor document exists.
- Use `advancement apply` for level, hit point, scale value, and feature/item grants.
- Prefer `advancement grant-feature`, `advancement grant-spell`, and
  `actor create-monster` for ruleset-backed features, spells, and monster stat
  blocks before manually creating equivalent Actor Items and Activities.
- Use `advancement grant-class` and `advancement grant-subclass` for compiled
  2014 class progression before individual grants.
- Before non-combat elapsed time, run `time preview --elapsed PT10M`; then use
  `time declare --elapsed PT10M --reason "..." --intent-id <stable-id>`.
  Never use wall-clock time or model latency. Retry the same declaration with
  the same intent ID after a transport failure; it is idempotent.
- An Activity with minute/hour activation returns a `scheduled_activity`. Complete
  it only with `time declare --operation <id> --elapsed <enough-time> ...`; use
  `time cancel --operation <id>` if the declared activity is interrupted.
- Use `scene activate --campaign <id> --scene <scene-id>` when the tactical scene
  changes; this advances `scene_end` durations for the previous active scene.
- Use `scene show` or `token show` to read prepared token runtime fields such as
  Actor summary, HP bar, token size, position, targetability, and vision derived
  from Actor senses.
- Use `rest short --payload '{"hit_dice":1}'` when a character spends hit dice,
  and `rest long` for full HP, spell slot, resource, death-save, and hit-dice recovery.
- Use `combat death-save --target-id <actor-id>` at 0 HP; the result is also
  synchronized to the Actor document's death-save state.
- Use `template place` before resolving area effects with a target template; it creates the scene Region the AI DM should reference.
- Region `metadata.triggers` is executable for period events such as
  `declared_minute`, `turn_start_inside`, and `turn_end_inside`; do not hand-roll
  its damage or saves after `time declare` or a combat period has resolved it.
- Use `cover check` before ranged attacks or Dexterity saves when map obstacles might matter.
- Use `region create --behavior apply_active_effect` for auras, hazards, and template
  zones that attach ActiveEffects to actors. `token move` reports created/removed
  `movement.region_effects`; handle those state changes before final movement narration.
