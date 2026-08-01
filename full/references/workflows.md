# Runtime Workflows

Full Runtime uses the `sagasmith_dnd` MCP server. See `mcp-contract.md` for the
complete public facade and mutation contract. Never call an internal/retired tool
name copied from an old prompt.

Read `long-form-narrative-architecture.md` for the complete cross-layer ownership
model and the distinction between immutable source, Agent adjudication, engine
settlement, continuity ledgers, the playthrough manifest, and Snapshot recovery.

## Exposure and session start

1. Call `storage_status`, then `campaign_query(view="list")` and select a campaign.
2. Call `exposure_open(campaign_id, principal_id)`. Its phase is authoritative.
3. Call `exposure_search` and `exposure_inspect`, then load only the groups needed
   for this step with `exposure_load`.
   Use player-safe read/action groups for player Agents. Load `*_control`,
   `combat.save`, or `combat.map` only in an owner/DM exposure.
4. A native dynamic client refreshes `tools/list`. A client that cannot refresh
   calls the loaded domain tool with `exposure_call`; consume its structured result
   exactly like a native call.
5. In `play`, read `module_query(view="current")`, expand the exact scene with
   `module_query(view="scene")`, read recent `campaign_event(action="list")`, and
   call `continuity_context` separately for each acting PC or NPC. For a DM
   deciding module-authored behavior, pass `related_refs` for every relevant
   actor, scene/location, active quest, and key item, then read the returned
   exact `module_evidence`.
6. Refresh `campaign_query(view="party")` and relevant
   `character_query(view="get")` cards. Never carry a card or revision across a
   write, phase transition, branch checkout, or restore.

An exposure belongs to one MCP session and principal. Every other Agent opens its
own exposure. Loading a group for one Agent must not expose it to another.

## Module-authored narrative behavior

1. Search and expand the authoritative module chunks. Keep exact excerpts for
   the whole nearby narrative sequence, not just the first sentence that happens
   to match the current query.
2. Upsert a stable, DM-only `context_anchor` whose subject and `related_refs`
   point to the NPC, scene/location, quest, faction, and key item. Store only the
   exact managed source bindings. Do not encode conditions or actions.
3. Immediately before adjudication, refresh actor cards, current scene,
   inventory/ownership, events, and `continuity_context`. The Agent interprets
   `module_evidence` against that live state.
4. Carry out the decision through existing public checks, movement, combat,
   inventory, time, event, fact, knowledge, and playthrough tools. Standard
   mechanics and random outcomes remain server-owned.
5. Commit only the path that happened. Do not copy DM source into player context,
   transfer one actor's knowledge to another, or persist hypothetical branches.
   After snapshot restore or branch checkout, discard the prior assembled context
   and retrieve it again.

## New campaign and module PDF

1. Without a campaign, open an exposure and load `lobby.bootstrap`; call
   `campaign_create`, then reopen the exposure with the new campaign id.
2. Lock the correct Core edition with `campaign_rules`. Do not silently use a
   different edition or optional publication.
3. Inventory every allowlisted file before importing. Call
   `character_query(view="document")` for character sheets, pregenerated-PC
   packets, and ability-score option files. Its classification and checksum are
   authoritative; these documents never enter `module_import`. Keep explicit
   `manual` score entry available even when the document supplies arrays.
   For a campaign directory, group every document below the same top-level
   campaign folder into one campaign while retaining one immutable module
   revision per physical document. A root-level adventure remains its own
   campaign. Do not create one campaign per appendix, map packet, or supplement.
4. Load `lobby.modules`. For each module PDF call `module_import` in this exact order:
   `stage` -> `inspect` -> `validate` -> `ingest` -> `activate`. Keep the same
   `job_id`; use a stable, stage-specific idempotency key for each write.
