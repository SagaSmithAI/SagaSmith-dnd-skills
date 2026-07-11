# Runtime 数据契约

Skill 不直接访问数据库。所有写入通过 `sagasmith-dnd --json`。

- SQL 是真实数据源。
- 规则、模组和 embedding 是静态索引。
- 战役、角色、场景进度、快照和记忆是可变状态。
- 角色的 `sheet v2` 是数值、资源、法术、效果和个人背包的权威；`notes v2` 是描述、
  重要对话记忆、关系和目标的权威。
- `character show` 的 `derived` 是只读计算结果，不能写回或手改。
- 队伍共享背包和钱包位于 `campaign.state.party.inventory`，且与个人背包使用相同结构。
- 物品、钱包、装备、准备法术、效果和 NPC 记忆必须使用对应 `character`/`party` CLI
  命令；不得用 `character update --sheet` 局部篡改。
- 快照不复制规则或模组原文。
- Dense 不可用时 Runtime 自动保留 exact/lexical 检索。
- 每个战役的 edition、locale 和 publications 由 rule profile 决定。

完整字段见 `../../../references/character-schema-v2.md`。
