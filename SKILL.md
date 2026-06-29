---
name: sagasmith
description: "Autonomous D&D 5e AI Dungeon Master — campaign lifecycle management with delta recaps and campaign-scoped long-term memory, module generation (5 types × 25 paradigms), rule adjudication, and immersive DM persona (Minthara Baenre). Bundles dnd-dm, dnd-campaign-manager, and dnd-module-gen skills."
version: 1.0.0
tags:
  - dnd
  - ttrpg
  - dungeon-master
  - campaign
  - module-generator
  - gaming
---

# SagaSmith — Autonomous D&D 5e AI Dungeon Master

Saga = epic campaign / Smith = autonomous creator. A cross-platform D&D DM skill suite.

## Skills Included

| Skill | Slug | Role |
|-------|------|------|
| **dnd-dm** | `sagasmith-dm` | Core DM persona, rule adjudication, dice engine, combat, SRD retrieval |
| **dnd-campaign-manager** | `sagasmith-campaign` | Campaign lifecycle, snapshot recap, campaign memory, save/load/undo, USER.md player mapping |
| **dnd-module-gen** | `sagasmith-modulegen` | Adventure module generation: one-shot / short / medium / long / sandbox × 25 paradigms |

## DM Persona

**明萨拉·班瑞 (Minthara Baenre)** — Lawful Evil, rules-absolutist, cold wit.
Strict adherence to 2024 rulebooks. Dice are the judge; the module script is inviolable.

## Platforms

Install on NanoBot, OpenClaw, or Hermes. See [README.md](README.md) for per-platform setup.

## SRD

Includes D&D 5e 2024 SRD 5.2.1 (CC-BY-4.0, 20 files). English and optional Chinese translation.

### Retrieval Architecture

```
Search (default, zero dependency)
├─ Exact match   ← keyword exact match, highest weight
├─ Lexical       ← tokenization + bigram matching
└─ Dense vectors — optional  ← requires ChromaDB + BGE-M3

First-time setup
├─ database.upgrade_schema()            ← automatic
└─ ensure_bundled_rules_ingested()     ← auto-ingest SRD (text only)
    └─ embed = False (default)        ← BGE-M3 not loaded
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CHROMA_DB_DISABLED` | disabled (unset) | Set to `0` to enable ChromaDB (uses `<skill>/data/chroma_db/`) |
| `CHROMA_DB_URL` | - | Remote ChromaDB server address (auto-enables when set) |
| `CHROMA_DB_PATH` | - | Custom ChromaDB path (auto-enables when set) |
| `DND_DENSE_DISABLED` | disabled (`=1`) | Set to `0` to enable BGE-M3 dense vectors (slow on non-GPU) |
| `DND_DATABASE_URL` | `<skill>/data/dnd.db` | SQLite database path (overrides default) |

### Recommended Configurations

```bash
# Zero dependency, pure lexical search (ready immediately after first use)
# No environment variables needed

# With ChromaDB but no GPU: embeddings stored in SQLite for NumPy fallback
set CHROMA_DB_DISABLED=0

# Full dense vector experience with GPU + ChromaDB
set CHROMA_DB_DISABLED=0
set DND_DENSE_DISABLED=0
```

## Repository

https://github.com/dajiaohuang/SagaSmith-skills
