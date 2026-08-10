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
