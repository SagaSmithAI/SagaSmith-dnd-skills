# Agent rulebook editing loop

Use this loop for a Rulebook Pack that is not finalized. Core+D&D own document
extraction, the first mechanical candidate set, deterministic normalization,
and validation. The Agent owns semantic edits until explicit finalization.

## Loop

1. Call `rulebook_draft(action="start")`. Treat the returned candidates as
   editable material, never executable content.
2. Read `job.draft_workspace`, all `draft_issues`, and exact evidence through
   `rulebook_draft(action="evidence", kind="chunks" | "page")`.
3. Call `rulebook_draft(action="edit", operation="candidates")` to include,
   exclude, reopen, split, merge, or replace candidates. Use `operation="catalog"`
   for a missed source-bound entity. Keep every superseded candidate as an
   explicit exclusion.
4. After every edit, read the new issues. Fix blockers from exact evidence and
   repeat; accepted and rejected dispositions are not frozen.
5. Call `rulebook_draft(action="finalize")` with the latest revision, completion
   note, and final Pack manifest only when every candidate is resolved and the
   blocker count is zero. Finalization freezes, compiles, and saves the Pack.
6. Inspect or activate the saved Pack through `content_pack(get|activate,
   kind="core_rules"|"addon")`; no public build, test, or install step exists.

## Agent permissions before finalization

The Agent may revise identity, kind, ownership, boundaries, and structured
fields; replace or reopen its earlier decision; add a missed source-bound
candidate; split or merge through explicit replacement and exclusion; repair
source-proven transcription damage; and refuse finalization while evidence is
insufficient.

The Agent must not edit the source, parser, checksums, validators, or immutable
evidence; invent missing facts; suppress blockers; silently omit candidates;
or activate a Pack merely because it finalized successfully.

## Review patterns migrated from former special cases

Treat these as prompts to inspect evidence, never as automatic parser rules:

- Repeated headings or identical paths do not prove occurrence boundaries. Read
  the surrounding sections and split only when the source establishes separate
  entries.
- Physical proximity does not establish subclass, parent-feature, dependent
  statblock, or scene ownership. Bind an owner only from explicit source
  structure or a reviewed source-bound decision.
- Overlapping text spans do not prove two features are duplicates. Preserve
  distinct identities unless their complete evidence establishes one entry;
  otherwise replace/merge explicitly and exclude every superseded candidate.
- A generic-looking background or feature title is not grounds for rejection.
  Inspect its full source role, fields, and neighbors before including or
  excluding it.
- A damaged redundant modifier or missing label may be repaired mechanically
  only when the remaining values uniquely prove it. Otherwise record a bounded
  transcript correction from inspected evidence or leave the issue unresolved.
- A parser-produced candidate that is structurally valid can still be
  semantically wrong, and a missing candidate can still be source-established.
  The Agent must check both false positives and false negatives before final
  confirmation.

For every correction, save the exact candidate ids, evidence ids/page range,
replacement or disposition, decision, and reason. Re-run the draft checks after
each edit and inspect the resulting inventory rather than assuming the edit had
the intended downstream effect.

## Decision storage

Book-specific decisions remain with the draft/Pack as dispositions, artifacts,
issues, source bindings, and edit history. The Skill stores this workflow, not
book answers. Promote only a demonstrated cross-source deterministic invariant
into Core or D&D.

For a module book, use the same loop through `module_draft`. Save publication
decisions with `edit(operation="package")`; after reviewing the actual draft,
issues, evidence, and package decisions, finalize with the Agent's explicit
`confirmation`. Never fabricate a publication matrix. The server validates the
descriptor and records `metadata.agent_finalization`. Activation is always a later
`content_pack(kind="module")` operation.
When a progressed scene was renamed or restructured, the Agent records the
continuity decision at activation as `from_scene_id` -> finalized
`to_scene_key` plus its reason; runtime progress itself remains campaign state.

The Agent may keep editing for as many review passes as necessary before
finalization. “No current blocker” is not automatic publication approval: the
Agent must inspect the current revision and send the explicit confirmation.
After finalization, corrections require a new draft/version rather than a
mutation of the immutable Pack.
