---
name: sagasmith-dnd-suite
description: "Run or maintain D&D 5e 2014/2024 campaigns through SagaSmith's MCP-first game-master, actor, module, continuity-memory, and snapshot workflows. Use for live play, campaign setup, character management, module import, rules adjudication, durable facts, actor knowledge, branches, saves, and restores."
---

# SagaSmith D&D Suite

This repository is an Agent Skill, not a Python runtime. Full Runtime calls the
`sagasmith_dnd` MCP server; clients may expose raw tool names with a prefix such
as `mcp_sagasmith_dnd_`.

## Startup

1. Call `storage_status`; call `storage_migrate` only when schema setup is needed.
   Start every MCP session with `exposure_open`, then use `exposure_search`,
   `exposure_inspect`, and `exposure_load` for the current campaign phase. Before
   a campaign exists, load only `lobby.bootstrap`; reopen the exposure with the
   returned `campaign_id` before loading campaign-bound groups. There is one active
   exposure per MCP session/principal: calling `exposure_open` again replaces it.
   Load every compatible group needed for the current phase into that one exposure;
   never retain or call an older exposure id.
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
discoveries. Read `references/memory-ownership.md` before routing a "remember this"
request or persisting a scene. Do not use workspace memory as campaign state.

Module generation is maintained separately in `SagaSmith-module-gen-skills`.

## Invariants

- Keep the active `campaign_id`, edition, and locale explicit.
- Never mix 2014 and 2024 rules unless the user explicitly requests comparison.
- Search first, then expand only the selected rule or module chunk.
- Trust MCP tool results; do not emulate a successful write.
- Use `lobby` outside play, `play` for live non-combat scenes, and the automatic
  `combat_start`/`combat_end` transitions for combat. The MCP owns the session
  exposure; never keep all phase-specific tools visible merely for convenience.
- Runtime character state uses `sheet v2` / `notes v2`; load
  `references/character-schema-v2.md` before creating or mutating a PC, NPC, or
  monster. All three are full `Character` records, not abbreviated stat blocks.
- Use granular character / party MCP tools for inventory, wallet, equipment,
  prepared spells, effects, resources, and actor adventure state. Legacy
  `notes.memories` is import-only; new subjective information belongs to
  ActorKnowledge. `character update` is
  reserved for a reviewed replacement of the complete `sheet` or `notes` document.
- `character build` is the preferred player-character creation workflow: it creates
  a public template and a separate initial campaign instance atomically.
- Do not load entire rulebooks or modules into context.
- For user rulebooks, use the staged Core parser workflow in
  `references/rulebook-import.md`; never make an imported PDF executable without
  source-bound chunks, validation, and explicit DM activation. Inspection warnings
  require an explicit DM acknowledgement before ingest; never bypass that gate.
- For module maps or diagrams, follow `references/module-visual-atlas.md`.
  Text parsing remains fail-closed; only an inspected page image may support a
  `reviewed_image` connection.
- For a real campaign rehearsal or corpus regression, follow
  `skills/dnd-dm/references/CAMPAIGN_REGRESSION.md`; each campaign must exercise
  source-bound lobby preparation, play settlement, combat, continuity, and
  branch/Snapshot isolation instead of treating successful PDF import as play coverage.
- For creature cards present only as PDF images, follow
  `references/module-image-content-review.md`; review the managed page before
  creating an actor with `mode="module_statblock"`.
- For a new platform user, resolve a stable `principal_id` first. Never trust a
  prompt-provided role or `player_name` as permission.
- Supply `expected_revision` and an `idempotency_key` on retriable writes. Treat a
  revision conflict as a fresh read/review cycle, not as permission to overwrite.
- For rule-profile and rule-pack writes, obtain `campaign_revision` from
  `campaign_rules(action="get_profile")` and carry the returned revision forward one write at
  a time. Never silently relock a legacy snapshot or unavailable Core fingerprint.
- Keep each PC/NPC's `actor_id` explicit when reading or writing ActorKnowledge;
  never merge one actor's memories into another actor's context.
- Use `rule_seed_status` before the first rules lookup on a fresh server. Use
  `branch_query(view="compare")` before explaining divergent timelines.

See `references/mcp-contract.md` and `references/workflows.md`. The CLI contract is
legacy compatibility documentation only.
