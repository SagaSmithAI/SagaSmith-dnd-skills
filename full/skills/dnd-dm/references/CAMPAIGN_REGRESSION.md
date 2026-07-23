# Real Campaign Rehearsal and Corpus Regression

Use this workflow to rehearse one imported adventure or regression-test a corpus
without bypassing the same MCP exposure available to a live Agent. The run is a
gameplay audit, not only a parser benchmark. Never import server modules, read the
database directly, raw-patch actor sheets, or infer missing module facts.

## Per-campaign gate

Run every step through one campaign-bound MCP session/exposure at a time.

1. In `lobby`, verify storage, server capabilities, the campaign edition, the
   locked Core fingerprint, and the active module revision. Complete the staged
   module import and explicitly resolve any warning gate before activation. Give
   every parser behavior change a new parser version before refreshing. A refresh
   may enter `lobby`, but on any stage/inspect/validate/ingest/activate failure it
   must restore the phase that was exposed on entry.
2. Read `module_query(view="index")`. Visit every non-reference/non-overview
   scene through `module_query(view="scene")`; require readable content, valid PDF
   page ranges, and stable scene ids. Exercise every available atlas location by
   writing scoped progress on a disposable branch. Do not invent topology for a
   scene without reviewed or explicit connections.
3. Prepare at least one complete source-bound PC using only active content catalog
   ids. Use a level appropriate to the adventure segment. Exhaust advancement
   follow-ups, prepared spells, features, derived-state re-reads, and a verified
   snapshot before returning to `play`.
4. Prepare every important named NPC and every NPC/monster required by the
   selected encounter. When the module provides only a narrative identity and no
   combat statblock, use the public driver's `prepare-narrative-npc` path: cite
   the active module/scene/chunk/page/hash and an excerpt containing the exact
   name, enter `lobby`, create `character_create_from(mode="narrative_npc")`,
   verify `combat_eligible=false` plus the `narrative_only`/`source_bound` tags,
   restore `play`, register the actor in the manifest, and verify its checkpoint.
   Such a card supports identity, notes, relationships, and ActorKnowledge; its
   default mechanical shell is not an authored statblock and must never enter
   combat. For encounter participants, use exact rule statblocks or reviewed
   module image cards and retain all warnings. A descriptive passive or action is
   a DM boundary only when it becomes relevant; it does not authorize replacing
   the creature or blocking unrelated automatic attacks. Before any prepared
   monster enters combat, compare every printed Multiattack with
   `derived.multiattack_options`. If a deterministic printed composition is
   missing, stop at the quality gate and repair/reimport it; do not silently run
   one ordinary attack in place of the source-defined action. A generic
   “N melee/ranged [weapon] attacks” composition (where “weapon” may be omitted
   in the source) is deterministic only when the actor card has exactly one
   compatible weapon for that mode. Multiple compatible weapons remain an
   explicit DM-review boundary.
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
   Resolve Surprise from the source positioning and the authoritative actor cards:
   when multiple hostiles hide, call public `character_check` for each hostile's
   Dexterity (Stealth), preserving its derived skill modifier and automatic armor
   disadvantage, then compare every result with each opponent's passive
   Perception. An opponent is surprised only when it detects none of the hiding
   threats; a tied passive score detects that threat. Never hardcode a generic
   Stealth modifier or substitute one creature's profile for another. Use one
   shared hostile roll only when the exact encounter text explicitly says to roll
   once for the group, and require identical Stealth profiles before doing so.
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
   checkpoint before consuming or transferring any acquired item. For a reward
   promised in an earlier scene and paid at a later destination, cite the original
   promise chunk but validate the event against the current scene and its actual
   Scene Atlas location. Treat missing named businesses, inns, farms, or other
   authored locations as an import/atlas defect to repair and refresh, not as
   permission to reuse an unrelated fallback location.
10. Pay source-presented lodging, services, supplies, or other shared expenses
    through the public regression driver's `spend-coins` path. Supply one stable
    spend id, exact positive denominations, the current or explicitly separate
    source scene, actual Scene Atlas location, exact chunk `source_ref`, and the
    Core/Skill `rule_ref` or reviewed price basis. The public
    `campaign_change(action="currency_spend")` transaction must atomically reject
    insufficient funds or commit the full payment and branch-local spend audit.
    Commit witness ActorKnowledge, sync the manifest, and verify the checkpoint;
    never decompose one bill into negative `wallet_change` calls.
