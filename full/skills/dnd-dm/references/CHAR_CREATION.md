# 角色创建与升级

## 创建流程

1. 读取战役 edition、locale、可用 publication 和可选规则。
2. 确认玩家概念、等级及是否使用预设、标准数组、购点或掷骰。所有随机结果通过 CLI。
3. 按版本依次选择物种/种族、职业、背景、属性分配、熟练项、语言、工具、装备、
   职业特定选项与法术。每一步只展示该规则档案允许的选项。
4. 计算熟练加值、豁免、技能、AC、HP、速度、攻击、法术 DC/攻击和资源上限。
5. 补充姓名、外观、动机、关系、信念及玩家名。
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

角色 sheet 至少包含 abilities、level、armor_class、hit_points、
max_hit_points、class、species、background、proficiencies、inventory 和 spells。

## 背包与物品账本

`sheet.inventory` 只作为创建/导入时的兼容入口。正式角色创建后，背包由
`sagasmith-dnd item ... --json` 维护；不要再通过 `character update --sheet`
手动改背包。

标准物品对象字段：

```json
{
  "name": "Potion of Healing",
  "quantity": 2,
  "category": "consumable",
  "equipped_slot": null,
  "attunement": "none",
  "identified": true,
  "charges": {},
  "condition": "normal",
  "state": {
    "source": "starting equipment"
  },
  "template": {
    "source_key": "srd:potion-healing",
    "rarity": "common",
    "value": {"gp": 50},
    "weight": 0
  }
}
```

金币使用 `category="currency"`，并在模板或 state 中记录 denomination：
`cp`、`sp`、`ep`、`gp`、`pp`。装备槽位使用稳定英文键，例如 `main_hand`、
`off_hand`、`armor`、`shield`、`ring_1`、`ring_2`、`neck`、`belt`、`pack`。

创建角色时可在 `--sheet` 中附带初始 `inventory`；CLI 会导入物品账本并返回
`inventory_managed=true`。之后拾取、购买、转移、装备、消耗、同调、鉴定都必须使用
`item` 子命令。
