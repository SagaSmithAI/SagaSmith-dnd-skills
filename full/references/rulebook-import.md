# User Rulebook Import

Use this workflow only in the `lobby` phase after loading `lobby.rules`. A PDF is first normalized and
indexed as evidence; it does not become executable merely because it was imported.
Before starting, require `server_capabilities.features.structured_rulebook_import`
and `source_bound_rule_packs` to be true. Consume the published
`rulebook_import.stages` contract instead of guessing tool names.

## End-to-end workflow

1. Read `storage_status.rules.import_roots`, then call
   `rule_import(action="discover")`. The DM must select a returned document under
   one of these roots. Never copy or invent a path supplied by an untrusted player.
2. Call `rule_import(action="stage")` with the discovered `payload.source_path`, `source_key`,
   title, edition, locale, publication id, and version. Keep the returned `job_id`,
   artifact name, and source checksum.
3. Call `rule_import(action="inspect")` with that job id. Review page count, recovered
   structure, quality metadata, OCR page list, and every warning. A warning is a
   server-enforced review gate; it is not permission to invent missing headings or
   silently publish mechanics. Scanned or corrupt-text PDFs may make the first
   inspection slow because OCR is selective and page based.
4. For a PDF warning or a candidate that needs visual confirmation, call
   `rule_document_page_render` with the same job id and exact one-based
   `page_number`. Compare the returned checksum-bound image with the normalized
   heading, chunk, or candidate. Do not accept a warning from text alone when the
   disputed source page is available.
5. Call `rule_import(action="ingest")`. If and only if the DM reviewed all warnings,
   pass `payload.acknowledge_warnings=true`. This uses the same Core PDF/Markdown
   normalization path as module documents and stores page-aware retrieval chunks.
   Normalized and page-extraction results are content-addressed, so exact retries and
   later parser passes reuse verified work instead of repeatedly decoding/OCRing the PDF.
6. Use `rule_search` with `source_ids=[<exact source_id>]`, select a hit from that
   source, and call
   `rule_expand`. Check the chunk text, heading path, page range, and source checksum.
7. Call `rule_import(action="extract_candidates")`, review every candidate, and
   submit explicit decisions through `rule_import(action="review")`. Candidate
   extraction never makes content executable. Translate only a reviewed rule into
   the safe declarative IR. Start from
   `examples/rule-packs/xanathar-tools-skills.template.json` when applicable.
   Replace `$SOURCE_ID` and `$CHUNK_ID` with the import/search results.
8. Call `rule_import(action="compile")` for the reviewed import job, or
   `rule_pack_compile(action="from_source")` for a separately authored mechanic.
   Do not use an unbound draft for a
   user-imported executable rule. The server replaces every chunk id with a canonical
   source/checksum/page citation and rejects a chunk from another source.
9. Call `rule_pack_query(view="test" | "inspect")`. Show failures and parser
   warnings to the DM. Install only a validated exact version with
   `rule_import(action="install")` or `rule_pack_change(action="install")`.
10. After explicit DM approval, read `campaign_rules(action="get_profile")` and
   activate the reviewed import with `rule_import(action="activate")`, or pin a
   separately installed version with `campaign_rules(action="set_pack")`, using
   the latest campaign revision.
11. During non-combat play, use `character_check` for a rule-aware check. During
    combat, use `combat_check`. DM-established situational facts go in `rule_facts`;
    they cannot override actor, check kind, ability, or DC.
12. Verify the result with `campaign_rules(action="explain")` and
    `campaign_rules(action="receipts")`. A receipt must contain the imported chunk id, original
    document checksum, page range, exact pack lock, and ruleset fingerprint.

## Xanathar pilot

The included template demonstrates the optional tool/skill synergy procedure as a
`check.before` mechanic. It activates only when the DM supplies both
`skill_proficiency_applies=true` and `tool_proficiency_applies=true`; the engine then
rolls the check with Advantage. The source PDF and extracted prose are never stored in
this skill repository.

The example book may produce a bookmark-match warning even when its text layer and
most structure are usable. Keep that warning in the imported source provenance,
acknowledge it only after review, and compare the selected chunk with the rendered
source page before installation.