5. Review `module_query(view="index")`. Search only selects candidates; expand the
   chosen scene before using its facts. Verify scene boundaries, keeper/public
   visibility, encounter participants, exact source excerpts, spatial locations,
   explicit-evidence spatial connections, and parser warnings. Never treat room
   heading order as connectivity; an empty `spatial.connections` list means the
   parser found no source-backed topology.
   If a PDF map contains required topology, use
   `module-visual-atlas.md`: `module_query(view="assets")` ->
   `module_review(action="render_page")` -> visual inspection ->
   `module_set_progress(spatial_review=...)`. Never infer an edge from room order.
   If a 2014 appendix statblock is image-only, use
   `module-image-content-review.md`. First call
   `module_review(action="recover_statblock")`; the server performs
   layout OCR and independent critical-fact corroboration without requiring model
   vision. If it returns `requires_agent_fill=true`, the Agent reads the returned
   normalized OCR text and exact requirements, supplies the semantic
   `payload.agent_fill` under a fresh idempotency key, and only then re-reads the
   immutable review. Then use
   `character_create_from(mode="module_statblock")`. Only when recovery remains
   ambiguous may an image-capable reviewer render, inspect, and submit the page
   manually. Do not send a 2024 card through this 2014 OCR grammar. Submit a
   complete indexed 2024 candidate with
   `content_kind="dnd5e_2024_statblock"`, or use an image-capable literal visual
   transcription; otherwise leave the card unresolved.
    Also inspect `module_query(view="candidates")`. A `review_ready` candidate may
    be submitted to `module_review(action="submit_content")` only with its exact
     `source_chunk_ids`. Read its structured `ruling_requirement`: complete-text
     review defaults to the Agent, so do not pause merely because the workflow
     calls it a DM review. If `agent_fill_requirements.required` is true, the
     Agent must cover every listed Multiattack as `structured` or
     `agent_ruling`; parser-produced options are never authoritative for module
     creatures. A `blocked` candidate whose requirement names
    `missing_or_conflicting_source_review` is a stop condition. For 2014, first
    use `module_review(action="recover_statblock")` with its managed PDF page.
    For 2024, require complete edition-matching indexed text or capable visual
    review. If ambiguity remains, an image-capable
    reviewer may transcribe only observed fields, or leave it unresolved. A
    text-only Agent cannot claim to have inspected a returned image. Never repair
    OCR from rules memory or silently relabel blocked evidence as reviewed text.
6. Set scoped progress with `module_set_progress`, including
   `current_location_key` and `state.location_scene_id` when the spatial room is a
   separate scene. The location key must be copied from the expanded scene's
   `spatial.locations`; a slug, display label, or guessed room id is not valid.
   Never merge narrative text merely because two scenes refer to the same encounter.
7. If the scene changes by opening hours, daylight, watches, or travel duration,
   treat `state.game_time.elapsed_ticks` as the one elapsed-time authority and
   `state.world_time` as its optional calendar projection. Establish
   `campaign_change(action="clock_set")` before resolving a calendar-dependent branch.
   Advance only source- or DM-established elapsed time with
   `campaign_change(action="clock_advance")`; it updates the tick stream,
   optional anchored calendar, and timed effects atomically. Every minute, hour,
   or day advance should include the canonical target
   `payload.expected_elapsed_ticks`; derive it from the current public
   `game_time.elapsed_ticks` plus the reviewed interval. When a calendar is
   anchored, also include
   `payload.expected_world_time={day,hour,minute,elapsed_minutes}` as a projection
   guard. Never hand-copy a large tick or minute literal without deriving both
   targets. The MCP rejects a missing target or a duration that would land
   anywhere else before changing the timeline or any timed effect. Completed
   combat/chase rounds and out-of-combat spell/ritual casting use the same tick
   stream; do not add a second narrative clock write for them.
   For a completed Short or Long Rest, use
   `campaign_change(action="party_rest")` instead:
   it advances the clock once and settles all named members atomically. Never
   advance the rest clock separately or loop individual actor rests.
