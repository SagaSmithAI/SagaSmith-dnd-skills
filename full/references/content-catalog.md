# D&D Content Catalog Contract

The Content Catalog is the common character-option surface for bundled Core
rules and source-bound extension packs. It separates three facts that must not
be conflated:

1. **Catalogued**: a source-linked record is searchable and has a stable id.
2. **Available**: its core edition is locked by the campaign, or its extension
   pack is enabled on the current branch.
3. **Executable**: a reviewed rule mechanic covers the requested outcome.

For the bundled 2014 SRD, `dnd5e.content.srd2014@1.0.0` is installed during MCP
startup when the full D&D skill repository is configured. Its records retain a
`bundled:srd2014/...` reference to the original Markdown file. Optional books
must use `rule_pack_draft_from_source`; every artifact supplies imported
`source_chunk_ids`, which the MCP resolves to the exact document chunk/page
citations before the pack can be installed.

## Agent procedure

1. Read `campaign_rule_profile_get` and the current branch state.
2. Call `content_catalog_list(campaign_id, kind, query)`.
3. Present only returned options and their source references to the player.
4. For a supported card target, call `character_content_apply` with the
   character's latest revision and an idempotency key.
5. If the response is `pending_ruling`, obtain the required choices or resolve
   the effect as a DM decision. Do not bypass the result by editing raw sheets.

An imported extension is not automatically enabled, and installation is not a
mechanics claim. The DM selects its exact pack version per branch. Snapshots
then retain that version/checksum lock for replay and audit.
