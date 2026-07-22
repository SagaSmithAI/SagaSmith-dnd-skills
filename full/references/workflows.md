# Runtime Workflows

Full Runtime uses the `sagasmith_dnd` MCP server. See `mcp-contract.md` for the
complete public facade and mutation contract. Never call an internal/retired tool
name copied from an old prompt.

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
   call `continuity_context` separately for each acting PC or NPC.
6. Refresh `campaign_query(view="party")` and relevant
   `character_query(view="get")` cards. Never carry a card or revision across a
   write, phase transition, branch checkout, or restore.

An exposure belongs to one MCP session and principal. Every other Agent opens its
own exposure. Loading a group for one Agent must not expose it to another.

## New campaign and module PDF

1. Without a campaign, open an exposure and load `lobby.bootstrap`; call
   `campaign_create`, then reopen the exposure with the new campaign id.
2. Lock the correct Core edition with `campaign_rules`. Do not silently use a
   different edition or optional publication.
3. Load `lobby.modules`. For a user PDF call `module_import` in this exact order:
   `stage` -> `inspect` -> `validate` -> `ingest` -> `activate`. Keep the same
   `job_id`; use a stable, stage-specific idempotency key for each write.
4. Review `module_query(view="index")`. Search only selects candidates; expand the
   chosen scene before using its facts. Verify scene boundaries, keeper/public
   visibility, encounter participants, exact source excerpts, spatial locations,
   explicit-evidence spatial connections, and parser warnings. Never treat room
   heading order as connectivity; an empty `spatial.connections` list means the
   parser found no source-backed topology.
   If a PDF map contains required topology, use
   `module-visual-atlas.md`: `module_query(view="assets")` ->
   `module_page_render` -> visual inspection ->
   `module_set_progress(spatial_review=...)`. Never infer an edge from room order.
   If an appendix statblock is image-only, use `module-image-content-review.md`:
   render and inspect the page, submit `module_content_review`, re-read the
   immutable evidence, then use `character_create_from(mode="module_statblock")`.
5. Set scoped progress with `module_set_progress`, including
   `current_location_key` and `state.location_scene_id` when the spatial room is a
   separate scene. Never merge narrative text merely because two scenes refer to
   the same encounter.
6. If the scene changes by opening hours, daylight, watches, or travel duration,
   establish `campaign_change(action="clock_set")` before resolving the branch.
   Advance only source- or DM-established elapsed time with
   `campaign_change(action="clock_advance")`; it updates the branch-local clock
   and timed effects atomically.
   For a completed long rest, use `campaign_change(action="party_rest")` instead:
   it advances the clock once and settles all named members atomically. Never
   loop an eight-hour clock advance per character or call individual long rests.
7. Load `lobby.characters`. Use `character_create_from(mode="build")` for confirmed
   PCs and `mode="direct"`, `mode="template"`, `mode="statblock"`, or
   `mode="module_statblock"` for NPCs and monsters. Either statblock mode must
   cite exact imported evidence; unsupported or absent creatures remain unresolved
   instead of being replaced by a similar one.
   When the module modifies a named standard creature, import that exact rule source
   and use its source-bound `variant` whitelist; never replace the whole actor sheet.
   Read `module-image-content-review.md` for the distinction between an image-only
   full card and a standard card with module instance changes.
8. Apply every confirmed class/subclass feature and complete species/background
   card, then re-read each actor's `derived` values and unresolved rules.
9. Prepare legal spells with `character_spell_prepare(mode="replace_all")`.
10. Record the opening with `campaign_event(action="add")`, persist objective facts
   with `memory_change`, and call `snapshot_create`. These writes require the
   owner/DM `lobby.memory_control` and `lobby.campaign` groups.

## Scene readiness and temporary combat map

1. Build a source-grounded participant manifest from the expanded encounter scene.
   Each group has a stable key, role (`combatant`, `reinforcement`, or `optional`),
   required count, canonical campaign actor ids, same-module `source_scene_id`, and
   an exact normalized `source_excerpt`.
2. Call `module_query(view="readiness")`. Do not start while a required actor is
   missing, Dead/at 0 HP, lacks an executable card, or carries unresolved required
   rules. Review surfaced manual rulings rather than hiding them.
3. Required `combatant` actors go into initial `participant_ids`.
   `reinforcement` actors must stay out and join later through `combat_join`.
4. Call `combat_start` only after readiness succeeds. Let it compile a temporary
   combat map from the recorded spatial scene and location. Load the owner/DM
   `play.combat_control` group for this transition. If it falls back to a
   12-by-12 canvas, do not narrate those dimensions as module-authored facts.
5. Resolve surprise before `combat_start`, but do not turn an adventure's approach
   prerequisite into automatic surprise. A requirement such as "approach carefully
   and without light" only avoids the adventure's automatic alert unless its text
   explicitly promises more. Under 2014 rules, roll each hiding creature's Stealth
   with its canonical card, including armor disadvantage, and compare those results
   against each opposing creature's passive Perception. An opponent that notices
   any approaching threat is not surprised. Determine `surprised` separately for
   every participant; never replace these comparisons with the general "at least
   half succeed" group-check rule unless the imported source explicitly calls for
   a group check. Record the comparisons and source condition in a campaign event.
