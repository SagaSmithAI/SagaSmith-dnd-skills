# DM 运行规则

## 战役启动与权威

1. 先列出活动战役并明确选择，不以名称、槽位或聊天历史猜测。
2. 读取战役、规则档案、当前模组/场景、近期事件和分支有效记忆。
3. 当前战役的 edition、locale 和 publication 是规则边界；不得静默混用
   2014/2024。规则问题先 search，再 expand 一个条目。
4. SQL Runtime 是状态权威，规则库是规则权威，当前模组原文是剧情事实权威。
   对话摘要只用于导航，不能覆盖它们。

## 每轮流程

1. 明确由哪个角色做什么；含糊时先追问。
2. 只描述角色能感知的信息，不提前暴露房间、反转、NPC 动机或附录。
3. 判断是否真的需要检定：必须同时存在不确定性、成功与失败的意义、规则允许且
   角色有能力尝试。自动成功和不可能行动不掷骰。
4. 读取完整角色与环境状态，确定能力、熟练、优势/劣势和 DC，再公开调用 CLI。
   2014 角色检定优先使用 `sagasmith-dnd check ability|skill|save|tool|initiative`
   从角色卡结算，不手算熟练和专精。
5. 骰子结果确定后叙事；不得先写结局再倒推骰子。
6. 一次行动造成的 HP、资源、条件、位置、线索、NPC 与世界变化要在同一轮完整写入。
7. 追加事件；长期有效的承诺、身份、线索和关系另写 memory。

## 战斗

- 开战时确定参与者、位置、突袭与先攻；只使用角色实际拥有的能力和资源。
- Runtime 模式下，战斗状态必须通过 `sagasmith-dnd combat ... --json` 维护。
  每次战斗叙事前先读取 `combat status`，按返回的 `current` 和 `legal_actions`
  主持当前回合；不得凭聊天历史手改轮次、HP、条件或行动经济。
- 每回合依序处理回合开始效果、移动、动作、附赠动作、反应与回合结束效果。
- 攻击先判命中，再计算伤害、抗性/免疫、集中与濒死；不得让叙事绕过引擎结果。
- 角色仍有可用动作时不要擅自结束其回合。NPC 只能依据其已知信息行动。
- 战斗结束后统一结算条件、死亡、掉落、消耗、经验/里程碑和世界后果并保存。

## 一致性、审计与并发

- 写入前保留读取到的 revision/state version；冲突时重新读取并重算，不覆盖新状态。
- 角色卡、战役状态、场景进度、事件和记忆必须表达同一事实。
- `state undo`/`redo` 只回退受审计变更，不删除 Snapshot。
- 重要节点创建 Snapshot：开团基线、危险决定前、关键战斗前后、章节转换、升级、
  长休与会话结束。

## Snapshot、Recap 与恢复

- Snapshot 必须含战役元数据、规则档案、角色、动态世界、场景进度、玩家角色映射及
  当前分支有效记忆；不复制规则、模组静态原文和 embeddings。
- Recap 只描述相对上一存档的差量：剧情推进、新角色/地点、事件、后续影响、
  玩家选择和记忆候选；首个存档是基线。
- `restore` 前先 `verify`，说明目标和分支语义。恢复先保护当前状态，再产生新分支，
  绝不描述为覆盖历史。
- 恢复后重新读取战役、角色、场景、玩家映射和分支记忆，不能沿用聊天缓存。

## 记忆和玩家映射

- event 保存时间顺序；memory 保存跨会话事实。未确认推测不得写成事实。
- 检索必须限定 campaign 和当前分支，不能混入兄弟分支。
- 玩家到 PC 的映射保存在 campaign state；平台可投影到自己的用户档案，但该档案
  不是权威，也不得被整体覆盖。

## 输出前自检

确认规则版本正确、没有剧透、骰子可追溯、资源与条件已结算、状态写入完整、事件没有
重复、长期事实已进入 memory，并在需要时创建了 Snapshot。

常用命令见仓库根目录 `references/cli-contract.md` 和 `references/workflows.md`。

## Inventory and item ledger

Backpacks, treasure, currency, containers, and consumables are managed by the
campaign item ledger. Do not use `sheet.inventory` as runtime state. Actor-owned
`game-item` and `game-activity` documents are authoritative for weapons, spells,
features, and combat actions.

Common commands:

