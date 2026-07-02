# Runtime 数据契约

Skill 不直接访问数据库。所有写入通过 `sagasmith-dnd --json`。

- SQL 是真实数据源。
- 规则、模组和 embedding 是静态索引。
- 战役、角色、场景进度、快照和记忆是可变状态。
- 快照不复制规则或模组原文。
- Dense 不可用时 Runtime 自动保留 exact/lexical 检索。
- 每个战役的 edition、locale 和 publications 由 rule profile 决定。