11. If a source-cited bargain, tribute, gift, handoff, or destruction removes a
    non-consumable shared item, use the public regression driver's `spend-item`
    path. Supply one stable spend id, exact item id and positive quantity, actual
    Scene Atlas location, exact source excerpt and chunk reference, and every
    witness actor id. Verify the atomic stash decrement, branch-local
    `item_spends` audit, ActorKnowledge, manifest sync, and checkpoint. Never
    represent the disposition only in prose while the item remains in inventory.
12. Exercise a source-acquired standard healing potion when a living PC is
    wounded: call `campaign_change(action="consumable_use")` once, then verify the
    stack decrement, service-owned `2d4+2` random receipt, HP clamp, Core rule
    receipt, ActorKnowledge recipients, manifest sync, and checkpoint. A dead PC
    is not a valid recipient and must not gain knowledge from the use.
13. Give every source-cited scene event a stable identity derived from the run,
    scene, event type, and resolved summary. Before writing progress, merge the
    new entry into the existing `full_playthrough_events` map; never replace the
    map or reuse a run-only key. Re-read progress after the checkpoint and verify
    that earlier events from the same run and scene remain present.
14. When a resolved event changes an NPC, quest, clue, or machine-verifiable
    world condition, use the public regression driver's `record-outcome` path.
    Give it a stable outcome id and exact source reference. It must atomically
    commit the event, stable world facts, and witness-scoped ActorKnowledge,
    upsert (not replace) the manifest NPC/quest/clue projections, merge world
    state, then sync and verify a checkpoint containing the resulting manifest.
    The driver must validate the complete prospective manifest before the first
    mutation. If transport fails after scene progress commits, retry the same
    stable outcome id and identical outcome/fact payload: matching saved progress
    is a resume boundary, not a reason to rewrite it with a changed state version.
    Narrative event text alone is not a restorable NPC or quest state.
15. Award one source-defined XP parcel to every exact eligible recipient in one
    public `award-xp` call when possible. Never include a dead, departed, or
    otherwise ineligible actor merely to match the original party count. The
    stable award and manifest-sync identities must include the sorted recipient
    actor ids as well as scene and amount, so a deliberately split award cannot
    collide with another actor's transaction.
16. Advance each eligible survivor through the public regression driver's
    `advance-level` path one target level at a time. Supply the exact source
    reference that established the XP or milestone, an explicit fixed/rolled HP
    method, the intended return phase, and every caller-owned choice. The driver
    must enter `lobby`, replay the stable level transaction when resuming,
    exhaust all returned and newly applicable class/subclass feature artifacts,
    validate any subclass and known/spellbook choices against the active catalog,
    replace the complete prepared-spell list when the follow-up requires it,
    re-read and verify the actor, restore `play`, sync the manifest, and verify a
    checkpoint. Never edit the raw sheet, silently choose a subclass or feature,
    advance an ineligible/dead actor, or treat the level integer alone as a
    complete advancement.
17. Advance campaign time through the public regression driver's
    `advance-time` path whenever travel, waiting, or a source-triggered interval
    matters. Cite the exact scene chunk and excerpt, supply a positive
    minute/hour/day count, and state any DM ruling used to turn narrative timing
    such as "late in the day" into a duration. The service-owned campaign clock,
    continuity event, actual-witness ActorKnowledge, snapshot, and manifest sync
    must all agree. Never update only the manifest's projected clock or invent a
    duration without an explicit audited ruling.
18. When a manifest PC is dead or departed, build one replacement through the
    public party driver. Prefer an applicable unused module pregen; otherwise
    select one legal audited profile, give it a new identity, enter `lobby`
    through `game_phase`, and restore the entry phase even when construction
    fails. Then use `register-replacement` in `play` at the current source-cited
    Scene Atlas location. The new actor must start with empty ActorKnowledge; the
    joining event may add only its witnessed join and explicit `told_by` handoff
    facts. Keep the predecessor actor and its independent knowledge unchanged,
    replace only its active manifest party slot, append the predecessor,
    replacement, and handoff-event ids to replacement history, and verify a
    checkpoint after the manifest update.

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

When replaying an objective outcome on a sibling branch, reuse its deterministic
`fact_key`. The commit must create or revise a branch-local head for the shared
stable fact identity while leaving the sibling head unchanged. A visibility
error is a branch-isolation defect to fix; inventing a branch-suffixed key is not
a valid workaround. Verify both branches through `memory_query` or
`branch_query(view="compare")` after the replay checkpoint.

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