8. Load `lobby.characters`. Use `character_create_from(mode="build")` for confirmed
   PCs and `mode="direct"`, `mode="template"`, `mode="statblock"`, or
   `mode="module_statblock"` for mechanically authoritative NPCs and monsters.
   Either statblock mode must cite exact imported evidence; unsupported or absent
   creatures remain unresolved instead of being replaced by a similar one. For
   an important named NPC whose exact module chunk supplies identity but no
   statblock, use `mode="narrative_npc"` with the active source reference and a
   name-bearing excerpt. Its `narrative_only` default mechanics are sentinels and
   cannot be used for a check or combat.
   When the module modifies a named standard creature, import that exact rule source
   and use its source-bound `variant` whitelist; never replace the whole actor sheet.
   If rule-source statblock creation fails because the indexed text split a card
   across columns, retry `mode="statblock"` with the source-established
   page/neighborhood `chunk_ids` and the exact printed heading in
   `payload.source_statblock_name`. Keep the campaign instance name in
   `payload.name`. The deterministic text-layout result must cite only chunks
   from that heading through the next creature core and report
   `source.text_layout_recovery`; it does not require Agent vision. If required
   facts are still absent or conflicting on a 2014 card, use
   `import_query(view="list", kind="rulebook")` to find the retained `job_id`
   whose `source_id` exactly matches the selected source, then call
   `rule_import(action="recover_statblock", payload={job_id, name, page_number?})`.
   `name` is the exact printed creature heading, not a differently named campaign
   instance.
   The server performs 2014 local layout OCR, uses the adjacent creature core to
   disambiguate repeated decorative/narrative copies of the same heading, and
   corroborates critical facts without asking the Agent to inspect an image. Retry with
   `mode="reviewed_rule_statblock"` and the returned `review_id`. Stop for explicit
   source review on low confidence or disagreement. A 2024 card instead uses
   exact edition-matching indexed text with `review_mode="agent_text"`, or an
   edition-matching visual review; `recover_statblock` must reject it. If OCR is structurally
   ambiguous but one exact indexed page still contains the complete card as an
   ordered contiguous chunk segment, a text-only Agent may normalize that segment
   through `rule_import(action="review_statblock",
   payload={job_id,page_number,normalized_content,observation,
   review_mode:"agent_text",evidence_chunk_ids:[...]})`. The MCP verifies source,
   page, ordinal continuity, no invented normalized fact, and no omitted selected
   evidence. This path is forbidden when the indexed facts themselves are missing
   or conflicting. That remaining boundary may require an image-capable reviewer;
   never fill the card from memory or substitute a similar creature.
   Read `module-image-content-review.md` for the distinction between an image-only
   full card and a standard card with module instance changes.
   For an already reviewed shared actor, use
   `character_create_from(mode="portable_card")`. PC, NPC, and monster share one
   card format; import creates a new runtime identity and an empty ActorKnowledge
   ledger. Browse bundled standard monsters/NPCs as `actor_card` catalog entries,
   or export/import a complete `preset_pack` through
   `rule_pack_query(view="actor_presets")`. Never choose a creature by a
   host-maintained name table.
9. Apply every confirmed class/subclass feature and complete species/background
   card, then re-read each actor's `derived` values and unresolved rules.
10. Prepare legal spells with `character_spell_prepare(mode="replace_all")`.
    When setup or advancement should finish with a completed long rest, use one
    atomic `campaign_change(action="party_rest")` for all named members; an
    anchored calendar is not a prerequisite. Do not call individual long rests.
11. Only after every campaign resource has activated and all actors have passed
    their completeness checks, record the opening with one `memory_change(action="commit")`:
    include the opening event,
    deterministic-key objective facts, per-actor knowledge only for actual
    witnesses, and the initial snapshot. Supply a fresh `idempotency_key` and the
    current campaign revision. This requires the owner/DM `lobby.memory_control`
    and `lobby.campaign` groups.

## Share or migrate structured content

1. Enter Lobby and load `lobby.characters`, `lobby.modules`, and, for preset
   libraries, `lobby.rules`. Export a PC/NPC/monster with
   `character_query(view="portable_card")`; review its sheet and notes before
   distributing the managed `.sagasmith.json` artifact.
2. Before module export, call `module_import(action="bind_actor")` for every cast
   NPC, encounter monster, and pregenerated PC. Use stable portable actor ids,
   binding kinds, roles, and actual Scene Atlas keys. Verify
   `module_query(view="actors")`.
