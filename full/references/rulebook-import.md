# User Rulebook Import

Use this workflow only in the `lobby` phase after loading `lobby.rules`. A PDF is first normalized and
indexed as evidence; it does not become executable merely because it was imported.
Before starting, require `server_capabilities.features.structured_rulebook_import`
and `source_bound_rule_packs` to be true. Consume the published
`rulebook_import.stages` contract instead of guessing tool names.

## End-to-end workflow

1. Read `storage_status.rules.import_roots`, then call
   `rule_import(action="discover")`. The SagaSmith Agent acting as DM selects an
   exact returned document under one of these roots. Ask the owner only when the
   requested document is ambiguous or activation authority is missing. Never
   copy or invent a path supplied by an untrusted player.
2. Call `rule_import(action="stage")` with the discovered `payload.source_path`, `source_key`,
   title, edition, locale, publication id, and version. Keep the returned `job_id`,
   artifact name, and source checksum.
3. Call `rule_import(action="inspect")` with that job id. Review page count, recovered
   structure, quality metadata, OCR page list, and every warning. A warning is a
   server-enforced review gate; it is not permission to invent missing headings or
   silently publish mechanics. Scanned or corrupt-text PDFs may make the first
   inspection slow because OCR is selective and page based.
4. For an inspection warning that genuinely requires page-image evidence, an
   image-capable Agent or human DM may call
   `rule_import(action="render_page")` with the exact one-based page. A text-only
   Agent must not call page rendering and pretend it inspected the image; keep
   the warning blocked until a capable reviewer records the decision.
5. Call `rule_import(action="ingest")`. If and only if the Agent acting as DM
   reviewed all warnings from available exact evidence, pass
   `payload.acknowledge_warnings=true`. This uses the same Core PDF/Markdown
   normalization path as module documents and stores page-aware retrieval chunks.
   Normalized and page-extraction results are content-addressed, so exact retries and
   later parser passes reuse verified work instead of repeatedly decoding/OCRing the PDF.
6. Use `rule_search` with `source_ids=[<exact source_id>]`, select a hit from that
   source, and call
   `rule_expand`. Check the chunk text, heading path, page range, and source checksum.
7. If `character_create_from(mode="statblock")` fails because the indexed text
   split one card across columns or attached its headings to an adjacent
   creature, retry it with the source-established page/neighborhood `chunk_ids`
   and `payload.source_statblock_name` set to the exact printed creature heading.
   Keep the differently named campaign instance in `payload.name`. The server
   deterministically selects that creature's core chunk, stops at the next
   creature core, reconstructs the ability and action sections, and returns
   `source.text_layout_recovery` with the exact retained chunk ids. This is the
   first recovery path for a text-only Agent and requires neither page rendering
   nor image understanding. If the indexed text still omits or conflicts on a
   required fact, call `import_query(view="list", kind="rulebook")` and match
   the exact `source_id` to its retained import `job_id`; this also works when
   the source was indexed in an earlier process. Then call
   `rule_import(action="recover_statblock")` with that `job_id`, exact printed
   creature heading as `name`, and a fresh idempotency key. If the exact card
   contains Multiattack, include `payload.agent_fill` in this same recovery
   call; local OCR remains authoritative for card facts, while the Agent maps
   only the returned/diagnosed activity id and exact action prose to canonical
   weapon ids, modes, and counts. A mismatched submission reports both expected
   and received activity ids; correct the generic fill rather than adding a
   creature-specific parser rule. Do not use a
   campaign instance name such as a named dragon in place of
   `Adult Blue Dragon`. Supply `page_number` only when it is already
   source-established; otherwise let the server read the printed-page index hint
   and scan nearby physical pages. Core then runs local layout OCR, isolates the
   target column, distinguishes repeated decorative/narrative copies of the
   creature name by the adjacent size/type/alignment core, rejects
   low-confidence critical fields, and requires either
   target-segment embedded-text corroboration or agreement from an independent
   OCR scale. The result is a checksum-bound reviewed statblock with any
   validated Agent action fill retained atomically; retry actor
   creation with `mode="reviewed_rule_statblock"` and its returned `review_id`.
8. If layout OCR cannot isolate a card but the already-indexed chunks still contain
   the complete card as one ordered, contiguous segment on an exact page, a
   text-only Agent acting as DM may normalize only that segment. Require
   `server_capabilities.features.indexed_text_statblock_review` and one
   unambiguous retained artifact identity whose jobs match the `source_id`.
   Equivalent historical jobs with the same artifact name and checksum are safe
   to select deterministically; if those identities differ, stop and use an
   explicitly reviewed `--source-job-id`. Then call
   `rule_import(action="review_statblock")` with
   `review_mode="agent_text"`, the exact `page_number`, normalized full card,
   observation, and ordered `evidence_chunk_ids`. The MCP independently requires
   every chunk to belong to that source, cover the page, and have contiguous
   ordinals; it rejects both facts absent from the evidence and selected evidence
   omitted from the normalized card. Use the returned `review_id` with
   `character_create_from(mode="reviewed_rule_statblock")`.
   Any reviewed passive that is not one of the engine's structured source traits
   must remain on the actor card as
   `choices.manual_ruling.kind="descriptive_passive"` with
   `default_resolver="agent"` and its exact description as `source_excerpt`.
   A warning without this typed entry is an importer defect: repair and recreate
   the actor through the public review path rather than adding a creature-name or
   phrase-specific settlement exception.
   This is layout normalization, not model-memory reconstruction. If the indexed
   facts themselves are missing or conflicting, stop at explicit source review.
   An image-capable reviewer may instead render the exact page and transcribe only
   observed fields through the same action with `review_mode="visual"` (the
   default). A text-only Agent must not claim visual review, repair tokens from
   rules memory, select a similar SRD creature, or acknowledge a conflict as
   success.
9. Call `rule_import(action="extract_candidates")`, review every candidate, and
   submit explicit decisions through `rule_import(action="review")`. Candidate
   extraction never makes content executable. Translate only a reviewed rule into
   the safe declarative IR. Start from
   `examples/rule-packs/xanathar-tools-skills.template.json` when applicable.
   Replace `$SOURCE_ID` and `$CHUNK_ID` with the import/search results.
10. Call `rule_import(action="compile")` for the reviewed import job, or
   `rule_pack_compile(action="from_source")` for a separately authored mechanic.
   Do not use an unbound draft for a
   user-imported executable rule. The server replaces every chunk id with a canonical
   source/checksum/page citation and rejects a chunk from another source.
11. Call `rule_pack_query(view="test" | "inspect")`. The Agent acting as DM
   reviews failures and parser warnings from exact evidence. Install only a validated exact version with
   `rule_import(action="install")` or `rule_pack_change(action="install")`.
12. After explicit campaign-owner/DM approval, read
   `campaign_rules(action="get_profile")` and
   activate the reviewed import with `rule_import(action="activate")`, or pin a
   separately installed version with `campaign_rules(action="set_pack")`, using
   the latest campaign revision.
13. During non-combat play, use `character_check` for a rule-aware check. During
    combat, use `combat_check`. For a 2014 opposed check, use the atomic
    `character_check(action="contest")` tool and supply rule facts independently for each side.
    Agent-as-DM-established situational facts go in `rule_facts`; they cannot override
    actor, check kind, ability, or DC.
14. Verify the result with `campaign_rules(action="explain")` and
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
