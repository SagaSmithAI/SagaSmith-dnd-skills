# Runtime Workflows

Full Runtime uses the `sagasmith_dnd` MCP server. See `mcp-contract.md` for the
complete operation mapping.

## Session start

1. `storage_status`, then `campaign_list` and `campaign_get`.
2. Call `module_current` for the acting scope and `module_read_scene` if one exists.
3. Read recent `event_list`, use `continuity_context` for the acting actor, then
   refresh `party_show` and relevant `character_get` cards.

## New campaign

1. `campaign_create`.
2. `module_write` → `module_inspect` → `module_import` → `module_index`.
3. `character_build` for each confirmed PC; use `character_create` or
   `character_instantiate` for NPCs and monsters.
4. Record the opening `event_add` and `snapshot_create`.

## Rule question

1. `rule_search` with edition and locale.
2. Select a result, then `rule_expand`.
3. Answer with source metadata rather than unverified recollection.

## User rulebook to executable optional pack

1. Read `rulebook-import.md`, then stage and inspect with
   `rule_document_stage` and `rule_document_inspect`.
2. Import through `rule_document_import`; use `rule_search` and `rule_expand` to
   select evidence from its exact `source_id`.
3. Translate the reviewed boundary into safe IR and call
   `rule_pack_draft_from_source` with imported chunk ids.
4. Test, inspect, install inactive, and obtain explicit DM approval before
   `campaign_rule_pack_set`.
5. Settle a non-combat check with `character_check` or a combat check with
   `combat_check`, then audit `campaign_rule_receipts`.

## Restore

1. `snapshot_verify`, then `snapshot_lineage`.
2. Explain that `snapshot_restore` forks history.
3. Refresh actors, `party_show`, `module_current`, `event_list`, and
   `continuity_context`; discard pre-restore assumptions.

## Session close

1. Reconcile actor and party state with granular MCP tools.
2. Append `event_add`, `memory_add`, and actor knowledge as appropriate.
3. `snapshot_create`; use `snapshot_regenerate_recap` when needed.

## Audit recovery

- `state_history` inspects audited mutations.
- `state_undo` and `state_redo` change audited state without deleting snapshots.
- `memory_list` and `branch_list` show current-branch continuity; use explicit
  `branch_id` to inspect another timeline.
