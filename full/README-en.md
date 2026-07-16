# SagaSmith D&D Skills — Full MCP Runtime

[中文](README.md) · [English](README-en.md) · [Repository overview](../README.md)

Full mode is the SagaSmithAI D&D 5e 2014/2024 table-running workflow. It requires the `sagasmith_dnd` MCP server. Persistence, retrieval, rule packs, modules, characters, branches, knowledge, and combat state all belong to that server.

This directory contains agent orchestration only. It never writes SQLite/ChromaDB directly, calls a local D&D CLI, or pretends a natural-language state change has been committed.

## Startup

1. Call `server_capabilities` and `storage_status`.
2. Open a session exposure with `exposure_open`; do not model-supply a principal when the Host injects identity. One session/principal has one current exposure, reopening replaces the old id, and same-phase groups belong in that one exposure.
3. For lobby work, use `exposure_search` → `exposure_inspect` → `exposure_load`.
4. Read [`references/mcp-contract.md`](references/mcp-contract.md) and the relevant child skill.
5. On fresh storage, verify core-rule seed state. For an existing campaign, read campaign, branch, and continuity context first.

## Phase surfaces

| Phase | Typical work | Forbidden shortcut |
|---|---|---|
| `lobby` | campaigns, characters, rules/modules, access, branches, initial knowledge | settling from an uninstalled rule pack |
| `play` | scenes, checks, resources, events, memory, actor knowledge | mutating combat state through ordinary character writes |
| `combat` | preflight, attacks/spells/reactions/movement, choices, temporary map | inventing targets, sight lines, or distance |

Campaign state, not the prompt, owns the phase. The MCP refreshes session exposure when the phase changes.

## Minimal turn loop

1. Read the active branch, continuity context, current scene, and caller-visible actor knowledge.
2. Separate player statement, character intent, and missing rules inputs.
3. Use `rule_search` then `rule_expand`; use the same search/expand pattern for modules.
4. Let the engine settle deterministic mechanics and the agent/human GM rule on targets, sight, exceptions, and narrative cost.
5. Use controlled tools for state, scene progress, events, memory, and actor knowledge.
6. Snapshot major divergence, danger, chapter transitions, and combat boundaries.

## Non-negotiable boundaries

- Every PC, NPC, and monster has an independent complete Character card.
- Actor knowledge is explicitly scoped to actor/campaign/branch; players cannot read another player's private scope.
- Sibling branches never auto-merge; reads and retrieval follow active ancestry.
- Retriable writes use current revisions and stable idempotency keys.
- Retrieval supplies evidence; the campaign's core/extension lock determines executable rules.
- Combat results always use the player-safe projection; hidden actors, mechanics, and map data remain hidden.
- Generated modules become editable artifacts before inspection and import.

`../standalone/` is an explicit fallback, not a transparent replacement. See [`references/mcp-contract.md`](references/mcp-contract.md) for schemas, error semantics, grouped tools, and the exposure protocol.