3. Call `module_query(view="package", include_package=true)` for an inline
   transfer or use its managed artifact. Do not package progress, continuity,
   ActorKnowledge, branches, random state, or Snapshots as source content.
4. On the target installation, call
   `module_import(action="import_package")` with exactly one inline package,
   managed artifact, or allowlisted source path. Treat returned actor ids as new
   identities. Re-read the imported index, actor bindings, assets, reviews, and
   readiness before play.
5. For a default actor library, export
   `rule_pack_query(view="actor_presets", edition=..., include_package=true)`.
   Import one nested card through portable character creation with the same pack
   plus its exact `artifact_id`. Optional rule dependencies still need normal
   reviewed installation and campaign activation.

## Scene readiness and temporary combat map

1. Build a source-grounded participant manifest from the expanded encounter scene.
   Each group has a stable key, role (`combatant`, `reinforcement`, or `optional`),
   required count, canonical campaign actor ids, same-module `source_scene_id`, and
   an exact normalized `source_excerpt`.
2. Call `module_query(view="readiness")`. Do not start while a required actor is
   missing, Dead/at 0 HP, lacks an executable card, or carries unresolved required
   rules. `source_excerpt` is an evidence assertion and must be an exact normalized
   substring of the expanded same-module scene; use a verified `module_search` hit
   when needed, never a paraphrase. Review surfaced manual rulings and their
   structured `ruling_requirements` rather than hiding them. The Agent resolves
   entries marked `default_resolver="agent"`; missing/conflicting source and
   player-owned choices keep their declared boundary. `ready=true` authorizes
   entry only: automatic effect settlement and component, targeting, passive, or
   on-hit rulings remain separate.
3. Required `combatant` actors go into initial `participant_ids`.
   `reinforcement` actors must stay out and join later through `combat_join`.
   A source group that must climb, cross, arrive, or otherwise spend time before
   joining is a delayed reinforcement, not an initially distant combatant. Pass
   its exact entry excerpt and canonical actor reports to the full-playthrough
   encounter driver; it queues the actors through public `combat_join`. If the
   source names a later round, pass `--reinforcement-round`; otherwise they enter
   at the next round boundary. They are neither targetable nor acting before
   their queued round.
   Use separate hostile and ally reinforcement reports so source-authored
   rescuers remain friendly without becoming party members. When the source
   uses a semantic arrival condition, have the Agent inspect the live combat and
   submit `--agent-reinforcement-trigger-json` with the exact excerpt, future
   round, decision, and observed-state reason. Keep the semantic judgment at
   the Agent boundary and the actual entry in generic `combat_join`.
4. Call `combat_start` only after readiness succeeds. Let it compile a temporary
   combat map from the recorded spatial scene and location. Load the owner/DM
   `play.combat_control` group for this transition. If it falls back to a
   12-by-12 canvas, do not narrate those dimensions as module-authored facts. If
   the selected source chunk explicitly says a participant starts under a
   condition, pass it in that actor's `source_conditions` with
   `duration="encounter"`, the service-returned immutable `source_ref`, and the
   exact excerpt. Apply the group in this one start mutation; do not issue
   per-actor sheet replacements. The server retains the condition through combat
   synchronization and removes the encounter-added condition on `combat_end`.
   If an ordinary removable object is the exact authored cause of that condition,
   the Agent acting as DM may later declare
   `combat_common_action(action="interact_object")` with the object, `interaction`
   `"remove"`, the exact active condition, unchanged `source_ref` and excerpt, and
   a bounded `agent_dm_adjudication` decision and reason. This spends that actor's
   one object interaction for the turn rather than its main action. The server,
   not the driver, verifies ownership and deactivates only the matching source
   condition; the Agent must not patch the sheet or infer that unrelated owners
   of the same condition also ended.
