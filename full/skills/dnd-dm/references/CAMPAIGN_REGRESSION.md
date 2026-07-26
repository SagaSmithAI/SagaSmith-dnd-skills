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
   must restore the phase that was exposed on entry. Its stage idempotency identity
   must include the logical source key, active parent module id, actual source
   content hash, and resolved title. An exact retry reuses the staged job; changed
   content or a later active parent creates a new revision instead of colliding
   with the earlier refresh.
   For every repeatable driver mutation whose authored content may legitimately
   be identical later, supply a non-empty stable `--occurrence-id`. Reuse it only
   to retry that exact occurrence; use a new id for a later identical check,
   scene advance, event, stand, source-state initialization, time advance,
   Short/Long Rest, stable recovery, XP award, narrative-NPC creation,
   source-item transfer, explicit checkpoint, or manual manifest sync. Human
   summary/reason text and mutable request fields are payload, not occurrence
   identity. Actions with an explicit business
   id (`roll-id`, outcome, acquisition, spend, consumable-use, damage-event, or
   activity-event id) use that id instead.
2. Read `module_query(view="index")`. Visit every non-reference/non-overview
   scene through `module_query(view="scene")`; require readable content, valid PDF
   page ranges, and stable scene ids. After `module_search`, take the immutable
   chunk `source_ref` from `module_expand`, including the service-owned
   `content_sha256`; a missing reference is an import/exposure defect, not
   permission to calculate or invent a client-side hash. Exercise every available atlas location by
   writing scoped progress on a disposable branch. Do not invent topology for a
   scene without reviewed or explicit connections. A campaign may revisit the
   same scene after world, quest, party, or objective state changes. Give each
   `advance-scene` visit a stable `--occurrence-id`: an exact transport retry
   must reuse that id and the same target manifest, while a later visit receives
   a new id even if the payload is identical. A target-scene-only or
   target-manifest-derived key is invalid because hubs, towns, and headquarters
   are intentionally revisited and mutable payload is not occurrence identity.
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
   source reference and document checksum. If extraction cannot find a party-size
   range, search the complete normalized document, expand every plausible hit, and
   visually inspect the introduction and character-creation pages. A semantic
   search miss or unrelated numeral hit is not a source range. If the module is
   genuinely silent, stop the source-confirmed gate and have the SagaSmith Agent
   acting as DM record an explicit review before building any PC. The review
   must retain the reviewed module
   pages, search terms, exact fallback rule reference and checksum, selected
   count, and `represented_as_module_recommendation=false`. A completed review
   may use an exact enabled-Core design baseline, but it must not relabel that
   number as the module's recommendation; never silently default to four. Use a
   level appropriate to the adventure segment. Exhaust advancement follow-ups,
   prepared spells, features, derived-state re-reads, and a verified snapshot
   before returning to `play`.
   The same gate must verify complete class and background equipment, starting
   wallet, and background characteristics. If the enabled 2014 catalog exposes
   only a sample background, use the Core custom-background rule through the
   public content-apply path to create distinct legal backgrounds; do not either
   clone the sample across the whole party or import an inactive setting option.
4. Prepare every important named NPC and every NPC/monster required by the
   selected encounter. When the module provides only a narrative identity and no
   combat statblock, use the public driver's `prepare-narrative-npc` path: cite
   the active module/scene/chunk/page/hash and an excerpt containing the exact
   name, assign the creation a stable `--occurrence-id`, enter `lobby`, create
   `character_create_from(mode="narrative_npc")`,
   verify `combat_eligible=false` plus the `narrative_only`/`source_bound` tags,
   restore `play`, register the actor in the manifest, and verify its checkpoint.
   For a source-counted anonymous group, create one actor per actual instance.
   Set `--narrative-npc-source-identity` to the exact printed group label and a
   distinct stable `--narrative-npc-instance-key`; the actor name must be
   `<source identity> [<instance key>]`. Require the additional
   `anonymous_source_instance` tag. Do not invent proper names merely to satisfy
   database uniqueness, and do not collapse several NPCs into one knowledge scope.
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
   action is an Agent-as-DM boundary only when it becomes relevant; it does not authorize
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
   compatible weapon for that mode. When multiple compatible weapons remain,
   the Agent performs the DM review from the exact statblock and current
   loadout; missing or conflicting source evidence remains external review.