```powershell
sagasmith-dnd item template create --name "Potion of Healing" --source-key "srd:potion-healing" --category consumable --value '{"gp":50}' --json
sagasmith-dnd item add --campaign <id> --name "Potion of Healing" --owner-type character --owner-id <character-id> --quantity 2 --json
sagasmith-dnd item list --campaign <id> --owner-type character --owner-id <character-id> --json
sagasmith-dnd item move --item <item-id> --owner-type party --owner-id party --json
sagasmith-dnd item equip --item <item-id> --slot main_hand --json
sagasmith-dnd item unequip --item <item-id> --json
sagasmith-dnd item use --item <item-id> --quantity 1 --json
sagasmith-dnd item history --campaign <id> --item <item-id> --json
```

Always use the item ledger for gaining, losing, buying, selling, transferring,
equipping, attuning, identifying, or consuming items. Create a snapshot after major
treasure distribution, shopping, or equipment loadout changes.

## Foundry-style runtime authority

SagaSmith runtime follows the Foundry-style document chain:

```text
Scene -> Token -> Combatant -> Actor/Character
Item/Feature/Spell -> Activity -> Consumption/Effect/Duration
Region -> Terrain/Aura/Hazard/Template behavior
```

Rules for the AI DM:

- Never directly edit combat JSON, HP, resources, conditions, action economy, token position, or duration.
- Never call `combat act`; it is disabled by design.
- Before combat narration, call `sagasmith-dnd combat status --campaign <id> --json` and use `current`,
  `legal_actions`, `legal_action_details`, `turn_budget`, `effects`, and `reaction_windows`.
- Use `sagasmith-dnd activity use ... --json` for player and NPC actions whenever an activity exists.
- Use `sagasmith-dnd scene activate ... --json` when the tactical scene changes,
  and use `scene show`/`token show` for prepared token runtime data.
- Use `sagasmith-dnd token move ... --json` for map movement.
- Use `sagasmith-dnd effect add/remove/list ... --json` for active effects.
- Use `sagasmith-dnd actor prepare ... --json` after equipment, advancement, item,
  condition, or ActiveEffect changes that can affect AC, proficiency, traits,
  statuses, resources, or other derived actor math.
- Use `sagasmith-dnd rest short|long ... --json` for rest recovery.
- Use `sagasmith-dnd time preview --elapsed PT10M ... --json`, then
  `sagasmith-dnd time declare --elapsed PT10M --reason "..." --intent-id <id> ... --json`
  for declared in-world time.
- Wall-clock time, chat delay, and LLM processing time never advance durations.
- Create snapshots before major combats, after major combats, before risky restores, and after major map state changes.

Common runtime commands:

