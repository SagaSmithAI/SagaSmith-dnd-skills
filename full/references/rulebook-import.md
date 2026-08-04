# User Rulebook Import

Use this workflow only in the `lobby` phase after loading `lobby.rules`. A PDF is first normalized and
indexed as evidence; it does not become executable merely because it was imported.
Before starting, require `server_capabilities.features.structured_rulebook_import`
and `source_bound_rule_packs` to be true. Consume the published
`rulebook_import.stages` contract instead of guessing tool names.

## Portable package path

If the owner already has a reviewed `sagasmith.portable` `rule_pack`, do not
repeat PDF normalization, OCR, candidate extraction, or semantic review. Call
`rule_import(action="import_package")` with exactly one inline package, managed
artifact, or allowlisted source path and a stable idempotency key. Confirm that
the package system and source editions match the campaign and inspect every
exact dependency status. Import reconstructs source/chunk ids and citations but
only creates a validated inactive draft. Continue with explicit install and
Owner/DM activation; never infer either from package validity.

Rule dependencies are locked by the dependency package's
`metadata.definition_checksum`, not its installation-local rule-pack checksum
or its full distribution-envelope checksum. The full envelope checksum is used
when that package appears as a component of a thin release manifest.

Export a locally reviewed version through `rule_pack_query(view="package")`.
Default to private distribution; shareable output requires owner-confirmed
license and attribution. Keep rule, preset, and module packages independent and
compose them through `rule_pack_query(view="release")`. The receiver may inspect
that thin manifest with `rule_import(action="inspect_release")`; it has no fetch,
import, install, activation, or access authority.

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
4. For a damaged or structurally ambiguous page, call
   `rule_import(action="render_page")` with the exact one-based page. The result
   contains the immutable rendered image plus checksum-bound normalized text,
   native PDF text, and local OCR variants. A text-only Agent may use those text
   fields but must not claim it inspected the image. After `inspect`, it may call
   `rule_import(action="review_text")` for that page, batching every known exact,
   unique replacement and supplying the returned normalized-text hash, current
   job revision, rationale, and evidence basis:
   - `cross_text` requires the replacement text in two returned text sources;
   - `agent_context` permits only a small spelling/case/Markdown-structure repair
     and cannot change any digit sequence;
   - `rendered_page` requires a reviewer that actually inspected the returned
     image and its exact checksum.
   The server never edits the PDF or cached OCR. It stores the reviewer,
   evidence hashes, before/after page hashes, and replacements, then reruns
   inspection against the revised view. Copy heading depth from adjacent entries
   of the same content kind; capitalization alone does not establish hierarchy.
   If reparsing exposes a mistake, an unpublished inspected job may submit a
   further version bound to the new page hash (at most eight per page). Refresh
   the job revision after every version. Once ingested, create a new import job
   instead of mutating history. Missing or conflicting mechanics remain blocked; do not use
   transcript review to reconstruct them from model memory.
   For a durable whole-book regression, copy every accepted repair into that
   document's source-review manifest under `text_reviews`. Retain the one-based
   page, current `base_text_sha256`, exact `old`/`new` replacements, rationale,
   evidence basis, review method, and (for `rendered_page`) the exact image
   checksum. `regression_rulebooks.py --catalog-manifest ...` re-renders the
   evidence, rejects hash drift, and replays each revision through the public
   `review_text` facade before ingest. Never refresh a stored hash merely to make
   a stale correction apply; re-review the changed normalized page instead.
5. Call `rule_import(action="ingest")`. If and only if the Agent acting as DM
   reviewed all warnings from available exact evidence, pass
   `payload.acknowledge_warnings=true`. This uses the same Core PDF/Markdown
   normalization path as module documents and stores page-aware retrieval chunks.
   Normalized and page-extraction results are content-addressed, so exact retries and
   later parser passes reuse verified work instead of repeatedly decoding/OCRing the PDF.