5. Resolve surprise before `combat_start`, but do not turn an adventure's approach
   prerequisite into automatic surprise. A requirement such as "approach carefully
   and without light" only avoids the adventure's automatic alert unless its text
   explicitly promises more. Under 2014 rules, roll each hiding creature's Stealth
   with its canonical card, including armor disadvantage, and compare those results
   against each opposing creature's passive Perception. An opponent that notices
   any approaching threat is not surprised. Determine `surprised` separately for
   every participant; surprise never uses the general group-check rule. Outside
   surprise, when the party is making one collective attempt whose consequence
   applies to everyone, the Agent acting as DM may explicitly call a 2014 group
   ability check. Resolve all participants atomically with
   `character_check(action="group")`; the engine, not the Agent, applies each
   actor card and the "at least half succeed" threshold. Record the comparisons
   and source condition in a campaign event.
6. After `combat_start`, reopen exposure. The server phase is now `combat`; load
   `combat.observe`, `combat.turn`, or `combat.actions` for an acting player.
   Load `combat.control`, `combat.save`, or `combat.map` only for an owner/DM.

## Combat turn loop

1. Read `combat_query(view="status")` and
   `combat_query(view="available_actions", actor_id=...)`. Use the returned current
   actor, revision, budgets, conditions, positions, and derived attacks.
   For automated execution, the Agent must also declare the actor's tactics.
   `--agent-target-priority-json` lists every opponent in exact order and works
   for PCs, allies, and hostiles. `--agent-spell-priority-json` orders supported
   structured spells and their target policies.
   `--agent-weapon-priority-json` orders exact weapon/mode pairs and any selected
   structured Multiattack. These are Agent decisions retained in the report,
   not driver defaults. If none applies, return `pending_ruling` instead of
   selecting an inventory entry or spell by hidden code order.
2. For every attack, use `combat_preflight_attack` immediately before
   `combat_resolve_attack`. Never supply replacement attack bonuses or damage
   formulas. Multiattack is a distinct action choice, not a passive increase to
   `derived.attacks_per_action`. To choose a source statblock Multiattack, pass one
   `derived.multiattack_options` id on the first attack and consume only its
   remaining source-defined entries. Omit the id to choose one ordinary Attack.
   An unstructured/descriptive Multiattack remains an Agent-as-DM adjudication
   boundary but never blocks that ordinary single weapon attack.
   If an exact reviewed passive makes the triggering attack deal conditional
   extra damage, first query its source card with
   `content_solution(action="query")`. On the first use, the Agent must read the
   complete card and exact managed source, then persist one schema-v2
   `attack.after_hit` solution containing the printed `damage.apply` expression
   through `content_solution(action="compile")`. Keep the occurrence-specific
   condition decision with the attack: the owner/DM
   Agent supplies one `source_conditional_extra_damage` ruling containing the
   reviewed feature id, persisted plan fingerprint, explicit eligible target ids,
   its exact stored excerpt and printed dice expression, typed trigger facts,
   decision, and reason. The encounter driver uses
   `--source-extra-damage-ruling-json` to bind the same evidence to eligible
   melee or ranged weapons, eligible targets, rounds, and a bounded application
   count. For a printed advantage-or-adjacent-ally condition, declare the
   reusable applicability mode rather than predicting later map state. At each
   attack the driver derives the current branch (attack advantage, or no
   disadvantage plus an adjacent non-incapacitated ally), records the exact
   qualifying ally ids, and the server verifies those positions and conditions.
   Do not apply the rider
   later with `combat_hp_change`: all damage dice must resolve simultaneously,
   double together on a critical hit, and share one target-state mutation.
   When exact scene evidence and current relative position give the target Half,
   Three-Quarters, or Total Cover, the Agent supplies the attacker, distinct
   target, attack mode, exact source reference/excerpt, decision, and reason.
   Send only the cover degree; the server verifies the current-scene evidence
   and the D&D engine derives +2 AC, +5 AC, or an untargetable target. Never
   calculate a numeric cover bonus in the Agent or encounter driver.
