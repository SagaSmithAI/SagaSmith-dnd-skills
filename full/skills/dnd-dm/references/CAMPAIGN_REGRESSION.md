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
   page ranges, and stable scene ids. After `module_search`, take the immutable
   chunk `source_ref` from `module_expand`, including the service-owned
   `content_sha256`; a missing reference is an import/exposure defect, not
   permission to calculate or invent a client-side hash. Exercise every available atlas location by
   writing scoped progress on a disposable branch. Do not invent topology for a
   scene without reviewed or explicit connections. A campaign may revisit the
   same scene after world, quest, party, or objective state changes. Derive each
   `advance-scene` replace identity from the complete normalized target manifest:
   an exact transport retry must reproduce the same key and payload, while a
   later stateful revisit must receive a different key. A target-scene-only key
   is invalid because hubs, towns, and headquarters are intentionally revisited.
   In a public full-playthrough run, use the driver's `read-scene` action when
   the indexed scene id is already known. It calls
   `module_query(view="scene", scope_id="dm")` directly and validates the returned
   id. Reserve `query-source` for locating an unknown chunk, then expand only the
   selected hits. Do not inflate `top_k` or repeat broad searches to reconstruct
   a scene that the exact scene query can return in one request.
3. Classify and import every module-supplied PC document before building seats.
   Fill every applicable party seat from those pregenerated PCs first, up to the
   module's source-cited maximum recommended party size; only then build the
   remaining legal seats from active content catalog ids. A present applicable
   pregen may not be skipped for a generated optimization. Preserve each pregen's
   source reference and document checksum. Use a level appropriate to the
   adventure segment. Exhaust advancement follow-ups, prepared spells, features,
   derived-state re-reads, and a verified snapshot before returning to `play`.
4. Prepare every important named NPC and every NPC/monster required by the
   selected encounter. When the module provides only a narrative identity and no
   combat statblock, use the public driver's `prepare-narrative-npc` path: cite
   the active module/scene/chunk/page/hash and an excerpt containing the exact
   name, enter `lobby`, create `character_create_from(mode="narrative_npc")`,
   verify `combat_eligible=false` plus the `narrative_only`/`source_bound` tags,
   restore `play`, register the actor in the manifest, and verify its checkpoint.
   A `prepare-statblock` failure at candidate lookup, visual review, validation,
   creation, or verification must restore both the entry branch and entry phase
   before surfacing the error. Re-read the public phase after a failed review;
   never leave the campaign in `lobby` and repair it out of band.
   Such a card supports identity, notes, relationships, and ActorKnowledge; its
   default mechanical shell is not an authored statblock and must never enter
   combat. For encounter participants, use exact rule statblocks or reviewed
   module image cards and retain all warnings. When one reviewed statblock must
   create several source-identical actors, create every actor separately with an
   idempotency identity scoped by the run, review, actor name, actor type, and
   source variant. Retrying one actor must recover that actor, while the next
   actor must not collide with the previous creation. A descriptive passive or
   action is a DM boundary only when it becomes relevant; it does not authorize
   replacing the creature or blocking unrelated automatic attacks. Before any prepared
   spellcaster enters combat, a printed `Spellcasting` entry must have parsed as
   structured spellcasting rather than a descriptive passive. Compare its
   source-printed ability, slot maxima, and exact spell-name set with the created
   card, and require the prepared spell ids to cover that executable set. OCR
   tokens such as a broken ordinal are an importer regression to fix and refresh,
   not permission to accept an empty spell list or patch the actor manually.
   Before any prepared
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
   Derive each check's retry identity from the run, scene, Scene Atlas location,
   check kind, ability/skill, actor, DC, proficiency, advantage/disadvantage, and
   exact source chunk. Checks that share a scene, actor, ability, and DC but occur
   at different locations or cite different chunks are separate rolls and must
   not reuse progress, dice, continuity, knowledge, or manifest-sync keys.
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
   obligation is pending. When a hostile selects a structured Multiattack, pass
   its option id only on the first attack, resolve every remaining source-defined
   attack separately, and do not end that actor's turn while its Multiattack
   attack budget/remaining sequence is nonempty.
   Resolve Surprise from the source positioning and the authoritative actor cards.
   When the encounter text itself explicitly says that this route surprises a
   named participant, preserve the exact excerpt and use the driver's
   source-declared-surprise input for only that participant; do not invent a
   Stealth or scout check. Otherwise, when multiple hostiles hide, call public
   `character_check` for each hostile's Dexterity (Stealth), preserving its
   derived skill modifier and automatic armor disadvantage, then compare every
   result with each opponent's passive Perception. An opponent is surprised only
   when it detects none of the hiding
   threats; a tied passive score detects that threat. Never hardcode a generic
   Stealth modifier or substitute one creature's profile for another. Use one
   shared hostile roll only when the exact encounter text explicitly says to roll
   once for the group, and require identical Stealth profiles before doing so.
   Preserve detection separately for every hostile-observer pair: a hidden
   combatant's `visible_to_actor_ids` includes each opponent whose passive score
   detected that combatant. Detecting one hider neither reveals the others nor
   makes the detected hider untargetable.
   Preserve source-authored NPC tactics as ordered opening casts with exact
   excerpts. Charged item spells must call `combat_cast_spell` with the actual
   `source_item_id`; never copy the spell into the NPC's ordinary prepared list
   or patch charges. If a source says a living NPC surrenders at an HP threshold
   only when escape is impossible, confirm both predicates from current state and
   end with `status="surrender"` before another attack. Do not relabel surrender
   as defeat, death, or a generic truce.
   When surrender or defeat moves a unique equipped item into party custody,
   use the public `transfer-source-item` path (`character_to_party`) with both
   current revisions and the exact scene evidence. Do not create a duplicate
   loot record; preserve the original item's charges, condition, and source key.
