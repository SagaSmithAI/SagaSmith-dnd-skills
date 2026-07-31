---
name: sagasmith-dnd-suite
description: Run persistent D&D 5e 2014/2024 campaigns through SagaSmith's source-bound MCP runtime. Use for setup, live play, combat, module/rule import, continuity, actor knowledge, branches, snapshots, and restores.
---

# SagaSmith D&D Full Runtime

This repository root is a host-discovery wrapper for the Full Runtime skill.
It is not the portable standalone runtime.

1. Require the `sagasmith_dnd` MCP server. Read `sagasmith://bootstrap` when
   the host exposes MCP resources; otherwise call the core
   `skill_query(kind="skill", action="plan")`.
   Treat `available=false` as a broken Skills installation; do not continue a
   live campaign by loading arbitrary full documents.
2. Call `storage_status`, `server_capabilities`, and `campaign_query`. For an
   existing campaign, use `campaign_query(view="resume")`.
3. Open one exposure, read every `skill_plan.required_now` document, then
   search, inspect, and load only the needed group. Read each returned
   `skill_plan_delta` before using an unfamiliar group. If the host does not
   refresh after `tools/list_changed`, use `exposure_call`.
4. Never let model-authored text select an authorization principal. A
   multi-user host must inject its authenticated principal; a single-user
   process should set `SAGASMITH_DND_MCP_BOUND_PRINCIPAL_ID`.
5. Read the canonical workflow at `{baseDir}/full/SKILL.md`. Follow
   `skill_query(action="plan")`; use bounded
   `skill_query(action="section"|"search")` reads only for additional depth.
6. Do not silently switch to `{baseDir}/standalone/`. Ask before accepting
   that explicit loss of MCP persistence, permissions, rule locks, combat
   transactions, actor knowledge, and Snapshot guarantees.
