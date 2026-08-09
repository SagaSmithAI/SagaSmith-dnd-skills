# Rulebook import

Use the three-tool authoring contract:

- `rulebook_draft`: source-to-finalized Rule/Addon Pack;
- `module_draft`: source-to-finalized Module Pack;
- `content_pack`: finalized Pack import, export, test, install, and activation.

## Start the mechanical pass

Call `rulebook_draft(action="start")` with:

- `campaign_id` and an idempotency key;
- `source_path` under a configured import root;
- stable `source_key`, `title`, and `edition`;
- optional locale, publication/version, authority, and warning acknowledgement.

Core stages the immutable source and normalizes pages. D&D inspects, indexes,
extracts the first candidate set, performs safe deterministic repairs, and
returns the editable job plus `draft_workspace` issues. Do not reproduce these
steps with separate MCP calls.

If source inspection has warnings and they were not acknowledged, `start`
returns `source_review_required` without indexing. Use
`rulebook_draft(action="evidence", kind="page")` to inspect a checksum-bound
page. Submit a bounded transcript repair with
`rulebook_draft(action="edit", operation="source_text")`, then call
`operation="advance"`. Only acknowledge warnings after examining their exact
evidence.

## Inspect evidence

Use `rulebook_draft(action="get")` to list jobs or retrieve one job. Use
`rulebook_draft(action="evidence")` with:

- `kind="chunks"` for indexed source chunks, optionally filtered by page/query;
- `kind="page"` for a managed rendered page and its transcription evidence.

The source PDF, normalized page cache, OCR variants, source checksum, and
indexed chunks remain immutable. A rendered-page claim requires the returned
image checksum. A text-only Agent must not claim visual inspection.

## Edit the draft

All writes use `rulebook_draft(action="edit")` and one operation:

| Operation | Purpose |
|---|---|
| `advance` | Resume mechanical indexing/extraction after source review |
| `source_text` | Save a bounded checksum-bound transcript repair |
| `statblock_recovery` | Recover one named or a page-bounded statblock catalog |
| `statblock_review` | Save reviewed normalized statblock content/fill |
| `catalog` | Add a mechanically missed, source-bound candidate |
| `candidates` | Include, exclude, reopen, split, merge, or replace candidates |

Candidate edits may remain temporarily incomplete. Core+D&D persist the edit,
append history, and return deterministic issues. Resolve all blockers before
finalization; do not turn one book's decision into a global parsing heuristic.

For a split or merge, add the reviewed replacement and explicitly exclude each
superseded candidate. Preserve exact chunk/page evidence. Never invent missing
numbers, identities, ownership, selection rules, or executable mechanics.

## Finalize and save

Call `rulebook_draft(action="finalize")` with:

- current `expected_revision` and a fresh idempotency key;
- `job_id` and a completion note;
- final manifest plus optional mechanics/provenance.

Finalization requires every candidate to be included or excluded and every
blocker to be resolved. It atomically freezes the candidate fingerprint,
compiles the source-bound artifacts, validates the Pack, and saves its immutable
version. There is no separate public compile step and no draft mutation after
finalization.

Use `content_pack(action="test", kind="rule")`, then `install` and `activate`
with the same explicit `kind`. Activation
is an explicit Owner/DM decision and requires campaign revision/idempotency
contracts. For an already reviewed portable archive, skip parsing and call
`content_pack(action="import", kind="rule")`.
