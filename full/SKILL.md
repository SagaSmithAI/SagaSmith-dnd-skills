---
name: sagasmith-dnd-suite
description: "Run or maintain D&D 5e 2014/2024 campaigns through SagaSmith's MCP-first game-master, actor, module, continuity-memory, and snapshot workflows. Use for live play, campaign setup, character management, module import, rules adjudication, durable facts, actor knowledge, branches, saves, and restores."
---

# SagaSmith D&D Suite

This repository is an Agent Skill, not a Python runtime. Full Runtime calls the
`sagasmith_dnd` MCP server; clients may expose raw tool names with a prefix such
as `mcp_sagasmith_dnd_`.

## Startup

1. On a zero-knowledge host, first read `sagasmith://bootstrap`. If resources
   are unavailable, call the always-visible
   `skill_query(kind="skill", action="plan")`. Read every document in
   `required_now`; the plan is derived from the server-owned phase, trusted
   campaign role, and loaded tool groups. Use `outline`, `section`, and
   `search` only for task-specific depth. Do not load the entire DM skill or
   MCP contract by default. If the plan reports `available=false`, stop live
   campaign work and repair the installed Skills pack. Use `refresh=true` once
   after a Skills update, not on every turn.
2. Call `storage_status`; call `storage_migrate` only when schema setup is needed.
   Call `server_capabilities` and `campaign_query`. Resume an existing campaign
   with `campaign_query(view="resume")`, which reloads its current branch,
   manifest, scene, continuity, and a signed context receipt.
3. Start every MCP session with `exposure_open`, then use `exposure_search`,
   `exposure_inspect`, and `exposure_load` for the current campaign phase. Pass
   `tool_id` and an optional `selector` to `exposure_inspect` before using a
   compact facade whose payload is unfamiliar. Follow an
   `exact_field_contract` as a strict whitelist; treat a
   `runtime_field_guide` as discovery guidance and still obey any validation
   error returned by the selected action. Before a campaign exists, load only
   `lobby.bootstrap`; reopen the exposure with the returned `campaign_id` before
   loading campaign-bound groups. There is one active exposure per MCP
   session/principal: calling `exposure_open` again replaces it. Load every
   compatible group needed for the current phase into that one exposure; never
   retain or call an older exposure id. Read the `skill_plan` or
   `skill_plan_delta` returned by `campaign_query(view="resume")`,
   `exposure_open`, `exposure_inspect`, `exposure_load`, `game_phase`,
   `combat_start`, and `combat_end`.
4. Use Full Runtime only when the `sagasmith_dnd` MCP tools are available. The
   bounded Skill-group fragments under `references/skill-groups/` are the
   operational loading surface; use the child Skills and
   `references/mcp-contract.md` only as task-specific deep references.
5. If MCP is unavailable, use the separate `standalone/` skill. Do not silently
   switch this full skill to shell CLI commands.
6. Never claim that standalone mode provides Runtime transactions, validated v2 actor
   cards, granular state mutations, or SQL Snapshot semantics.

## Included Skills

- `skills/dnd-dm`: play, adjudication, rule/module retrieval, and narration.
- `skills/dnd-campaign-manager`: campaign, character, save, and memory lifecycle.

Runtime continuity is branch-aware: use world facts for durable truth, actor knowledge
for one PC/NPC/monster's subjective information, and scoped scene state for private
discoveries. Read `references/memory-ownership.md` before routing a "remember this"
request or persisting a scene. Do not use workspace memory as campaign state.

Module generation is maintained separately in `SagaSmith-module-gen-skills`.
The machine-readable phase/tool-group mapping is
`data/skill-plan.v1.json`; do not maintain a host-specific duplicate.

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
- When one source-defined treasure parcel contains both currency and items, use
  `campaign_change(action="loot_acquire")` with one stable acquisition id and the
  exact expanded module chunk reference. Do not split that parcel into independent
  wallet and inventory writes. If a promised reward is paid later, keep the exact
  promise chunk as the source while separately recording and validating the scene
  and Scene Atlas location where payment actually occurs; never relabel the old
  source location as the payout location.
