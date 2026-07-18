---
name: dnd-campaign-manager
description: "Create and maintain D&D campaigns through the SagaSmith D&D MCP server."
---

# D&D Campaign Manager

`full/` is MCP-first. Use the raw MCP names below (a client may prefix them), not
shell `sagasmith-dnd` commands. Read `../../references/mcp-contract.md` and
`../dnd-dm/references/DM_RULES.md` before mutating a campaign.

Open an MCP session exposure when resuming a campaign. Use `lobby` groups for
setup, module, import, indexing, and character-building workflows; load `play`
groups only when live in-character play begins. Use `exposure_call` only when
the host cannot refresh the native MCP tool list. One session/principal has one
active exposure. Load multiple compatible groups into it; a later `exposure_open`
replaces the earlier exposure, so discard every older exposure id.

## Start and Modules

1. Call `campaign_create` with name, edition, locale, and optional description.
   Choose `2014` or `2024` from the adventure and table contract before importing
   modules or building actors; never accept the server default without checking it.
2. Persist the returned `campaign_id`.
3. Read `campaign_rules(action="get_profile")` and
   `campaign_rules(action="explain")`. Confirm the locked Core provider matches
   the selected edition. For an existing campaign, change the profile only in
   `lobby`, with the fresh campaign revision and an explicit DM decision, before
   any character option is applied.
4. Resolve the caller's stable `principal_id`; use
   `access_grant(scope="campaign" | "actor")` for access instead of treating
   `player_name` as authorization.
5. Use the `module_import` state machine in order: `stage`, `inspect`, `validate`,
   `ingest`, then `activate`. For generated content, stage with `payload.name` and
   `payload.content`. For a user PDF/Markdown/text module, stage with
   `payload.source_path`; it must be inside the server-configured module import
   roots. The server copies it into checksum-addressed MCP storage, and Core
   performs PDF-to-Markdown normalization. Never bypass staging with a direct path.
6. Review `preview.valid`, parser warnings, scene/spatial evidence, and the revision
   diff before ingesting. For a PDF, reject the preview if any scene lacks a valid
   `page_start`/`page_end` within the source page count. Treat document checksum,
   `parser_profile`, and `parser_version` together as the normalized module
   revision identity: a parser-version change requires a new inspect, validate,
   ingest, and activate cycle even when the source checksum and scene diff are
   unchanged. Activation additionally requires the fresh campaign
   `expected_revision`; keep a stable idempotency key per stage.
7. After ingest and again after activation, use `module_query(view="index")` to
   confirm chapter/scene counts, stable keys, page ranges, and spatial evidence.
   Room-heading order is not a map. A `spatial.connections` edge is usable only
   when it carries explicit source evidence (for example, prose stating that a
   stair leads from D4 to D5); an empty connection list means topology is still
   unknown, not that the listed rooms are isolated or sequentially adjacent.
   When a PDF map carries the missing topology, follow
   `../../references/module-visual-atlas.md`: list managed assets, render and
   inspect the exact page, then submit only visually observed edges through
   `module_set_progress(spatial_review=...)`. The returned review is scoped,
   branch-aware, and snapshot-restorable; do not edit immutable import metadata.
   Then choose a scene and use `module_set_progress` with an explicit
   `scope_id` to enter it. Do not narrate from a `module_search` snippet until
   `module_expand` or `module_query(view="scene")` has been called. If an
   encounter scene occurs in a room indexed under a different scene, set its
   same-module, unique spatial key as `current_location_key`; never copy the
   room geometry into the encounter or guess an ambiguous key. Also persist the
   exact spatial scene id as `state.location_scene_id`. At combat start, keep
   the current progress scene, that spatial evidence scene, and any explicit
   encounter `scene_id` distinct; do not recover a duplicate key by scanning a
    different scene when `location_scene_id` was recorded.

Before combat, create a participant manifest from the expanded source scene and
call `module_query(view="readiness")`. Every group records its role, required
count, canonical actor ids, same-module `source_scene_id`, and exact normalized
`source_excerpt`. Initial participants must satisfy every required `combatant`
group. Actors in a `reinforcement` group stay out of the initial list and may enter
only through `combat_join` after the source condition succeeds. Missing, Dead/0 HP,
or mechanically unresolved required actors block combat start; surfaced manual
rulings require review but must not be erased from the readiness report.