6. Use `rule_search` with `source_ids=[<exact source_id>]`, select a hit from that
   source, and call
   `rule_expand`. Check the chunk text, heading path, page range, and source checksum.
   For whole-book compilation, call
   `rule_import(action="recover_statblocks")` once after indexing. It first accepts
   deterministic complete cards assembled from checksum-bound indexed chunks, then
   uses cached local layout OCR only for the remaining 2014 pages. One malformed
   card must become an itemized failure and must not abort the rest of the book.
   Treat out-of-range ability scores, ambiguous identities, and incomplete
   multi-page cards as unresolved evidence, never as values to clamp or recall from
   model knowledge. The internal `indexed_text` review mode is reserved for this
   server batch action and is not a caller-selectable shortcut.
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
   the source was indexed in an earlier process. For a 2014 source, then call
   `rule_import(action="recover_statblock")` with that `job_id`, exact printed
   creature heading as `name`, and a fresh idempotency key. Standard rulebook
   mechanics are engine-authoritative: do not include `payload.agent_fill`.
   A parsed standard Multiattack is used directly; an unparsed standard
   Multiattack or other mechanical card is an engine implementation gap that
   must be fixed and tested before retrying. Do not add a creature-name special
   case or ask the Agent to redefine the printed rule. Do not use a
   campaign instance name such as a named dragon in place of
   `Adult Blue Dragon`. Supply `page_number` only when it is already
   source-established; otherwise let the server read the printed-page index hint
   and scan nearby physical pages. Core then runs local layout OCR, isolates the
   target column, distinguishes repeated decorative/narrative copies of the
   creature name by the adjacent size/type/alignment core, rejects
   low-confidence critical fields, and requires either
   target-segment embedded-text corroboration or agreement from an independent
   OCR scale. The result is a checksum-bound reviewed 2014 statblock; retry actor
   creation with `mode="reviewed_rule_statblock"` and its returned `review_id`.
   `recover_statblock` must reject a 2024 source instead of applying 2014 layout
   grammar. For 2024, use a complete exact-page indexed segment with the
   edition-matching `review_statblock` text path below, or an image-capable
   edition-matching visual review.
8. If 2014 layout OCR cannot isolate a card, or a 2024 card bypasses OCR, but the
   already-indexed chunks still contain
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
   omitted from the normalized card. The source and campaign must agree on 2014
   or 2024, and that edition selects the statblock parser. Use the returned `review_id` with
   `character_create_from(mode="reviewed_rule_statblock")`.
   The engine continues to own generic D&D transactions: action economy,
   attacks, saves, damage, resource payment, timing windows, and structured
   Multiattack composition. Missing or conflicting source facts, or a generic
   printed mechanic that the engine cannot execute, must reject the review as
   `engine_implementation_required`; implement it with a source-backed
   regression test before recreating the actor. Exact creature-, spell-, item-,
   or feature-specific prose may instead carry a persisted exact-source direct
   Agent-ruling clause created during import/review. That clause settles only
   the authored content outcome and cannot replace the engine-owned transaction.
   This is layout normalization, not model-memory reconstruction. If the indexed
   facts themselves are missing or conflicting, stop at explicit source review.
   Bounded text-layer repair may accept `l/I` for `1`, `o` for `0`, and `f`
   between two numeric range components for `/`, but only at the matching
   numeric positions. Changed DCs, bonuses, dice, damage types, and other
   numeric facts remain rejected.
   The bounded 2014 layout path may recompute a damaged redundant ability
   modifier only from a clearly printed score. It may restore one missing
   ability label only when all six source scores are present and the remaining
   five labels uniquely establish the canonical column order; it never supplies
   a missing score or clamps an illegal value. Explicit `Actions for Type ...`
   sections are source-defined variants and must become separate actor cards,
   each with only its own action set.
   During a private Monster Manual build, an exact bundled SRD actor may be
   reused only when edition is 2014, publication id is exactly `mm2014`, and one
   unique OCR-confusable identity matches. The private card is rebuilt in the
   target preset namespace and records the source card version/checksum. Never
   apply this optimization to supplements or same-name variants. A visibly
   damaged heading may be superseded only by one reviewed actor on the same
   source page.
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
   Every otherwise-unclaimed mechanically signaled chunk must remain represented
   during extraction by an `agent_resolution_required` catalog candidate with
   exact citations. This is a transient Lobby review state and must never appear
   in a compiled pack or addon. It is not descriptive merely because no parser
   handled it. Accepting that candidate immediately stores a direct exact-source
   Agent-ruling clause unless the reviewer supplies a native mechanic or typed
   plan. The compiled artifact must be `ruling_ready` (or
   `descriptive_ready` for proven descriptive material). Missing identity, core
   statistics, or source text may not be relabeled as semantic deferral.
   Dice procedures, numbered random-effect tables, and adjudication guidance
   count as mechanical signals even when no specialized entity parser matches.
   They require one exact-source build-time Agent-ruling artifact per retained
   chunk. A no-candidate whole-book regression applies the same conservative
   per-chunk fallback; it must never publish one descriptive sample as coverage
   for the rest of a mechanical source.
10. Call `rule_import(action="compile")` for the reviewed import job, or
   `rule_pack_compile(action="from_source")` for a separately authored mechanic.
   Do not use an unbound draft for a
   user-imported executable rule. The server replaces every chunk id with a canonical
   source/checksum/page citation and rejects a chunk from another source.
11. Call `rule_pack_query(view="test" | "inspect")`. The Agent acting as DM
   reviews failures and parser warnings from exact evidence. Install only a validated exact version with
   `rule_import(action="install")` or `rule_pack_change(action="install")`.
   Standalone package export/import/install and composed-addon export/import
   must all report and independently recompute
   `resolution_readiness.complete=true`, an empty `unresolved` list, and
   `first_use_compilation_required=false`.
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
