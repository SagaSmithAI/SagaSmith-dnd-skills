# Portable cards and packages

Use `sagasmith.portable` schema v1 for sharing. PC, NPC, and monster are the
same `actor_card` envelope; `payload.actor_type` selects the role. Export with
`character_query(view="portable_card")`. Import with
`character_create_from(mode="portable_card")`, providing exactly one inline
`card`, managed `artifact`, or allowlisted `source_path`. Import creates a fresh
Character identity and never copies campaign membership, revision, or
ActorKnowledge. Review the exported sheet and notes before publishing them.

Browse the campaign edition's installed standard actors with
`rule_pack_query(view="content_catalog", kind="actor_card")`. Import one
bundled preset by its `artifact_id`. To share the default SRD library, call
`rule_pack_query(view="actor_presets", edition=..., include_package=true)`;
the returned `preset_pack` or managed artifact is portable. When importing a
card from any preset pack, supply the pack plus its exact `artifact_id`. Never
reconstruct or substitute a monster by name in host code.

Before exporting a structured module, bind every cast member, encounter
creature, and pregenerated PC with `module_import(action="bind_actor")`; inspect
the result with `module_query(view="actors")`. Export through
`module_query(view="package")`. A `module_pack` contains the normalized source,
signed Scene Atlas scene text and retrieval chunks, embedded checksum-bound
assets, reviewed source content, portable actor cards, and stable scene-key
bindings. Import replays that stored structure rather than applying the
receiver's current parser heuristics. It is not a campaign save and does
not contain progress, world state, Snapshot history, or ActorKnowledge.

Import with `module_import(action="import_package")` in Lobby. The MCP validates
the envelope, checks the locked edition, re-ingests through Core, materializes
assets in managed storage, creates fresh actors, and restores bindings. Re-read
the module index, actor bindings, assets, and readiness before activation or
play. Do not edit the database, preserve foreign Character IDs, or treat a
checksum-valid package as permission to execute uninstalled optional rules.
