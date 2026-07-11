# 角色创建与升级

## 创建流程

1. 读取战役 edition、locale、可用 publication 和可选规则。
2. 确认玩家概念、等级及是否使用预设、标准数组、购点或掷骰。所有随机结果通过 CLI。
3. 按版本依次选择物种/种族、职业、背景、属性分配、熟练项、语言、工具、装备、
   职业特定选项与法术。每一步只展示该规则档案允许的选项。
4. 填写 `sheet v2`：完整属性、技能、战斗数值、特质、资源、法术、特性、效果和背包。
   法术使用 `spellcasting` 与 `content.spells`，将当天已准备法术写入
   `preparation.selected_spell_ids`；法术书、始终准备和仪式资格不能混为一项。
5. 填写 `notes v2`：姓名、外观、动机、关系、信念及玩家名；NPC 必须有一段 summary，
   怪物也应有简短的外观/行为描述。
6. 展示完整草稿和来源；玩家最终确认前不得创建正式角色。
7. 创建后重新读取并验证派生值、资源上限和 campaign/player 绑定。

新玩家使用逐步对话；熟悉规则的玩家可一次提交草稿，但仍须逐项验证并最终确认。
NPC 可以不绑定战役作为资料库角色；进入某战役时再建立引用或战役实例。

## 升级

从当前角色增量升级，不得重建并覆盖既有选择。依次处理 HP、职业/子职能力、熟练加值、
技能与豁免、专长或属性提升、法术与法术位、装备资源及所有派生值。验证后提交，记录
升级事件并创建 Snapshot。

```powershell
sagasmith-dnd character create --campaign <id> --name "<name>" --type pc --player "<player>" --sheet '<json>' --json
sagasmith-dnd character list --campaign <id> --json
sagasmith-dnd character show --id <character-id> --json
```

角色 sheet 必须是 `schema_version: 2`。创建后执行 `character show`，核对完整 `sheet`
和只读 `derived` 中的熟练、豁免、技能、AC、HP、速度、法术 DC、已准备法术和负重。
字段与草稿例子见 `../../../references/character-schema-v2.md`。