3. When an attack returns `pending_reaction`, read the target's
   `combat_query(view="reactions")`, then use
   `combat_choice(action="resolve_defense")`. Do not roll or apply damage twice.
   Do not end the attacker's turn while this window is pending. Automated Shield
   tactics use the lowest offered slot only when +5 AC changes the hit to a miss;
   otherwise decline. Available Shield should block Magic Missile.
   When a committed hit instead returns `pending_on_hit_ruling_id`, the Agent
   reviews the complete card and exact excerpt. When the result also reports
   `semantic_solution.status="compilation_required"`, compile the reusable
   schema-v2 recipe through `combat_choice(action="compile_solution")`, then
   settle that same paid window through `combat_choice(action="execute_plan")`.
   Use `on_hit_ruling` only for an occurrence-specific effect that the registered
   engine calls cannot express. In that fallback, use `apply_condition` only
   for printed action/check escape terms. Use `saving_throw_condition` for an
   immediate save-gated timed condition with printed turn-end repeat saves; pass
   the exact condition, ability, DC, repeat timing, duration, and excerpt.
   `combat_end_turn` rolls those repeat saves automatically without spending an
   action. Use `saving_throw_damage` for printed save-dependent extra damage.
   Never classify by creature name, parse the prose inside the driver to invent
   the Agent settlement, silently dismiss a structured rider, repeat the already
   committed hit, or let the driver mistake a non-escape effect for an action
   escape.
4. Resolve movement with `combat_movement`, checks with `combat_check`, common
   actions with `combat_common_action`, spells with `combat_cast_spell`, activities
   with `combat_use_activity`, and damage/healing with `combat_hp_change`.
   For a structured monster point-radius, line, or Wing Attack area, include
   every living actor geometrically inside it in `target_contexts`, with the
   Agent-reviewed cover degree. Let the runtime apply Dexterity-save cover
   bonuses or Total Cover; never default all targets to no cover.
   A locked standard card that reports
   `semantic_solution.status="engine_implementation_required"` must stop before
   payment and be implemented in the engine; do not reinterpret it as custom
   prose. Do not require import to translate every custom card. A repeatable
   custom activity or spell first returns
   `semantic_solution.status="compilation_required"` without payment. Compile it
   once with `content_solution(action="compile")`, using exact managed evidence.
   The persisted solution is a strictly typed recipe of allowlisted engine
   function calls, not a second rules engine; then retry the action with that
   stored contract. A genuinely one-off
   descriptive activity, unstructured spell, or scene procedure with printed
   save damage remains a two-call recoverable transaction with one immutable
   semantic identity. Before paying, place the complete canonical
   `agent_ruling_commitment` in that action's declaration/payload. Settle it with
   `combat_hp_change(action="save_damage")` using the identical target order,
   source card, save/DC, damage terms, exact mechanics excerpt, and active-scene
   Agent ruling. The server alone rolls one shared damage result, rolls every
   target save, rounds half damage down, and applies all sheets atomically.
   Never roll the damage in the driver, divide it there, or follow the paid
   activity with separate per-target damage calls.
   Do not promote parseable prose to a standard engine rule without its complete
   timing transaction. False Appearance remains a descriptive Agent ruling;
   Legendary Resistance remains an Agent decision until a failed-save window
   can both replace the outcome and spend the limited use atomically.
   When a predeclared Agent object interaction ends an exact encounter-source
   condition, execute it before choosing the actor's action, re-read combat and
   character state, then continue the same turn with the remaining main-action
   budget. Do not encode the object, creature, or source phrase as driver logic.
   After movement, settle every returned opportunity-reaction window before the
   next action. A rescue move can damage or incapacitate the rescuer before a
   Medicine attempt, so re-read both actor cards after the reaction.
   For a structured multi-attack spell, cast once and keep its returned
   `resolution_id`. Resolve each attack separately with `combat_resolve_attack` and
   `action.spell_resolution_id`, refreshing `expected_revision` after every write.
   The cast spends its action and slot once; the individual attacks spend neither.
   Resolve any owned Shield window before the next attack. Do not end the caster's
   turn or the encounter until `remaining_attacks` is zero.
   Automated tactics must read the current prepared/known spell projection, not
   every spellbook card. Select the lowest available legal slot; when only a
   higher slot remains, pass that `cast_level` and preserve the spell's scaling.
5. A source offer such as “10 gp grants advantage on DC 15 Persuasion” requires
   the stated payment/offer fact and
   `combat_check(action="improvise", ability="persuasion", dc=15)`. Only on success
   call `combat_join` through `combat.control`; the canonical reinforcement
   appears at the next round boundary with a full turn.
