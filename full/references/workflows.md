# Runtime Workflows

## Session start

1. `doctor`
2. `campaign list`
3. `campaign show`
4. `campaign rules-get`
5. `module list`
6. Retrieve the current scene or recent memory.

## New campaign

1. `campaign start`
2. Optional `module inspect` and `module ingest`
3. Create confirmed characters
4. Record the opening event
5. Keep the initial snapshot returned by `start`

## Rule question

1. `rules search --campaign`
2. Select by title, edition and publication
3. `rules expand`
4. Answer with source metadata

## Restore

1. `save verify`
2. Explain the target slot and branch behavior
3. `save restore`
4. Refresh campaign, characters, scene and memory

## Session close

1. Reconcile character resources, conditions, scene progress and campaign state
2. Append the session events and durable memories
3. `save create`
4. Inspect the generated delta recap; use `save regenerate-recap --slot` if needed

## Module generator handoff

1. Ingest the existing generated Markdown without rewriting it
2. `module index --campaign`
3. `module export-scenes --campaign --output`

## Audit recovery

- `state history` inspects mutations.
- `state undo` and `state redo` change audited state without deleting snapshots.
- `memory scope` reports the memories effective on the current branch.
