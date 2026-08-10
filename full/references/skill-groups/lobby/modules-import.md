# Module drafts and packs

Call `module_draft(start)` with a managed module book. Core+D&D stage, inspect,
validate, and mechanically import an inactive editable module workspace in one
operation. A failed validation remains a draft; inspect its page with
`module_draft(evidence)`, submit a checksum-bound
`module_draft(edit, operation="source_text")`, then use `operation="advance"`.

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

Build `play_profile` with `party_size={minimum, maximum, source_refs}`,
`starting_level={value, source_refs}`, `expected_end_level={value, source_refs}`,
`advancement={modes, recommended, source_refs}`, and
`pregenerated_characters={available, applicability, source_refs}`. Obtain a
current chunk-evidence receipt and reuse each source reference verbatim as
`{source_key, page, chunk_hash, note}`; never retype, splice, or infer its
checksum. Re-read the draft after the edit and verify the complete stored
decision before finalization.

After reviewing the current draft, its issues, evidence, imported scenes, and
saved package decisions, call `module_draft(finalize)` with
`confirmation={confirmed: true, note: ...}`. The confirmation is the Agent's
final editorial decision; do not manufacture or submit publication dimensions.
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
