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

- 开战时先创建或读取 Actor，放置 Actor-linked Token，再用 `combat start --scene`
  从场景启动战斗；不要传 free-form participants。
- Runtime 模式下，战斗状态必须通过 `sagasmith-dnd combat ... --json` 维护。
  每次战斗叙事前先读取 `combat status`，按返回的 `current`、`legal_actions`、
  `legal_action_details`、`turn_budget` 和 `reaction_windows` 主持当前回合；
  不得凭聊天历史手改轮次、HP、条件或行动经济。
- 每回合依序处理回合开始效果、移动、动作、附赠动作、反应与回合结束效果。
- 常规行动必须优先通过 `activity use`；Action Surge、Second Wind、Extra Attack、
  bonus action、reaction、opportunity attack、Shield 等易错内容交给结构化运行时。
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

Use ruleset-backed creation before hand-authoring common content:
`advancement grant-feature` for core class features, `advancement grant-spell`
for core spells, and `actor create-monster` for ruleset monster stat blocks.

Common runtime commands:

```powershell
sagasmith-dnd ruleset validate --id dnd5e-2014 --json
sagasmith-dnd actor create --campaign <id> --name "Mira" --type character --payload '{"level":5}' --json
sagasmith-dnd advancement grant-feature --campaign <id> --actor <actor-id> --feature action-surge --json
sagasmith-dnd advancement grant-spell --campaign <id> --actor <actor-id> --spell fire-bolt --json
sagasmith-dnd actor create-monster --campaign <id> --monster goblin --json
sagasmith-dnd scene create --campaign <id> --name "Cellar" --width 1000 --height 800 --json
sagasmith-dnd scene activate --campaign <id> --scene <scene-id> --json
sagasmith-dnd token create --scene <scene-id> --name "Hero" --actor-type character --actor-id <actor-id> --x 0 --y 0 --json
sagasmith-dnd combat start --campaign <id> --scene <scene-id> --json
sagasmith-dnd combat status --campaign <id> --json
sagasmith-dnd activity use --campaign <id> --actor <actor-id> --item <item-id> --activity <activity-id> --target-id <target-actor-id> --json
sagasmith-dnd resolution list --campaign <id> --actor <actor-id> --json
sagasmith-dnd resolution resolve --campaign <id> --id <reaction-window-id> --payload '{"item_id":"<shield-item>","activity_id":"<shield-activity>"}' --json
sagasmith-dnd token move --token <token-id> --x 30 --y 20 --json
sagasmith-dnd time declare --campaign <id> --elapsed PT1M --reason "declared time" --intent-id declared-minute-001 --json
sagasmith-dnd rest short --campaign <id> --actor <actor-id> --payload '{"hit_dice":1}' --json
sagasmith-dnd rest long --campaign <id> --json
```

If `activity use` or `token move` returns pending reactions, resolve or decline
them before final narration. Wall-clock time, chat delay, and LLM processing time
never advance durations; only declared runtime periods do.
