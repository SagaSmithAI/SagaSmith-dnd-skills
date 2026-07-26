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
7. If `character_create_from(mode="statblock")` fails because the indexed text is
   missing the creature heading, identity line, AC, HP, Speed, or ability table,
   call `rule_import(action="recover_statblock")` with the same `job_id`, exact
   printed creature heading as `name`, and a fresh idempotency key. Do not use a
   campaign instance name such as a named dragon in place of `Adult Blue Dragon`.
   Supply `page_number` only when it
   is already source-established; otherwise let the server read the printed-page
   index hint and scan nearby physical pages. This path is designed for a
   text-only Agent: Core runs local layout OCR, isolates the target column,
   rejects low-confidence critical fields, and requires either target-segment
   embedded-text corroboration or agreement from an independent OCR scale. The
   result is a checksum-bound reviewed statblock; retry actor creation with
   `mode="reviewed_rule_statblock"` and its returned `review_id`.
8. If recovery cannot locate exactly one heading, the two evidence paths disagree,
   a critical field has low confidence, or no page can be inferred, stop at
   explicit missing/conflicting-source review. A capable reviewer may render the
   exact page and transcribe
   only observed fields through `rule_import(action="review_statblock")`. A
   text-only Agent must not repair tokens from rules memory, select a similar SRD
   creature, or acknowledge a conflict as success.
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
