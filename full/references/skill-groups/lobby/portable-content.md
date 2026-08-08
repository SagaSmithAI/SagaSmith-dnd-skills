# Portable cards and packages

## Actors and modules

PC, NPC, and monster share `actor_card`; `payload.actor_type` selects the role.
Export with `character_query(view="portable_card")`; import exactly one card,
artifact, or allowlisted path with `character_create_from(mode="portable_card")`.
Import creates a fresh identity and never copies campaign state or
ActorKnowledge. Browse standard cards with
`rule_pack_query(view="content_catalog", kind="actor_card")` and select an exact
`artifact_id`; never reconstruct a creature by name. Custom mechanics use their
reviewed plan or persist an Agent-compiled `content_solution` on first use.

Bind cast, encounters, and pregenerated PCs with `module_import(bind_actor)` and
inspect `module_query(view="actors")` before export. The v2 descriptor inside
`.sagasmith-module` locks play/continuity contracts, dependencies, source, Scene
Atlas, catalogs, narrative, reviews, actor cards, components, and readiness;
asset bytes use content-addressed archive paths. Runtime campaign state,
ActorKnowledge, random streams, branches, and snapshots are excluded.

Import one managed or allowlisted archive with `module_import(import_package)`.
The MCP validates descriptor, blobs, edition, and exact dependencies before Core
re-ingest and fresh actor creation. Only `playable`/`complete` may activate;
review `draft`/`indexed` inactive. Re-read all imported indexes and readiness.
Reject module-pack v1; never preserve foreign IDs or edit the database.

## Rule packs, releases, and addons

Export reviewed rules with `rule_pack_query(view="package", ...)`. Packages use
stable source/chunk keys and carry citation text. Keep distribution `private`
without an explicit shareable license and attribution.

Import with `rule_import(action="import_package")` from exactly one inline,
managed, or allowlisted source. Verify edition and dependencies. Import creates
an inactive draft with fresh local ids; install and Owner/DM activation remain
separate.

Rule packages require `resolution_policy="build_time_complete"` and a readiness
report recomputed from artifacts and mechanic providers. Export, import, and
install reject missing, stale, or deferred reports. Addons repeat this audit at
the envelope and each rule component.

Pin rule `definition_checksum` and release component `checksum`. Compose exact
ids and versions with `rule_pack_query(view="release")`; inspection never
imports, installs, activates, or grants access.

Build an addon from a complete source-bound rule pack and reviewed preset pack;
import with `rule_import(import_addon)`. Components install globally but stay
inactive until revision-safe Owner/DM `campaign_rules(set_addon)`. Branches and
snapshots retain exact versions. Policies are `branch` for rules, `library` for
presets, and `none` for modules; addons cannot contain module components or own
module activation. Editions and conflict ids must validate.

Partial preset export may contain proven cards and deferred templates, but the
rule component still covers every source section and candidate. Catalog-only
statblocks use rebound source/chunk ids and exact printed name through
`character_create_from(mode="statblock")`. Parameterized companions remain
templates until required owner/class context is provided and retained.

Before distribution, cold-start a fresh MCP home and run the addon regression:
publicly import, inspect, enable, catalog, disable, and exactly re-export. Keep
commercial-source reports private.
