---
name: dnd-campaign-manager
description: "Create and maintain source-bound SagaSmith D&D campaigns. Use for campaign setup, membership, modules and rules, characters, advancement, continuity, actor knowledge, manifests, snapshots, branches, restores, and audits."
---

# D&D Campaign Manager

Use the `sagasmith_dnd` MCP runtime. Campaign truth belongs to the server, not
workspace memory, prose, a local CLI, or direct database writes.

## Start with the runtime plan

1. Call `skill_query(kind="skill", action="plan")` and read every
   `required_now` document. Stop if `available=false`.
2. For an existing campaign, call `campaign_query(view="resume")` and discard
   pre-restore or pre-resume assumptions.
3. Open one campaign-bound exposure, inspect the selected facade action, load
   only the relevant group, and read its `skill_plan_delta`.
4. Use `exposure_call` only for hosts that do not refresh native tool schemas.
5. Use `refresh=true` once after updating the installed Skills pack, not during
   ordinary campaign operations.

## Route campaign work

| Work | Load | Read deeper only when needed |
|---|---|---|
| Create/list a campaign | `lobby.bootstrap` | `references/CAMPAIGN_MANAGER_DEEP_REFERENCE.md` |
| Rules, membership, manifest, snapshots, branches | `lobby.campaign` | `references/database-contract.md` |
| Build/import/advance characters | `lobby.characters` | `../dnd-dm/references/CHAR_CREATION.md` |
| Import and lock rules | `lobby.rules` | `../../references/rulebook-import.md` |
| Import modules and assets | `lobby.modules` | `../dnd-dm/references/MODULE_INDEX.md` |
| Read continuity/knowledge | `lobby.memory` | `../../references/memory-ownership.md` |
| Write continuity/knowledge | `lobby.memory_control` | `references/CAMPAIGN_MANAGER_DEEP_REFERENCE.md` |
| Save/restore during play | `play.scene_control` or `combat.save` | `references/database-contract.md` |

Use `skill_query(action="search"|"section")` for deep references. The
machine-readable `skill_plan` is the routing source of truth; do not maintain a
host-specific phase/group list.

## Campaign invariants

- Choose and verify 2014/2024 edition, locale, advancement mode, and locked Core
  provider before importing content or building characters.
- Use the module's source-backed recommended maximum party size. Prefer reviewed
  pregenerated PCs; fill only shortages through legal, diverse character
  creation.
- Keep PC/NPC/monster cards, actor access, and ActorKnowledge independent.
- Use exact source references for module metadata, scene progress, rewards, and
  endings.
- Keep rule/module import in Lobby and pass every review/readiness gate.
- Use current revisions and stable idempotency keys for retriable writes.
- Snapshot meaningful boundaries. Fork important alternatives from a parent
  snapshot; never let sibling branches contaminate each other.
- After restore, verify the new head, resume again, reopen exposure, and reread
  campaign, characters, module progress, continuity, actor knowledge, and any
  invalidated Skill fragment.
- Treat the playthrough manifest as progress/audit state, not an alternative
  mutation channel.
- Keep `standalone/` separate and never claim it has MCP persistence or
  transaction guarantees.

For exact facade contracts, inspect the chosen operation and consult
`../../references/mcp-contract.md`. For the retained detailed procedure,
section-read `references/CAMPAIGN_MANAGER_DEEP_REFERENCE.md`.
