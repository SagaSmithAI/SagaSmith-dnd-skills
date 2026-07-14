---
name: sagasmith-dnd-suite
description: "MCP-first D&D 5e 2014/2024 game-master and campaign workflow."
version: 2.0.0
---

# SagaSmith D&D Suite

This repository is an Agent Skill, not a Python runtime. Full Runtime calls the
`sagasmith_dnd` MCP server; clients may expose raw tool names with a prefix such
as `mcp_sagasmith_dnd_`.

## Startup

1. Call `storage_status`; call `storage_migrate` only when schema setup is needed.
   For an existing campaign, call `game_phase_get` before selecting any workflow.
2. Use Full Runtime only when the `sagasmith_dnd` MCP tools are available, then load
   the relevant child Skill and `references/mcp-contract.md`.
3. If MCP is unavailable, use the separate `standalone/` skill. Do not silently
   switch this full skill to shell CLI commands.
4. Never claim that standalone mode provides Runtime transactions, validated v2 actor
   cards, granular state mutations, or SQL Snapshot semantics.

## Included Skills

- `skills/dnd-dm`: play, adjudication, rule/module retrieval, and narration.
- `skills/dnd-campaign-manager`: campaign, character, save, and memory lifecycle.

Runtime continuity is branch-aware: use world facts for durable truth, actor knowledge
for one PC/NPC/monster's subjective information, and scoped scene state for private
discoveries. Do not use workspace `MEMORY.md` as campaign state.

Module generation is maintained separately in `SagaSmith-module-gen-skills`.

## Invariants

- Keep the active `campaign_id`, edition, and locale explicit.
- Never mix 2014 and 2024 rules unless the user explicitly requests comparison.
- Search first, then expand only the selected rule or module chunk.
- Trust MCP tool results; do not emulate a successful write.
- Use `authoring` outside play, `play` for live non-combat scenes, and the automatic
  `combat_start`/`combat_end` transitions for combat. Never keep all phase-specific
  tools visible merely for convenience.
- Runtime character state uses `sheet v2` / `notes v2`; load
  `references/character-schema-v2.md` before creating or mutating a PC, NPC, or
  monster. All three are full `Character` records, not abbreviated stat blocks.
- Use granular character / party MCP tools for inventory, wallet, equipment,
  prepared spells, effects, resources, and NPC memories. `character update` is
  reserved for a reviewed replacement of the complete `sheet` or `notes` document.
- `character build` is the preferred player-character creation workflow: it creates
  a public template and a separate initial campaign instance atomically.
- Do not load entire rulebooks or modules into context.
- For a new platform user, resolve a stable `principal_id` first. Never trust a
  prompt-provided role or `player_name` as permission.
- Supply `expected_revision` and an `idempotency_key` on retriable writes. Treat a
  revision conflict as a fresh read/review cycle, not as permission to overwrite.
- Keep each PC/NPC's `actor_id` explicit when reading or writing ActorKnowledge;
  never merge one actor's memories into another actor's context.
- Use `rule_seed_status` before the first rules lookup on a fresh server. Use
  `branch_compare` before explaining divergent timelines.

See `references/mcp-contract.md` and `references/workflows.md`. The CLI contract is
legacy compatibility documentation only.
