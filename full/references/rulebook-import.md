# User Rulebook Import

Use this workflow only in the `authoring` profile. A PDF is first normalized and
indexed as evidence; it does not become executable merely because it was imported.
Before starting, require `server_capabilities.features.structured_rulebook_import`
and `source_bound_rule_packs` to be true. Consume the published
`rulebook_import.stages` contract instead of guessing tool names.

## End-to-end workflow

1. Read `storage_status.rules.import_roots`. The DM must place or select the source
   under one of these roots. Never copy a path supplied by an untrusted player.
2. Call `rule_document_stage(campaign_id, source_path)`. Keep the returned artifact
   name and source checksum.
3. Call `rule_document_inspect(campaign_id, artifact)`. Review page count, recovered
   structure, and every warning. A warning is a review gate; it is not permission to
   invent missing headings or silently publish mechanics.
4. Call `rule_document_import` with an explicit `source_key`, title, edition,
   publication id, and idempotency key. This uses the same Core PDF/Markdown
   normalization path as module documents and stores page-aware retrieval chunks.
5. Use `rule_search`, select a hit from that exact `source_id`, and call
   `rule_expand`. Check the chunk text, heading path, page range, and source checksum.
6. Translate only the reviewed rule into the safe declarative IR. Start from
   `examples/rule-packs/xanathar-tools-skills.template.json` when applicable.
   Replace `$SOURCE_ID` and `$CHUNK_ID` with the import/search results.
7. Call `rule_pack_draft_from_source`. Do not call the unbound draft tool for a
   user-imported executable rule. The server replaces every chunk id with a canonical
   source/checksum/page citation and rejects a chunk from another source.
8. Call `rule_pack_test` and `rule_pack_inspect`. Show failures and parser warnings to
   the DM. Install only a validated exact version with `rule_pack_install`.
9. After explicit DM approval, read `campaign_rule_profile_get` and pin the installed
   version using `campaign_rule_pack_set` with the latest campaign revision.
10. During non-combat play, use `character_check` for a rule-aware check. During
    combat, use `combat_check`. DM-established situational facts go in `rule_facts`;
    they cannot override actor, check kind, ability, or DC.
11. Verify the result with `campaign_rules_explain` and
    `campaign_rule_receipts`. A receipt must contain the imported chunk id, original
    document checksum, page range, exact pack lock, and ruleset fingerprint.

## Xanathar pilot

The included template demonstrates the optional tool/skill synergy procedure as a
`check.before` mechanic. It activates only when the DM supplies both
`skill_proficiency_applies=true` and `tool_proficiency_applies=true`; the engine then
rolls the check with Advantage. The source PDF and extracted prose are never stored in
this skill repository.

The example book may produce a bookmark-match warning even when its text layer and
most structure are usable. Keep that warning in the imported source provenance and
review the selected chunk against the rendered source page before installation.
