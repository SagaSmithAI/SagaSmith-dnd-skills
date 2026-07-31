# SagaSmith D&D Skills — Full MCP Runtime

[中文](README.md) · [English](README-en.md) · [Repository overview](../README.md)

Full mode is the SagaSmithAI D&D 5e 2014/2024 table-running workflow. It requires the `sagasmith_dnd` MCP server. Persistence, retrieval, rule packs, modules, characters, branches, knowledge, and combat state all belong to that server.

This directory contains agent orchestration only. It never writes SQLite/ChromaDB directly, calls a local D&D CLI, or pretends a natural-language state change has been committed.

## Startup

1. Call `skill_query(kind="skill", action="plan")` and read every `required_now` fragment.
2. Call `server_capabilities` and `storage_status`.
3. Open a session exposure with `exposure_open`; do not model-supply a principal when the Host injects identity. One session/principal has one current exposure, reopening replaces the old id, and same-phase groups belong in that one exposure.
4. For lobby work, use `exposure_search` → `exposure_inspect` → `exposure_load` and read each returned `skill_plan_delta`.
5. Search or bounded-read [`references/mcp-contract.md`](references/mcp-contract.md) and child Skills only for task-specific depth; do not load them in full by default.
6. On fresh storage, verify core-rule seed state. For an existing campaign, call `campaign_query(view="resume")` first and read its phase plan, campaign, branch, and continuity context.

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
4. Let the engine settle deterministic mechanics and let the SagaSmith Agent perform ordinary GM rulings on targets, sight, exceptions, and narrative cost. Stop externally only for player-owned choices, owner approvals, permission changes, or missing/conflicting source evidence.
5. Use controlled tools for state, scene progress, events, memory, and actor knowledge.
6. Snapshot major divergence, danger, chapter transitions, and combat boundaries.

## Non-negotiable boundaries

- Every PC, NPC, and monster has an independent complete Character card.
- Named NPC portrayal uses a signed actor-scoped bundle and, when the host can
  enforce it, a fresh zero-tool non-persistent model call. The proposal still
  needs public mechanical resolution and an explicit MCP continuity commit.
- Actor knowledge is explicitly scoped to actor/campaign/branch; players cannot read another player's private scope.
- Sibling branches never auto-merge; reads and retrieval follow active ancestry.
- Retriable writes use current revisions and stable idempotency keys.
- Retrieval supplies evidence; the campaign's core/extension lock determines executable rules.
- Combat results always use the player-safe projection; hidden actors, mechanics, and map data remain hidden.
- Generated modules become editable artifacts before inspection and import.

`../standalone/` is an explicit fallback, not a transparent replacement. See [`references/mcp-contract.md`](references/mcp-contract.md) for schemas, error semantics, grouped tools, and the exposure protocol.
