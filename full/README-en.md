# 🐉 SagaSmith D&D Skills

[中文](README.md) | [English](README-en.md)

<p align="center"><img src="images/Sagasmith.png" alt="SagaSmith" width="200"></p>

**Cross-platform D&D 5e 2014/2024 Agent Skills** — provide Dungeon Master capabilities to Claude Code, NanoBot, Codex, Cursor, and any agent platform supporting the SKILL.md standard.

> *"The dice have been cast."*

Runtime mode requires the `sagasmith_dnd` MCP server. The MCP server owns the
database, vector index, bundled rules, module artifacts, and durable state; this
repository contains the orchestration skills and does not call a local D&D CLI.

---

## Ecosystem

| Repo | Role |
|------|------|
| 📦 **SagaSmith-dnd-skills** (this repo) | D&D agent skill definitions |
| ⚔️ [SagaSmith-dnd-mcp](https://github.com/SagaSmithAI/SagaSmith-dnd-mcp) | D&D MCP runtime and owned storage |
| 🧮 [sagasmith-dnd](https://github.com/dajiaohuang/sagasmith-dnd) | D&D schema and rules library |
| 🏗️ [sagasmith-core](https://github.com/dajiaohuang/sagasmith-core) | General engine — DB, docs, RAG |
| 🎲 [SagaSmith-agent](https://github.com/dajiaohuang/SagaSmith-agent) | Complete AI DM runtime |
| ✍️ [SagaSmith-module-gen-skills](https://github.com/dajiaohuang/SagaSmith-module-gen-skills) | Module generator |

---

## Skill Inventory

| Skill | File | Role |
|-------|------|------|
| 🎲 **dnd-dm** | `skills/dnd-dm/SKILL.md` | Core DM persona (always-on), rule adjudication, combat engine, SRD retrieval, scoped scene tracking |
| 📋 **dnd-campaign-manager** | `skills/dnd-campaign-manager/SKILL.md` | Campaign lifecycle, Snapshot save/load, module import, USER.md sync |

---

## Usage

### Runtime Mode (persistence)

```text
Call module_current(campaign_id, scope_id, principal_id),
module_set_progress(...), and snapshot_create(...) through the D&D MCP.
Use rule_search -> rule_expand and module_search -> module_read_scene for facts.
```

### Portable Mode (no installation)

Without the Runtime, the bundled SRD files remain available for static, non-persistent reference — useful for quick rule lookups and session preparation.

---

## Rules Corpus

| Corpus | Edition | Locale |
|--------|---------|--------|
| SRD 5.2.1 | 2024 | EN |
| SRD 5.1 | 2014 | EN |
| SRD 5.1 | 2014 | ZH-CN (convenience translation) |

---

## DM State Machine

Each player action:

1. **Scope resolution** — select `party` / `group:<id>` / `player:<id>` for the acting player, call `module current`
2. **Scene reading** — read that scope's current scene; search candidates only when needed, always `expand` before using
3. **Type-based execution** — `intro` setups / `combat` engagements / `dungeon` subsection progression / `transition` condition checks
4. **State merge** — read existing `progress.state`, merge new facts, write back atomically
5. **Record & save** — event log, memory, snapshot at decision points

---

## Quick Install

```bash
# Claude Code / Codex / Cursor
npx skills add dajiaohuang/SagaSmith-dnd-skills
```

---

## License

MIT
