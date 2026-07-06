# 🐉 SagaSmith D&D Skills

[中文](README.md) | [English](README-en.md)

<p align="center"><img src="full/images/Sagasmith.png" alt="SagaSmith" width="150"></p>

**跨平台 D&D 5e 2014/2024 Agent Skills** — 为 Claude Code、NanoBot、Codex、Cursor、OpenClaw、Hermes 等所有支持 SKILL.md 标准的 AI Agent 平台提供完整的地下城主持能力。

本仓库是一个 **SKILL 定义包**，不包含数据库、游戏引擎或平台专属工具。它提供两种使用方式，Agent 请根据本机环境**自动选择**：

| 方式 | 目录 | 依赖 | 适用场景 |
|------|------|------|---------|
| 📦 完整版 | `full/` | `sagasmith-dnd` Python 包 | 持久化战役、PDF 导入、FTS5 搜索、Snapshot DAG |
| 🪶 轻量版 | `standalone/` | Python 3.11+（标准库） | 即装即用，纯文件系统，零依赖 |

---

## 📦 full/ — 完整版（推荐）

要求本机安装 `sagasmith-dnd` CLI。所有操作通过 `sagasmith-dnd --json` 命令完成。

```bash
pip install "sagasmith-dnd[documents]"
sagasmith-dnd doctor --json
```

加载 `full/SKILL.md`。

### 特性

- 🏛️ **战役管理** — SQLite 持久化、Alembic 迁移、FTS5 全文检索
- 📖 **模组导入** — PDF/Markdown 解析、结构感知分块、场景索引、中英双语场景合并
- 🎲 **战斗引擎** — d20 掷骰、先攻/命中/伤害/豁免/暴击、回合追踪、XP 计算
- 🔍 **规则检索** — 三层混合搜索（精确 + FTS5 BM25 + 语义），ChromaDB 可选加速
- 🧩 **场景进度** — `party`/`group:<id>`/`player:<id>` 作用域追踪，透明继承
- 💾 **Snapshot** — 不可变 DAG 存档树、分支读档、战役记忆
- 🌐 **中英双解** — 中文查询自动扩展英文等价物（"豁免" → "saving throw"）

### 生态

| 仓库 | 定位 |
|------|------|
| ⚔️ [sagasmith-dnd](https://github.com/SagaSmithAI/Sagasmith-dnd) | D&D 5e 运行时 + CLI |
| 🏗️ [sagasmith-core](https://github.com/SagaSmithAI/Sagasmith-core) | 通用引擎 — DB、文档、RAG、FTS5 |
| 🎲 [SagaSmith-agent](https://github.com/SagaSmithAI/SagaSmith-agent) | 完整 AI DM 运行时（基于 NanoBot） |
| ✍️ [SagaSmith-module-gen-skills](https://github.com/SagaSmithAI/SagaSmith-module-gen-skills) | 冒险模组生成器（25 种范式） |

### Skill 清单

| Skill | 路径 | 职责 |
|-------|------|------|
| 🎲 **dnd-dm** | `full/skills/dnd-dm/SKILL.md` | 核心 DM 人格（always-on）、规则裁判、战斗引擎、SRD 检索 |
| 📋 **dnd-campaign-manager** | `full/skills/dnd-campaign-manager/SKILL.md` | 战役生命周期、Snapshot 存档/读档、模组导入 |

### 规则语料

| 语料 | 版本 | 语言 |
|------|------|------|
| SRD 5.2.1 | 2024 | EN |
| SRD 5.1 | 2014 | EN |
| SRD 5.1 | 2014 | ZH-CN |

---

## 🪶 standalone/ — 轻量版

不需要安装任何 Python 包。Python 3.11+ 标准库即可运行。

切换到 `standalone/` 目录后操作：

```powershell
python portable.py doctor
python portable.py campaign start --name "Campaign" --edition 2024
python portable.py roll check --dc 15 --score 16 --proficient --level 5
```

加载 `standalone/SKILL.md`。

数据存储在 `~/.sagasmith/`，全部为纯文本文件（JSON / Markdown / JSONL）。

### 支持的操作

| 命令组 | 覆盖能力 |
|--------|---------|
| 战役 | `campaign start/list/get` |
| 角色 | `character create/list/get` |
| 模组 | `module ingest/index/current/read-scene/search/set-progress` |
| 规则 | `rules search` — 捆绑 SRD 词法搜索，中英扩展 |
| 掷骰 | `roll dice/check/attack` — d20 加熟练/优势 |
| 事件 | `event add/list` |
| 记忆 | `memory add/list/search` |
| 存档 | `save create/list/restore` — 全目录快照 |

### 限制

- ❌ 不支持 PDF 导入（需用户先转为 Markdown）
- ❌ 不支持 FTS5 全文索引（使用 Python 词法评分）
- ❌ 不支持 ChromaDB 语义搜索
- ❌ 无事务原子性（单文件 atomic write，多文件 best-effort）

---

## 带团状态机

每次处理玩家行动：

1. **Scope 决议** — 根据行动者选择 `party` / `group:<id>` / `player:<id>`，调用 `module current`
2. **场景读取** — 读取该 scope 的当前场景；搜索候选后务必 `expand` 再使用
3. **按类型执行** — `intro` 铺垫 / `combat` 开战 / `dungeon` 按 subsection 推进 / `transition` 核验条件
4. **State 合并** — 先读取 `progress.state`，合并新事实后整体写回，不删除未知键
5. **记录与保存** — 事件日志、战役记忆、关键节点创建 Snapshot

---

## Agent 安装流程

```powershell
# 1. 检测 sagasmith-dnd 是否可用
sagasmith-dnd doctor --json 2>nul

# 2a. 可用 → 完整版
#     加载 full/SKILL.md
#     所有命令使用 sagasmith-dnd xxx --json

# 2b. 不可用 → 轻量版
#     切换到 standalone/ 目录
#     加载 SKILL.md
#     所有命令使用 python portable.py xxx
```

---

## 许可

MIT