6. At the start of a death-save combatant's turn, if its card is at 0 HP and has
   neither Dead nor Stable, require
   `combat_query(view="available_actions", actor_id=...) == ["death_save"]`, then
   call `combat_check(kind="death_save")` without an `ability` or target. Do not
   require or write a synthetic Dying condition. Refresh state before continuing;
   a revived actor may still act, while a pending result may only end its turn.
7. End each completed turn with `combat_end_turn`, using the latest revision and a
   fresh idempotency key. Refresh status after every write.
8. Call `combat_end` through owner/DM `combat.control` with a structured outcome
   only when the encounter is actually over. Do not end while a death-save
   participant is at 0 HP without Dead or Stable. The server returns the campaign
   to `play`; reopen exposure before further play writes.
9. After combat, a Stable actor at 0 HP cannot rest. If the scene permits the party
   to wait, call `campaign_change(action="stable_recovery")` once with every
   simultaneously waiting Stable actor; the engine rolls each `1d4`-hour delay,
   advances the campaign timeline by the longest wait, and restores 1 HP. Do not
   patch HP, supply a roll, or run separate per-character clocks.
   When conscious and above 0 HP, clear the retained Prone condition only with
   `character_state_change(action="stand")`.

## Source-bound level advancement

1. Read the campaign's explicit advancement mode. For a milestone module, verify
   the exact trigger and do not synthesize encounter XP. For XP mode, atomically
   apply the reviewed source-bound PC awards with
   `campaign_change(action="experience_award")`, fresh campaign/actor revisions,
   and a fresh idempotency key. It does not auto-level; use its returned
   `eligible` status. Add a campaign event with the same exact source reference.
2. Settle the trigger before entering a later sourced scene. End combat, switch
   to `lobby`, re-read the actor revision, and call
   `character_state_change(action="level_advance")`. This advances an exact
   2014 or 2024 single-class actor by one level; multiclass remains a stop
   condition. Use the fixed HP value unless
   the table selected rolled HP; the engine owns that roll, so never supply a roll
   value. XP mode rejects advancement below its cumulative threshold.
3. Inspect `advancement.follow_up`. Apply its base-class and existing-subclass
   feature ids through `character_content_apply`. Resolve a listed subclass choice
   with the player, apply it, then query the catalog again for subclass features.
4. Select only the reported number of legal cantrips, prepared-list additions,
   known spells, or spellbook spells from the active edition's catalog. Apply
   Wizard additions as `method: spellbook`. A prepared-class
   `method: class_prepared` selection hydrates a legal card only and must remain
   unprepared until selected through the rest workflow.
5. Do not change a prepared list during advancement. Re-read the actor and
   verify all resources and derived values; submit any revised complete list
   through the next completed `campaign_change(action="party_rest")`.
6. Create a snapshot, switch back to `play`, and reopen phase exposure. Stop if
   the runtime reports unsupported multiclass state or any catalog item
   remains unresolved.

## Feature settlement examples

- For 2014 or 2024 Sneak Attack, declare `use_sneak_attack: true` in preflight and resolve;
  let the engine validate eligibility and its once-per-turn token.
- For the canonical 2014 or 2024 Action Surge feature id, call `combat_use_activity` on
  the Fighter's turn. The committed result consumes its card use and grants one
  current-turn `extra_action`; never patch the turn budget, and never carry an
  unused extra action into a later turn.
- For 2014 or 2024 Second Wind, call `combat_use_activity` with its exact
  edition-bound feature id. The same
  transaction pays its bonus action and use, rolls the source formula, and applies
  clamped healing. Never roll it externally or follow it with `combat_hp_change`.
- For healing from a levelled spell, send rolled base `amount`, `source_actor_id`,
  `spell_id`, and actual `spell_level`; do not pre-add source-linked modifiers.
- Halfling Lucky needs no extra write. Preserve returned reroll evidence and
  narrate only the selected final d20.
- For 2024 Heroic Inspiration, immediately reroll exactly one recorded die with
  `character_check(action="reroll", resolution_id=..., roll_index=...,
  expected_original_roll=...)`. The replacement is mandatory; never replay the
  whole check or keep the better result.
