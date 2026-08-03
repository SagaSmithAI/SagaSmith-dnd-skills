---
name: dnd-dm
description: "Run D&D 5e 2014 or 2024 sessions through SagaSmith's source-bound MCP runtime. Use for live scenes, checks, combat, rests, character resources, module evidence, DM rulings, continuity, actor knowledge, and campaign playthrough regression."
---

# D&D Dungeon Master

Use the `sagasmith_dnd` MCP runtime. Do not emulate a successful state change,
roll, rule settlement, or snapshot in prose or through direct database/CLI
access.

## Start with the runtime plan

1. Call `skill_query(kind="skill", action="plan")` and read every
   `required_now` document.
2. Stop if the plan reports `available=false`; repair the Skills/MCP
   installation before live play.
3. Resume with `campaign_query(view="resume")`, then open one campaign-bound
   exposure. Search, inspect, and load only the task-relevant group.
4. Read every returned `skill_plan` or `skill_plan_delta`. A checksum-satisfied
   group does not need to be read again. Use `refresh=true` once only after an
   installed Skills update.
5. Use `exposure_call` only when the host cannot refresh native tools after
   `tools/list_changed`.

The plan is context-loading guidance. Server-owned phase, trusted principal,
campaign role, actor grants, revision, idempotency, source validation, and
transactions remain authoritative.

## Route by phase and capability

| Work | Load | Read deeper only when needed |
|---|---|---|
| Resume, scene evidence, narration | `play.scene` | `references/RUNTIME_DEEP_REFERENCE.md` |
| Scene/world/knowledge writes | `play.scene_control` | `../../references/memory-ownership.md` |
| Checks and contests | `play.resolution` | `references/DM_RULES.md` |
| Character resources and rests | `play.characters` | `../../references/character-schema-v2.md` |
| Enter combat | `play.combat_control` | `references/RUNTIME_DEEP_REFERENCE.md` |
| Observe, turns, and actions | `combat.observe`, `combat.turn`, `combat.actions` | `references/DM_RULES.md` |
| End combat or add actors | `combat.control` | `references/RUNTIME_DEEP_REFERENCE.md` |
| Tactical map changes | `combat.map` | `references/DM_MAP_SYS.md` |
| Campaign/module preparation | matching `lobby.*` group | `references/MODULE_INDEX.md`, `references/MODULE_ARC.md` |
| Character creation/advancement | `lobby.characters` | `references/CHAR_CREATION.md` |
| Full campaign regression | groups required by the current phase | `references/CAMPAIGN_REGRESSION.md` |

Use `skill_query(action="search"|"section")` for these deep references; do not
load a whole large document by default.

## Keep the adjudication boundary explicit

- Standard locked mechanics execute in the engine. Do not reinterpret them
  from prose.
- Module, addon, and homebrew semantics use exact source evidence. Import,
  review, or card construction must persist a constrained typed plan or direct
  Agent-ruling clause before the content can be published or used; live play
  never authors that reusable resolution on first use.
- Unique narrative situations default to Agent DM reasoning, followed by
  ordinary public MCP mutations.
- Player choices, owner approval, permission changes, and missing/conflicting
  source evidence remain external-input boundaries.
- Module-authored behavior is DM context, not an executable trigger language.
  Retrieve current context, decide from live state, and persist only the outcome
  that actually occurred.

## Preserve campaign truth

- Treat every PC, NPC, and monster as an independent Character with independent
  ActorKnowledge.
- Carry current campaign/character revisions and stable idempotency keys.
- Let `combat_start` and `combat_end` own Combat phase transitions.
- Use server dice and the campaign random stream.
- Snapshot meaningful boundaries and branches, not every roll or turn.
- After restore, discard old context and revisions, resume again, reopen the
  exposure, and reread invalidated guidance.
- Keep `standalone/` separate; never silently downgrade Full Runtime.

For exact facade payloads, inspect the selected tool/action and use
`../../references/mcp-contract.md`. For the retained detailed operating manual,
search or section-read `references/RUNTIME_DEEP_REFERENCE.md`.