8. Back in `play`, persist the public outcome and only the knowledge actually
   gained by each PC/NPC/monster. Re-read actor cards rather than treating the
   historical final combat projection as current state. On `record-event` or
   `record-outcome`, keep `--event-knowledge-cause witnessed` only for actors
   directly present and capable of perceiving the information. If the party
   later briefs an absent, unconscious, newly joined, or replacement actor, run
   a separate source-cited handoff with `--event-knowledge-cause told_by` and
   name only the actual recipient actor ids. Update the manifest clue's
   `known_by_actor_ids` projection to match the resulting ledgers; never copy
   knowledge merely because the party collectively has it.
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
    For a charged magic item that grants spells, add one source-bound item with
    its exact charge maximum, recovery and last-charge formulas, casting-time
    overrides, attunement/class-list restrictions, and active spell artifact ids.
    Cast it through the public spell tool with `source_item_id`; verify one atomic
    action/charge payment, automatic effect, Core receipts, and any service-owned
    last-charge roll. At an actually reached printed recovery trigger, call
    `inventory_change(action="recharge")` and verify its random-stream receipt.
    Never add the item spell to the actor's ordinary spell list, pay a spell slot,
    pre-roll the resource, or patch the charge count.
13. Give every source-cited scene event a stable identity derived from the run,
    scene, event type, and resolved summary. Before writing progress, merge the
    new entry into the existing `full_playthrough_events` map; never replace the
    map or reuse a run-only key. Re-read progress after the checkpoint and verify
    that earlier events from the same run and scene remain present.
14. When a resolved event changes an NPC, quest, clue, or machine-verifiable
    world condition, use the public regression driver's `record-outcome` path.
    Give it a stable outcome id and exact source reference. It must atomically
    commit the event, stable world facts, and cause-scoped ActorKnowledge,
    upsert (not replace) the manifest NPC/quest/clue projections, merge world
    state, then sync and verify a checkpoint containing the resulting manifest.
    For an outcome fulfilled in a later scene, pass the actual occurrence scene
    and Scene Atlas location separately from `source_scene_id`: validate the
    excerpt and exact reference against the original source scene, but write
    progress and location only to the occurrence scene. Preserve both scene ids
    in the continuity event; never move the party back to the source scene merely
    to make a delayed rescue, delivery, promise, or return condition validate.
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
18. Before advancing time for a Short Rest, preflight every participant through
    `character_query(view="rest")` with that actor's exact Hit Die keys/counts
    and optional Arcane Recovery allocation. All preflights must report ready
    before the first write. Use the keys currently exposed by each authoritative
    actor card; never derive a class-prefixed key from an older fixture or another
    actor. The server rolls spent Hit Dice, applies Constitution, checks remaining
    dice and the once-per-day Arcane Recovery allowance, and records the random
    receipt. A failed preflight must leave both clock and actors unchanged. Give
    each Short Rest a stable identity derived from the complete normalized member
    choices, duration, and reason. Reuse that identity across its clock, actor,
    knowledge, continuity, and manifest-sync mutations, but never reuse those
    keys for a later rest with different choices or narrative occurrence.