## Characters

| Need | MCP tool |
|---|---|
| Create a direct actor | `character_create_from(mode="direct")` |
| List templates | `character_query(view="library")` |
| Instantiate a template | `character_create_from(mode="template")` |
| Atomically create PC template + instance | `character_create_from(mode="build")` |
| Create from an imported exact statblock | `character_create_from(mode="statblock")` |
| List or read live actors | `character_query(view="list" | "get")` |
| Full reviewed replacement | `character_sheet_replace` |
| Ability generation | `dnd_ability_roll`, `character_ability_apply` |

All live actors use `sheet v2` and `notes v2`. Read
`../../references/character-schema-v2.md` before creation or mutation. Do not
persist an unconfirmed draft. Build mode requires one stable
`idempotency_key`; an exact retry must return the original template and campaign
instance rather than creating another pair.

For imported modules, distinguish narrative and mechanical provenance. The module
may name an NPC, describe its intent, and assign fixed possessions while an
inspected rule source provides its statblock. If the supplied module has no
pregenerated PCs, label separately built PCs as player or regression fixtures
instead of claiming module provenance. Use statblock mode only with an exact
imported source and retain its source/chunk provenance. If the exact creature is
absent, ambiguous, spell-only, 2024, or unsupported by the current parser, keep it
unresolved; never substitute a similar SRD creature. Do not pre-resolve random treasure.

For normal play, mutate only the affected structure:

```text
inventory_change | inventory_transfer | wallet_change
character_state_change | character_action | character_metadata_update
campaign_query(view="party")
```

Use `character_spell_prepare(mode="replace_all")` only in `lobby` for setup and level advancement.
During live play, pass the complete
new list as `prepared_spell_ids` on the actor's
`character_state_change(action="rest")` long-rest
transaction; do not toggle preparations one by one.

When the campaign awards a level, first record the source-bound milestone event,
then switch to `lobby` and use
`character_state_change(action="level_advance")`. Never patch the actor sheet.
Treat the returned `advancement.follow_up` as a blocking checklist: all eligible
features, subclass/player choices, spell gains, and the complete `level_up`
prepared list must be reconciled from the active catalog before returning to
`play`. Re-read the actor and create a post-advancement snapshot. Current 2014
single-class support is explicit; unsupported multiclass or 2024 advancement
stops for review.

After each actor or party mutation call `character_query(view="get")` or
`campaign_query(view="party")`. Use their
derived values rather than recalculating weapon attacks, AC, or encumbrance in
prose.

Before every write, read the matching optimistic token and send a fresh
`idempotency_key`. Character tools use the actor revision; scene progress uses
`state_version`; knowledge revisions use `revision_id`; branch/snapshot tools use
the campaign revision plus the current branch or snapshot-head ID. The exact
fields are listed in `../../references/mcp-contract.md`.

## Saves, Branches, and Audit

| Need | MCP tool |
|---|---|
| Create/list a save | `snapshot_create`, `snapshot_query(view="list")` |
| Validate / inspect lineage | `snapshot_query(view="verify" | "lineage")` |
| Restore without deleting future history | `snapshot_restore` |
| Regenerate recap | `snapshot_query(view="recap")` |
| List/create/switch timeline | `branch_query`, `branch_change` |
| Audit / undo / redo | `state_revision(history/undo/redo)` |

Restore is a branch fork, never destructive overwrite. Verify the target first,
then refresh campaign actors, party state, scene progress, events, and continuity
context after restoring.

## Continuity

Use `memory_change/query` for branch-scoped durable world facts and
`campaign_event(add/list)` for immutable chronology. Use
`actor_knowledge_change/query` only for one PC/NPC/monster's subjective knowledge.

For player-safe retrieval, call `continuity_context` with `actor_id`, `scope_id`,
audience, and optionally `branch_id`. Do not substitute broad
`memory_query(view="search")` for
that context; it can expose DM facts or sibling-branch history.
