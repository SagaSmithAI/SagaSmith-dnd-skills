# SagaSmith D&D Skills（Full MCP Runtime）

Full 模式是 MCP-first 的 D&D 5e 2014/2024 带团技能包。持久化状态、规则索引、
模组制品、角色卡、分支和角色认知全部由 `sagasmith_dnd` MCP 服务负责。

本目录是 Agent Skill，不是 Python runtime，也不直接访问 SQLite、Chroma 或本地 CLI。

## 仓库职责

| 仓库 | 职责 |
|---|---|
| `SagaSmith-dnd-skills` | DM 判断、叙事与 MCP 调用编排 |
| `SagaSmith-dnd-mcp` | MCP 工具、状态存储、规则和模组读取 |
| `sagasmith-core` | 分支、快照、事务、权限和 ActorKnowledge |
| `sagasmith-dnd` | 角色 schema、D&D 规则库和运行时模型 |
| `SagaSmith-module-gen-skills` | 生成可导入的模组制品 |

## 启动

1. 确认 `sagasmith_dnd` MCP 可用，调用 `server_capabilities`。
2. 调用 `storage_status`；fresh storage 使用 `rule_seed_status` 检查规则。
3. 解析平台用户的稳定 `principal_id`，不要把 `player_name` 当作权限。
4. 读取 `full/references/mcp-contract.md` 和对应 child skill。

## 运行原则

- 规则：`rule_search` → `rule_expand`。
- 模组：`module_search` → `module_expand` 或 `module_read_scene`。
- 角色：每个 PC/NPC/monster 都使用独立的完整 `Character` card。
- 认知：每次读写都显式提供 `actor_id`、`campaign_id`、`branch_id`。
- 写入：可重试的操作提供 `expected_revision` 和 `idempotency_key`。
- 分支：用 `branch_compare` 检查差异，不自动合并兄弟时间线。
- 玩家视图：使用带 `principal_id` 的 MCP 读取，让服务端过滤 keeper 内容。
- 战斗：当前仍使用可审计的通用状态工具；结构化战斗引擎暂不在本版本范围内。

## Portable 模式

`standalone/` 是独立的无持久化参考流程。MCP 不可用时必须显式切换到该模式，
不能假装 portable 写入了 Runtime 状态。

详细工具字段和错误语义见 `references/mcp-contract.md`。