- For a looted weapon, set `mechanics.proficient` explicitly for the intended
  recipient from current rule-backed proficiencies. Do not inherit the defeated
  monster's proficiency or attack bonus; use `false` when the recipient or
  proficiency is not yet proven.
- Use `campaign_change(action="consumable_use")` for a shared standard healing
  potion outside combat so item consumption, server-side `2d4+2`, healing, the
  random-stream position, and their rule receipt commit together.
- When a source-cited bargain, handoff, tribute, or destruction permanently
  removes a non-consumable shared item, use
  `campaign_change(action="item_spend")` with a stable spend id, exact item id
  and quantity, and the expanded source chunk reference. Do not leave the item
  in inventory while recording only a narrative outcome.
- `character build` is the preferred player-character creation workflow: it creates
  a public template and a separate initial campaign instance atomically.
- Do not load entire rulebooks or modules into context.
- For user rulebooks, use the staged Core parser workflow in
  `references/rulebook-import.md`; never make an imported PDF executable without
  source-bound chunks, validation, and explicit campaign-owner activation. The
  Agent acting as DM reviews inspection warnings from exact text or page evidence
  before acknowledging ingest; missing/conflicting evidence remains an external
  review boundary. A returned `normalization_notes` entry records source text or
  page furniture that the parser safely excluded; retain it for audit, but never
  turn it into a ruling requirement or source-review blocker. Never bypass either
  gate.
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
- For an important named module NPC with no combat statblock, use
  `character_create_from(mode="narrative_npc")` with an exact active
  module/scene/chunk/page/hash and name-bearing excerpt. Keep the resulting
  `narrative_only` actor out of checks and combat.
- For a new platform user, resolve a stable `principal_id` first. Never trust a
  prompt-provided role or `player_name` as permission. A multi-user host must
  hide and inject the authenticated principal. A single-user process should set
  `SAGASMITH_DND_MCP_BOUND_PRINCIPAL_ID`; never expose authorization identity as
  a model choice.
- Supply `expected_revision` and an `idempotency_key` on retriable writes. Treat a
  revision conflict as a fresh read/review cycle, not as permission to overwrite.
- For rule-profile and rule-pack writes, obtain `campaign_revision` from
  `campaign_rules(action="get_profile")` and carry the returned revision forward one write at
  a time. Never silently relock a legacy snapshot or unavailable Core fingerprint.
  If a verified snapshot needs an older unavailable Core, inspect it with
  `snapshot_query(view="core")` and use the explicit
  `branch_change(action="create_core_upgrade")` conversion only after recording a
  reviewed reason and both old/new fingerprints.
- Keep each PC/NPC's `actor_id` explicit when reading or writing ActorKnowledge;
  never merge one actor's memories into another actor's context.
- Keep module-authored narrative behavior as exact DM context, not an executable
  trigger language. Link the verbatim source through a DM-only
  `kind="context_anchor"` fact, retrieve it with `continuity_context.related_refs`,
  let the Agent adjudicate from the live actor/scene/quest/item state, and execute
  only the resulting standard public operations. Persist only what actually
  happened; never encode hypothetical `if/then` behavior in memory metadata.
  When a continuity commit cites a source pinned by a matching context anchor,
  include the current `continuity_context.context_receipt`. A stale, wrong
  branch/principal, unsigned, or source-incomplete receipt is rejected; reread
  context after any revision or restore before committing the ruling.
- Use `rule_seed_status` before the first rules lookup on a fresh server. Use
  `branch_query(view="compare")` before explaining divergent timelines.

For the complete cross-repository ownership, persistence, adjudication, retrieval,
time, knowledge, manifest, and restore model, read
`references/long-form-narrative-architecture.md`. See
`references/mcp-contract.md` and `references/workflows.md` for the exact public
contract and ordered operations. The CLI contract is legacy compatibility
documentation only.
