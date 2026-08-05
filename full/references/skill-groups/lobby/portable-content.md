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
presets, and non-`none` for modules; editions and conflict ids must validate.

Partial preset export may contain proven cards and deferred templates, but the
rule component still covers every source section and candidate. Catalog-only
statblocks use rebound source/chunk ids and exact printed name through
`character_create_from(mode="statblock")`. Parameterized companions remain
templates until required owner/class context is provided and retained.

Before distribution, cold-start a fresh MCP home and run the addon regression:
publicly import, inspect, enable, catalog, disable, and exactly re-export. Keep
commercial-source reports private.