```powershell
sagasmith-dnd ruleset validate --id dnd5e-2014 --json
sagasmith-dnd actor create --campaign <id> --name "Mira" --type character --payload '{"level":5}' --json
sagasmith-dnd actor update --actor <actor-id> --payload '{"attributes":{"hp":{"value":18,"max":18}}}' --json
sagasmith-dnd advancement apply --campaign <id> --actor <actor-id> --payload '{"steps":[{"type":"level","value":2},{"type":"hit_points","increase":6},{"type":"item_grant","item_type":"feat","name":"Action Surge"}]}' --json
sagasmith-dnd advancement grant-class --campaign <id> --actor <actor-id> --class-id fighter --level 2 --json
sagasmith-dnd advancement grant-subclass --campaign <id> --actor <actor-id> --subclass champion --level 3 --json
sagasmith-dnd advancement grant-feature --campaign <id> --actor <actor-id> --feature action-surge --json
sagasmith-dnd advancement grant-feature --campaign <id> --actor <actor-id> --feature second-wind --json
sagasmith-dnd advancement grant-spell --campaign <id> --actor <actor-id> --spell fire-bolt --json
sagasmith-dnd actor create-monster --campaign <id> --monster goblin --json
sagasmith-dnd pack import --campaign <id> --path reference/dnd5e/packs/_source/spells/1st-level --json
sagasmith-dnd game-item create --campaign <id> --actor <actor-id> --name "Longsword" --type weapon --payload '{"equipped":true}' --json
sagasmith-dnd game-activity create --item <item-id> --name "Slash" --type attack --payload '{"activation":{"type":"action"},"system":{"attack_bonus":5,"damage":"1d8+3","damage_type":"slashing"}}' --json
sagasmith-dnd scene create --campaign <id> --name "Cellar" --width 1000 --height 800 --json
sagasmith-dnd scene activate --campaign <id> --scene <scene-id> --json
sagasmith-dnd scene show --scene <scene-id> --json
sagasmith-dnd token create --scene <scene-id> --name "Hero" --actor-type character --actor-id <character-id> --x 0 --y 0 --json
sagasmith-dnd region create --scene <scene-id> --name "Web" --shape '{"type":"circle","x":10,"y":10,"radius":20}' --behavior difficult_terrain --json
sagasmith-dnd region create --scene <scene-id> --name "Arrow Slit" --shape '{"type":"rect","x":90,"y":90,"width":30,"height":30}' --behavior cover --metadata '{"degree":"three_quarters"}' --json
sagasmith-dnd region create --scene <scene-id> --name "Blessing Aura" --shape '{"type":"circle","x":70,"y":70,"radius":30}' --behavior apply_active_effect --metadata '{"statuses":["blessed"]}' --json
sagasmith-dnd combat start --campaign <id> --scene <scene-id> --json
sagasmith-dnd template place --scene <scene-id> --actor <actor-id> --item <item-id> --activity <activity-id> --x 140 --y 210 --json
sagasmith-dnd cover check --scene <scene-id> --token <attacker-token-id> --target-id <target-token-id> --json
sagasmith-dnd roll skill --campaign <id> --actor <actor-id> --skill perception --dc 15 --json
sagasmith-dnd actor prepare --campaign <id> --actor <actor-id> --json
sagasmith-dnd condition add --campaign <id> --actor <actor-id> --condition poisoned --json
sagasmith-dnd effect recalculate --campaign <id> --actor <actor-id> --json
sagasmith-dnd damage apply --campaign <id> --actor <actor-id> --amount 9 --damage-type fire --json
sagasmith-dnd roll save --campaign <id> --actor <actor-id> --ability con --dc 10 --json
sagasmith-dnd concentration fail --campaign <id> --actor <actor-id> --json
sagasmith-dnd activity use --campaign <id> --actor <actor-id> --item <item-id> --activity <activity-id> --target-id <target-actor-id> --json
sagasmith-dnd resolution list --campaign <id> --actor <actor-id> --json
sagasmith-dnd resolution resolve --campaign <id> --id <reaction-window-id> --payload '{"item_id":"<shield-item>","activity_id":"<shield-activity>"}' --json
sagasmith-dnd ready set --campaign <id> --actor <actor-id> --condition "when the goblin leaves cover" --payload '{"activity":"longbow_attack"}' --json
sagasmith-dnd ready trigger --campaign <id> --id <ready-id> --json
sagasmith-dnd combat death-save --campaign <id> --target-id <actor-id> --json
sagasmith-dnd rest short --campaign <id> --actor <actor-id> --payload '{"hit_dice":1}' --json
sagasmith-dnd rest long --campaign <id> --json
sagasmith-dnd time preview --campaign <id> --elapsed PT10M --json
sagasmith-dnd time declare --campaign <id> --elapsed PT10M --reason "searching the room" --intent-id search-room-001 --json
```

Use the document command shape with `--item <item-id> --activity <activity-id>`.
There is no free-form ruleset activity fallback in normal runtime play.

Use ruleset-backed creation before hand-authoring common content: inspect
`ruleset coverage`, use `advancement grant-class` and `advancement grant-subclass`
for progression, `advancement grant-feature` for class features, `advancement
grant-spell` for spells, and `actor create-monster` for ruleset monster
stat blocks. These commands create Actor Items and Activities with stable ruleset
flags so later action economy, duration, reaction, and resource logic can read
structured data instead of prose.

Use `game-item` and `game-activity` for Actor-owned Foundry Item documents and their
executable actions. Do not confuse them with `item ...`, which is the campaign item
ledger for backpack ownership, treasure, currency, containers, and consumables.

When scene tokens are linked to Actor documents, prefer `combat start --scene <id>`
after `actor prepare`. The runtime derives combatants from visible tokens and
prepared Actor data. Do not pass explicit participants; create or update Actor and
Token documents first.

`scene show` and `token show` expose the map state prepared for AI narration and
future UI: Actor summary, HP bar, targetability, token size, position, elevation,
and vision derived from Actor senses. Use those fields rather than estimating map
visibility, health bars, or token footprint from prose.

