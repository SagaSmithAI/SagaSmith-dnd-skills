# Real Campaign Rehearsal and Corpus Regression

Use this workflow to rehearse one imported adventure or regression-test a corpus
without bypassing the same MCP exposure available to a live Agent. The run is a
gameplay audit, not only a parser benchmark. Never import server modules, read the
database directly, raw-patch actor sheets, or infer missing module facts.

## Per-campaign gate

Run every step through one campaign-bound MCP session/exposure at a time.

1. In `lobby`, verify storage, server capabilities, the campaign edition, the
   locked Core fingerprint, and the active module revision. Complete the staged
   module import and explicitly resolve any warning gate before activation.
2. Read `module_query(view="index")`. Visit every non-reference/non-overview
   scene through `module_query(view="scene")`; require readable content, valid PDF
   page ranges, and stable scene ids. Exercise every available atlas location by
   writing scoped progress on a disposable branch. Do not invent topology for a
   scene without reviewed or explicit connections.
3. Prepare at least one complete source-bound PC using only active content catalog
   ids. Use a level appropriate to the adventure segment. Exhaust advancement
   follow-ups, prepared spells, features, derived-state re-reads, and a verified
   snapshot before returning to `play`.
4. Prepare every NPC/monster required by the selected encounter in `lobby`. Use
   exact rule statblocks or reviewed module image cards and retain all warnings.
   A descriptive passive or action is a DM boundary only when it becomes relevant;
   it does not authorize replacing the creature or blocking unrelated automatic
   attacks.
5. In `play`, select one source-printed non-combat check. Read the exact scene,
   preserve its ability/skill and DC, resolve it through `character_check`, and
   commit the event, stable facts, per-witness ActorKnowledge, and snapshot with
   one `continuity_commit`. A skill label belongs in the cited evidence; use
   `kind="check"` unless the tool contract explicitly defines another kind.
6. Before combat, read the exact encounter scene and its location. Call
   `module_query(view="readiness")` with every source/DM-established group.
   `required_count` is the complete group count, not `len(actor_ids)`: derive it
   from an exact printed count, a persisted random-table roll, or an explicit
   branch-local DM composition fact, and prepare all required cards. Include other
   printed hostiles as initial, reinforcement, or optional groups, or record the
   scene-supported reason they are absent.
7. Start combat from `play` and require the automatic transition to `combat` plus
   an encounter-local temporary map whose encounter, spatial scene, module, and
   location provenance agree. Exercise at least one structured automatic path
   and any relevant owned reaction/choice window. End with a structured outcome;
   never stop while a spell resolution, reaction, death save, or concentration
   obligation is pending.
8. Back in `play`, persist the public outcome and only the knowledge actually
   gained by each PC/NPC/monster. Re-read actor cards rather than treating the
   historical final combat projection as current state.
9. When the resolved scene yields treasure, select and expand the exact treasure
   chunk and acquire the complete parcel through
   `campaign_change(action="loot_acquire")`. Use one stable acquisition id,
   stable item ids, the printed denominations and quantities, and the exact
   content hash. Currency, items, and the branch-local audit record must commit
   in one public transaction. Record the discovery only for living or otherwise
   present witnesses, sync the playthrough manifest, and verify the resulting
   checkpoint before consuming or transferring any acquired item.
10. Exercise a source-acquired standard healing potion when a living PC is
    wounded: call `campaign_change(action="consumable_use")` once, then verify the
    stack decrement, service-owned `2d4+2` random receipt, HP clamp, Core rule
    receipt, ActorKnowledge recipients, manifest sync, and checkpoint. A dead PC
    is not a valid recipient and must not gain knowledge from the use.

## Exact scene evidence

`module_search` selects a document chunk; `module_expand` proves what that chunk
contains. Neither proves that the text belongs to a chosen scene. A PDF chunk can
have no scene id, overlap adjacent headings, or match another occurrence of the
same room name. Before using a DC, participant excerpt, or map location:

1. select the scene from `module_query(view="index")`;
2. read it with `module_query(view="scene")`;
3. verify module id, scene id, page range, and location key;
4. copy the evidence substring from that returned scene content.

The readiness check normalizes PDF control characters, soft hyphens, typographic
quotes, dash variants, case, and whitespace. This only compensates for extraction
artifacts. It never makes a paraphrase, translation, truncated count, or text from
another scene acceptable.

## Snapshot and branch-isolation audit

Run destructive rehearsal steps on a disposable branch created from a verified
source checkpoint. Carry fresh campaign/actor/scene revisions and idempotency keys
through every mutation. Create and verify branch checkpoints during long scene
walks and after continuity/combat. Then:

1. end combat and switch the disposable branch to `lobby`;
2. create and verify its closing snapshot;
3. checkout the original source branch through `branch_change`;
4. restore its original phase;
5. create and verify a new source-branch head;
6. compare branches and re-read current scene/progress, actor HP/resources,
   campaign facts, ActorKnowledge, and active combat.

The source branch passes only when its scene/progress, actor state, facts, and
knowledge are unchanged and no combat remains active. Interrupted disposable
branches must be closed and returned through the same public MCP sequence before
retrying; do not delete them or repair the database.

## Corpus completion report

For every campaign, retain machine-readable reports for import/index, all-scene
walk, PC preparation, hostile preparation, non-combat resolution, combat, and
final read-only audit. A corpus is complete only when all campaigns satisfy:

- every non-reference/non-overview scene was read and progressed on an isolated
  branch;
- a source-bound PC and all selected encounter actors are complete;
- one source-cited non-combat check and one structured combat path committed;
- ActorKnowledge exists only on the rehearsal branch for actual witnesses;
- HP/resources, scene progress, current scene, facts, and knowledge are restored
  on the source branch;
- the final branch is the expected source branch in `play`, with no active combat
  and a valid head snapshot.

Keep parser warnings and review-only candidates in the report. They are evidence
of fail-closed behavior, not permission to fabricate missing content. A successful
corpus result means the exercised public workflows passed; it does not claim that
every optional rule or every possible encounter path was executed.
