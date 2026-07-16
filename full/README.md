# SagaSmith D&D Skills — Full MCP Runtime

[中文](README.md) · [English](README-en.md) · [仓库说明](../README.md)

Full 模式是 SagaSmithAI 的 D&D 5e 2014/2024 带团工作流。它要求 `sagasmith_dnd` MCP 可用；所有持久化、检索、规则包、模组、角色、分支、知识与战斗状态均由 MCP 服务拥有。

本目录只包含 Agent orchestration：它不会直接访问 SQLite/ChromaDB，不调用本地 D&D CLI，也不会用自然语言假装一次状态变更已经落库。

## 启动顺序

1. 调用 `server_capabilities` 与 `storage_status`，确认服务和存储。
2. 为当前 MCP session 调用 `exposure_open`；身份由 Host 注入时不要让模型填写 principal。同一 session/principal 只有一个当前 exposure，再次 open 会替换旧 id；同阶段所需 group 应加载进同一个 exposure。
3. lobby 任务先 `exposure_search` → `exposure_inspect` → `exposure_load`。
4. 读取 [`references/mcp-contract.md`](references/mcp-contract.md) 与当前 child skill。
5. fresh storage 检查 core rules seed；进入既有团先读取 campaign、branch 与 continuity context。

## 三阶段工具面

| 阶段 | 典型任务 | 不允许的越界 |
|---|---|---|
| `lobby` | 建团、车卡、规则/模组导入、权限、分支、初始知识 | 未安装规则包直接参与结算 |
| `play` | 场景、非战斗检定、资源、事件、记忆、actor knowledge | 活跃战斗中直接改 HP/资源绕过引擎 |
| `combat` | preflight、攻击/法术/反应/移动、选择窗口、临时地图 | 从叙事猜距离/视线或替 GM 选择目标 |

阶段由 campaign state 决定，不由提示词声明。阶段变化后 MCP 会刷新会话 exposure。

## 每轮主持最小闭环

1. 获取 current branch、continuity context、当前 scene 和调用者可见 actor knowledge。
2. 区分玩家陈述、角色意图和仍缺失的规则输入。
3. 必要时 `rule_search`，随后 `rule_expand`；模组同样先 search 再 expand。
4. 让引擎处理确定性 mechanics，让 Agent/GM 裁决目标、视线、例外与叙事代价。
5. 通过受控工具更新 state、scene progress、event、memory 和 actor knowledge。
6. 重大分歧、危险决定、章节切换或战斗前后创建 Snapshot。

## 不可破坏的边界

- 每个 PC、NPC、monster 使用独立完整 Character card；不要把所有人塞入 party JSON。
- actor knowledge 每次显式关联 actor/campaign/branch；玩家不能读取其他玩家私有 scope。
- 不自动合并兄弟分支；读取与检索只沿当前分支祖先链。
- 可重试写入使用最新 revision 与稳定 idempotency key。
- 规则检索只提供证据；campaign 的 core/extension lock 决定可执行规则。
- 战斗写结果始终经过 player-safe projection；隐藏敌人、机制与地图信息不能泄露。
- 模组生成先写 artifact，再 inspect/import；导入失败不丢失可编辑源文件。

## Portable fallback

`../standalone/` 是显式降级模式。MCP 不可用时，先告知能力差异并取得用户同意；不能让 portable 的文件写入冒充 Full campaign state。

详细 schema、错误语义、工具合并和 exposure 流程见 [`references/mcp-contract.md`](references/mcp-contract.md)。