19. Resolve every Long Rest through the atomic public
    `campaign_change(action="party_rest")` surface, and derive occurrence-scoped
    ActorKnowledge and manifest-sync identities from the complete normalized
    member choices, duration, and reason. If that rest commits but its following
    continuity checkpoint fails, retry the exact request first. A stale-revision
    idempotency conflict means the rest may already exist: read its owner/DM-only
    receipt with `state_revision(action="receipt")`. Require its branch and
    before/after entity-revision evidence to match the current campaign and
    actors, reconstruct the exact pre-rest request from those before revisions
    and all member choices, and require its hash to match the receipt. Then
    require the receipt's members, duration, campaign revision, and world clock
    to equal current public state. Also require every member's `rest_history`
    completion/start minutes and any prepared-spell receipt to match the
    authoritative card. Only after all checks pass may the driver commit the
    missing continuity event and checkpoint.
    Never run the rest twice, edit the database, or accept a receipt from an
    intervening campaign mutation.
20. When a manifest PC is dead or departed, build one replacement through the
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

When a source rule calls for a random encounter check or table roll, use the
public driver's `roll-source` action with a stable occurrence-specific roll id,
the exact dice expression, Scene Atlas location, expanded chunk reference, and
verbatim rule excerpt. The action advances the server-owned random stream,
records the receipt and result in scene progress and continuity, and syncs the
manifest. Use a DM audience for hidden encounter checks. If the result triggers
a second table roll, give that roll a different id and perform it through the
same action; never generate either result client-side.

## Snapshot and branch-isolation audit

Run destructive rehearsal steps on a disposable branch created from a verified
source checkpoint. Carry fresh campaign/actor/scene revisions and idempotency keys
through every mutation.

Use scene-level checkpoint batching on a campaign's main timeline. Pass
`--defer-checkpoint` only to repeated `prepare-statblock` calls on the main
timeline and to these public playthrough-driver actions:
`prepare-narrative-npc`, `resolve-check`, `record-event`, an intermediate
`record-outcome`, `advance-time`, `roll-source`, `stand-up`, `provision-source-item`,
`transfer-source-item`, `acquire-loot`, `spend-coins`, `spend-item`, and
`use-consumable`. Each action must still commit its authoritative state, exact
source reference where applicable, event/facts, ActorKnowledge, and manifest
mutation before returning; only its action-local snapshot is omitted. After the
related preparation, checks, events, loot, expenses, consumables, and ordinary
time advances are complete, call the public `checkpoint` action once with a
stable label that identifies the scene and outcome, then verify that snapshot.
Re-read the public manifest and require the returned snapshot id in
`snapshot_dag.nodes` and as `snapshot_dag.head_snapshot_id`; seeing it only in
the separate runtime projection does not close the scene. A deferred scene is
not complete until this terminal checkpoint exists. If transport or the process
stops first, resume the same idempotent actions, re-read public state, and create
the missing scene checkpoint; never repair the database or fabricate a manifest
head.

Never defer a combat-end checkpoint, PC death or stable recovery, replacement
handoff, level advance, Short or Long Rest, major branch point, module
transition, or campaign ending. Never combine both `--defer-checkpoint` and an
isolated `prepare-statblock` branch: an isolated branch requires its own actor
checkpoint so it can close and return without contaminating the source branch.
For branch regression, keep only a verified parent checkpoint and the completed
branch checkpoint unless an intervening key event above requires another one.
Do not create one snapshot for every ordinary roll, narrative note, loot line,
or repeated source-identical actor.

Create and verify additional checkpoints after key combat and during genuinely
long scene walks where recovery would otherwise require repeating substantial
play. Then:

An exact checkpoint retry may encounter a newer manifest revision after its
sync. Reuse only the verified snapshot with the same stable label on the current
branch; do not create a semantically different checkpoint under that label or
silently select a same-named snapshot from another branch.

If the parent snapshot's built-in Core fingerprint is unavailable in the current
runtime, do not relock the live branch and retry a normal restore. Inspect the
target with `snapshot_query(view="core")`, review the old/new fingerprints, and
rerun `branch-from-snapshot` with an explicit Core-conversion reason. The public
driver must use `branch_change(action="create_core_upgrade")`, preserve the old
snapshot checksum, and verify the converted child checkpoint before play resumes.
A snapshot with no recorded Core lock remains blocked for an edition migration.

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

Keep parser warnings and review-only candidates in the report. A warning that
demotes source-printed Spellcasting to a descriptive passive blocks that
spellcaster from combat until the importer is repaired and the actor is recreated
from a clean parent snapshot. Warnings are evidence of fail-closed behavior, not
permission to fabricate missing content. A successful
corpus result means the exercised public workflows passed; it does not claim that
every optional rule or every possible encounter path was executed.