6. After `combat_start`, reopen exposure. The server phase is now `combat`; load
   `combat.observe`, `combat.turn`, or `combat.actions` for an acting player.
   Load `combat.control`, `combat.save`, or `combat.map` only for an owner/DM.

## Combat turn loop

1. Read `combat_query(view="status")` and
   `combat_query(view="available_actions", actor_id=...)`. Use the returned current
   actor, revision, budgets, conditions, positions, and derived attacks.
2. For every attack, use `combat_preflight_attack` immediately before
   `combat_resolve_attack`. Never supply replacement attack bonuses or damage
   formulas. Multiattack is a distinct action choice, not a passive increase to
   `derived.attacks_per_action`. To choose a source statblock Multiattack, pass one
   `derived.multiattack_options` id on the first attack and consume only its
   remaining source-defined entries. Omit the id to choose one ordinary Attack.
   An unstructured/descriptive Multiattack remains a DM boundary but never blocks
   that ordinary single weapon attack.
3. When an attack returns `pending_reaction`, read the target's
   `combat_query(view="reactions")`, then use
   `combat_choice(action="resolve_defense")`. Do not roll or apply damage twice.
4. Resolve movement with `combat_movement`, checks with `combat_check`, common
   actions with `combat_common_action`, spells with `combat_cast_spell`, activities
   with `combat_use_activity`, and damage/healing with `combat_hp_change`.
   After movement, settle every returned opportunity-reaction window before the
   next action. A rescue move can damage or incapacitate the rescuer before a
   Medicine attempt, so re-read both actor cards after the reaction.
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
   to wait, call `character_state_change(action="stable_recovery")`; the engine
   rolls the `1d4`-hour delay and restores 1 HP. Do not patch HP or supply the roll.
   When conscious and above 0 HP, clear the retained Prone condition only with
   `character_state_change(action="stand")`.

## Source-bound level advancement

1. Verify the XP or milestone in the imported module/campaign evidence and add a
   campaign event with that exact source reference.
2. End combat, switch to `lobby`, re-read the actor revision, and call
   `character_state_change(action="level_advance")`. Use the fixed HP value unless
   the player actually rolls; never supply a fabricated roll.
3. Inspect `advancement.follow_up`. Apply its base-class and existing-subclass
   feature ids through `character_content_apply`. Resolve a listed subclass choice
   with the player, apply it, then query the catalog again for subclass features.
4. Select only the reported number of legal cantrips/known/spellbook spells from
   the active catalog. Apply Wizard additions as `method: spellbook`.
5. Submit the complete prepared list with
   `character_spell_prepare(mode="replace_all", event="level_up")`, re-read the
   actor, and verify all resources and derived values.
6. Create a snapshot, switch back to `play`, and reopen phase exposure. Stop if
   the runtime reports unsupported edition/multiclass state or any catalog item
   remains unresolved.

## Feature settlement examples

- For 2014 Sneak Attack, declare `use_sneak_attack: true` in preflight and resolve;
  let the engine validate eligibility and its once-per-turn token.
- For the canonical 2014 Action Surge feature id, call `combat_use_activity` on
  the Fighter's turn. The committed result consumes its card use and grants one
  current-turn `extra_action`; never patch the turn budget, and never carry an
  unused extra action into a later turn.
- For Second Wind, call `combat_use_activity` to pay its bonus action and use, roll
  the source formula, then call `combat_hp_change(action="heal")` with that result.
- For healing from a levelled spell, send rolled base `amount`, `source_actor_id`,
  `spell_id`, and actual `spell_level`; do not pre-add source-linked modifiers.
- Halfling Lucky needs no extra write. Preserve returned reroll evidence and
  narrate only the selected final d20.

## Rulebook to executable optional pack

1. Load `lobby.rules`; run `rule_import` in order:
   `stage` -> `inspect` -> `ingest` -> `extract_candidates` -> `review` ->
   `compile` -> `install` -> `activate`.
2. Review exact imported chunks and provenance. Candidate extraction is not
   approval; unsupported content remains pending.
3. Compile only safe declarative IR through `rule_pack_compile` when a separate
   reviewed mechanic is needed. Arbitrary code is never executable rule content.
4. Use `rule_pack_query(view="test")` and inspect the installed inactive pack.
   Activation requires explicit DM approval and a fresh campaign revision.
5. Settle checks with `character_check` in play or `combat_check` in combat, then
   audit `campaign_rules(action="receipts")`.

## Post-scene continuity and save

Load owner/DM `play.scene_control` before the following chronology and save
writes. A player Agent uses `play.scene` and receives only audience-safe events,
continuity, and its authorized actor knowledge.

1. Persist the structured `combat_end` outcome as a `campaign_event`.
2. Use `audience_scope="actor"`, `known_by_actor_ids`, and
   `knowledge_disclosure_scope="owner"` for a witnessed subset. Use `party` only
   when every party actor may know the event.
3. Persist objective world facts with `memory_change`. Persist each PC/NPC's
   subjective belief, secret, inference, or misinformation separately with
   `actor_knowledge_change`.
4. Call `snapshot_create` with the current campaign revision, branch id, and head
   snapshot id. The restorable payload is full; the recap is the delta.
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
