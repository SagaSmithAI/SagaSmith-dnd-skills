# Source-backed opposition hydration

Use this workflow when a campaign preflight or regression reports missing
source-backed opposition, or when a narrative actor must become mechanical.
Do not treat that gap alone as proof that the active Pack needs a new review.

## Establish the source path

1. Move to `lobby` before authoring or actor creation. Consume
   `tools/list_changed`, refresh the native list, and use
   `exposure(search/set)` to load `rule_search`, `rule_seed_status`,
   `rulebook_draft`, `character_create_from`, and module authoring tools only as
   needed.
2. Re-read `character_query(view="list")`. Reuse every existing actor whose
   returned `statblock.source_identity` matches the required printed card, and
   create only the exact shortfall.
3. Search the exact printed creature identity first with only `campaign_id`,
   `query`, and optional `top_k`. Campaign binding already scopes the default
   edition, locale, and enabled sources. If a filtered search misses, retry this
   minimal shape once before starting any draft.
4. Use `rule_seed_status` only when source-level inventory is necessary. A
   returned rule hit `source_id`, or the matching source inventory `id`, is the
   only valid rule-source id. Module, Pack, scene, and document ids are different
   namespaces.

## Hydrate from a canonical rule source

1. Treat every returned `source_id` and `chunk_id` as an opaque exact value.
   Copy complete ids character-for-character from one latest successful
   `rule_search` result. Never retype, normalize, splice, or reconstruct them.
2. Call `character_create_from(mode="statblock")` with that exact `source_id`,
   selected evidence in `payload.chunk_ids`, and the exact printed identity in
   `payload.source_statblock_name`. There is no `exact_chunks` field. Give
   repeated instances distinct `payload.name` values.
3. If creation reports a source/chunk mismatch, search again and compare the
   submitted JSON to one result. A one-character mismatch is Agent input error,
   not missing evidence and not grounds to weaken validation.
4. If an exact localized hit is readable but not mechanically hydratable, do
   not hand-copy its numbers and do not move a standard creature into a
   module-specific review. When current module evidence also prints the
   canonical English identity, make one explicit same-edition English lookup,
   for example:

   ```json
   {
     "campaign_id": "<campaign id>",
     "query": "<exact canonical English identity>",
     "filters": {"edition": "2014", "locale": "en"}
   }
   ```

   Verify the returned heading is the same creature, then hydrate only from
   that one English result's exact `source_id` and `chunk_ids`. This selects an
   enabled canonical source; it does not permit translation or a remembered
   substitute. If no mechanically usable equivalent exists, retain the source
   diagnostic and use the reviewed rulebook-draft lifecycle.
5. If module evidence applies a narrow instance change to that canonical card,
   pass it through the existing `variant` boundary. Cite the exact managed
   `module-chunk:<id>` or immutable `module-review:<id>` in `variant.source_ref`
   and include only the printed override, such as `creature_type`. Do not copy
   the entire card into Pack data or use a generic sheet patch.
6. Re-read every created actor and require `statblock.source_identity` to match
   the intended source card.

## Hydrate module-only opposition

1. When the exact creature exists only in the active module, inspect the
   current Pack's immutable content reviews. Use
   `character_create_from(mode="module_statblock")` only with the returned
   `review_id`, and pass the exact printed card as `payload.source_identity`.
2. If a finalized Pack lacks the required review, create an explicit new
   draft/version from the same managed source. Add only the evidence-backed
   missing review, re-read it, finalize it, import the new artifact, and
   activate only the module id returned by that import. Never edit a finalized
   Pack in place or guess a review id.
3. An ending entry, dossier, encounter label, or `module_set_progress` value is
   narrative metadata and never substitutes for a mechanical content review.

## Verify and return to play

Run `module_query(view="preflight")`. Its `ready`, `card_valid`,
`hard_blockers`, and `disabled_capabilities` fields are the combat gate. A
usable attack card is not wholly blocked merely because unrelated source-backed
spells are disabled; retain those diagnostics and avoid only the unavailable
capability. Repair first when the whole card is invalid, the intended action is
disabled, or indispensable evidence is absent or conflicting.

Restore the entry phase after preparation, consume the native tool-list change,
refresh the list, and use `exposure(search/set)` for the next phase. Stop for
external input only after the exact rule, reviewed rulebook, and module-review
paths are absent, contradictory, or unavailable.