5. In `play`, select one source-printed non-combat check. Read the exact scene,
   preserve its ability/skill and DC, resolve it through `character_check`, and
   commit the event, stable facts, per-witness ActorKnowledge, and snapshot with
   one `memory_change(action="commit")`. A skill label belongs in the cited evidence; use
   `kind="check"` unless the tool contract explicitly defines another kind.
   Assign each check an explicit stable `--occurrence-id`. The run, scene, Scene
   Atlas location, check kind, ability/skill, actor, DC, proficiency,
   advantage/disadvantage, and exact source chunk are immutable retry payload,
   not identity. Separate rolls must never reuse progress, dice, continuity,
   knowledge, or manifest-sync keys even when every payload field is identical.
   If the 2014 source directly opposes two creatures' efforts, use
   `character_check(action="contest")` (or the driver's `resolve-contest`) with both actors and
   both abilities/skills. Never replace the contest with an invented fixed DC.
   The target and source roll modes are independent; a source instruction such
   as "the enemies make a check with advantage for the group" applies advantage
   only to that enemy side. Compare totals atomically, and preserve
   `tie_no_change` rather than declaring either side successful on a tie.
5a. When the active route invokes the 2014 DMG chase rules, run
   `scripts.regression_chase` through the public stdio MCP exposure. Bind
   `chase(action="start")` to the exact expanded scene `source_ref`, excerpt, quarry,
   pursuers, and printed starting distance. Require `mode="theater_of_the_mind"`
   and the absence of a battle map. Let `chase(action="take_turn")` own initiative,
   distance, Dash counts, extra-Dash Constitution checks, chase exhaustion,
   Urban Chase Complications, damage, and the server random stream. A module
   transition such as a quarry ducking into a destination is legal only when
   the `close_transition` carries its own exact same-scene `source_ref` and
   `source_excerpt`; require its `summary` to equal that normalized excerpt,
   including when the transition is stored in a different chunk from the
   starting-distance evidence. Seal the
   completed chase and its manifest/world-state update with one checkpoint;
   never checkpoint each chase turn or replace the chase with a fabricated
   outcome event.
6. Before combat, read the exact encounter scene and its location. Call
   `module_query(view="readiness")` with every source/DM-established group.
   `required_count` is the complete group count, not `len(actor_ids)`: derive it
   from an exact printed count, a persisted random-table roll, or an explicit
   branch-local DM composition fact, and prepare all required cards. Include other
   printed hostiles as initial, reinforcement, or optional groups, or record the
   scene-supported reason they are absent.
   When the source says a group starts outside the fight and must climb, cross,
   arrive, or otherwise spend time before joining, pass those actor reports as
   delayed reinforcements. Keep them out of `combat_start`; queue each through
   public `combat_join` immediately after the trigger so the engine admits them
   only at the next round boundary. Do not place them on the initial map or let
   the auto-runner target them before they enter.
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
   In a mixed group, pass only the source-named hiders as hidden actor ids. Do
   not hide their visible allies or include those allies in a shared Stealth
   profile. If the source says a present NPC waits until a later round, keep it
   in the initial participant set but suppress its earlier turns with the exact
   delayed-action excerpt; reserve `combat_join` for actors actually outside the
   fight.
   Preserve source-authored NPC tactics as ordered opening casts with exact
   excerpts. Charged item spells must call `combat_cast_spell` with the actual
   `source_item_id`; never copy the spell into the NPC's ordinary prepared list
   or patch charges. A spell printed as cast before initiative must instead use
   public noncombat `character_action(action="cast_spell")` before
   `combat_start`, paying its slot and starting concentration. Bind a printed
   Invisibility effect to the exact spell card and condition; it ends after the
   invisible actor makes an attack or casts any spell, when its duration expires,
   or whenever concentration ends, while the triggering attack still receives
   the unseen-attacker benefit. Incapacitated—and therefore Paralyzed,
   Petrified, Stunned, or Unconscious—ends concentration. Bind a printed
   first attack to that actor's reviewed weapon rather than allowing generic
   weapon preference to override it. When an effect-only weapon hit opens an
   on-hit ruling, settle its printed condition and escape terms through
   `combat_choice(action="on_hit_ruling")`; a restrained target uses
   `combat_check(action="escape")`, spends its action, and clears the condition
   only on success. When the reviewed on-hit text instead prints a saving throw
   plus damage, use `combat_choice(action="on_hit_ruling")` with
   `payload.selection.id="saving_throw_damage"` and the exact ability, DC, dice,
   damage type, success treatment, and excerpt. Include
   a structured zero-HP effect when printed; for a giant spider Bite this keeps
   a target reduced to 0 HP stable, Poisoned for 1 hour, and Paralyzed while
   poisoned. Never dismiss that explicit save-and-damage clause or apply only
   its final condition. If a source says a living NPC surrenders at an HP threshold
   only when escape is impossible, confirm both predicates from current state and
   end with `status="surrender"` before another attack. Do not relabel surrender
   as defeat, death, or a generic truce.
   A module-specific encounter procedure does not need a new Core mechanic.
   Preserve its exact source excerpt, invoke the reviewed action through the
   public tool, and let the SagaSmith Agent perform the resulting DM ruling.
   If the action returns `pending_ruling`, inspect its payment and latest
   revision before applying any generic public dice/state/continuity writes;
   never pay the action twice or invent a `combat_choice` window.
   When module prose establishes relative placement without a numeric map (for
   example, creatures "clustered tightly" around a door), the Agent may map that
   fact onto the temporary combat grid through repeated
   `--agent-position-json` declarations. Every declaration must name a canonical
   participant, unique in-bounds cell, exact encounter excerpt, and explicit
   ruling reason. The driver passes those positions only through public
   `combat_start`, records the ruling in its report, and rejects unsupported,
   overlapping, or uncited placements.
   For a source-authored abstract casualty cohort, pass the printed initial
   count, hostile activity, casualty dice, and recharge instruction through the
   encounter driver's source-casualty declaration. Require a descriptive
   activity card, server-side recharge/casualty rolls, a bounded idempotent
   manifest projection, and no attacks against PCs while that procedure is
   active. For a source-authored minimum separation, pass the exact distance
   excerpt through the source-separation declaration; keep the hostile at or
   beyond it and do not make melee-only actors approach illegally.
   Before initiative, have the Agent inspect the canonical equipped attacks
   against that geometry. If an owned ranged or thrown weapon is present but
   not equipped, declare a pre-combat party loadout and let the driver call the
   public inventory facade in Play. Do not treat backpack ownership as an
   executable combat attack, equip during active combat for free, or bypass
   ammunition and range settlement. Keep the short participant-identity excerpt
   distinct from any longer multi-sentence encounter-procedure excerpt.
   If a source designates one actor to retreat after any printed number of other
   hostiles fall, configure that actor with the defeated-count threshold. The
   threshold neither ends combat nor skips intervening turns: the designated
   actor attempts to leave on its own turn, and other living hostiles keep
   fighting. Bind retreat to one defeated actor id only when the source names
   that exact trigger. A downstream encounter receives the actor as a
   reinforcement only when the recorded source departure actually succeeded.
   If retreat instead triggers after cumulative server-settled damage or a
   single critical hit, configure the printed damage threshold and critical
   trigger together with their exact excerpt. Count only committed applied
   damage and server-confirmed critical attacks, resume from the bounded combat
   log after interruption, and let the actor depart on its own turn.
   Automated party spell tactics may select only currently prepared spells or
   spells the actor actually knows. A spellbook entry alone is not castable.
   Choose the lowest available legal slot at or above the spell's level; when
   lower slots are empty, preserve the public higher-slot cast and its scaling
   rather than falling back to a weapon while usable slots remain.
   If an attack returns a defensive reaction window, stop before ending the
   attacker's turn. For automated Shield tactics, use the lowest offered slot
   only when the candidate's projected AC changes the hit to a miss; decline an
   attack that still hits, but use available Shield against Magic Missile.
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
13. Give every source-cited scene event an explicit stable `--occurrence-id`.
    Before writing progress, merge the
    new entry into the existing `full_playthrough_events` map; never replace the
    map or reuse an occurrence id for a later event, even when its scene, event
    type, and summary are identical. Re-read progress after the checkpoint and
    verify that earlier events from the same run and scene remain present.
    Every `advance-scene` must cite the exact transition text from the manifest's
    current scene through `--source-scene-id`, `--source-ref-json`, and
    `--source-excerpt`. The driver persists that evidence under the occurrence id
    and rejects an arbitrary jump, a stale source scene, or a changed retry.
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
15. Award one source-defined encounter XP parcel to the exact actors who
    participated in earning it. A participant who dies later in that encounter
    keeps the earned share on the retained actor record; death does not erase XP.
    A replacement or relief party earns only the creatures and objectives that
    group actually resolves, and never inherits a predecessor's award or
    progression. Exclude an actor who did not participate, left before the
    encounter, or joined afterward rather than using a historical party count.
    Use one public `award-xp` call when all recipients receive the same integer
    share. If equal division produces a fractional result but the public schema
    accepts only integer XP, have the Agent acting as DM select and record an
    explicit rounding policy from the locked advancement rules.
    A total-conserving deterministic remainder is acceptable only when the
    audit records the ordered remainder recipients, no two shares differ by
    more than one XP, and no allocation is silent. Give each public award call a
    stable `--occurrence-id`; a split remainder therefore uses distinct ids for
    its distinct recipient groups. The exact retry keeps the same id and payload.
16. Advance each eligible survivor through the public regression driver's
    `advance-level` path one target level at a time. Supply the exact source
    reference that established the XP or milestone, an explicit fixed/rolled HP
    method, the intended return phase, and every caller-owned choice. The driver
    must enter `lobby`, replay the stable level transaction when resuming,
    exhaust all returned and newly applicable class/subclass feature artifacts,
    validate any subclass and known/spellbook choices against the active catalog,
    verify that newly level-eligible always-prepared subclass spells were
    materialized, add any newly chosen prepared-class spell cards with
    `method="class_prepared"`, replace the complete prepared-spell list when the
    follow-up requires it,
    re-read and verify the actor, and restore `play`. For a single advancement,
    sync the manifest and verify its checkpoint. For a contiguous group of
    eligible party members advancing from the same source-cited scene or
    downtime boundary, pass `--defer-checkpoint` only after each actor's complete
    advancement can be verified, then call one public `checkpoint` after the
    final actor and verify the aggregate party state before entering another
    sourced scene. Raising maximum HP does not heal current HP. A newly built
    replacement advanced to the module's source gate therefore keeps its
    pre-advancement current HP until a legal rest, spell, feature, potion, or
    other public healing path changes it. Never edit the raw sheet, silently
    choose a subclass or feature, advance an ineligible/dead actor, treat the
    level integer alone as a complete advancement, or patch current HP to the
    new maximum.
17. Advance campaign time through the public regression driver's
    `advance-time` path whenever travel, waiting, or a source-triggered interval
    matters. Give each interval a stable `--occurrence-id`, cite the exact scene
    chunk and excerpt, supply a positive
    minute/hour/day count, and state any Agent-as-DM ruling used to turn narrative timing
    such as "late in the day" into a duration. The service-owned campaign clock,
    continuity event, actual-witness ActorKnowledge, snapshot, and manifest sync
    must all agree. Never update only the manifest's projected clock or invent a
    duration without an explicit audited ruling.
    Treat that count as actual elapsed time rather than an effect-unit selector.
    `60 minute`, `1 hour`, and two consecutive `30 minute` advances must expire
    the same one-hour actor and world effects exactly once. The public receipt and
    subsequent actor/campaign reads must agree on every advanced or expired effect;
    never round or directly patch a sub-hour/sub-day remainder.
18. Before advancing time for a Short Rest, preflight every participant through
    `character_query(view="rest")` with that actor's exact Hit Die keys/counts
    and optional Arcane Recovery or Natural Recovery allocation. Natural
    Recovery also requires declared meditation in `rest_activity_minutes`; it
    resets on a Long Rest rather than on a campaign-day boundary. A source-bound
    level-20 Sorcerer's four-point Sorcerous Restoration is automatic. When a
    conscious source-bound 2014
    Bard performs Song of Rest, include that participating Bard's actor id as
    `song_of_rest_source_actor_id` only for members who spend at least one Hit
    Die and can hear the performance. Include the complete
    `rest_schedule`, any Ki meditation under `rest_activity_minutes`, and one
    `attune_item_id` when that rest is devoted to a source-required item. The DM
    must verify the item's exact source prerequisite against the actor card and
    pass `attunement_prerequisite_confirmed=true`; an unproven prerequisite
    blocks the rest mutation.
    All preflights must report ready
    before the first write. Use the keys currently exposed by each authoritative
    actor card; never derive a class-prefixed key from an older fixture or another
    actor. The server rolls spent Hit Dice, applies Constitution, checks remaining
    dice, Arcane Recovery's once-per-day allowance, Natural Recovery's
    once-per-Long-Rest allowance, Sorcerous Restoration's capped four points,
    and the level-scaled single Song of Rest die per eligible creature, and
    records the random receipt. A
    failed preflight must leave both clock and actors unchanged. Give
    each Short Rest a stable `--occurrence-id` and reuse it across that
    occurrence's clock, actor, knowledge, continuity, and manifest-sync
    mutations. A later rest needs a new id even when its normalized members,
    duration, and reason are exactly identical. Reusing an id with changed
    choices must fail rather than create another rest.
    The Short Rest's minute clock write must also advance minute/hour/day actor
    and world effects by the actual elapsed minutes. In particular, an established
    one-hour Giant Spider poison/paralysis effect expires after a legal 60-minute
    Short Rest (or two 30-minute advances), while unrelated conditions remain.
19. Resolve every Long Rest through the atomic public
    `campaign_change(action="party_rest")` surface. Supply one stable
    `--occurrence-id` and use it for party-rest, clock, ActorKnowledge,
    continuity, and manifest-sync identities. A later rest needs a new id even
    when its complete normalized member choices, duration, and reason are
    identical. If that rest commits but its following
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
    Each member needs a schedule whose minutes equal the shared clock advance.
    The normal 2014 path is at least 480 minutes with at least 360 minutes of
    sleep, no more than 120 minutes of light activity, and less than 60 minutes
    of strenuous activity. A 240-minute path is legal only when every included
    actor using it has a source-bound `Trance` feature and records 240
    `trance_minutes`; never infer the exception from a race name. For a changed
    2014 prepared list, put the complete selected list in that member request
    and reserve `rest_schedule.light_activity_minutes` equal to at least the sum
    of every selected spell's level. This is the full-list preparation cost, not
    the levels of only the replacements. A minimal 240-minute Trance therefore
    cannot also change the list without extending the schedule.
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
    checkpoint after the manifest update. Re-read every ending condition after
    registration: an active-party `sheet.progression.level` check must follow
    the replacement party slot, while every other actor check remains attached
    to the predecessor unless its own source condition says otherwise.
21. Advance to the exact indexed conclusion scene only after its source-defined
    prerequisites are true in authoritative runtime state. Record the decisive
    conclusion facts, NPC state, quest state, world state, and actual-witness
    ActorKnowledge through public outcome and manifest paths with exact source
    references. Narrative prose by itself is not a machine-verifiable ending.
22. Configure each source-defined ending through the public regression driver's
    `configure-ending` action. Its `source_ref` must use the manifest source
    schema and preserve the asset/checksum, module, scene, chunk, page, content
    hash, and exact excerpt used as evidence. Define checks against specific
    manifest paths, world facts, actor/NPC state, quest state, and other public
    projections needed by that ending; do not use a broad narrative string as a
    substitute for the printed conditions. After a parser-backed module refresh,
    require each ending citation for that same source asset to resolve to exactly
    one new chunk with the same content hash and excerpt. Re-read the condition
    and require its module, scene, chunk, pages, heading, and any
    `current.scene_id` check to reference the active revision. The refresh must
    fail closed on zero or multiple matches and must scope idempotency to the
    exact refreshed manifest payload.
23. Call `verify-ending` without deferral. Require every returned check to pass,
    the selected ending id to be achieved, the manifest and ending state to be
    `completed`, and a verified terminal checkpoint to become the Snapshot DAG
    head. Only `combat.active=true` is an active-combat blocker. A retained
    `combat_query(view="status")` projection with
    `snapshot_role="historical_final_encounter"` and
    `combatant_state_is_current=false` is audit evidence and must not block a
    conclusion.
24. Ordinary final-scene outcome writes may defer their individual checkpoints
    only as one immediately closed terminal batch. End that batch with one
    public checkpoint before `verify-ending`. Never defer ending verification
    or its terminal checkpoint, and never reuse a final-scene batch key for a
    later retrospective correction.

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

For scene advances, narrative-NPC creation, source-cited noncombat checks,
`record-event`, `stand-up`, `initialize-source-state`, `advance-time`,
`transfer-source-item`, and XP awards, pass the explicit
`--occurrence-id` described above. For environmental damage, use one distinct
`--damage-event-id` for each actual damage occurrence; for `use-activity`, use
one distinct `--activity-event-id` for each use, including another use after a
legal rest in the same scene. Retry an interrupted occurrence with the same id
and unchanged payload. Never derive these ids only from scene, actor, expression,
activity, summary, reason, member choices, or recipient set.

## Snapshot and branch-isolation audit

Run destructive rehearsal steps on a disposable branch created from a verified
source checkpoint. Carry fresh campaign/actor/scene revisions and idempotency keys
through every mutation.

Use scene-level checkpoint batching on a campaign's main timeline. Pass
`--defer-checkpoint` only to repeated `prepare-statblock` calls on the main
timeline and to these public playthrough-driver actions:
`prepare-narrative-npc`, `resolve-check`, `record-event`, an intermediate
`record-outcome`, `advance-time`, `roll-source`, `initialize-source-state`,
`stand-up`, `use-activity`, `provision-source-item`, `transfer-source-item`,
`acquire-loot`, `spend-coins`, `spend-item`, and `use-consumable`.
`apply-damage` may defer only while the authoritative resulting HP remains above
0; the driver must force a snapshot when damage reaches 0 HP even when deferral
was requested. `advance-level` may also defer only as part of one contiguous
same-scene or same-downtime party-advancement batch; every actor must complete
and verify all required follow-up before the next actor, and one aggregate public
checkpoint must immediately close the batch. Each action must still commit its authoritative state, exact
source reference where applicable, event/facts, ActorKnowledge, and manifest
mutation before returning; only its action-local snapshot is omitted. After the
related preparation, checks, events, loot, expenses, consumables, and ordinary
time advances are complete, call the public `checkpoint` action once with a
stable label that identifies the scene and outcome plus a distinct stable
`--occurrence-id`, then verify that snapshot. Reuse the occurrence id only for an
exact retry. A later visit may reuse the reader-facing label, but it must use a
new occurrence id and create a new DAG node.
Re-read the public manifest and require the returned snapshot id in
`snapshot_dag.nodes` and as `snapshot_dag.head_snapshot_id`; seeing it only in
the separate runtime projection does not close the scene. A deferred scene is
not complete until this terminal checkpoint exists. If transport or the process
stops first, resume the same idempotent actions, re-read public state, and create
the missing scene checkpoint; never repair the database or fabricate a manifest
head.

Never defer a combat-end checkpoint, PC death or stable recovery, replacement
handoff, standalone level advance, Short or Long Rest, major branch point,
module transition, or campaign ending. A deferred party-advancement batch is
incomplete and must not enter another sourced scene until its aggregate
checkpoint verifies. Never combine both `--defer-checkpoint` and an
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
sync. Recover only through that occurrence's owner/DM idempotency receipt. Require
the receipt request hash, label, branch, response snapshot id/slot/parent, current
branch head, public snapshot list, integrity verification, and manifest DAG head
to agree. Never recover by label search: distinct checkpoints may legitimately
share a reader-facing label, and a same-named older or sibling-branch snapshot is
not the retried occurrence.

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
