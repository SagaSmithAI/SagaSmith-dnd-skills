# Portable cards and packages

## Actors and modules

Use `sagasmith.portable` schema v1. PC, NPC, and monster share the same
`actor_card` envelope; `payload.actor_type` selects the role. Export with
`character_query(view="portable_card")`. Import with
`character_create_from(mode="portable_card")`, providing exactly one inline
`card`, managed `artifact`, or allowlisted `source_path`. Import creates a fresh
Character identity and never copies campaign membership, revision, or
ActorKnowledge. Actor-card writes must already contain a source-bound plan or
direct Agent ruling for unresolved prose; first use is not a content-authoring
phase.

Browse installed standard actors with
`rule_pack_query(view="content_catalog", kind="actor_card")` and import a
bundled preset by exact `artifact_id`. Export the default SRD library through
`rule_pack_query(view="actor_presets", edition=..., include_package=true)`.
When importing from a preset pack, supply the pack and exact artifact id; never
reconstruct a monster by name in host code.

Before exporting a module, bind every cast member, encounter creature, and
pregenerated PC with `module_import(action="bind_actor")`, then inspect
`module_query(view="actors")`. Export through `module_query(view="package")`.
A `module_pack` contains normalized source, signed Scene Atlas text and chunks,
checksum-bound assets, reviews, actor cards, and scene bindings. It deliberately
excludes campaign progress, world state, snapshots, and ActorKnowledge.

Import with `module_import(action="import_package")` in Lobby. The MCP validates
the edition, re-ingests through Core, creates fresh actors, and restores managed
assets/bindings. Re-read index and readiness before activation. Never edit the
database, preserve foreign Character IDs, or run uninstalled optional rules.

## Rule packs, releases, and addons

Export a reviewed extension with `rule_pack_query(view="package",
payload={campaign_id, pack_id, version, metadata, include_package?})`. The
package uses stable source/chunk keys and carries indexed text for citations.
Keep `metadata.distribution="private"` unless the owner supplied an explicit
license and attribution for shareable distribution.

Import through `rule_import(action="import_package")` using exactly one
`package`, managed `artifact`, or allowlisted `source_path`, plus a stable
idempotency key. Verify the edition and exact dependencies. Import creates an
inactive draft with fresh local source/chunk ids; installation, Owner/DM
approval, and activation are separate steps.

Every standalone D&D rule package must contain
`resolution_policy="build_time_complete"` and the exact readiness report
recomputed from its artifacts and proven mechanic providers. Export, import,
and install all fail on a missing, stale, or deferred report. An addon repeats
that audit at the outer envelope and inside every embedded rule component; one
layer never substitutes for the other.

Pin `metadata.definition_checksum` for rule dependencies and the envelope
`checksum` for release components. Keep rules, presets, and modules separate,
then compose exact ids, versions, and checksums with
`rule_pack_query(view="release")`. Inspect with
`rule_import(action="inspect_release")`; a release manifest never imports,
installs, activates, or grants access.

For a complete locally owned book, combine its complete source-bound
`rule_pack` and reviewed `preset_pack` with
`rule_pack_query(view="addon_package")`; import through
`rule_import(action="import_addon")`. Components install globally but remain
inactive until Owner/DM calls `campaign_rules(action="set_addon")` with the
current revision. Branch locks and snapshot restore retain exact versions.
Activation policies must match component kinds (`branch` for rules, `library`
for presets, non-`none` for modules); rule components must support all declared
editions, and conflict ids must be unique and non-self-referential.

`rule_pack_query(view="preset_package", allow_partial=true)` may return proven
cards plus deferred templates in `summary.failures`. The rule component must
still contain every source section and candidate. A catalog-only statblock
advertises `character_create_from(mode="statblock")`; use its rebound
`source_id`, `chunk_ids`, and exact `source_statblock_name` so the engine performs
normalization. Parameterized companions remain source templates until supplied
their required owner/class context, which must be retained in provenance.

Before distributing detached addons, cold-start a fresh MCP home and run
`scripts/regression_addons.py`. The audit must publicly import, inspect, enable,
catalog, disable, and exactly re-export every package. Keep reports containing
commercial source metadata beside private packages, not in a public repository.
