# Module drafts and packs

Call `module_draft(start)` with a managed module book. Core+D&D stage, inspect,
validate, and mechanically import an inactive editable module workspace in one
operation. A failed validation remains a draft; inspect its page with
`module_draft(evidence)`, submit a checksum-bound
`module_draft(edit, operation="source_text")`, then use `operation="advance"`.
Retain the returned `result.job_id` and `result.module_id` from the receipt and
reuse them for the rest of this draft lifecycle. Do not start another draft or
guess either identifier when a large inspection payload is persisted by the
host.

Before activation, retrieve draft evidence only through these exact shapes:

```json
{"action":"evidence","payload":{"job_id":"<job id>","kind":"chunks","limit":100}}
{"action":"evidence","payload":{"job_id":"<job id>","kind":"chunks","query":"<one short exact source phrase>","limit":20}}
{"action":"evidence","payload":{"job_id":"<job id>","kind":"page","page_number":2,"include_ocr_text":true}}
```

There are no `top_k`, `queries`, `pages`, or `scene_keys` fields on this action.
Do not call `module_search` with the editable draft id: search/expand addresses
an imported active Pack revision after finalization. For Pack decisions, copy a
returned chunk's complete `source_ref` object verbatim. A page result includes
`citation_candidates`; choose a candidate whose excerpt/heading supports the
reviewed fact and copy its `source_ref` verbatim. Never construct a reference
from the page transcription checksum, image checksum, editable module id, or a
separately copied `content_hash`.

Use `module_draft(edit)` for reviewed content, statblocks, assets, and actor
bindings. Extract party range, levels, advancement, endings, scenes,
encounters, actors, items, maps, clues, and exact references. Prose is not
executable; incomplete editorial coverage remains visible advice unless it
causes structural corruption, missing/conflicting source identity, explicit
test failure, or compilation failure.

Save manifest, catalogs, narrative, dependencies, and metadata with
`module_draft(edit, operation="package")`. Each write enters the Pack edit
history, so one-book Agent decisions travel with the draft instead of becoming
parser heuristics. Reserve revision/idempotency requirements for durable start
and finalization boundaries, not every fine edit.

The public request shape puts each decision field directly beside `operation`
inside `payload`; do not add a `package` wrapper and do not put request-control
fields inside `payload`. The only package decision fields are `manifest`,
`catalogs`, `narrative`, `dependencies`, `metadata`, and `version`. Each supplied
field is a complete replacement, not a deep patch. Therefore a manifest edit
must preserve or submit the full reviewed manifest: `title`, `classification`,
`compatibility`, `play_profile`, `continuity`, `activation`, and
`content_summary`. Its structural request is
`module_draft(action="edit", payload={job_id, operation:"package", manifest:{title, classification, compatibility, play_profile, continuity, activation, content_summary}}, expected_revision=..., idempotency_key=...)`.
Use the current module schema inside that manifest: `classification` is the
source-reviewed `adventure` or `campaign`; `compatibility` contains `editions`
and `required_capabilities`; `continuity` contains `series_id`, `order`,
`continues_from`, and `state_policy`; and `activation` contains `mode` and
`default_active`. Omit `catalogs` or `narrative` when there is no reviewed
structured decision to replace. If supplied, every catalog value is an array,
and `narrative` contains the two arrays `dossiers` and `endings`; never replace
those structures with opening/ending prose strings. A source-defined legal
ending belongs as a cited structured entry in `narrative.endings`.

Build `play_profile` with `party_size={minimum, maximum, source_refs}`,
`starting_level={value, source_refs}`, `expected_end_level={value, source_refs}`,
`advancement={modes, recommended, source_refs}`, and
`pregenerated_characters={available, applicability, source_refs}`. Obtain a
current chunk-evidence receipt (or a page `citation_candidates` entry) and reuse
each `source_ref` verbatim as
`{source_key, page, chunk_hash, note}`; never retype, splice, or infer its
checksum. Re-read the draft after the edit and verify the complete stored
decision before finalization.

After reviewing the current draft, its issues, evidence, imported scenes, and
saved package decisions, call `module_draft(finalize)` with
`payload={job_id, pack_id, confirmation:{confirmed:true, note:...}}`; `version`
is optional and defaults to `1.0.0`. `pack_id` is the stable Agent-selected Pack
identity (for example `dnd5e.module.<slug>`), is required only at finalization,
and is not a package-edit field. Keep `idempotency_key` at the tool-call top
level. The confirmation is the Agent's final editorial decision; do not
manufacture or submit any other publication dimensions.
The server validates the finalized workspace, stores `metadata.agent_finalization`
with the reviewer and note, freezes the workspace, and writes the immutable
Module Pack archive. A module may carry or depend on a companion Addon Pack for
new monsters, items, spells, or rules; do not duplicate those entries as scene
prose.

Use `content_pack(import, kind="module")` for an existing archive and
`content_pack(activate, kind="module")` only with Owner/DM authority. Packs exclude campaign
progress, memories, branches, and Snapshots. If an active revision has progress
on a scene whose stable key changed, the Agent must review both indexes and pass
`progress_remaps` entries containing the old `from_scene_id`, the finalized
candidate `to_scene_key`, and an evidence-backed `reason`; never copy a draft
scene id into an immutable Pack activation.