- For 2024 Divine Spark, select heal or damage in one Channel Divinity activity
  call. The engine owns level scaling, Wisdom, the Constitution save, half
  damage, target HP, and the resource receipt.
- For Turn Undead, use the edition-bound Channel Divinity card. In 2024 a failed
  save produces Frightened plus Incapacitated and depends on the source remaining
  alive and capable; `sear_undead=true` is legal only with the source-bound level
  5 feature. In 2014 use the Turned action/movement procedure instead.
- Preserve Life uses one complete allocation. Apply the Undead/Construct
  exclusion only to the 2014 card; the 2024 text has no such exclusion.
- Cunning Strike is currently an explicit engine-implementation gate, not an
  Agent permission to subtract Sneak Attack dice or patch post-hit conditions.

## Rulebook to executable optional pack

1. Load `lobby.rules`; run `rule_import` in order:
   `stage` -> `inspect` -> `ingest` -> `extract_candidates` -> `review` ->
   `compile` -> `install` -> `activate`.
2. Review exact imported chunks and provenance. Candidate extraction is not
   approval; unsupported content remains pending.
3. Compile only safe declarative IR through `rule_pack_compile` when a separate
   reviewed mechanic is needed. Arbitrary code is never executable rule content.
4. Use `rule_pack_query(view="test")` and inspect the installed inactive pack.
   Activation requires explicit campaign-owner/DM approval and a fresh campaign
   revision; the Agent must not infer that approval from its own adjudication.
5. Settle checks with `character_check` in play or `combat_check` in combat. For
   a 2014 opposed check, use one atomic `character_check(action="contest")` call instead of
   inventing a DC or comparing client-side rolls. Then audit
   `campaign_rules(action="receipts")`.

## Post-scene continuity and save

Load owner/DM `play.scene_control` before the following chronology and save
writes. A player Agent uses `play.scene` and receives only audience-safe events,
continuity, and its authorized actor knowledge.

1. Build one `memory_change(action="commit")` payload from the structured `combat_end` or scene
   outcome. Include exactly one event, accepted objective fact changes, each
   affected actor's knowledge changes, and the snapshot request.
2. Use `audience_scope="actor"` and owner-scoped ActorKnowledge for a witnessed
   subset. Use `party` only when every party actor may know the event. Never infer
   actor knowledge from a world fact.
3. Give objective facts deterministic keys such as
   `location:cellar:door-state`. Existing keys and knowledge revisions require
   their current `expected_revision_id`; the commit itself requires a fresh
   `idempotency_key` and the current campaign revision.
4. Submit once. If any write fails, refresh all affected revisions and rebuild the
   entire unit; do not retry only the missing tail or claim a partial save.
5. Verify with `snapshot_query(view="verify")` and inspect
   `snapshot_query(view="lineage")`.

## Restore, branches, and audit recovery

1. Before restore call `snapshot_query(view="verify")` and inspect lineage.
2. Explain that `snapshot_restore` forks history; perform it with current guards.
3. Verify the new head, then refresh campaign, characters, party, module progress,
   events, and each actor's continuity context. Discard pre-restore assumptions.
4. Use `branch_query(view="compare")` before discussing alternate timelines.
   There is no implicit merge of world facts or actor knowledge.
5. `state_revision(action="history")` inspects audited mutation groups.
   `state_revision(action="undo" | "redo")` uses the latest history sequence and
   does not delete snapshots.

For destructive or stateful regression, enter `lobby`, create and verify a source
checkpoint, then create-and-checkout a disposable branch. Return to `play`, reopen
exposure, run the scene/combat workflow, record actor-scoped knowledge and a full
snapshot, then return through `lobby`. The phase change dirties the disposable
branch, so create and verify a second lobby checkpoint before checkout; otherwise
the clean-branch guard must reject the switch. Checkout the source branch. Reopen
exposure after every phase or branch change. Verify source HP/resources and query
each actor's knowledge on both branches; a branch comparison must show the test
memory and subjective knowledge only on the disposable branch. There is no merge.
