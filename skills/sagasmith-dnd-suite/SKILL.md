---
name: sagasmith-dnd-suite
description: Run persistent D&D 5e 2014/2024 campaigns through SagaSmith's source-bound MCP runtime. Use for setup, live play, combat, module/rule import, continuity, actor knowledge, branches, snapshots, and restores.
---

# SagaSmith D&D Full Runtime

This is a Claude/Hermes/plugin discovery wrapper. Require the
`sagasmith_dnd` MCP server and read `sagasmith://bootstrap`. Then call
`skill_query(kind="skill", action="plan")`, read every `required_now`
document, and read the canonical workflow at
`{baseDir}/../../full/SKILL.md`.
Stop if the plan reports `available=false`; the installed Skills/MCP pairing
must be repaired before live play.

Resume with `campaign_query(view="resume")`; open one campaign-bound
exposure, read its phase plan, and load only relevant groups. Read every
returned `skill_plan_delta`. Use `exposure_call` when the host
does not refresh `tools/list_changed`. Never trust a model-authored
principal. Do not silently switch to `standalone/`.
