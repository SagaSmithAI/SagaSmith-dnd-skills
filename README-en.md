# 🐉 SagaSmith D&D Skills

[中文](README.md) | [English](README-en.md)

<p align="center"><img src="full/images/Sagasmith.png" alt="SagaSmith" width="150"></p>

**Cross-platform D&D 5e 2014/2024 Agent Skills** — provide Dungeon Master capabilities to Claude Code, NanoBot, Codex, Cursor, OpenClaw, Hermes, and any agent platform supporting the SKILL.md standard.

This repository is a **SKILL definition package** — it contains no database, game engine, or platform-specific tools. It ships two variants — **you choose** which to install:

| Variant | Directory | Dependency | When to use |
|---------|-----------|------------|-------------|
|---------|-----------|------------|-------------|
| 📦 Full | `full/` | `sagasmith-dnd` Python package | Persistent campaigns, PDF import, FTS5 search, Snapshot DAG |
| 🪶 Standalone | `standalone/` | Python 3.11+ (stdlib only) | Instant setup, file-based storage, zero pip deps |

---

## 📦 full/ — Full Runtime (Recommended)

Requires the `sagasmith-dnd` CLI on the local machine. All operations via `sagasmith-dnd --json` commands.

```bash
pip install "sagasmith-dnd[documents]"
sagasmith-dnd doctor --json
```

Load `full/SKILL.md`.

### Features

- 🏛️ **Campaign Management** — SQLite persistence, Alembic migrations, FTS5 full-text search
- 📖 **Module Import** — PDF/Markdown parsing, structure-aware chunking, scene indexes, bilingual CN/EN scene merging
- 🎲 **Combat Engine** — d20 rolls, initiative/hit/damage/save/crit, turn tracking, XP
- 🔍 **Rule Retrieval** — 3-layer hybrid search (exact + FTS5 BM25 + semantic), optional ChromaDB acceleration
- 🧩 **Scene Progress** — scoped to `party`/`group:<id>`/`player:<id>` with transparent inheritance
- 💾 **Snapshot** — immutable DAG save tree, branch-aware restore, campaign memory
- 🌐 **Bilingual Expansion** — Chinese queries auto-expand with English equivalents ("豁免" → "saving throw")

### Ecosystem

| Repo | Role |
|------|------|
| ⚔️ [sagasmith-dnd](https://github.com/SagaSmithAI/Sagasmith-dnd) | D&D 5e runtime + CLI |
| 🏗️ [sagasmith-core](https://github.com/SagaSmithAI/Sagasmith-core) | General engine — DB, docs, RAG, FTS5 |
| 🎲 [SagaSmith-agent](https://github.com/SagaSmithAI/SagaSmith-agent) | Complete AI DM runtime (NanoBot-based) |
| ✍️ [SagaSmith-module-gen-skills](https://github.com/SagaSmithAI/SagaSmith-module-gen-skills) | Adventure module generator (25 paradigms) |

### Skill Inventory

| Skill | Path | Role |
|-------|------|------|
| 🎲 **dnd-dm** | `full/skills/dnd-dm/SKILL.md` | Core DM persona (always-on), rule adjudication, combat, SRD retrieval |
| 📋 **dnd-campaign-manager** | `full/skills/dnd-campaign-manager/SKILL.md` | Campaign lifecycle, Snapshot save/load, module import |

### Rules Corpus

| Corpus | Edition | Locale |
|--------|---------|--------|
| SRD 5.2.1 | 2024 | EN |
| SRD 5.1 | 2014 | EN |
| SRD 5.1 | 2014 | ZH-CN |

---

## 🪶 standalone/ — Lightweight

No pip dependencies. Python 3.11+ stdlib only.

Switch to the `standalone/` directory and run:

```powershell
python portable.py doctor
python portable.py campaign start --name "Campaign" --edition 2024
python portable.py roll check --dc 15 --score 16 --proficient --level 5
```

Load `standalone/SKILL.md`.

Data is stored at `~/.sagasmith/`, all plain text (JSON / Markdown / JSONL).

### Supported Operations

| Command | Coverage |
|---------|----------|
| Campaign | `campaign start/list/get` |
| Characters | `character create/list/get` |
| Modules | `module ingest/index/current/read-scene/search/set-progress` |
| Rules | `rules search` — bundled SRD lexical search with CN/EN expansion |
| Dice | `roll dice/check/attack` — d20 with proficiency/advantage |
| Events | `event add/list` |
| Memory | `memory add/list/search` |
| Saves | `save create/list/restore` — full directory snapshots |

### Limitations

- ❌ No PDF import (convert to Markdown first)
- ❌ No FTS5 full-text index (Python lexical scoring)
- ❌ No ChromaDB semantic search
- ❌ No transaction atomicity (single-file atomic write, multi-file best-effort)

---

## DM State Machine

Each player action:

1. **Scope resolution** — select `party` / `group:<id>` / `player:<id>`, call `module current`
2. **Scene reading** — read that scope's current scene; always `expand` before using search candidates
3. **Type-based execution** — `intro` setups / `combat` engagements / `dungeon` subsection progression / `transition` condition checks
4. **State merge** — read existing `progress.state`, merge new facts, write back atomically
5. **Record & save** — event log, campaign memory, snapshot at decision points

---

## Agent Install Flow

```powershell
# 1. Check if sagasmith-dnd is available
sagasmith-dnd doctor --json 2>nul

# 2a. Available → Full mode
#     Load full/SKILL.md
#     All commands use sagasmith-dnd xxx --json

# 2b. Not available → Standalone mode
#     Switch to standalone/ directory
#     Load SKILL.md
#     All commands use python portable.py xxx
```

---

## License

MIT