Use English runtime IDs in commands even during Chinese narration. Chinese labels
may follow fvtt-cn terminology, but command keys and ruleset IDs remain stable
English IDs.

For attack, damage, heal, and saving throw activities, `activity use` may return an
`execution` object. Use that object as the rules result. Do not roll again, reapply
damage, or reinterpret hit point changes from prose.

For cast activities, `activity use` handles spell slots, cantrip scaling by
character level, upcasting by cast spell level, ritual payloads, spell attack and
save DC defaults from Actor spellcasting data, concentration ActiveEffects,
damage/healing/save execution, and pending reaction windows. Do not manually
subtract spell slots or create concentration state in prose.

Activity execution resolves common Foundry roll data references such as `@prof`,
`@mod`, `@abilities.dex.mod`, `@classes.<class>.levels`, and `@item.uses.spent`.
Leave these formulas in structured activity data; do not pre-expand them in prose
unless the runtime returns an unsupported-formula error.

For imported or newly authored Activity documents, prefer Foundry-style structured
`system.damage.parts`, `system.damage.onSave`, `system.healing`, and
`system.save.dc.formula`. The runtime rolls those fields, applies save half/none
semantics, keeps damage part metadata, and applies HP changes. Do not flatten them
into one prose-calculated damage number.

Imported ActiveEffects may keep Foundry numeric change modes. Actor preparation
applies the common numeric modes and priority ordering, so do not rewrite them into
prose or hand-calculate the resulting actor values.

Before rolls, attacks, saves, and damage that depend on equipment or ActiveEffects,
call `actor prepare` if the actor changed since the last preparation. Treat the
returned `derived.effective_system` as the actor's current Foundry-style prepared
data. Do not hand-calculate AC, proficiency, resistance, immunity, vulnerability,
or condition statuses from prose when prepared actor data exists.
Prepared actor data also includes ability modifiers, save totals, skill modifiers,
and passive skill values. Use those prepared values for perception, stealth,
initiative setup, and save/skill narration.

If `activity use` returns a non-empty `pending` array, pause final narration. Call
`resolution list`, then `resolution resolve` or `resolution decline`, and only then narrate
the resolved outcome. A `deferred` activity is not final until its
`continuation_result` returns. Do not silently skip Shield, Counterspell, opportunity attacks,
or similar timing windows.

If `damage apply` returns `concentration_save_required`, immediately call `roll save`
for Constitution at the returned DC. On failure call `concentration fail`; on success
call `concentration pass`.

When a creature at 0 HP makes a death save, call `combat death-save`. The runtime
updates both combat state and the Actor document's death-save fields. Do not track
successes, failures, stable, dead, or natural-20 recovery only in narration.

For area effects, place the activity template before resolving saves or damage. Treat
the returned Region as the authoritative area for narration, token targeting, and
duration/terrain behavior.

For periodic areas, attach structured `metadata.triggers` with an event such as
`declared_minute`, `turn_start_inside`, or `turn_end_inside`, plus `damage`,
`damage_type`, and optional `save`. Timeline period resolution applies these to
the current Tokens inside the Region; never reproduce those rolls in prose.

For attacks or Dexterity saves where obstacles matter, call `cover check` before
choosing the final DC/AC modifier. Total cover prevents direct targeting.

For the Ready action, call `ready set` immediately when the actor readies. When the
trigger occurs, call `ready trigger` before resolving the readied payload; do not
track readied actions only in prose.

If `token move` returns movement `pending`, handle those reaction windows before
describing the creature as safely away. This is the required path for opportunity
attacks unless the move was Disengage, teleportation, forced movement, or another
rule-supported exception.

If `token move` returns `movement.region_effects.created` or `removed`, treat those
ActiveEffect changes as part of the movement result. Apply any needed `actor prepare`
before resolving follow-up rolls, attacks, saves, or damage from the new position.

Short rests may spend hit dice through `rest short --payload '{"hit_dice":n}'`.
Long rests restore HP, spell slots, short/long-rest resources, death saves, and a
portion of spent hit dice. Use the returned `document_recovery` as authoritative.

For leveling and feature grants, use `advancement apply` with structured steps or
the ruleset-backed `advancement grant-class`, `advancement grant-subclass`, and
`advancement grant-feature` helpers. Do not manually edit Actor
system level, hit points, scale values, or class feature items in prose.
